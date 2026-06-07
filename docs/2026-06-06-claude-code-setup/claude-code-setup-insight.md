# Claude Code Setup 插件 - 分析与应用指南

> 来源：https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-code-setup  
> 插件作者：Isabella He (Anthropic)  
> 分析日期：2026-06-06

## 快速摘要

**Claude Code Setup 插件** 是 Anthropic 官方提供的**项目感知型自动化推荐引擎**，通过 `claude-automation-recommender` 技能扫描代码库特征信号，将 Hooks、Skills、MCP Servers、Subagents、Plugins 五类 Claude Code 扩展能力映射为**每类 1-2 条最高价值推荐**。其核心方法论是**只读分析 + 信号驱动决策 + 渐进式披露 + 参考库与动态搜索混合**，解决开发者在 Claude Code 生态中"选项爆炸、不知从何配置"的冷启动与持续优化难题。

这是一套面向 AI 编码智能体**基础设施选型与落地编排**的方法论，与本项目此前探索的 Claude Skills 构建指南、CLAUDE.md 管理、Superpowers 工程纪律、GSD 工作流形成"配置 → 记忆 → 纪律 → 执行"的完整闭环。

## 核心方法论

### 解决的问题

Claude Code 提供了丰富的可扩展能力（Hooks、Skills、MCP、Subagents、Plugins、Slash Commands），但真实使用中普遍存在：

- **选项爆炸（Option Overload）**：数十种 MCP 服务器、官方插件、自定义技能模板并存，新手和团队难以判断优先级。
- **配置与代码库脱节（Context Mismatch）**：通用推荐清单无法反映项目实际技术栈（React vs Django vs Convex）、工具链（Prettier/Ruff/Jest）和安全边界（.env、lock files）。
- **过早实施风险（Premature Automation）**：在未理解项目特征前批量安装插件/钩子，导致 token 膨胀、权限冲突、团队不同步。
- **缺乏编排视角（No Orchestration View）**：用户孤立看待单一能力类型，未建立"外部集成 → 工作流封装 → 事件自动化 → 并行审查 → 预打包套件"的分层思维。
- **团队配置漂移（Team Config Drift）**：个人全局配置与项目需求不一致，缺少 `.mcp.json` 入库等团队共享机制。

插件直击"**如何为这个项目选对 Claude Code 自动化组合**"这一冷启动与持续优化问题，且坚持**只读分析、用户自主实施**，避免静默修改项目。

### 核心概念

1. **五类自动化分类法（Automation Taxonomy）**

   | 类型 | 职责 | 典型场景 |
   |------|------|----------|
   | **MCP Servers** | 连接外部工具/服务（数据库、API、浏览器、文档） | context7 查库文档、GitHub MCP 管 PR |
   | **Skills** | 封装领域知识、可重复工作流、模板与脚本 | api-doc、gen-test、project-conventions |
   | **Hooks** | 工具事件触发的自动动作（格式化、lint、阻断） | PostToolUse 自动 Prettier、PreToolUse 阻断 .env |
   | **Subagents** | 并行运行的专业化审查/分析实例 | security-reviewer、code-reviewer |
   | **Plugins** | 预打包的技能/命令/代理/钩子集合 | frontend-design、commit-commands |

2. **三阶段推荐工作流（Analyze → Recommend → Report）**
   - **Phase 1 代码库分析**：检测语言/框架、依赖、目录结构、测试/CI 配置、现有 `.claude/` 与 `CLAUDE.md`。
   - **Phase 2 生成推荐**：基于信号映射表，跨五类生成候选；每类仅保留 Top 1-2。
   - **Phase 3 输出报告**：结构化 Markdown 报告，含 Codebase Profile、每类 Why + Install/Create 指引、扩展入口。

3. **信号驱动映射（Signal-Driven Mapping）**
   - 将可观测项目特征（`package.json` 依赖、`tsconfig.json`、`.env` 存在、`convex/` 目录等）映射到具体自动化建议。
   - 参考文件提供通用模式库（`references/mcp-servers.md` 等），**同时要求 web search 补充代码库特定集成**，避免目录僵化。

4. **渐进式披露约束（Top-N Disclosure）**
   - 默认每类仅推荐 **1-2 条**最高价值项，防止信息过载。
   - 用户点名某类时，可扩展至 3-5 条。
   - 报告末尾明确"可要求更多推荐"与"可请求帮助实施"。

