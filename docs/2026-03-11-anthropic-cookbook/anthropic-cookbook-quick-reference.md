# Anthropic Cookbook - 快速参考指南

> 34.7k Stars 的官方实践库 | 可直接运行的代码参考 | 覆盖从基础到生产的完整路径

---

## 一句话总结

**Anthropic 官方的"Claude API 开发者操作手册"**，提供五大工作流模式 + 四阶段 SDK 教程，从单行研究智能体到企业级 SRE 智能体的完整参考实现。

---

## 核心模式速查表

| 模式名 | 适用场景 | 核心 Notebook |
|--------|----------|---------------|
| Prompt Chaining | 多步骤顺序处理 | `patterns/agents/basic_workflows.ipynb` |
| Routing | 意图分类，差异化处理 | `patterns/agents/basic_workflows.ipynb` |
| Parallelization | 大批量并行任务 | `patterns/agents/basic_workflows.ipynb` |
| Orchestrator-Workers | 复杂任务分解编排 | `patterns/agents/orchestrator_workers.ipynb` |
| Evaluator-Optimizer | 迭代质量优化 | `patterns/agents/evaluator_optimizer.ipynb` |

---

## 目录导航速查

```
需要...              看这里
─────────────────────────────────────
工具调用基础       → tool_use/calculator_tool.ipynb
结构化输出         → tool_use/extracting_structured_json.ipynb
Pydantic 工具定义  → tool_use/tool_use_with_pydantic.ipynb
并行工具调用       → tool_use/parallel_tools.ipynb
工具数量过多       → tool_use/tool_search_with_embeddings.ipynb
客服智能体完整示例 → tool_use/customer_service_agent.ipynb
记忆/持久化状态    → tool_use/memory_cookbook.ipynb
图像/视觉处理      → multimodal/getting_started_with_vision.ipynb
复杂推理           → extended_thinking/extended_thinking.ipynb
成本优化           → misc/prompt_caching.ipynb
上下文压缩         → tool_use/automatic-context-compaction.ipynb
评估体系建立       → misc/building_evals.ipynb
RAG 知识库         → capabilities/retrieval_augmented_generation/
分类任务           → capabilities/classification/
SDK 入门           → claude_agent_sdk/00_*.ipynb
生产级特性         → claude_agent_sdk/01_*.ipynb
外部系统集成       → claude_agent_sdk/02_*.ipynb
向量数据库集成     → third_party/Pinecone/
```

---

## Claude Agent SDK 四阶段学习路径

```
[阶段 0] 单行研究智能体
  核心：query() + async iteration + WebSearch
  时间：半天
  产出：能独立运行的研究智能体

[阶段 1] 执行助理智能体
  核心：CLAUDE.md记忆 + Plan Mode + Hooks + 子智能体编排
  时间：1-2天
  产出：有记忆、有审计、可规划的助理

[阶段 2] 可观测性智能体
  核心：Git MCP(13工具) + GitHub MCP(100工具) + 事件响应
  时间：2-3天
  产出：能监控 CI/CD 并自动响应的运维智能体

[阶段 3] 站点可靠性智能体
  核心：生产 SRE 场景完整实现
  时间：3-5天
  产出：可接入真实运维流程的智能体
```

---

## 工作流模式决策树

```
我的任务是什么？
│
├─ 单一线性流程 → Prompt Chaining
│   例：文章 → 摘要 → 翻译 → 格式化
│
├─ 多种不同输入类型 → Routing
│   例：投诉/咨询/退款 → 不同处理逻辑
│
├─ 大量相似任务 → Parallelization
│   例：100篇文章同时分类
│
├─ 复杂多步骤任务 → Orchestrator-Workers
│   例：写研究报告（搜索+分析+撰写+校对）
│
└─ 需要高质量输出 → Evaluator-Optimizer
    例：生成代码直到所有测试通过
```

---

## 生产化检查清单

**安全性**
- [ ] Plan Mode：生产操作前规划确认
- [ ] Hooks：所有操作自动审计追踪
- [ ] 权限最小化：每个子智能体只有必要工具

**性能**
- [ ] Prompt Caching：重复内容启用缓存
- [ ] Context Compaction：长对话自动压缩
- [ ] 动态工具注入：工具数 > 20 时用 Embedding 搜索

**质量**
- [ ] Evaluator-Optimizer：关键输出有评估循环
- [ ] Extended Thinking：复杂推理任务启用
- [ ] Building Evals：建立自动化评估基线

**可观测性**
- [ ] 日志记录：所有 LLM 调用有追踪
- [ ] 成本监控：追踪 token 使用和缓存命中率
- [ ] 错误处理：智能体有降级和恢复策略

---

## 常见问题快速修复

| 问题 | 解决方案 | 参考 |
|------|----------|------|
| 工具太多，智能体选错工具 | 工具 Embedding 搜索动态注入 | `tool_search_with_embeddings.ipynb` |
| 长对话成本越来越高 | 启用上下文自动压缩 | `automatic-context-compaction.ipynb` |
| 智能体忘记之前的对话 | CLAUDE.md 持久记忆 | `memory_cookbook.ipynb` |
| 输出格式不稳定 | Pydantic 结构化工具定义 | `tool_use_with_pydantic.ipynb` |
| 复杂任务推理不足 | 启用 Extended Thinking | `extended_thinking.ipynb` |
| 需要接入外部系统 | MCP Server 集成 | `claude_agent_sdk/02_*.ipynb` |
| 生产操作风险高 | Plan Mode 两阶段执行 | `claude_agent_sdk/01_*.ipynb` |

---

## 业务场景 → 推荐起点

| 你想构建... | 从这里开始 |
|-------------|-----------|
| 智能客服 | `customer_service_agent.ipynb` |
| 文档问答（RAG）| `capabilities/retrieval_augmented_generation/` |
| 数据提取 | `extracting_structured_json.ipynb` |
| 代码助手 | `extended_thinking_with_tool_use.ipynb` |
| 研究智能体 | `claude_agent_sdk/00_*.ipynb` |
| 运维自动化 | `claude_agent_sdk/02_*.ipynb` + `03_*.ipynb` |
| 内容生成 + 审核 | `patterns/agents/evaluator_optimizer.ipynb` |
| 多部门协作系统 | `patterns/agents/orchestrator_workers.ipynb` |

---

## 快速启动（3 分钟）

```bash
# 1. 克隆
git clone https://github.com/anthropics/anthropic-cookbook
cd anthropic-cookbook

# 2. 安装依赖
pip install anthropic jupyter

# 3. 启动 Notebook
jupyter notebook tool_use/customer_service_agent.ipynb
```

---

*参考来源：https://github.com/anthropics/anthropic-cookbook | 2026-03-11*
