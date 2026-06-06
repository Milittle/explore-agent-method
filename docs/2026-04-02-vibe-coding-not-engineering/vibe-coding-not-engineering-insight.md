# Vibe Coding Is Not Engineering - 分析与应用指南

## 快速摘要

本文系统性论证了“Vibe Coding”（通过提示让LLM或非软件工程师自动生成代码）与真正“Engineering”（工程）之间的根本区别：前者只能产出可立即运行的 demo 代码，后者通过在任何代码存在之前完成的系统性决策（不变量、身份同一性、约束、失败模式、边界、契约等），确保生成的代码能在真实生产运行时系统中长期安全、一致、可维护地运行。这不是反对 AI，而是主张必须把工程判断放在代码生成之前。对于当前所有 AI 智能体辅助开发方法论而言，这是一份重要的第一性原理警示和补全框架。

## 核心方法论

### 解决的问题

LLM（包括各类 coding agent）擅长统计模式延续，能快速生成“看起来正确”的代码片段，但无法感知：

- 运行时系统的真实上下文（已有的不变量、状态转换规则、跨组件耦合）
- 业务需求的完整集合（尤其是隐式约束、边缘案例、合规与非功能需求）
- 变更的因果后果（修改现有系统 vs 新增代码的根本差异）

结果是：demo 容易做出，生产系统却因缺失关键工程决策而迅速出现身份歧义、状态不一致、功能碰撞、部署脆弱等问题。文章核心论断：“Vibe coding gets you a demo. This demo is disposable. To take that demo forward into production requires engineering.”

### 核心概念

1. **Vibe Coding**：非软件工程师通过自然语言提示自动生成代码的过程。核心特征是“跳过前置工程决策，直接要结果代码”。
2. **Engineering（工程）**：在编写/生成任何代码之前，人类（工程师）必须完成的使系统保持连贯的所有工作。代码只是这些决策的实现载体。
3. **系统 ≠ 生成的代码文本**：真正的系统是“代码 + 它所运行的环境 + 操作规则 + 不变量 + 约束 + 失败行为”的整体。
4. **Additive vs Transformative（来自 companion 文章）**：
   - Additive（添加式）：新增自包含功能，LLM 较强（pattern extension）。
   - Transformative（转换式）：修改已有系统中的代码/配置，会改变系统因果结构，需要因果推理（causal reasoning），LLM 根本弱项。
5. **LLM 的本质限制**：LLM 做的是 token 预测 / 模式匹配，而不是构建和维护一个“必须被一致修改的系统”的内部因果模型。

### 架构概览

本文并未提出新的智能体架构，而是给出了一个**工程前置决策框架**，必须在任何代码生成（无论人工还是 agent）之前执行。

关键结构可理解为两个并行的活动层：

- **工程决策层**（人类/强约束过程主导）：问题框架 → 需求工程 → 系统建模 → 架构设计 → NFR 定义 → 风险识别 → 接口契约 → 规划排序。
- **代码实现层**（LLM/agent 可加速）：仅在上述决策明确后，才进行模式填充。

文章用两个核心表格清晰呈现了差距：

**工程师在代码前完成的工作（What vibe coding skips）**

| Topic | Description |
|-------|-------------|
| Problem framing | 定义问题、用户、约束、预期结果 |
| Requirements engineering | 引出行为、不变量、边缘案例、验收标准 |
| System modelling | 状态模型、数据流、序列图、因果推理 |
| Architectural design | 边界、职责、接口、失败模式、权衡 |
| Non-functional requirement definition | 性能、可靠性、安全、合规、可运维性、成本 |
| Risk identification | 未知、依赖、失败点、缓解措施 |
| Interface and contract design | 定义 API、Schema、行为保证 |
| Planning and sequencing | 将工作拆分为连贯、可交付单元 |

**具体被跳过的工程领域（最致命的 9 大类）**

