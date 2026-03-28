# 智能体方法论探索 - 文档索引

> 本目录包含通过agent-methodology-explorer技能生成的所有洞察报告

---

## 📊 探索报告列表

### 2026年3月

| # | 洞察目录 | 来源 | 核心主题 | 关键洞察 |
|---|----------|------|----------|----------|
| 01 | [上下文工程洞察](context-engineering-insight-2026-03-06/) | GitHub | 上下文工程、多智能体、记忆系统 | 87% token减少、工具整合3.5倍提升 |
| -- | ├── [深度洞察报告](context-engineering-insight-2026-03-06/context-engineering-agent-skills-insight.md) | | 完整方法论分析 | 6大设计模式详解 |
| -- | └── [快速参考指南](context-engineering-insight-2026-03-06/context-engineering-quick-reference.md) | | 原则速查表 | 5分钟立即行动 |
| 02 | [文件系统上下文工程洞察](file-system-context-engineering-2026-03-07/) | arXiv论文 | 文件系统抽象、上下文工程架构 | AIGNE框架、PCR存储模型 |
| -- | ├── [深度洞察报告](file-system-context-engineering-2026-03-07/file-system-context-engineering-insight.md) | | AFS架构范式 | 可验证、可维护的GenAI系统 |
| -- | └── [快速参考指南](file-system-context-engineering-2026-03-07/file-system-context-engineering-quick-reference.md) | | 架构决策树 | 工业级实施指南 |
| 03 | [Anthropic Cookbook 洞察](anthropic-cookbook-2026-03-11/) | GitHub（官方） | 五大工作流模式、SDK四阶段教程、生产级智能体 | 34.7k Stars、10大可复用模式、完整实施路径 |
| -- | ├── [深度洞察报告](anthropic-cookbook-2026-03-11/anthropic-cookbook-insight.md) | | 三层递进架构模型 | 五大模式+SDK完整分析 |
| -- | └── [快速参考指南](anthropic-cookbook-2026-03-11/anthropic-cookbook-quick-reference.md) | | 目录导航+决策树 | 3分钟快速启动 |
| 04 | [AutoHarness LLM智能体框架](autoharness-llm-agent-2026-03-13/) | arXiv论文 | 自动代码框架合成、非法操作消除、小模型超越大模型 | 145款游戏100%合法率、Flash超越Pro、Policy模式推理零成本 |
| -- | ├── [深度洞察报告](autoharness-llm-agent-2026-03-13/autoharness-insight.md) | | 6大设计模式、业务迁移路径 | Thompson采样+迭代精化架构 |
| -- | └── [快速参考指南](autoharness-llm-agent-2026-03-13/autoharness-quick-reference.md) | | 三类Harness选择决策树 | 实施模板+性能对比表 |
| 05 | [Claude Skills 完整构建指南](claude-skill-building-guide-2026-03-14/) | 官方PDF文档 | Skills系统架构、五大设计模式、MCP+Skills完整方案 | 渐进式三层加载、15-30分钟构建首个技能、90%触发率基准 |
| -- | ├── [深度洞察报告](claude-skill-building-guide-2026-03-14/claude-skill-building-insight.md) | | 完整技术规范+5大模式+实施路线图 | YAML规范+反模式表+量化成功标准 |
| -- | └── [快速参考指南](claude-skill-building-guide-2026-03-14/claude-skill-building-quick-reference.md) | | Description公式+排查树+30分钟速建 | 禁止项清单+三端场景速查 |
| 06 | [Superpowers 智能体技能框架](superpowers-skill-framework-2026-03-20/) | GitHub | AI编码智能体工作流、15+组合技能、工程纪律强制注入 | 101k Stars、10大可复用模式、设计门控+子智能体驱动开发 |
| -- | ├── [深度洞察报告](superpowers-skill-framework-2026-03-20/superpowers-insight.md) | | 完整7阶段工作流+10大模式+实施路线图 | 两阶段审查+新鲜子智能体+TDD刚性约束 |
| -- | └── [快速参考指南](superpowers-skill-framework-2026-03-20/superpowers-quick-reference.md) | | 工作流速查+决策树+30分钟接入 | 反模式清单+场景速查表 |
| 07 | [Browser-Use Web 智能体框架](browser-use-web-agent-2026-03-24/) | GitHub | 四阶段DOM管道、认知循环Schema、LLM驱动浏览器自动化 | 9大可复用模式、升级干预循环检测、CDP并行采集架构 |
| -- | ├── [深度洞察报告](browser-use-web-agent-2026-03-24/browser-use-insight.md) | | 完整架构分析+5大核心洞察+实施路线图 | DOM-to-LLM管道+认知循环+安全凭证隔离 |
| -- | └── [快速参考指南](browser-use-web-agent-2026-03-24/browser-use-quick-reference.md) | | 5分钟快速启动+决策树+生产配置清单 | 常见问题修复+与其他方法论结合点 |
| 08 | [GSD 工作流系统](get-shit-done-gsd-workflow-2026-03-28/) | GitHub | 元提示、上下文工程、规范驱动开发系统 | 43.7k Stars、10大可复用模式、编排者-子智能体架构 |
| -- | ├── [深度洞察报告](get-shit-done-gsd-workflow-2026-03-28/gsd-workflow-insight.md) | | 上下文衰减解决方案+完整架构+实施路线图 | 波次并行执行+原子Git提交+验证模式 |
| -- | └── [快速参考指南](get-shit-done-gsd-workflow-2026-03-28/gsd-workflow-quick-reference.md) | | 命令速查表+决策树+快速实施模板 | Token优化清单+常见问题修复 |