5. **只读分析原则（Read-Only Analysis）**
   - 技能仅使用 Read、Glob、Grep、Bash 进行探测，**不创建或修改任何文件**。
   - 实施与用户审批分离：推荐 ≠ 自动落地，降低意外配置风险。

6. **技能调用控制矩阵（Invocation Control）**
   - 默认：用户与 Claude 均可调用。
   - `disable-model-invocation: true`：仅用户（副作用操作：deploy、commit、migration）。
   - `user-invocable: false`：仅 Claude（背景知识如 project-conventions）。
   - `context: fork` + `agent`：隔离子智能体中运行复杂工作流。

7. **团队共享与无头模式（Team Sharing & Headless）**
   - `.mcp.json` 入库实现团队统一 MCP 配置；`claude --mcp-debug` 排障。
   - Headless 模式（`claude -p "..." --allowedTools ...`）支持 CI/pre-commit 管道集成。

### 架构概览

```
用户触发（"recommend automations" / "help me set up Claude Code"）
       │
       ▼
┌──────────────────────────────────────────────────────────────┐
│           claude-automation-recommender (Skill)               │
│                    [READ-ONLY 只读分析]                        │
├──────────────────────────────────────────────────────────────┤
│  Phase 1: Codebase Analysis                                   │
│  ├─ 检测项目类型 (package.json / pyproject.toml / go.mod...)  │
│  ├─ 解析依赖与框架信号                                         │
│  ├─ 扫描 .claude/、CLAUDE.md 现有配置                         │
│  └─ 分析目录结构 (src/, tests/, api/, components/...)        │
├──────────────────────────────────────────────────────────────┤
│  Phase 2: Generate Recommendations                            │
│  ├─ MCP Servers    ← references/mcp-servers.md + web search  │
│  ├─ Skills         ← references/skills-reference.md          │
│  ├─ Hooks          ← references/hooks-patterns.md            │
│  ├─ Subagents      ← references/subagent-templates.md      │
│  └─ Plugins        ← references/plugins-reference.md       │
│       ↓ 每类 Top 1-2 过滤 + 跳过无关类别                      │
├──────────────────────────────────────────────────────────────┤
│  Phase 3: Output Recommendations Report                       │
│  ├─ Codebase Profile                                          │
│  ├─ 分类推荐 (Why + Install/Create + 配置位置)               │
│  └─ 扩展入口 ("Want more?" / "Want help implementing?")      │
└──────────────────────────────────────────────────────────────┘
       │
       ▼ (用户自主实施或另开对话请求 Claude 帮助搭建)
┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐
│ .mcp.json   │ .claude/    │ .claude/    │ .claude/    │ /plugin     │
│ MCP Servers │ skills/     │ settings    │ agents/     │ install     │
│             │             │ (hooks)     │             │             │
└─────────────┴─────────────┴─────────────┴─────────────┴─────────────┘
```

**决策框架摘要**：

| 何时推荐 | 判断条件 |
|----------|----------|
| MCP | 需外部服务集成、实时文档、浏览器自动化、团队工具（GitHub/Linear/Slack） |
| Skills | 重复性工作流、模板/脚本任务、项目特定约定、需 `/skill-name` 快捷入口 |
| Hooks | 编辑后重复动作（format/lint/test）、敏感文件保护、类型检查 |
| Subagents | 需并行专业化审查（安全/性能/可访问性）、大型代码库 |
| Plugins | 需多相关能力打包、团队标准化、首次 Claude Code 搭建 |

### 决策与交互模式

**分析决策逻辑**：
1. 先建立 Codebase Profile（语言、框架、关键库），再逐类匹配信号表。
2. 同类多条候选时，按**与项目痛点的直接关联度**排序（如检测到 Prettier → 自动格式化 Hook 优先于通用通知 Hook）。
3. 无关类别整类跳过（如无前端则跳过 Playwright MCP、ui-reviewer）。
4. 检测到已有 `.claude/` 配置时，推荐应**补缺口而非重复**（与 CLAUDE.md 管理插件的"审计"思维一致）。

**交互模式**：
- **触发短语**：`recommend automations for this project`、`help me set up Claude Code`、`what hooks should I use?`
- **输出格式**：固定 Markdown 模板，每类含 Why、Install/Create、Where。
- **扩展交互**：用户可追问单类更多选项，或请求实施协助（此时离开只读模式）。
- **工具边界**：分析阶段限 Read/Glob/Grep/Bash；不调用 Write/Edit。

