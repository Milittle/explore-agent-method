# CLAUDE.md 管理插件 - 分析与应用指南

> 来源：https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management  
> 插件作者：Isabella He (Anthropic)  
> 分析日期：2026-06-06

## 快速摘要

**CLAUDE.md 管理插件** 是 Anthropic 官方为 Claude Code 提供的**项目记忆生命周期管理工具集**，包含两个互补机制：`claude-md-improver` 技能（代码库驱动的周期性审计与结构化改进）和 `/revise-claude-md` 命令（会话结束时的学习捕获）。其核心方法论是**通过量化质量评分、报告先行审批门控、严格内容价值过滤和多层上下文文件体系**，系统性解决 AI 编码智能体长期使用中的"项目记忆腐烂"（stale context、bloat、缺失关键 gotchas）问题，确保 CLAUDE.md 始终成为高效、可信的常驻上下文来源。

这是一套直接面向生产级 AI 辅助开发实践的**上下文卫生（Context Hygiene）与记忆管理方法论**，与本项目此前探索的上下文工程、Superpowers 设计门控、GSD 工作流、Claude Skills 等高度互补且可直接落地。

## 核心方法论

### 解决的问题

AI 编码智能体（Claude Code、类似工具）依赖 CLAUDE.md 等持久化上下文文件作为"项目记忆"，但在真实使用中普遍出现以下退化：

- **过时（Staleness）**：构建、测试、部署命令已变更，架构描述与实际目录结构不符，引用的文件已删除。
- **膨胀与噪音（Bloat）**：收录显而易见的信息、通用最佳实践、一次性修复，导致 token 浪费且稀释关键信号。
- **缺失关键信息**：非显而易见的 gotchas、跨模块依赖、环境 quirks、有效测试模式从未被记录，导致每轮会话重复发现。
- **无治理的写入**：无约束的自动或随意更新，进一步加剧不一致和不可信。
- **分层混乱**：团队共享信息与个人偏好混杂，单体 vs monorepo 场景缺乏清晰策略。

结果是：上下文窗口被低价值内容占据，智能体重复犯已知错误，长期项目维护成本不降反升。插件直击"记忆腐烂"这一 AI 辅助开发的核心隐性成本。

### 核心概念

1. **分层 CLAUDE.md 体系（Layered Context）**
   - `./CLAUDE.md`：项目根，git 提交，团队共享的主要项目上下文。
   - `./.claude.local.md`：本地覆盖（gitignored），个人偏好与本地环境。
   - `~/.claude/CLAUDE.md`：用户全局默认，跨所有项目。
   - `./packages/*/CLAUDE.md`：monorepo 下的包/模块级上下文（自动被父级发现）。
   - Claude 自动向上发现父目录中的 CLAUDE.md，使 monorepo 天然可用。

2. **六维度质量评分模型（Quantitative Rubric）**
   - Commands/Workflows (20)：关键命令是否完整且带上下文。
   - Architecture Clarity (20)：目录结构、模块关系、入口点是否清晰。
   - Non-Obvious Patterns (15)：gotchas、workarounds、边缘案例、设计缘由。
   - Conciseness (15)：是否无填充、无冗余、每行都有价值。
   - Currency (15)：命令是否真实可执行、引用是否准确、栈是否当前。
   - Actionability (15)：指令是否可直接复制粘贴、路径真实、步骤具体。
   - 等级：A(90-100) / B(70-89) / C(50-69) / D(30-49) / F(0-29)。

3. **报告先行 + 审批门控（Report-First + Human Gate）**
   - **永远先输出完整质量报告**，列出逐文件评分、具体问题、推荐新增项。
   - 仅在用户明确批准后，才使用 Edit 工具进行最小化定向更新。
   - 每次变更必须展示 diff + "Why this helps future sessions" 说明。

4. **严格内容价值过滤器（Value Filter）**
   - **只加**：通过实际代码库分析发现的命令/工作流、gotchas、非显而易见模式、有效测试方法、配置 quirks、包间关系。
   - **绝不加**：代码中已明显的信息、通用最佳实践、一次性修复（one-off）、冗长解释、模板未定制内容、已过时引用。
   - 目标：让每一行都"赚回"它在提示词中占用的 token。

5. **双循环维护机制（Dual Maintenance Loops）**
   - **审计循环（Audit Loop，claude-md-improver）**：由代码库变更或用户请求触发，系统性发现 + 评分 + 报告 + 提议更新。
   - **捕获循环（Capture Loop，/revise-claude-md）**：会话结束时触发，反思"本次会话中什么上下文缺失会让未来 Claude 更有效"，并注入。
   - 两者互补：前者保证"与当前代码库对齐"，后者保证"捕获会话中产生的瞬时洞察"。