---

## 🔥 核心发现汇总

### 上下文工程 (Context Engineering)

**问题**：LLM上下文窗口有限，简单增加context长度会导致性能下降和成本激增

**解决方案**：
1. **渐进式披露** - 按需加载（三层结构）
2. **上下文隔离** - 多智能体分割上下文
3. **工具整合** - 合并相关工具减少选择困难
4. **观察掩码** - 工具输出写入文件系统而非上下文

**验证数据**：
- Digital Brain: 5000→650 tokens (87%减少)
- Vercel d0: 17→2工具，80%→100%成功率，3.5倍提速

### 文件系统抽象 (Agentic File System)

**问题**：GenAI系统上下文管理碎片化、不可追溯、难治理

**解决方案**：
1. **AFS架构** - "一切皆文件"哲学应用于上下文工程
2. **PCR存储** - History/Memory/Scratchpad三层持久化上下文库
3. **CEP管道** - Constructor/Updater/Evaluator自动化上下文工程
4. **可验证性** - 从ad-hoc实践转向系统化软件工程

**实现框架**：AIGNE（开源）提供工业级GenAI系统基础设施

### Anthropic Cookbook (官方最佳实践库)

**问题**：开发者缺乏智能体架构参考实现，无法从示例代码跨越到生产系统

**解决方案**：
1. **三层递进模型** - 基础能力 → 工作流模式 → 生产级智能体
2. **五大工作流模式** - Chaining/Routing/Parallelization/Orchestrator-Workers/Evaluator-Optimizer
3. **SDK 四阶段教程** - 单行研究智能体 → 执行助理 → 可观测性 → SRE
4. **MCP 集成** - 通过 MCP Server 接入外部系统（Git 13工具、GitHub 100工具）

**规模**：34.7k Stars，500+ Commits，MIT 开源

### AutoHarness (自动代码框架合成)

**问题**：LLM智能体在结构化任务中频繁产生非法操作（国际象棋竞赛中78%失败源于非法着法），且传统约束方法（提示词、微调）无法保证100%合法率

**解决方案**：
1. **自动Harness合成** - 让LLM自动生成任务特定的代码约束层，而非手工编写
2. **三种Harness类型** - Action-Filter（枚举合法集）/ Action-Verifier（验证+重试）/ Harness-as-Policy（纯代码策略）
3. **迭代精化搜索** - Thompson采样平衡代码优化的探索与利用，平均14.5次迭代收敛
4. **失败驱动学习** - 环境合法性反馈直接作为训练信号，无需人工标注

**验证数据**：145款游戏100%合法率；Gemini-2.5-Flash+Harness胜率56.3% vs Pro 38.2%；Policy模式单人游戏奖励0.870 vs GPT-5.2-High 0.844，推理成本≈$0 vs ~$640

### Claude Skills 构建方法论

**问题**：开发者缺乏标准化方法将领域知识和工作流封装为可复用的 Claude 能力，每次对话需重新解释偏好和流程，MCP 集成用户学习曲线高

**解决方案**：
1. **渐进式三层披露** - YAML frontmatter（始终加载）→ SKILL.md 正文（按需加载）→ references/ 文件（按需导航），最小化 token 消耗
2. **五大设计模式** - 顺序工作流 / 多MCP协调 / 迭代精化 / 上下文感知工具选择 / 领域特定智能
3. **MCP + Skills 完整方案** - MCP 提供"厨房"（工具连接），Skills 提供"菜谱"（工作流知识）
4. **skill-creator 元技能** - 使用 Claude 自身构建技能，15-30分钟完成首个技能

