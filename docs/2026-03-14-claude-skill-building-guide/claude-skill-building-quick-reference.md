# Claude Skills 构建指南 - 快速参考

> 基于 Anthropic 官方《The Complete Guide to Building Skills for Claude》

---

## 一句话总结

**Skill = SKILL.md 文件夹**，用 YAML 描述"做什么/何时用"，用 Markdown 写"怎么做"，三层渐进式加载在最小 token 消耗下注入专业能力。

---

## 核心结构速查

```
your-skill-name/          ← kebab-case 文件夹名
├── SKILL.md              ← 必需，大小写精确
├── scripts/              ← 可选，Python/Bash 脚本
├── references/           ← 可选，详细文档
└── assets/               ← 可选，模板/图标
```

```yaml
---
name: skill-name          # kebab-case，无空格无大写
description: |            # 核心！必须包含"做什么"+"何时用"
  功能描述。当用户说"[触发短语]"时使用。
---
```

---

## Description 字段写作公式

```
[功能描述] + [触发条件（用户会说什么）] + [关键能力/文件类型]
```

**✅ 好例子**：
```
Manages Linear project workflows. Use when user mentions "sprint",
"Linear tasks", "project planning", or asks to "create tickets".
```

**❌ 坏例子**：
```
Helps with projects.          ← 太模糊
Creates documentation.         ← 缺少触发条件
Implements entity model...     ← 太技术性，无用户触发语
```

**调试技巧**：问 Claude："When would you use the [skill name] skill?" Claude 会引用 description，根据缺失内容调整。

---

## 五大设计模式速查

| 模式 | 适用场景 | 核心结构 |
|------|---------|---------|
| **顺序工作流** | 必须按序执行的多步骤 | 步骤1→2→3，每步含验证 |
| **多MCP协调** | 跨服务工作流 | 阶段划分 + 数据传递 + 集中错误处理 |
| **迭代精化** | 质量需迭代提升的输出 | 草稿→检查→修复→验证→循环 |
| **上下文感知** | 同目标不同工具 | 决策树 + 清晰选择标准 |
| **领域智能** | 嵌入专业知识 | 合规前置 + 领域规则 + 审计追踪 |

---

## 触发问题排查树

```
技能不触发？
├── description 太通用？ → 加入具体触发短语
├── 缺少用户会说的词？ → 加入自然语言关键词
└── 技术术语过多？ → 改用用户视角描述

技能过度触发？
├── 加入负面触发
│   description: "...Do NOT use for [无关场景]"
├── 更具体的范围描述
└── 明确使用边界

MCP 调用失败？
├── 检查 Settings > Extensions > [服务] 是否"Connected"
├── API Key 是否有效/过期
├── OAuth Token 是否需要刷新
└── 先不用技能直接测试 MCP："Use [Service] MCP to fetch my projects"

指令不被遵守？
├── 指令太冗长 → 精简 + 移入 references/
├── 关键指令埋太深 → 置顶 + ## CRITICAL 标题
└── 语言模糊 → 改用脚本验证（确定性 > 语言）
```

---

## 量化成功标准

| 指标 | 目标 | 测量方法 |
|------|------|----------|
| 相关查询触发率 | ≥90% | 10-20个测试查询 |
| 工具调用次数 | 与基准相比减少 | 对比有/无技能 |
| API 错误率 | 0次/工作流 | 监控 MCP 日志 |
| 首次成功率 | 新用户首次能完成 | 用户测试 |

---

## YAML 禁止项

| 禁止 | 原因 |
|------|------|
| XML 尖括号 `< >` | 安全防注入 |
| 技能名含 `claude` 或 `anthropic` | 保留字 |
| SKILL.md 内放 README.md | 破坏标准结构 |
| 文件名大小写错误 (`skill.md`) | 上传会失败 |

---

## 三端使用场景速查

| 场景 | 推荐平台 |
|------|---------|
| 终端用户直接使用 | Claude.ai / Claude Code |
| 开发测试迭代 | Claude.ai / Claude Code |
| 个人/临时工作流 | Claude.ai / Claude Code |
| 程序化调用技能 | API |
| 生产环境大规模部署 | API |
| 自动化管道/智能体系统 | API |

---

## 30分钟构建第一个技能

```
1. 确定用例（5分钟）
   → 每周至少用3次的 Claude 工作流是什么？

2. 使用 skill-creator 生成初稿（10分钟）
   → "Use the skill-creator skill to help me build a skill for [用例]"

3. 检查核心字段（5分钟）
   → 文件夹 kebab-case ✓
   → 文件名 SKILL.md（精确）✓
   → description 含触发短语 ✓
   → 无 XML 标签 ✓

4. 上传并测试（10分钟）
   → Claude.ai Settings > Capabilities > Skills > Upload
   → 测试3种触发表达，1种无关查询
```

---

## 技能性能优化

| 问题 | 解决方案 |
|------|---------|
| 响应慢 | SKILL.md 控制在5000词内；详细内容移入 references/ |
| 质量下降 | 同时启用技能 ≤ 20-50个 |
| 关键验证不可靠 | 用 Python/Bash 脚本替代语言指令 |
| 模型"懒惰"跳步 | 在用户提示（不是SKILL.md）中加: "Take your time, quality > speed" |

---

## 参考资源

- 官方技能仓库：`github.com/anthropics/skills`
- skill-creator：Claude.ai 插件目录 / Claude Code 可下载
- 合作伙伴技能：Sentry、Asana、Figma、Canva、Zapier、Atlassian
- Bug 报告：`github.com/anthropics/skills/issues`
- 开发者社区：Claude Developers Discord