6. **模板驱动 + 交叉验证（Templates + Validation）**
   - 提供按项目类型（根项目、monorepo、package）的推荐章节模板。
   - 更新时必须通过实际文件系统/命令验证（read, run mentally or via bash, check existence）。

### 架构概览

```
用户/代码库事件
       │
       ▼
┌─────────────────────┐         ┌─────────────────────┐
│ claude-md-improver  │         │ /revise-claude-md   │
│ (Skill)             │         │ (Command)           │
├─────────────────────┤         ├─────────────────────┤
│ Phase1: Discovery   │         │ Step1: Reflect      │
│ (find all *.md)     │         │ (what was missing)  │
│ Phase2: Assessment  │         │ Step2: Locate files │
│ (rubric 6 dims)     │         │ Step3: Draft concise│
│ Phase3: Report      │◄───────►│ Step4: Show diff+why│
│ (always first)      │  user   │ Step5: Apply w/ OK  │
│ Phase4: Propose     │ approve │                     │
│ (minimal targeted)  │         │                     │
│ Phase5: Edit (gated)│         │                     │
└─────────────────────┘         └─────────────────────┘
       │                                 │
       └──────────────┬──────────────────┘
                      ▼
            多层 CLAUDE.md (git / local / global / pkg)
                      │
                      ▼
            未来 Claude 会话获得更高质量常驻上下文
```

工具受限：Read, Glob, Grep, Bash, Edit（有写权限但受审批保护）。

参考资料（打包在 skill 内）：
- quality-criteria.md：详细 100 分评分细则 + 红旗清单
- templates.md：最小/全面/包级/monorepo 模板
- update-guidelines.md：TO ADD / NOT TO ADD 决策表 + 验证 checklist

### 决策与交互模式

- **发现与验证驱动**：不信任 CLAUDE.md 自述，必须用 Glob/Read/Bash 交叉验证命令可执行性、文件存在性、结构准确性。
- **透明先行**：任何潜在变更前必须向用户呈现结构化报告，降低意外写入风险。
- **最小主义提议**：只提议高价值、会复现的增量；用 diff 形式 + 单句理由呈现。
- **用户作为最终守卫**：所有持久化写入都必须显式批准，体现了"人类判断中心化"。
- **会话闭环**：/revise-claude-md 形成"用完即捕获"的反馈回路，防止瞬时洞察流失。
- **与 Claude 生态集成**：支持 `#` 快捷键在会话中快速触发学习捕获；自然语言即可触发审计（"audit my CLAUDE.md files"）。

## 提取的模式

### 可复用模式

| 模式 | 描述 | 何时使用 |
|------|------|----------|
| 六维度量化质量门控 | 用可打分的 rubric（Commands 20 + Arch 20 + Patterns 15 + Concise 15 + Currency 15 + Actionable 15）对上下文文件做客观评估，并输出分级报告 | 任何需要评估或改进持久化 agent 上下文的场景；定期维护或 onboarding 新项目时 |
| 报告先行 + 审批三段式（Report → Diff+Why → Approve → Edit） | 永远先完整报告问题与建议，展示精确 diff 和收益理由，用户批准后才最小化写入 | 所有会对共享记忆产生副作用的 agent 写操作，防止静默损坏 |
| 严格价值过滤器（Non-obvious / Specific / Actionable / Recurring） | 建立清晰的"只加什么/绝不加什么"决策表，只保留能帮未来会话省时间的项目特有信息 | 防止 CLAUDE.md 成为垃圾堆；所有上下文维护工作流的核心守则 |
| 多层级上下文文件策略 | 明确区分 git 共享（项目级）、gitignored（个人本地）、全局用户级、monorepo 包级四层，并说明发现与优先级规则 | monorepo、大型团队、需要个人偏好隔离的场景 |
| 双循环互补维护（代码驱动审计 + 会话驱动捕获） | 一个循环保证"与当前代码库同步"，另一个循环保证"捕获会话中产生的隐性知识" | 长期活跃的 AI 辅助开发项目；两者结合形成完整记忆生命周期 |
| 模板 + 交叉验证更新 | 提供标准化章节模板 + 要求更新者必须用工具实际验证命令/路径/结构 | 新项目初始化或大规模重构后的记忆重建 |
| 内容极简主义（Dense & Human-readable） | 优先使用表格、一行描述、真实可复制命令；避免长篇解释 | 所有要进入 LLM 提示词的持久文件（不仅是 CLAUDE.md） |