**关键规范**：description 字段是成败关键（必须含触发短语）；脚本验证优于语言指令；同时启用技能上限 20-50 个

### Superpowers 智能体技能框架

**问题**：AI 编码智能体存在三类核心质量缺陷——急于实现（跳过设计）、跳过测试（测试后置无效）、虚假完成（无证据声明完成），导致高返工率和低代码质量

**解决方案**：
1. **设计门控模式** - 硬性阻断所有实施动作，直到设计文档获得用户审批；每个任务均有规格文档
2. **新鲜子智能体模式** - 每任务一个无历史污染的新会话子智能体，彻底隔离上下文污染
3. **两阶段审查** - 规格合规审查（方向正确？）先于代码质量审查（做得好？），顺序锁定
4. **刚性 TDD 约束** - 无测试的代码必须删除重来，非可选项；四阶段根因调试强制执行
5. **技能优先级体系** - Skills 优先级高于默认系统提示，"1%概率适用就必须调用"

**关键洞察**：将工程纪律从"最佳实践建议"升级为"智能体必须执行的刚性约束"；101k Stars 验证了这一方向的普遍性需求

### Browser-Use Web 智能体框架

**问题**：网页为人类视觉系统设计，原始 HTML 对 LLM 而言充满噪音；浏览器操作需要精确 DOM 引用，单靠 LLM 文本输出无法完成；生产环境需应对循环卡死、速率限制、上下文膨胀等挑战

**解决方案**：
1. **四阶段 DOM-to-LLM 管道** - CDP 并行采集 → EnhancedDOMTreeNode 构建 → 序列化过滤 → 分页检测，将网页转化为 LLM 友好的结构化表示
2. **认知循环 Schema** - 强制 `evaluation_previous_goal + memory + next_goal` 字段，迫使 LLM 每步自我评估、维护记忆、明确下步目标
3. **升级干预模式** - 5/8/12 步三级阈值渐进增强干预提示，平衡"坚持探索"与"放弃死路"
4. **Fallback LLM + MessageCompaction** - 主备 LLM 自动切换应对速率限制；历史消息压缩控制长任务 token 成本
5. **域名隔离凭证** - 架构层强制凭证仅注入白名单域名，不依赖提示词安全约束

**关键洞察**：DOM 处理管道质量决定 Web 智能体上限；认知 Schema 设计（而非单纯 prompt）是可靠性的核心；安全必须在架构层而非提示词层保证

### GSD (Get Shit Done) 工作流系统

**问题**：AI 辅助编程中的"上下文衰减"（context rot）现象——当 AI 的上下文窗口逐渐被填充时，输出质量会下降；Vibecoding 信誉危机——用户描述需求 → AI 生成代码 → 得到不一致的垃圾；现有工具过度复杂化（引入 sprint 仪式、story points、Jira 工作流）

**解决方案**：
1. **元提示（Meta-prompting）** - 系统生成结构化提示（XML 格式），而非用户直接输入自然语言
2. **上下文分层加载** - PROJECT.md（全局）+ CONTEXT.md（阶段）+ PLAN.md（任务），按需加载控制上下文窗口使用
3. **编排者-子智能体模式** - 轻量级编排者协调专业化子智能体，主上下文保持 30-40% 使用率，子智能体在全新 200k token 上下文中工作
4. **波次并行执行** - 基于依赖关系将计划分组，波次内并行，波次间串行；垂直切片（端到端功能）比水平切片（按层）更好并行化
5. **原子 Git 提交** - 每个任务独立提交，可追溯、可回滚、支持 git bisect
6. **验证模式** - 自动验证（代码存在、测试通过）+ 人工验证（功能符合预期），失败自动诊断生成修复计划

**关键架构组件**：
- **工作流循环**：new-project → discuss-phase → plan-phase → execute-phase → verify-work → ship → complete-milestone → new-milestone
- **阶段讨论模式**：识别灰色领域（视觉特性、API/CLI、内容系统、组织任务）针对性提问生成 CONTEXT.md
- **假设模式**：代码库完善时用假设模式（分析代码库提出假设，用户纠正），全新项目用讨论模式
- **快速模式**：临时任务跳过可选步骤，保留核心保证（原子提交、状态跟踪）
- **工作流（Workstreams）**：命名空间隔离的并行里程碑工作

