# Superpowers - 快速参考指南

> GitHub: https://github.com/obra/superpowers | 101k Stars | MIT

---

## 一句话总结

**Superpowers 通过 15+ 个可组合技能文件，将"设计先于编码、测试先于实现、验证先于声明完成"的工程纪律强制注入 AI 编码智能体行为。**

---

## 完整工作流速查

```
① Brainstorming    → 设计文档 + 用户审批（硬门控，无法跳过）
② Git Worktrees    → 独立工作区，干净测试基线
③ Writing Plans    → 2-5 分钟粒度任务，含精确文件路径
④ Subagent Dev     → 每任务新鲜子智能体 + 两阶段审查
⑤ TDD              → RED(失败测试) → GREEN(最少代码) → REFACTOR
⑥ Code Review      → 规格合规审查 → 代码质量审查
⑦ Finish Branch    → 验证测试 → 合并/PR/保留/丢弃（四选一）
```

---

## 技能优先级规则

```
用户明确指令 > Superpowers Skills > 默认系统提示

规则：如果有 1% 概率某技能适用 → 必须调用该技能
```

---

## 刚性技能（严格遵循，无例外）

### TDD 检查清单
- [ ] 先写测试，确认测试**失败**
- [ ] 写最少代码让测试通过
- [ ] 重构，保持测试通过
- [ ] **如果代码在测试之前已存在 → 删除代码，重新开始**

### 系统调试清单
- [ ] 阶段1：根因调查（读完整错误信息，复现步骤）
- [ ] 阶段2：模式分析（找可用参照，列出差异）
- [ ] 阶段3：假设测试（每次只做最小改动）
- [ ] 阶段4：实施（先写失败测试，再修复）
- [ ] **3次修复失败 → 停止，质疑架构**

### 验证完成清单
- [ ] 识别能证明完成的命令
- [ ] 实际运行该命令（不是"应该能运行"）
- [ ] 读取完整输出，检查退出码
- [ ] 输出确实支持完成声明，才能汇报完成

---

## 子智能体派发决策树

```
任务是否独立（互不影响）？
  ├─ 否 → 顺序执行，单智能体
  └─ 是 →
        独立任务数量？
          ├─ 1-2个 → 直接执行，不必开新子智能体
          └─ 3+个 → 并发派发，每个域一个子智能体
                      ↓
              派发后：收集结果 → 检查冲突 → 完整测试套件
```

---

## 设计门控规则

**绝对禁止**在设计审批前执行的操作：
- 写任何生产代码
- 创建项目脚手架
- 执行任何实施类操作

**触发 brainstorming 的时机**：
- 收到新功能需求
- 重构方向不明确
- 涉及多组件修改

---

## 计划文档规范

每个任务必须包含：
```markdown
### 任务 N: [任务名称]
**目标**：[一句话描述]
**文件**：`src/path/to/file.ts`
**失败测试**：[精确测试代码]
**实施**：[最少实现代码]
**命令**：`npm test -- --testPathPattern=xxx`
**预期输出**：[精确输出内容]
**提交**：`git commit -m "feat: xxx"`
**时间估算**：2-5 分钟
```

---

## 代码审查两阶段顺序

```
阶段1（规格合规）先于阶段2（代码质量）

阶段1 问题 → 实施者修复 → 重新阶段1 → 通过后 → 阶段2
              （不可跳到阶段2）
```

代码审查维度（Code Reviewer Agent）：
1. **计划对齐** - 实现与规格的偏差
2. **代码质量** - 模式、错误处理、命名、安全
3. **架构** - SOLID、关注点分离
4. **文档** - 注释和标准合规
5. **问题分级** - Critical / Important / Suggestions

---

## 快速接入步骤（30 分钟）

```bash
# 1. 在项目中创建技能目录
mkdir -p .claude/skills

# 2. 下载核心技能文件（从 github.com/obra/superpowers）
# 优先下载：
#   skills/using-superpowers/
#   skills/brainstorming/
#   skills/test-driven-development/
#   skills/systematic-debugging/
#   skills/writing-plans/
#   skills/subagent-driven-development/
#   skills/verification-before-completion/
#   agents/code-reviewer.md

# 3. 在 CLAUDE.md 中声明技能路径
```

**CLAUDE.md 最小配置**：
```markdown
## 技能系统
项目使用 Superpowers 技能框架。执行任何任务前必须检查 .claude/skills/ 中是否有适用技能。
技能优先级高于默认行为。
```

---

## 常见场景快速决策

| 场景 | 使用技能 |
|------|----------|
| 收到新功能需求 | `brainstorming` |
| 开始编写代码 | `test-driven-development` |
| 遇到 bug | `systematic-debugging` |
| 执行计划任务 | `subagent-driven-development` |
| 多个独立问题 | `dispatching-parallel-agents` |
| 声明任务完成 | `verification-before-completion` |
| 完成一个特性分支 | `finishing-a-development-branch` |
| 不确定用哪个技能 | `using-superpowers` |

---

## 反模式速查（立即停止）

- 收到需求 → 立刻写代码 ❌
- 先写代码 → 再补测试 ❌
- "应该可以了" → 不验证就声明完成 ❌
- bug 修复 → 同时修改多处 ❌
- 3次失败还在继续尝试 ❌
- 有依赖关系的任务 → 并发执行 ❌