### 需避免的反模式

| 反模式 | 为何有问题 | 替代方案 |
|--------|------------|----------|
| 无报告直接编辑 / 静默更新 | 用户/未来 agent 不知道记忆被改动过什么、为什么，信任度崩塌且难以审计 | 强制 Phase3 完整报告 + diff + why，所有写入走审批 |
| 收录"显而易见"或"通用最佳实践" | 浪费 token，稀释真正有价值的项目特有信号；新会话仍需重新学习真实约束 | 严格执行 NOT TO ADD 清单；只记录代码中看不出的 gotchas |
| 命令/架构未经验证就写入 | 记录了已失效的命令或过时结构，导致 agent 产生错误假设并执行失败操作 | 每次提议更新前必须 Glob/Read/Bash 交叉验证 currency |
| 把一次性修复或 commit 历史写进 CLAUDE.md | 这些信息不会复现，只会让文件膨胀且过时更快 | 只记录"可复现的模式/quirks"；一次性问题留在 git history 或 issue |
| 混淆共享与本地层级 | 个人 .local 偏好污染团队 CLAUDE.md，或反之，导致团队成员看到无关信息或缺少关键本地 setup | 明确使用 .claude.local.md + .gitignore；教育团队分层语义 |
| 模板直接复制未定制 | 产生大量占位符或不适用的章节，降低可信度 | 使用模板作为起点，但必须用本项目真实命令/结构/quirks 填充并验证 |
| 缺少会话闭环捕获 | 会话中发现的宝贵 gotchas 仅存在于当前上下文窗口，未来会话重复踩坑 | 养成 /revise-claude-md 或 `#` 键结束会话习惯 |

## 业务应用分析

### 直接适用性

**极高直接适用**（几乎零适配即可使用）：

- 任何使用 Claude Code（或兼容 Claude 插件系统的 AI 编码环境）的个人开发者、初创团队、企业内部 AI 辅助开发团队。
- 当前项目（explore-agent-method）本身：我们已经重度依赖 CLAUDE.md + AGENTS.md + 多个 skill 目录，定期用本插件审计将显著提升技能质量与索引一致性。
- monorepo 或多包项目：插件原生支持 package 级 CLAUDE.md 发现与管理。
- 长期维护项目（>3 个月活跃开发）：记忆腐烂速度快，收益最高。

**特别高价值场景**：
- 团队多人使用 AI 编码，需保持共享项目记忆一致。
- 频繁上下文切换（多个项目并行）的开发者。
- 希望降低"AI 每次都要重新熟悉项目"的摩擦成本。
- 结合 Superpowers/GSD 等工作流使用时，作为"记忆基础设施"层提供支撑。

### 适配需求

- **几乎无需代码改动**：作为官方插件直接安装使用。
- **团队 adopton**：需要简单约定（何时运行 audit、结束会话必 revise、CLAUDE.md 变更走 PR review）。
- **自定义 rubric**（可选进阶）：可 fork 插件的 references/ 目录，针对特定领域（安全、性能、合规）增加评分维度。
- **与现有技能结合**：本项目的 agent-methodology-explorer 等技能可在其输出末尾自动建议调用 revise-claude-md；或将 improver 的质量报告作为输入喂给其他分析技能。

### 投入与回报

- **复杂度**：低（安装即用，通过自然语言或斜杠命令触发）。
- **开发/设置时间**：5-15 分钟安装 + 第一次完整审计（取决于项目大小）。
- **资源需求**：仅依赖 Claude Code 内置工具（Read/Glob/Edit 等），无额外服务或付费。
- **预期收益**：
  - 显著降低重复发现成本（命令、gotchas）。
  - 提高 agent 首次尝试成功率，减少来回澄清。
  - 长期 token 节省（更 dense、更高信噪比的上下文）。
  - 团队知识沉淀加速，新成员/新会话 onboarding 更快。
  - 与 vibe-coding 洞察结合，可系统性防护"能跑但不可维护"的风险。

ROI 在第一个月内即可通过减少的"Claude 又问我 build 命令是什么"体现。

## 实施路线图

### 阶段 1：基础建设（1-2 天，立即见效）

