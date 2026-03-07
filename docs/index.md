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

---

## 📋 技能映射表

| 业务场景 | 推荐技能 | 预期收益 |
|----------|----------|----------|
| 客服智能体 | 上下文基础、记忆系统、工具设计 | 50-70%成本降低 |
| 研究分析 | 多智能体、上下文优化、评估 | 并行处理，深度分析 |
| 内容生成 | 项目开发、工具设计、评估 | 可扩展，一致质量 |
| 知识管理 | 记忆系统、BDI心智状态 | 持续学习，关系洞察 |

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

- **已探索资源**: 2个核心资源（GitHub仓库、arXiv论文）
- **生成报告**: 4份文档（2个洞察 × 2份文档）
- **提取模式**: 12个核心设计模式
- **识别反模式**: 12个需要避免的模式
- **业务场景**: 8个直接应用场景

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

*索引最后更新: 2026年3月7日*
