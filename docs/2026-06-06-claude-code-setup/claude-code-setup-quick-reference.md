# Claude Code Setup 插件 - 快速参考指南

## 一句话总结

**扫描代码库特征信号，将 Hooks / Skills / MCP / Subagents / Plugins 五类 Claude Code 能力映射为每类 Top 1-2 最高价值推荐；只读分析、渐进披露、用户自主实施，解决"选项爆炸、不知从何配置"的冷启动难题。**

---

## 核心原则速查表

| 原则 | 核心表述 | 实践要点 |
|------|----------|----------|
| 信号驱动，非通用清单 | 从 package.json、目录结构、配置文件等可观测信号推导推荐 | 先分析再推荐，拒绝与项目无关的 MCP/插件 |
| Top-N 渐进披露 | 默认每类 1-2 条，用户点名单类可扩展至 3-5 条 | 防止选项过载，保证可行动性 |
| 只读分析，实施分离 | 技能只探测不推荐即改；用户确认后再落地 | 降低意外配置风险 |
| 五类分层思维 | MCP(外部) → Skills(工作流) → Hooks(事件) → Subagents(并行) → Plugins(打包) | 按层补齐，而非孤立堆砌 |
| 参考库 + 动态搜索 | references/ 覆盖常见模式，web search 补充长尾集成 | 目录可维护且不失特异性 |
| 团队配置入库 | `.mcp.json` 提交 git，全团队共享 MCP | 避免个人全局配置漂移 |
| 调用边界明确 | 副作用技能 `disable-model-invocation: true`；背景知识 `user-invocable: false` | 防止 Claude 自动 deploy/commit |

---

## 触发与三阶段工作流

### 触发短语

```
"recommend automations for this project"
"help me set up Claude Code"
"what hooks should I use?"
"what MCP servers should I use?"
"show me more MCP server options"   ← 单类深挖
```

### 三阶段速查

| 阶段 | 动作 | 关键输出 |
|------|------|----------|
| **Phase 1 分析** | 检测项目类型、依赖、目录、现有 `.claude/` | Codebase Profile |
| **Phase 2 推荐** | 五类信号映射 + Top 1-2 过滤 | 分类候选清单 |
| **Phase 3 报告** | 结构化 Markdown + 扩展入口 | 可执行推荐报告 |

### Phase 1 探测命令模板

```bash
ls -la package.json pyproject.toml Cargo.toml go.mod pom.xml 2>/dev/null
cat package.json 2>/dev/null | head -50
cat package.json 2>/dev/null | grep -E '"(react|vue|next|fastapi|django|prisma|supabase|convex|stripe)"'
ls -la .claude/ CLAUDE.md 2>/dev/null
ls -la src/ app/ lib/ tests/ components/ pages/ api/ 2>/dev/null
```

---

## 五类自动化决策树

```
开始：已了解项目技术栈？
│
├─ 需要连接外部服务/API/数据库？
│   └─ YES → MCP Servers（优先 context7 / 栈专用 MCP）
│
├─ 有重复性工作流或项目约定？
│   └─ YES → Skills（官方插件技能 或 自定义 .claude/skills/）
│
├─ 编辑后需自动 format/lint/test 或保护敏感文件？
│   └─ YES → Hooks（.claude/settings.json）
│
├─ 需并行专业化审查（安全/性能/可访问性）？
│   └─ YES → Subagents（.claude/agents/）
│
└─ 需多能力打包或团队标准化？
    └─ YES → Plugins（/plugin install）
```

---

## 信号 → 推荐速查表

### MCP Servers

| 检测信号 | 推荐 MCP |
|----------|----------|
| React/Vue/Next + 常用 npm 库 | **context7** |
| 前端 + UI 测试需求 | **Playwright** |
| `@supabase/supabase-js` | **Supabase MCP** |
| `convex/` 或 `convex.json` | **Convex MCP** |
| GitHub remote | **GitHub MCP** |
| Linear 引用 (ABC-123) | **Linear MCP** |
| `@aws-sdk/*` | **AWS MCP** |
| `@sentry/*` | **Sentry MCP** |
| `docker-compose.yml` | **Docker MCP** |

**安装示例**：`claude mcp add context7`

### Skills

| 检测信号 | 推荐 Skill | 来源 |
|----------|------------|------|
| React/Vue/Angular | frontend-design | frontend-design 插件 |
| Git 工作流 | commit | commit-commands 插件 |
| API 路由 | api-doc（自定义） | `.claude/skills/api-doc/` |
| 测试目录存在 | gen-test（自定义） | `.claude/skills/gen-test/` |
| 团队编码约定 | project-conventions（Claude-only） | `.claude/skills/` |
| 新成员 onboarding | setup-dev（自定义） | `.claude/skills/setup-dev/` |

### Hooks

| 检测信号 | 推荐 Hook | 事件类型 |
|----------|-----------|----------|
| Prettier 配置 | 自动格式化 | PostToolUse on Edit/Write |
| ESLint/Ruff 配置 | 自动 lint/fix | PostToolUse on Edit/Write |
| tsconfig.json | 类型检查 | PostToolUse |
| tests/ 目录 | 运行相关测试 | PostToolUse |
| .env 文件 | 阻断编辑 | PreToolUse |
| lock 文件 | 阻断直接编辑 | PreToolUse |