| Area | What engineers decide | What LLMs generate |
|------|-----------------------|--------------------|
| **Invariants** | 什么必须始终为真 | 假设一切顺利的代码 |
| **Identity & Uniqueness** | 什么让一个实体成为“同一个东西” | 把一切当作可互换的 |
| **Constraints** | 系统绝不允许什么 | 在不知情的情况下违反边界 |
| **Failure Modes** | 系统在压力下如何行为 | 只在 happy path 工作的代码 |
| **Coupling & Sequencing** | 什么依赖什么、以什么顺序 | 完全忽略顺序 |
| **State & Transitions** | 状态如何变化、由什么触发 | 无规则地变异状态 |
| **Interfaces & Contracts** | 每个部分对其他部分承诺什么 | 暴露模型临时发明的任何东西 |
| **Boundaries** | 系统明确不做什么 | 代码持续膨胀直到崩溃 |
| **Error Handling** | 系统如何恢复 | 遇到第一个意外输入就崩塌 |

经典案例：“为网站添加用户账号登录” —— LLM 快速给出 Flask+SQLite demo，但完全不追问“邮箱是否唯一？是否需要验证？角色模型？密码重置策略？安全模型？”。缺少邮箱唯一性约束，会导致重复邮箱注册 → 登录身份歧义 → 密码重置影响所有同邮箱用户 → 删除账号波及多用户等生产灾难。

### 决策与交互模式

- **LLM 决策模式**：纯前向模式匹配，无系统状态的持久内部表征，无法追踪“这次变更将如何破坏哪些已有不变量”。
- **工程决策模式**：显式建模因果链、边界、压力场景，并将这些作为**代码必须遵守的硬约束**注入后续实现。
- **在智能体流程中的位置**：工程决策必须作为**前置门控**（pre-gate），由人类（或极强约束的 review agent）完成；代码生成 agent 只能在决策上下文中运行。

这与本项目此前探索的多个方法论（Superpowers 的设计门控 + 两阶段审查、GSD 的 discuss/plan/verify 阶段、Claude Skills 的渐进式披露）形成了高度互补：那些方法论正是试图用流程和技能把“本文所缺失的工程层”强制注入到 LLM 驱动的工作流中。

## 提取的模式

### 可复用模式

| 模式 | 描述 | 何时使用 |
|------|------|----------|
| 工程前置决策清单 | 在触发任何代码生成前，强制完成 invariants、identity、constraints、failure modes、boundaries、contracts 等领域的显式定义，并以结构化形式（文档、prompt 上下文、review checklist）固化 | 所有 AI 辅助开发场景，尤其是涉及状态、用户身份、跨组件交互或生产环境的任务 |
| 不变量与同一性优先建模 | 显式回答“什么必须永远为真？”“什么让这个实体是它自己（而非另一个）？”作为代码生成的顶级约束 | 用户系统、订单/支付、配置、任何有“唯一性”或“幂等”要求的地方 |
| 失败模式前置分析 | 在写 happy path 代码之前，先定义 top 3-5 种压力/错误/降级/恢复场景，并要求实现必须覆盖 | 高可用系统、关键业务流程、金融/医疗/合规相关 |
| 边界与“非功能”显式化 | 清晰写下“系统不做什么”和所有 NFR（性能、成本、安全、运维），防止功能蔓延和隐性技术债 | 长期演进的单体或分布式系统 |
| 阶段门控 + 人类判断中心 | 将工作流严格分为“工程决策阶段（人类/强审查）”和“实现阶段（agent 加速）”；人类始终保留对 intent、constraints、trade-offs 的最终判断权 | 任何规模的 AI 编码团队或 solo 开发者生产化路径 |
| Additive / Transformative 区分策略 | 新增独立模块 → 可更激进地用 agent；修改核心路径/已有组件 → 必须更重的工程前置 + 人工审查 + 影响模拟 | 遗留系统改造、核心重构、长期维护项目 |

### 需避免的反模式

