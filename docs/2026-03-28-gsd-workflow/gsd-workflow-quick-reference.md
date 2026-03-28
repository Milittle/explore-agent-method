# GSD (Get Shit Done) - 快速参考指南

## 一句话总结

GSD 是一个轻量级的元提示和上下文工程系统，通过分层加载、结构化提示和编排者-子智能体模式，解决 AI 辅助编程中的上下文衰减问题，让 solo 开发者能可靠地构建复杂系统。

---

## 核心原则速查表

| 原则 | 说明 | 实践 |
|------|------|------|
| **上下文分层加载** | 根据阶段动态加载相关上下文 | PROJECT.md（全局）+ CONTEXT.md（阶段）+ PLAN.md（任务） |
| **XML 结构化提示** | 使用 XML 格式定义任务 | `<task><name><files><action><verify><done>` |
| **波次并行执行** | 基于依赖关系的并行执行 | 无依赖 → 同一波次；有依赖 → 后续波次 |
| **原子 Git 提交** | 每个任务独立提交 | 一个任务 = 一个提交，支持 git bisect |
| **编排者-子智能体** | 轻量级编排，专业化执行 | 主上下文 30-40%，子智能体全新上下文 |
| **验证不可或缺** | 自动 + 人工验证 | 代码存在 + 测试通过 + 功能验收 |

---

## Token 优化检查清单

### 上下文加载策略