## 提取的模式

### 可复用模式

| 模式 | 描述 | 何时使用 |
|------|------|----------|
| **信号驱动推荐引擎** | 将可观测代码库特征映射到具体自动化建议，而非静态清单 | 任何需"按项目定制"的配置推荐场景 |
| **五类自动化分类法** | MCP（外部）→ Skills（工作流）→ Hooks（事件）→ Subagents（并行）→ Plugins（打包） | 设计 AI 编码智能体扩展体系时的分层架构 |
| **Top-N 渐进披露** | 默认每类 1-2 条，按需扩展，防止选项过载 | 复杂工具生态的 onboarding 与顾问式推荐 |
| **只读分析技能** | 分析/推荐与实施严格分离，技能本身不修改文件 | 降低自动化建议的意外副作用风险 |
| **参考库 + 动态搜索混合** | 内置 reference 文件覆盖常见模式，web search 补充长尾集成 | 平衡可维护目录与代码库特异性 |
| **检测模式速查表** | `If You See X → Recommend Y` 表格化决策 | 构建规则引擎、lint 规则、脚手架生成器 |
| **技能调用控制矩阵** | 通过 frontmatter 区分用户触发/Claude 自动/副作用隔离 | 设计 Skills/Commands 的权限与触发策略 |
| **团队配置入库** | `.mcp.json` 等配置文件 git 共享 | 企业团队 AI 工具链标准化 |
| **Headless CI 集成** | `claude -p` + `--allowedTools` + `--output-format stream-json` | 将智能体能力嵌入 pre-commit/CI 管道 |

### 需避免的反模式

| 反模式 | 为何有问题 | 替代方案 |
|--------|------------|----------|
| **一次性倾倒全部选项** | 用户认知过载，无法行动 | Top-N 披露 + 按需深挖单类 |
| **推荐即自动实施** | 未经审批修改配置，可能破坏现有工作流 | 只读分析 → 用户确认 → 另步实施 |
| **无视代码库信号的通用清单** | 推荐与项目无关的 MCP/插件，浪费 token 与维护成本 | 先 Phase 1 分析，再信号映射 |
| **忽略现有 .claude/ 配置** | 重复推荐已安装项，造成冗余 | 分析阶段检测已有配置，推荐补缺口 |
| **Hooks 无权限规划** | PostToolUse 跑测试/format 因权限被拒而静默失败 | 同步推荐 `.claude/settings.json` permissions |
| **MCP 仅个人全局配置** | 团队环境不一致，onboarding 痛苦 | 优先推荐项目级 `.mcp.json` 入库 |
| **Subagent 工具权限过宽** | 审查类代理获得 Write/Bash 可能引入风险 | 审查类只读；生成类按需开放 Write |
| **Skills 无调用边界** | 副作用操作被 Claude 自动触发（如 deploy） | `disable-model-invocation: true` 保护 |

## 业务应用分析

### 直接适用性

| 场景 | 应用方式 | 预期收益 |
|------|----------|----------|
| **新项目 Claude Code 冷启动** | 克隆仓库后立即运行 recommender，按报告逐项落地 Top 推荐 | 30-60 分钟完成基础设施选型，避免盲目试错 |
| **团队 AI 编码标准化** | 将 recommender 报告作为团队 checklist，`.mcp.json` + 共享 skills 入库 | 跨成员输出一致性↑，onboarding 时间↓ |
| **技术栈迁移后重配** | 框架升级（如 CRA→Next.js）后重新分析，更新 MCP/Hooks | 自动化配置与新技术栈对齐 |
| **企业 AI 工具顾问服务** | 将三阶段工作流产品化为"项目 AI 就绪度评估"交付物 | 可复制的咨询方法论 |
| **Monorepo 多包项目** | 按包特征分别分析（结合 CLAUDE.md 分层），推荐包级 skills/conventions | 避免单体配置污染子项目 |
| **安全敏感项目** | 优先落地 protection hooks + security-reviewer + security-guidance 插件 | 降低密钥泄露与 OWASP 风险 |
| **CI/CD 智能化** | 基于 Headless 模式推荐，将 lint-fix、测试生成嵌入管道 | 减少人工干预环节 |

### 适配需求