**验证数据**：43.7k+ Stars，3.5k+ Forks，98+ 贡献者，41 个 releases；支持 8 个 AI 运行时（Claude Code、OpenCode、Gemini CLI、Codex、Copilot、Cursor、Windsurf、Antigravity）

---

## 📋 技能映射表

| 业务场景 | 推荐技能 | 预期收益 |
|----------|----------|----------|
| 客服智能体 | 上下文基础、记忆系统、工具设计 | 50-70%成本降低 |
| 研究分析 | 多智能体、上下文优化、评估 | 并行处理，深度分析 |
| 内容生成 | 项目开发、工具设计、评估 | 可扩展，一致质量 |
| 知识管理 | 记忆系统、BDI心智状态 | 持续学习，关系洞察 |
| API调用/SQL生成智能体 | AutoHarness代码护栏模式 | 消除非法调用，100%合规率 |
| 高频规则化任务（RPA/交易） | Harness-as-Policy模式 | 推理零LLM成本，极低延迟 |
| 企业团队标准化工作流 | Claude Skills + 工作区级部署 | 跨团队输出一致性↑，onboarding↓ |
| SaaS产品 MCP 集成增值 | MCP + Skills 套装模式 | 用户上手时间↓，支持票量↓，竞争优势↑ |
| 生产级自动化管道 | Skills API + container.skills | 程序化调用，大规模稳定部署 |
| AI 辅助软件开发质量保障 | Superpowers 完整工作流 | 设计偏差↓、TDD覆盖↑、假阳性完成率↓ |
| 复杂遗留系统 Bug 修复 | Superpowers systematic-debugging | 四阶段根因调查，防止随机修复扩散 |
| 并行特性开发 | Superpowers dispatching-parallel-agents | 独立任务并发，缩短开发周期 |
| 无 API 的 Web 数据采集/操作 | Browser-Use DOM管道+认知循环 | 替代人工操作，80-95% 时间节省 |
| 传统 RPA 替代（表单/申报） | Browser-Use + 自定义 Controller | 开发成本↓60-70%，自然语言任务比 CSS 选择器更易维护 |
| LLM 驱动 E2E UI 测试 | Browser-Use + AgentHistory 审计 | 测试编写↓50%，对 UI 变更更鲁棒 |
| Solo 开发者/独立黑客快速原型到产品 | GSD 完整工作流 | 企业级系统化能力，无需假装大公司，50-80% 手动编码时间减少 |
| 现有代码库系统性添加新功能 | GSD map-codebase + 假设模式 | 理解现有模式，新代码一致，不破坏现有功能 |
| AI 编程团队一致性和质量保障 | GSD 编排者-子智能体 + 验证模式 | 上下文衰减↓，代码质量↑，清晰的 Git 历史 |
| 多并行项目/功能分支管理 | GSD 工作流（Workstreams） | 命名空间隔离，独立可验证，系统化合并 |
| 契约驱动外包/团队协作 | GSD 规范驱动开发（XML提示+验证） | 明确需求/计划/验收标准，可追溯，减少沟通成本 |

---

## 🎯 快速导航

### 按需求查找

**想了解基础概念**
→ [上下文工程深度洞察报告 - 核心方法论](context-engineering-agent-skills-insight.md#一核心方法论)

**想立即动手优化**
→ [快速参考 - 立即可做清单](context-engineering-quick-reference.md#token优化检查清单)

**想选择架构模式**
→ [快速参考 - 决策树](context-engineering-quick-reference.md#架构选择决策树)

**遇到具体问题**
→ [快速参考 - 常见问题修复](context-engineering-quick-reference.md#常见问题快速修复)

**想了解实施路径**
→ [深度洞察 - 实施路线图](context-engineering-agent-skills-insight.md#五实施路线图)

---

## 📈 探索统计

- **已探索资源**: 8个核心资源（GitHub仓库 x5、arXiv论文 x2、官方PDF文档 x1）
- **生成报告**: 16份文档（8个洞察 × 2份文档）
- **提取模式**: 64个核心设计模式
- **识别反模式**: 59个需要避免的模式
- **业务场景**: 38个直接应用场景

---

## 🔜 待探索

以下方向值得进一步探索：

- [ ] AutoGen框架的多智能体协调模式
- [ ] CrewAI的角色流程设计
- [ ] LangGraph的状态机架构
- [ ] GraphRAG的知识图谱检索
- [ ] AgentOps的生产部署模式
- [ ] 企业级智能体安全与合规

---

*索引最后更新: 2026年3月28日*