- [ ] **PROJECT.md** - 始终加载（项目愿景）
- [ ] **research/** - 阶段规划时加载（生态系统知识）
- [ ] **CONTEXT.md** - 阶段执行时加载（实施决策）
- [ ] **PLAN.md** - 任务执行时加载（XML 结构化任务）
- [ ] **STATE.md** - 跨会话记忆（决策、阻塞、位置）

### 大小控制

| 文档 | 大小限制 | 说明 |
|------|----------|------|
| PROJECT.md | ~50k tokens | 项目愿景，始终加载 |
| CONTEXT.md | ~30k tokens | 阶段决策，执行时加载 |
| PLAN.md | ~10k tokens | 单个任务，独立执行 |
| 研究文档 | 每个 ~20k tokens | 并行研究，按需加载 |

### 最佳实践

✅ **DO** - 垂直切片（Plan 1: 用户端到端，Plan 2: 产品端到端）
❌ **DON'T** - 水平切片（Plan 1: 所有模型，Plan 2: 所有 API）

✅ **DO** - 每个计划独立可执行
❌ **DON'T** - 计划间相互依赖

---

## 常见问题快速修复

### 问题 1：上下文衰减（质量下降）

**症状**：
- "我现在会更简洁"
- 遗忘早期内容
- 决策不一致

**解决方案**：
```bash
# 检查上下文使用率
/gsd:stats

# 启用上下文监控（显示使用率警告）
/gsd:settings
# 设置 hooks.context_warnings = true
```

### 问题 2：权限确认频繁

**症状**：每次 `git commit`, `date` 等命令都需要确认

**解决方案**：
```bash
# 方案 A：推荐（跳过权限模式）
claude --dangerously-skip-permissions

# 方案 B：细粒度权限
# 编辑 .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(git commit:*)",
      "Bash(git add:*)",
      # ... 添加其他常用命令
    ]
  }
}
```

### 问题 3：子智能体失败

**症状**：计划执行失败，子智能体无响应

**解决方案**：
```bash
# 1. 检查子智能体日志
cat .planning/phases/{phase_num}/{plan_num}-SUMMARY.md

# 2. 重新执行特定计划
/gsd:execute-phase {phase_num}

# 3. 系统化调试
/gsd:debug "描述问题"
```

### 问题 4：验证失败

**症状**：功能不符合预期，UAT 失败

**解决方案**：
```bash
# 1. 重新验证
/gsd:verify-work {phase_num}

# 2. 系统自动诊断
# 失败时自动生成修复计划

# 3. 重新执行修复计划
/gsd:execute-phase {phase_num}
```

### 问题 5：Git 历史混乱

**症状**：提交不清晰，难以定位问题

**解决方案**：
```bash
# 检查提交历史
git log --oneline

# 如果出现问题，使用 git bisect 定位
git bisect start
git bisect bad HEAD
git bisect good {好的提交}
# Git 会自动定位问题提交

# 独立回滚
git revert {提交哈希}
```

---

## 架构选择决策树

```
开始
  ↓
是否是全新项目？
  ├─ 是 → /gsd:new-project
  │         ├─ 想要并行研究？→ 等待研究完成
  │         └─ 快速启动？→ 使用 --auto 跳过研究
  │
  └─ 否 → 是否已有代码？
           ├─ 是 → /gsd:map-codebase（先分析现有代码）
           │        ↓
           │      /gsd:new-milestone（添加新功能）
           │
           └─ 否 → 是否是临时任务？
                    ├─ 是 → /gsd:quick
                    │        ├─ 需要讨论？→ /gsd:quick --discuss
                    │        ├─ 需要研究？→ /gsd:quick --research
                    │        └─ 完整验证？→ /gsd:quick --full
                    │
                    └─ 否 → /gsd:new-project
```

---

## 技能推荐学习顺序

### 第 1 周：核心工作流

1. **安装和配置**
   ```bash
   npx get-shit-done-cc@latest
   /gsd:help
   ```

2. **第一个项目**（选择小型项目）
   ```bash
   /gsd:new-project          # 理解问题收集
   /gsd:discuss-phase 1      # 理解决策捕获
   /gsd:plan-phase 1         # 理解研究规划
   /gsd:execute-phase 1      # 理解波次执行
   /gsd:verify-work 1        # 理解验证流程
   ```

3. **完成里程碑**
   ```bash
   /gsd:complete-milestone   # 归档和标记版本
   ```

### 第 2 周：高级功能

1. **现有代码库分析**
   ```bash
   /gsd:map-codebase         # 并行分析栈、架构、约定
   ```

2. **快速任务**
   ```bash
   /gsd:quick                # 跳过完整规划
   /gsd:fast "描述"          # 内联执行
   ```

3. **配置和优化**
   ```bash
   /gsd:settings             # 调整工作流
   /gsd:set-profile budget   # 切换模型配置
   ```

### 第 3 周：团队和协作

1. **工作流管理**
   ```bash
   /gsd:workstreams create feature-a
   /gsd:workstreams switch feature-a
   ```

2. **代码审查和 PR**
   ```bash
   /gsd:ship 2               # 创建 PR
   /gsd:pr-branch            # 过滤 .planning/ 提交
   ```

3. **监控和统计**
   ```bash
   /gsd:stats                # 项目统计
   /gsd:health               # 健康检查
   ```

---

## 实际案例效果

### 案例 1：Solo 开发者构建 SaaS 产品

**场景**：一个人构建完整的 B2B SaaS

**使用 GSD 前**：
- 3 个月完成 MVP
- 代码质量不一致
- 频繁返工
- 难以追踪进度

**使用 GSD 后**：
- 6 周完成 MVP（快 2 倍）
- 代码质量一致
- 清晰的 git 历史
- 每个阶段可验证

**关键因素**：
- `/gsd:new-project` 系统化需求收集
- `/gsd:map-codebase` 理解现有模式
- 波次并行执行加速开发
- `/gsd:verify-work` 确保功能正确

### 案例 2：团队协作开发

**场景**：5 人团队，3 个并行功能分支

**使用 GSD 前**：
- 代码冲突频繁
- 功能不集成
- 难以追踪进度

**使用 GSD 后**：
- 工作流隔离并行开发
- 清晰的集成点
- 每个工作流独立可验证

**关键因素**：
- `/gsd:workstreams create` 创建隔离
- 垂直切片减少依赖
- `/gsd:workstreams complete` 系统化合并

### 案例 3：遗留系统现代化

**场景**：5 年历史的代码库，添加新功能

**使用 GSD 前**：
- 不了解现有模式
- 新代码不一致
- 破坏现有功能

**使用 GSD 后**：
- `/gsd:map-codebase` 分析现有架构
- 新代码遵循现有模式
- 不破坏现有功能

**关键因素**：
- 并行分析（栈、架构、约定、关注点）
- 假设模式（`workflow.discuss_mode = 'assumptions'`）
- 验证确保不破坏现有功能

---

## 快速实施模板

### 模板 1：新项目初始化

```bash
# 1. 安装 GSD
npx get-shit-done-cc@latest

# 2. 启动新项目
/gsd:new-project

# 3. 回答问题（直到系统完全理解）
# - 目标和约束
# - 技术偏好
# - 边缘情况

# 4. 等待研究（可选但推荐）
# - 4 个并行研究智能体
# - 栈、功能、架构、陷阱

# 5. 批准路线图

# 6. 开始第一阶段
/gsd:discuss-phase 1
/gsd:plan-phase 1
/gsd:execute-phase 1
/gsd:verify-work 1
```

### 模板 2：现有代码库添加功能

```bash
# 1. 分析现有代码
/gsd:map-codebase

# 2. 等待并行分析完成
# - 栈和依赖
# - 架构和设计模式
# - 约定和风格
# - 技术债务和关注点

# 3. 启动新里程碑
/gsd:new-milestone

# 4. 描述要添加的功能

# 5. 系统生成新路线图

# 6. 执行阶段（同模板 1）
```

### 模板 3：快速任务

```bash
# 简单任务（无研究、无验证）
/gsd:quick

# 需要讨论的任务
/gsd:quick --discuss

# 需要研究的任务
/gsd:quick --research

# 完整验证
/gsd:quick --full

# 组合
/gsd:quick --discuss --research --full
```

### 模板 4：故障排查

```bash
# 1. 检查项目健康
/gsd:health

# 2. 修复问题（如果需要）
/gsd:health --repair

# 3. 系统化调试
/gsd:debug "描述问题"

# 4. 如果工作流失败
/gsd:forensics "失败描述"
```

---

## 核心命令速查

### 工作流核心（5 个命令）

```bash
/gsd:new-project          # 初始化新项目
/gsd:discuss-phase N      # 捕获实施决策
/gsd:plan-phase N         # 研究和规划
/gsd:execute-phase N      # 执行计划
/gsd:verify-work N        # 验证功能
```

### 导航和状态

```bash
/gsd:progress             # 当前位置和下一步
/gsd:next                 # 自动下一步
/gsd:stats                # 项目统计
/gsd:help                 # 所有命令
```

### 代码库分析

```bash
/gsd:map-codebase         # 分析现有代码
```

### 快速任务

```bash
/gsd:quick                # 快速任务
/gsd:fast "描述"          # 内联执行
```

### 里程碑管理

```bash
/gsd:complete-milestone   # 完成里程碑
/gsd:new-milestone        # 新里程碑
/gsd:ship N               # 创建 PR
```

### 配置

```bash
/gsd:settings             # 配置工作流
/gsd:set-profile budget   # 切换模型配置
```

---

## 配置文件速查

### `.planning/config.json` 核心设置

```json
{
  "mode": "interactive",           // yolo | interactive
  "granularity": "standard",        // coarse | standard | fine
  "model_profile": "balanced",      // quality | balanced | budget | inherit

  "git": {
    "branching_strategy": "none"   // none | phase | milestone
  },

  "workflow": {
    "discuss_mode": "discuss",     // discuss | assumptions
    "research": true,
    "plan_check": true,
    "verifier": true,
    "auto_advance": false
  },

  "parallelization": {
    "enabled": true
  }
}
```

### 模型配置

| 配置 | 规划 | 执行 | 验证 | 适用场景 |
|------|------|------|------|----------|
| `quality` | Opus | Opus | Sonnet | 高质量，不限制成本 |
| `balanced` | Opus | Sonnet | Sonnet | 平衡质量和速度（默认） |
| `budget` | Sonnet | Sonnet | Haiku | 经济实惠，快速迭代 |
| `inherit` | 继承 | 继承 | 继承 | 非 Anthropic 提供商 |

---

## 社区和资源

### 快速链接

- **GitHub**：https://github.com/gsd-build/get-shit-done
- **Discord**：https://discord.gg/gsd-build
- **NPM**：`npx get-shit-done-cc@latest`

### 更新和升级

```bash
# 检查更新
/gsd:update

# 手动更新
npx get-shit-done-cc@latest
```

### 获取帮助

```bash
# 显示所有命令
/gsd:help

# 加入社区
/gsd:join-discord
```

---

**文档元信息**：
- 生成日期：2026-03-28
- 配套文档：`gsd-workflow-insight.md`
- 方法论类型：AI 辅助编程工作流
- 核心价值：上下文工程 + 结构化流程