| 维度 | 原插件能力 | 典型业务适配 |
|------|------------|--------------|
| 参考库覆盖 | 官方 Anthropic 生态为主 | 补充企业内部 MCP、私有插件市场、合规白名单 |
| 信号检测 | Bash + 文件存在性检测 | 接入 SBOM、依赖扫描、架构图谱增强信号 |
| 输出格式 | Markdown 报告 | 可转为 JSON/YAML 直接生成 `.claude/` 脚手架 |
| 只读原则 | 不自动实施 | 企业可增设"审批后一键应用"层（需显式门控） |
| 语言支持 | 以 JS/Python/Go/Rust 为主 | 扩展 Java/Kotlin/移动端检测模式 |

### 投入与回报

- **复杂度**：低（插件即用，单技能驱动）
- **开发时间**：个人项目 30-60 分钟按报告落地；团队标准化 0.5-1 天（含评审与入库）
- **资源需求**：Claude Code 环境、项目读权限、可选 gh CLI / Playwright 等 MCP 前置依赖
- **预期收益**：
  - 冷启动配置时间减少 50-70%（相比自行摸索文档）
  - 减少无关 MCP/插件导致的 token 与维护开销
  - 团队配置一致性显著提升
  - 与 CLAUDE.md 管理、Skills 构建形成可迭代的"配置 → 维护"闭环

## 实施路线图

### 阶段 1：基础建设（15-30 分钟）

1. 安装 `claude-code-setup` 插件：`/plugin install claude-code-setup`
2. 在项目根运行：`recommend automations for this project`
3. 审阅 Codebase Profile 是否准确反映技术栈
4. 记录每类 Top 1-2 推荐及 Why，标记与现有 `.claude/` 配置的重叠项

### 阶段 2：核心实施（1-2 小时）

1. **MCP**：创建/更新 `.mcp.json` 并提交 git（如 context7、GitHub MCP）
2. **Hooks**：在 `.claude/settings.json` 配置 PostToolUse 格式化/lint 与 PreToolUse 保护规则
3. **Skills**：创建 1-2 个高 ROI 自定义技能（如 `project-conventions` Claude-only + `gen-test` user-only）
4. **Subagents**：在 `.claude/agents/` 添加 1 个审查代理（如 security-reviewer 或 code-reviewer）
5. **Plugins**：安装 1-2 个官方插件（如 commit-commands、frontend-design）
6. 配置 `permissions.allow` 确保 Hooks 所需命令可执行

### 阶段 3：集成与优化（持续）

1. 运行 `claude --mcp-debug` 验证 MCP 连通性
2. 重大依赖/架构变更后重新触发 recommender，迭代推荐
3. 配合 `claude-md-management` 插件维护 CLAUDE.md，记录已落地自动化与团队约定
4. 评估 Headless 模式是否适合 pre-commit/CI 场景
5. 按需追问单类更多推荐（如 "show me more MCP server options"）
6. 度量效果：配置后首周返工率、格式化/lint 手动干预次数、审查发现缺陷数

## 快速启动检查清单

- [ ] 已安装 `claude-code-setup` 插件
- [ ] 已运行 `recommend automations for this project` 并保存报告
- [ ] Codebase Profile 与技术栈一致
- [ ] `.mcp.json` 已创建并考虑入库（团队项目）
- [ ] 至少 1 个 PostToolUse Hook（format 或 lint）已配置
- [ ] 至少 1 个 PreToolUse 保护 Hook（.env 或 lock files）已配置（如适用）
- [ ] 至少 1 个项目级 Skill 已创建（project-conventions 或 gen-test）
- [ ] permissions.allow 已覆盖 Hook 所需命令
- [ ] 已与 CLAUDE.md 管理流程衔接，记录自动化配置摘要

## 关键资源

- [原始插件 README](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/README.md)
- [claude-automation-recommender SKILL.md](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/SKILL.md)
- [MCP Servers 参考](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/references/mcp-servers.md)
- [Hooks 模式参考](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/references/hooks-patterns.md)
- [Skills 参考](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/references/skills-reference.md)
- [Subagents 模板](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/references/subagent-templates.md)
- [Plugins 参考](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/skills/claude-automation-recommender/references/plugins-reference.md)
- 关联探索：[CLAUDE.md 管理插件](../2026-06-06-claude-md-management/claude-md-management-insight.md)、[Claude Skills 构建指南](../2026-03-14-claude-skill-building-guide/claude-skill-building-insight.md)