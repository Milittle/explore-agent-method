# Agent Methodology Explorer

> 系统化探索和分析AI智能体方法论的Claude Code技能项目

## 📚 探索报告文档库

| # | 洞察目录 | 来源 | 核心主题 | 发布日期 |
|---|----------|------|----------|----------|
| 01 | [上下文工程洞察](docs/context-engineering-insight-2026-03-06/) | GitHub | 上下文工程、多智能体架构、记忆系统 | 2026-03-06 |
| -- | └── [深度报告](docs/context-engineering-insight-2026-03-06/context-engineering-agent-skills-insight.md) | | 完整方法论与模式提取 | |
| -- | └── [快速参考](docs/context-engineering-insight-2026-03-06/context-engineering-quick-reference.md) | | 原则速查与优化清单 | |

---

## 项目简介

这是一个用于系统化探索和分析AI智能体方法论的Claude Code技能项目。每次探索会生成结构化的洞察报告，包含可复用的模式、业务应用分析和实施路线图。

## 项目结构

```
.
├── .claude/
│   └── skills/
│       └── agent-methodology-explorer/
│           ├── SKILL.md           # 核心技能定义
│           ├── evals/             # 测试用例
│           ├── scripts/           # 辅助脚本
│           └── references/        # 参考资料
├── docs/                          # 探索报告
│   ├── context-engineering-agent-skills-insight.md
│   ├── context-engineering-quick-reference.md
│   └── index.md                  # 文档索引
└── README.md
```

## 使用方法

直接让Claude探索任何智能体相关的方法论：

```bash
# 示例提示词：
- "帮我探索这个GitHub项目的方法论：https://github.com/..."
- "分析这篇关于agent的论文"
- "研究这个agent框架的架构模式"
- "探索这个URL中的智能体设计思想"
```

## 技能特性

`agent-methodology-explorer` 技能提供：

1. **多源支持** - GitHub仓库、网页URL、论文、文档、代码
2. **结构化分析** - 一致的输出格式便于对比
3. **模式提取** - 识别可复用的模式和反模式
4. **业务聚焦** - 始终连接到实际应用
5. **实施指导** - 具体、可操作的步骤
6. **中文输出** - 所有报告以中文生成，便于团队理解

## 输出格式

每次探索生成结构化文档，包含：

- **快速摘要** - 一句话概述核心价值
- **核心方法论** - 问题、概念、架构
- **模式提取** - 可复用的模式和需要避免的反模式
- **业务应用分析** - 直接适用场景和投资回报
- **实施路线图** - 分阶段的实施计划
- **快速检查清单** - 立即可行的行动项

## 最新洞察

### 🔥 上下文工程 (Context Engineering)

**核心发现**：
- 上下文工程不同于提示工程，它关注管理进入LLM的**所有信息**
- 通过渐进式披露可实现**87%的token减少**
- 多智能体的主要价值是**上下文隔离**，而非角色拟人化
- 工具整合原则：17个工具→2个工具可实现**3.5倍性能提升**

**关键数字**：
- 83.9% - 工具输出在上下文中的占比（优化重点）
- 70% - 触发上下文压缩的阈值
- 15× - 多智能体系统相对于单智能体的token倍数

**立即行动**：
1. 学习`context-fundamentals`技能
2. 实施渐进式披露（三层加载结构）
3. 整合重叠的工具定义

---

## 开始使用

1. 技能在此项目目录中自动可用
2. 使用示例提示词或创建自己的探索请求
3. 结果自动保存到`docs/`目录

## 示例

尝试探索一个方法论：

```
帮我探索 https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering
```

这将在`docs/`目录中生成一份全面的中文分析报告。