### Subagents

| 检测信号 | 推荐 Subagent | 模型 |
|----------|---------------|------|
| >500 文件 | code-reviewer | sonnet |
| auth/payment 代码 | security-reviewer | sonnet（只读工具） |
| API 路由无文档 | api-documenter | sonnet |
| 前端组件库 | ui-reviewer | sonnet |
| 测试覆盖不足 | test-writer | sonnet |

### Plugins

| 检测信号 | 推荐 Plugin |
|----------|-------------|
| 首次搭建 | anthropic-agent-skills |
| PR 工作流 | pr-review-toolkit / commit-commands |
| 前端项目 | frontend-design |
| 安全敏感 | security-guidance |
| TypeScript | typescript-lsp |
| Python | pyright-lsp |

---

## 技能 Frontmatter 速查

```yaml
---
name: skill-name
description: 做什么 + 何时触发（含触发短语）
disable-model-invocation: true   # 仅用户：deploy/commit/migration
user-invocable: false            # 仅 Claude：project-conventions
allowed-tools: Read, Grep, Glob  # 限制工具
context: fork                    # 隔离子智能体运行
agent: Explore                   # fork 时使用的代理类型
---
```

| 设置 | 用户 | Claude | 适用 |
|------|------|--------|------|
| 默认 | ✓ | ✓ | 通用工作流 |
| `disable-model-invocation: true` | ✓ | ✗ | 有副作用操作 |
| `user-invocable: false` | ✗ | ✓ | 背景知识/约定 |

---

## 配置落点速查

| 能力类型 | 配置文件位置 | 团队共享 |
|----------|--------------|----------|
| MCP Servers | `.mcp.json` 或 `~/.claude.json` | `.mcp.json` 入库 ✓ |
| Skills | `.claude/skills/<name>/SKILL.md` | 入库 ✓ |
| Hooks | `.claude/settings.json` | 入库 ✓ |
| Subagents | `.claude/agents/<name>.md` | 入库 ✓ |
| Plugins | `/plugin install <name>` | 文档化安装清单 |

### Permissions 示例（Hooks 前置）

```json
{
  "permissions": {
    "allow": ["Edit", "Write", "Bash(npm test:*)", "Bash(git commit:*)"]
  }
}
```

### Headless CI 示例

```bash
claude -p "fix lint errors in src/" --allowedTools Edit,Write
claude -p "<prompt>" --output-format stream-json | your_command
```

---

## 30 分钟快速实施模板

### 0-5 分钟：触发分析

1. `/plugin install claude-code-setup`
2. 输入：`recommend automations for this project`
3. 保存报告，核对 Codebase Profile

### 5-15 分钟：高 ROI 落地（任选组合）

| 优先级 | 动作 | 耗时 |
|--------|------|------|
| P0 | context7 MCP（几乎任何 JS/Python 项目） | 2 min |
| P0 | Prettier/ESLint PostToolUse Hook | 5 min |
| P1 | PreToolUse 阻断 .env 编辑 | 3 min |
| P1 | project-conventions Skill（Claude-only） | 10 min |
| P2 | security-reviewer 或 code-reviewer Subagent | 10 min |

### 15-30 分钟：验证与文档化

1. `claude --mcp-debug` 检查 MCP
2. 故意编辑源文件，验证 format/lint Hook
3. 在 CLAUDE.md 记录已安装自动化摘要
4. 标记下次技术栈变更时需重新运行 recommender

---

## 反模式快速排查

| 症状 | 可能反模式 | 修复 |
|------|------------|------|
| 报告太长无法行动 | 一次性倾倒全部选项 | 只实施每类 Top 1-2 |
| Hook 不生效 | 无 permissions 或未配置路径 | 检查 settings.json + allow 列表 |
| 团队 MCP 不一致 | 仅个人全局配置 | 改用 `.mcp.json` 入库 |
| Claude 自动执行 deploy | 副作用技能无调用限制 | 加 `disable-model-invocation: true` |
| 推荐与项目无关 | 跳过 Phase 1 分析 | 重新运行并核对 Profile |
| 审查代理改坏代码 | Subagent 工具权限过宽 | 审查类限 Read/Grep/Glob |

---

## 与其他方法论的衔接

| 已探索方法论 | 衔接点 |
|--------------|--------|
| **CLAUDE.md 管理** | Setup 负责"选对配置"，CLAUDE.md 管理负责"维护记忆"；报告摘要应写入 CLAUDE.md |
| **Claude Skills 构建指南** | Setup 推荐创建哪些 Skills；构建指南教如何写好 SKILL.md |
| **Superpowers** | 可推荐工程纪律类工作流；Setup 选型，Superpowers 执行 |
| **GSD 工作流** | GSD 的编排者-子智能体与 Setup 的 Subagent 推荐互补 |
| **Vibe Coding 工程前置** | Setup 推荐的 project-conventions Skill 可承载 invariants/constraints |

---

## 立即行动清单

- [ ] 运行 `recommend automations for this project`
- [ ] 实施 1 个 MCP（通常 context7 或栈专用）
- [ ] 实施 1 个 Hook（format 或 protection）
- [ ] 创建 1 个 Claude-only Skill（project-conventions）
- [ ] `.mcp.json` / settings 考虑入库（团队项目）
- [ ] 配合 CLAUDE.md 管理记录配置摘要