| 反模式 | 为何有问题 | 替代方案 |
|--------|------------|----------|
| 提示词工程万能论 | 认为只要 prompt 写得足够聪明，LLM 就能自动发现缺失的不变量和约束 | 改为前置结构化工程决策 + checklist 门控；prompt 只能传递已决策的内容 |
| “能跑就是好系统”幻觉 | 看到 demo 代码执行通过，就默认生产就绪 | 要求通过“不变量清单 + 失败场景 + 身份规则”的显式验证才能进入下一阶段 |
| 纯 Additive 思维 | 只关注“加新功能”，忽略对已有系统因果结构的破坏性影响 | 强制在计划阶段产出“变更对现有不变量和下游的影响分析” |
| 缺少因果链内部模型 | LLM 仅做文本延续，无法维护“系统当前状态 + 变更后状态”的一致因果表征 | 人类负责维护高层次系统模型（文档、架构图、INVARIANTS.md），agent 仅在局部使用 |
| Demo 即产品思维 | 把快速生成的原型直接当生产基线，跳过所有长期运行所需的工程制品 | 原型之后必须走完整的工程决策 + 验证门控流程 |
| 边界隐式膨胀 | LLM 会为了完成当前 prompt 而自然扩展职责，导致单点越来越胖、耦合越来越乱 | 显式“系统边界声明”作为前置产物，agent 生成代码必须遵守 |

## 业务应用分析

### 直接适用性

本文洞察可**最小适配直接应用**的场景包括：

- 所有使用 Claude Code、Cursor、Windsurf、GitHub Copilot Workspace、Replit Agents 等工具的个人和团队。
- 本项目此前已探索的多个智能体方法论的**直接补强**：
  - Superpowers：其“设计门控 + 新鲜子智能体 + 两阶段审查”正是本文工程前置思想的刚性落地。
  - GSD 工作流：discuss-phase → plan-phase → verify 阶段，本质是在做本文要求的“问题框架 + 需求 + 模型 + 规划”。
  - 上下文工程系列：通过文件系统持久化 invariants、架构决策、系统模型，正是解决“LLM 看不到完整系统上下文”的工程化方案。
  - Claude Skills：可将本文的工程决策清单直接封装为常驻 skill，在每次 coding 会话中强制加载。
- Solo 开发者从原型快速走向可维护产品的路径。
- 企业想规模化使用 AI 编码但不想积累隐性技术债的场景。

### 适配需求

典型业务使用需要做的改变：

- 工作流习惯：从“直接扔需求给 AI”改为“先花 10-30 分钟完成工程决策文档/清单”。
- 制品要求：在代码仓库中增加与代码同等重要的工程制品（INVARIANTS.md、CONSTRAINTS.md、FAILURE-MODES.md、ARCHITECTURE-DECISIONS.md）。
- Agent 集成：把工程决策作为 agent 的**只读高优先级上下文**（类似 Superpowers 的 skills 优先级或 GSD 的 PROJECT.md + CONTEXT.md）。
- 审查文化：把“是否做了工程前置决策”作为 PR / 变更的必检项，而非可选。

### 投入与回报

- **复杂度**：中（主要是思维模式和流程习惯的转变，不是技术实现难度）
- **开发时间**：单任务前置增加 15-40% 时间；但全生命周期返工、事故修复、理解成本可降低 50-80%
- **资源需求**：低（主要是 checklist、模板、少量文档制品）；可通过 Skill / meta-prompt / CI 检查点固化
- **预期收益**：
  - 生产事故率和“神秘 bug”大幅下降
  - 系统可维护性与演进速度提升
  - AI 真正成为“工程产能放大器”而非“技术债加速器”
  - 团队对 AI 输出的信心从“看起来能跑”提升到“知道它为什么安全”

## 实施路线图

### 阶段 1：基础建设（1-3 天，个人或小团队）

1. 将本文两个核心表格（工程师前置工作 + 9 大被跳过领域）制作成可打印 / Notion / Obsidian 模板，作为每次 AI coding 任务的标准前置检查单。
2. 在当前正在使用的 agent 工作流（Superpowers / GSD / 自定义 Claude 项目 / Cursor rules 等）中，**硬性插入 Engineering Decision Gate**：列出至少 invariants、identity/uniqueness、top constraints、top 3 failure modes 后，才允许进入实现阶段。
3. 针对高频领域（用户/账号体系、订单/支付、配置管理、数据实体）预制 3-5 个工程决策模板，降低每次的认知负担。
4. 阅读 companion 文章《Agents Cannot Maintain Systems》，理解 Additive vs Transformative 的实战区分。