1. 在 Claude Code 环境中安装 `claude-md-management` 插件（通过官方插件市场或仓库）。
2. 在当前仓库根执行第一次审计："audit my CLAUDE.md files" 或直接调用 improver 技能。
3. 仔细阅读生成的 Quality Report，按评分优先级处理最差文件（通常是 root CLAUDE.md）。
4. 对每项推荐更新，逐条评估是否符合"只加高价值"原则，批准后应用。
5. 建立个人习惯：任意会话结束前运行 `/revise-claude-md`（或按 `#` 键）。
6. 将 `.claude.local.md`（如有）加入 `.gitignore`。

### 阶段 2：核心实施与习惯固化（1 周）

1. 把 CLAUDE.md 审计纳入常规节奏：重大代码变更后、发布前、或每 1-2 周主动运行一次。
2. 在团队内部共享"CLAUDE.md 维护公约"（参考本报告反模式表）。
3. 对于 monorepo，识别需要独立 CLAUDE.md 的关键包，初始化最小模板并填充真实内容。
4. 将 improver 报告中的高频问题类型反馈给团队，驱动上游改进（如统一命令命名、文档化 quirks）。
5. 结合本项目其他技能：在 agent-methodology-explorer 等探索类技能的输出模板中增加"建议的 CLAUDE.md 增量"段落，并自动触发 revise 提议。
6. 验证效果：对比使用前后的会话中"需要 Claude 重新发现项目信息"的次数。

### 阶段 3：集成与优化（持续，2-4 周后）

1. 针对领域特点扩展质量 criteria（例如为安全敏感项目增加"威胁模型/权限边界"维度）。
2. 探索自动化触发点：git hook 或 CI 中轻量检查（检测明显 stale 命令），失败时提示人工运行 improver。
3. 建立 CLAUDE.md 变更的 PR 模板，要求变更者附上 improver 报告片段或 revise 理由。
4. 在更大范围内推广：为常用脚手架项目预置高质量初始 CLAUDE.md（使用本插件的模板 + 本项目实践）。
5. 度量与迭代：跟踪平均质量分数随时间的变化、典型会话中 CLAUDE.md 命中率、用户主观"Claude 懂项目程度"评分。
6. 与上下文工程、Superpowers、GSD 等方法论深度融合，形成"工程前置 + 记忆卫生 + 流程门控"的完整栈。

## 五、关键洞察总结

- **记忆不是静态文档，而是需要主动治理的活系统**。CLAUDE.md 的价值取决于其 currency、density 与 specificity，而非长度。
- **写操作必须有护栏**。报告先行 + 人类审批 + 最小 diff 是防止"好心办坏事"的关键架构决策。
- **双循环缺一不可**。仅靠审计会丢失会话中产生的宝贵瞬时知识；仅靠会话捕获会与代码库逐渐脱节。
- **过滤比收集更重要**。在 token 宝贵的时代，"不写什么"的纪律比"写什么"更能决定长期系统健康。
- **本插件是基础设施层**，而非独立方法论。它为 Superpowers 的设计门控、GSD 的验证模式、Claude Skills 的常驻约束、上下文工程的持久化层提供了可落地的记忆维护执行机制。

## 六、资源与参考

- 原始插件仓库：https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management
- 核心文件：
  - SKILL.md（完整 5 阶段工作流）
  - commands/revise-claude-md.md（会话捕获流程）
  - references/quality-criteria.md（100 分 rubric 详解 + 红旗）
  - references/templates.md（各类项目模板）
  - references/update-guidelines.md（TO ADD / NOT TO ADD + 验证 checklist）
- 推荐结合阅读：
  - 本项目之前的 "Vibe Coding Is Not Engineering"（工程前置决策与本插件的记忆治理形成闭环）
  - "Superpowers 智能体技能框架"（设计门控 + 两阶段审查可与 improver 报告联动）
  - "Claude Skills 完整构建指南"（技能本身也需要良好的 CLAUDE.md 作为底层支撑）
  - "GSD 工作流系统"（plan/verify 阶段可产出高质量的 revise 材料）

---

**质量检查清单（完成确认）**：
- [x] 识别了解决的根本问题（记忆腐烂）而非仅描述功能
- [x] 提取了可复用的模式（量化门控、价值过滤、双循环、审批三段式等）
- [x] 提供了具体实施步骤（三阶段路线图 + 立即可执行的阶段1动作）
- [x] 考虑了业务背景（直接适用于所有 Claude Code 用户，ROI 清晰）
- [x] 做出了现实复杂度与投入评估（低复杂度、高即时回报）
- [x] 包含了用户可立即采取的第一步（安装 + 首次审计 + 结束会话 revise）
- [x] 将更新 `docs/index.md` 和 `README.md`（本报告生成后立即执行）
