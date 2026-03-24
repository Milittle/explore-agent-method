# Agent Methodology Explorer

> 系统化探索和分析AI智能体方法论的Claude Code技能项目

[![探索报告](https://img.shields.io/badge/📚_探索报告-查看索引-blue)](docs/index.md)
[![技能定义](https://img.shields.io/badge/🔧_技能定义-SKILL.md-orange)](./claude/skills/agent-methodology-explorer/SKILL.md)

---

## 📚 探索报告文档库

> 所有洞察报告的完整索引：**[docs/index.md](docs/index.md)**

| # | 洞察主题 | 来源 | 快速预览 |
|---|----------|------|----------|
| 07 | [Browser-Use Web 智能体框架](docs/browser-use-web-agent-2026-03-24/) | GitHub | 四阶段DOM管道+认知循环Schema+升级干预循环检测，LLM驱动浏览器自动化 |
| 06 | [Superpowers 智能体技能框架](docs/superpowers-skill-framework-2026-03-20/) | GitHub | 设计门控+子智能体驱动+TDD刚性约束，101k Stars |
| 05 | [Claude Skills 完整构建指南](docs/claude-skill-building-guide-2026-03-14/) | 官方PDF文档 | 渐进式三层加载、五大设计模式、MCP+Skills完整方案、15-30分钟速建 |

**📊 查看完整索引** → [docs/index.md](docs/index.md)

---

## 项目简介

这是一个用于系统化探索和分析AI智能体方法论的Claude Code技能项目。每次探索会生成结构化的洞察报告，包含可复用的模式、业务应用分析和实施路线图。

## 项目结构

```
.
├── .claude/
│   └── skills/
│       └── agent-methodology-explorer/
│           └── SKILL.md           # 核心技能定义
├── docs/                          # 探索报告
│   ├── index.md                   # 📑 文档索引
│   ├── context-engineering-insight-2026-03-06/
│   └── file-system-context-engineering-2026-03-07/
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

---

## 快速导航

| 想要... | 前往 |
|---------|------|
| 📖 查看所有探索报告 | [docs/index.md](docs/index.md) |
| 🔧 了解技能定义 | [SKILL.md](.claude/skills/agent-methodology-explorer/SKILL.md) |
| 🚀 开始第一次探索 | 参考下方示例 |

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

---

*最后更新: 2026-03-24*