### 阶段 2：核心实施（1-2 周）

1. **系统上下文工程化**（强烈推荐结合此前“上下文工程”与“GSD”报告）：在仓库根部维护 INVARIANTS.md + BOUNDARIES.md + KEY-DECISIONS.md，作为 agent 的必加载只读上下文。
2. 区分任务类型实施不同策略：
   - 新增独立、边界清晰的功能 → 可更激进地使用 agent + 较轻前置。
   - 修改核心路径、跨组件、已有状态的变更 → 必须重度工程前置 + 人工 + 可能的影响模拟。
3. 在 plan / design 阶段强制产出“此变更对现有不变量和下游系统的影响分析”（可用 agent 辅助草稿，但必须人工确认）。
4. 引入轻量验证：任何涉及身份/唯一性/约束的变更，必须有对应测试或断言作为门控。

### 阶段 3：集成与优化（持续 + 团队级）

1. 将本文洞察 + 配套 checklist 打包为可复用的 **Claude Skill**（命名为 engineering-decision-enforcer 或 vibe-coding-guard 或类似），在所有 coding 会话中以最高优先级加载。
2. 结合 MCP 工具、linter、架构测试（ArchUnit / 自定义脚本）实现部分工程约束的自动化守护。
3. 团队/组织级：将“工程决策制品”纳入 Definition of Done 和代码审查 checklist；对关键系统建立“变更因果模拟”复盘文化。
4. 持续追踪指标：生产 incident 中“因缺失工程决策导致”的占比是否下降；AI 生成代码后的平均返工次数；新功能从 idea 到可维护生产的时间。

## 快速启动检查清单

- [ ] 完整阅读主文章 + companion《Agents Cannot Maintain Systems》
- [ ] 为你下一个 AI 辅助任务，**在写任何 prompt 之前**，至少显式回答以下问题：
  - 这个实体的同一性规则是什么？（什么让它成为它自己）
  - 有哪些必须永远为真的不变量？
  - 系统绝对不允许发生什么？
  - 前 3 个最可能的失败/压力场景是什么？系统该如何反应？
- [ ] 在你当前使用的 agent 工作流（无论 GSD、Superpowers、自定义 prompt 还是 IDE agent）中，增加一个不可跳过的“Engineering Gate”步骤
- [ ] 挑选一个最近用 AI 生成的模块，用本文 9 大领域表格做一次事后审查，找出至少 2-3 处被跳过的工程决策
- [ ] 在仓库中创建或更新 INVARIANTS.md（或等价文件），至少写入 5 条当前系统的核心不变量
- [ ] 下一次做 Transformative 变更（修改已有代码而非纯新增）时，额外花时间做一次“变更对现有系统的影响”显式分析

## 关键资源

- **原始来源**：https://phroneses.com/articles/build/notes/vibe-coding-is-not-engineering.html
- **核心 companion**（强烈推荐）：https://phroneses.com/articles/build/notes/agents-cannot-maintain-systems.html （Additive–Transformative Gap，解释了为什么 agent 难以自主交付真实系统变更）
- 同作者相关文章（同系列第一性原理思考）：
  - Agents Cannot Maintain Systems
  - What Software Engineers Need to Know About LLMs
  - Latency Is Architectural
  - The Big AI Gains Come From Teams, Not Individuals
- **与本项目其他探索的直接关联**：
  - Superpowers 智能体技能框架（设计门控、刚性约束、两阶段审查）
  - GSD 工作流系统（元提示、上下文分层、验证模式、原子提交）
  - 上下文工程系列（系统模型的持久化与按需加载）
  - Claude Skills 构建方法论（可将工程决策清单封装为常驻能力）
  - Anthropic Cookbook（工作流模式中的评估与优化器可用于工程约束验证）

---

*本报告基于 2026-04-02 对 https://phroneses.com/articles/build/notes/vibe-coding-is-not-engineering.html 及 companion 文章的系统分析生成。核心价值在于为 AI 智能体方法论提供缺失的“工程判断层”第一性原理支撑。*
