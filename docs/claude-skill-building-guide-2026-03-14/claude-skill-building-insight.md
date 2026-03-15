# Claude Skills 完整构建指南 - 深度洞察报告

> 来源：[The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) - Anthropic 官方文档
> 分析日期：2026-03-14
> 文档页数：33页

---

## 执行摘要

Claude Skills 是一套**结构化的指令封装系统**，允许开发者将领域知识、工作流程和最佳实践打包成可复用的文件夹，在不修改底层模型的前提下让 Claude 掌握特定任务能力。本指南是 Anthropic 官方发布的完整构建手册，涵盖从规划设计到测试分发的全流程，其核心价值在于将"一次教学、多次受益"的范式系统化落地——通过渐进式披露机制，在最小化 token 消耗的前提下实现专业化能力注入。

---

## 一、核心方法论

### 1.1 解决的根本问题

| 痛点 | 无 Skills 的现状 | 有 Skills 的改变 |
|------|-----------------|-----------------|
| 知识重复输入 | 每次对话需重新解释偏好和流程 | 一次定义，永久复用 |
| 工作流不一致 | 不同用户提示方式导致输出质量参差 | 嵌入最佳实践，结果标准化 |
| MCP 工具难上手 | 用户不知道如何组合使用工具 | 技能提供"食谱"，工具提供"厨具" |
| 多步骤流程易出错 | 依赖用户手动编排复杂步骤 | 自动化编排，验证门控 |

### 1.2 核心概念矩阵

**技能（Skill）的组成**：
```
your-skill-name/
├── SKILL.md          # 必需 - 主指令文件（含 YAML frontmatter）
├── scripts/          # 可选 - 可执行代码（Python、Bash等）
├── references/       # 可选 - 按需加载的参考文档
└── assets/           # 可选 - 模板、字体、图标等资源
```

**三大设计原则**：

1. **渐进式披露（Progressive Disclosure）**
   - 第一层（YAML frontmatter）：始终加载到系统提示，提供最小必要信息用于判断是否激活
   - 第二层（SKILL.md 正文）：Claude 判断相关时才加载完整指令
   - 第三层（链接文件）：技能目录内的附加文件，仅在需要时才被导航发现
   - **目标**：在最小化 token 消耗的前提下保持专业能力

2. **可组合性（Composability）**
   - Claude 可同时加载多个技能
   - 技能设计应考虑与其他技能协同，而非假设自己是唯一能力

3. **可移植性（Portability）**
   - 技能在 Claude.ai、Claude Code、API 三端完全一致运行
   - 创建一次，全平台可用

### 1.3 架构概览

```
┌─────────────────────────────────────────────────────────┐
│                    Claude 系统提示                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │Skill A   │  │Skill B   │  │Skill C   │  ← YAML层     │
│  │frontmatter│  │frontmatter│  │frontmatter│  (始终加载)  │
│  └──────────┘  └──────────┘  └──────────┘               │
└─────────────────────────────────────────────────────────┘
         ↓ 相关时加载
┌─────────────────────────────────────────────────────────┐
│              SKILL.md 正文（完整指令）                    │
│  步骤说明 / 工作流 / 错误处理 / 示例                      │
└─────────────────────────────────────────────────────────┘
         ↓ 按需加载
┌─────────────────────────────────────────────────────────┐
│         references/ + scripts/ + assets/                 │
│  详细文档 / 可执行脚本 / 模板资产                          │
└─────────────────────────────────────────────────────────┘
         ↓ 工具调用
┌────────────┐    ┌─────────────────────────────────────┐
│ 内置能力    │    │   MCP 服务器（外部工具连接器）        │
│代码执行/文档│    │Notion/Linear/Figma/Sentry/Asana...  │
└────────────┘    └─────────────────────────────────────┘
```

**Kitchen 类比**（官方文档原话）：
- **MCP** = 专业厨房：提供工具、食材和设备
- **Skills** = 菜谱：提供如何创造有价值成果的逐步指令

### 1.4 三大使用场景分类

**Category 1：文档与资产创建**
- 用途：生成一致、高质量的文档、演示文稿、应用、设计、代码等
- 典型示例：frontend-design 技能、docx/pptx/xlsx 技能
- 核心技术：内嵌风格指南和品牌标准、模板结构、质量检查清单、无需外部工具

**Category 2：工作流自动化**
- 用途：需要一致方法论的多步骤流程，可跨多个 MCP 服务器协调
- 典型示例：skill-creator 技能（自助创建技能的元技能）
- 核心技术：带验证门的分步工作流、通用结构模板、内置审查和改进建议、迭代精化循环

**Category 3：MCP 增强**
- 用途：为 MCP 工具访问提供工作流指导
- 典型示例：Sentry 代码审查技能（自动分析和修复 GitHub PR 中的 bug）
- 核心技术：序列化 MCP 调用协调、嵌入领域专业知识、常见 MCP 错误处理

---

## 二、可复用模式提取

### 2.1 五大设计模式

#### 模式 1：顺序工作流编排（Sequential Workflow Orchestration）

```markdown
## 工作流：客户入职

### 步骤 1：创建账户
调用 MCP 工具：`create_customer`
参数：name, email, company

### 步骤 2：设置支付
调用 MCP 工具：`setup_payment_method`
等待：支付方式验证完成

### 步骤 3：创建订阅
调用 MCP 工具：`create_subscription`
参数：plan_id, customer_id（来自步骤1）

### 步骤 4：发送欢迎邮件
调用 MCP 工具：`send_email`
模板：welcome_email_template
```

**适用场景**：用户需要特定顺序的多步骤流程
**关键技术**：显式步骤排序、步骤间依赖、每阶段验证、失败回滚指令

#### 模式 2：多 MCP 协调（Multi-MCP Coordination）

**适用场景**：工作流跨越多个服务
**示例**：设计到开发交接（Figma → Drive → Linear → Slack）

```markdown
### 阶段1：设计导出（Figma MCP）
### 阶段2：资产存储（Drive MCP）
### 阶段3：任务创建（Linear MCP）
### 阶段4：通知（Slack MCP）
```

**关键技术**：清晰的阶段划分、MCP 间数据传递、进入下一阶段前的验证、集中错误处理

#### 模式 3：迭代精化（Iterative Refinement）

**适用场景**：输出质量通过迭代提升
**示例**：报告生成（生成草稿 → 质量检查 → 问题修复 → 重新验证）

```markdown
### 质量检查
运行验证脚本：`scripts/check_report.py`
识别问题：缺失章节 / 格式不一致 / 数据错误

### 精化循环
1. 解决每个识别的问题
2. 重新生成受影响的部分
3. 重新验证
4. 重复直至达到质量阈值
```

**关键技术**：明确质量标准、迭代改进、验证脚本、设置停止条件

#### 模式 4：上下文感知工具选择（Context-Aware Tool Selection）

**适用场景**：相同目标但根据上下文使用不同工具
**示例**：智能文件存储

```markdown
### 决策树
1. 检查文件类型和大小
2. 确定最佳存储位置：
   - 大文件（>10MB）：使用云存储 MCP
   - 协作文档：使用 Notion/Docs MCP
   - 代码文件：使用 GitHub MCP
   - 临时文件：使用本地存储
```

**关键技术**：清晰决策标准、备选方案、对用户解释选择原因

#### 模式 5：领域特定智能（Domain-Specific Intelligence）

**适用场景**：技能添加超越工具访问的专业知识
**示例**：金融合规 + 支付处理

```markdown
### 处理前（合规检查）
1. 通过 MCP 获取交易详情
2. 应用合规规则：
   - 检查制裁名单
   - 验证司法管辖区许可
   - 评估风险级别
3. 记录合规决策

### 处理
IF 合规通过：调用支付处理 MCP → 欺诈检查
ELSE：标记审查 → 创建合规案例

### 审计追踪
- 记录所有合规检查
- 生成审计报告
```

**关键技术**：领域专业知识嵌入逻辑、行动前合规检查、全面文档记录、清晰治理

### 2.2 关键反模式

| 反模式 | 为何有问题 | 替代方案 |
|--------|------------|----------|
| 描述过于模糊（"Helps with projects"） | Claude 无法判断何时加载技能，导致触发失败 | 明确"做什么"+"何时用"，包含具体触发短语 |
| 缺少触发条件 | 技能永远无法自动激活 | 在 description 中加入用户实际可能说的短语 |
| 过于技术性的描述 | 不匹配用户自然语言 | 用用户视角描述，而非开发者视角 |
| SKILL.md 指令过于冗长 | Claude 可能忽略关键指令 | 精简至核心，详细文档移入 references/ |
| 重要指令埋在文件底部 | 可能被截断或忽略 | 关键指令置顶，使用 `## CRITICAL` 标题 |
| 语言模糊 | Claude 解释不确定 | 代码/脚本执行确定性验证，而非依赖语言指令 |
| 在技能文件夹内放 README.md | 破坏标准结构 | 文档放在 SKILL.md 或 references/；README 放在仓库根目录 |
| 技能名包含 "claude" 或 "anthropic" | 保留字，上传会失败 | 使用其他描述性名称 |

---

## 三、技术规范详解

### 3.1 YAML Frontmatter 完整格式

```yaml
---
name: skill-name-in-kebab-case          # 必需：kebab-case，不含空格和大写
description: |                           # 必需：<1024字符，包含做什么+何时用
  What it does. Use when user says [specific phrases].
  Mention file types if relevant.
license: MIT                             # 可选：开源许可证
allowed-tools: "Bash(python:*) WebFetch" # 可选：限制工具访问
compatibility: Requires Python 3.8+      # 可选：1-500字符，环境要求
metadata:                                # 可选：自定义键值对
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [automation, workflow]
  documentation: https://example.com/docs
  support: support@example.com
---
```

**Description 字段黄金模板**：
```
[技能做什么] + [何时使用它（触发条件）] + [关键能力]
```

**好的 Description 示例**：
```yaml
description: Analyzes Figma design files and generates developer handoff
  documentation. Use when user uploads .fig files, asks for "design specs",
  "component documentation", or "design-to-code handoff".
```

**安全限制**：
- 禁止使用 XML 尖括号（`< >`）—— 安全防注入
- 禁止在 YAML 中执行代码（使用安全 YAML 解析）
- 禁止以 "claude" 或 "anthropic" 作为技能名前缀（保留）

### 3.2 成功衡量标准

**量化指标**：
| 指标 | 目标 | 测量方法 |
|------|------|----------|
| 相关查询触发率 | ≥90% | 运行10-20个测试查询，统计自动加载次数 |
| 工具调用次数 | 减少50%+ | 对比有/无技能时完成同一任务的工具调用数 |
| API 调用失败率 | 0次/工作流 | 监控 MCP 服务器日志的重试率和错误码 |

**质化指标**：
- 用户无需提示 Claude 下一步该做什么
- 工作流无需用户纠正即可完成
- 新用户首次尝试即能成功完成任务

---

## 四、业务应用分析

### 4.1 直接适用场景

**场景 A：企业知识库标准化**
- 问题：不同员工使用 Claude 产出质量不一
- 解法：创建公司级技能包（报告模板、邮件规范、代码审查流程）
- 部署：管理员通过工作区级技能统一下发（2025年12月已支持）
- 预期效果：跨团队输出一致性提升，onboarding 时间缩短

**场景 B：SaaS 产品 MCP + Skills 套装**
- 问题：用户连接 MCP 后不知道如何有效使用
- 解法：随 MCP 服务器捆绑配套技能包
- 竞争优势：提供"MCP+技能"完整包的产品 vs 仅提供 MCP 的产品
- 典型案例：Sentry、Asana、Atlassian、Figma、Zapier 已有伙伴技能

**场景 C：自动化流水线构建**
- 问题：复杂多步骤业务流程难以稳定自动化
- 解法：通过 API 的 `container.skills` 参数程序化调用技能
- 适合场景：生产环境大规模部署、代理系统、自动化管道

**场景 D：垂直领域专家系统**
- 问题：Claude 缺乏特定领域的专业知识和规则（如金融合规、医疗协议）
- 解法：将合规规则、审计流程、领域决策树内嵌到技能中
- 关键优势：领域知识与工具调用深度融合

### 4.2 投入与回报分析

| 维度 | 评估 |
|------|------|
| **复杂度** | 低（简单技能15-30分钟可完成）到中（复杂多MCP协调） |
| **开发时间** | 首个技能：15-30分钟（使用 skill-creator）；复杂技能：数小时 |
| **资源需求** | 无需编码基础（Markdown即可）；复杂技能需Python/Bash脚本能力 |
| **分发成本** | GitHub 开源免费分发；企业级通过控制台管理 |
| **预期收益** | 工作流一致性↑、用户学习曲线↓、支持票量↓、用户满意度↑ |

---

## 五、实施路线图

### 阶段 1：首个技能构建（1-2天）

1. **选择高频、重复性场景** - 找到每周至少用3次的 Claude 工作流
2. **定义2-3个具体用例** - 明确触发条件、步骤、成功标准
3. **使用 skill-creator 生成初稿** - "Use the skill-creator skill to help me build a skill for [your use case]"
4. **检查必要字段**：
   - [ ] 文件夹名称 kebab-case
   - [ ] 文件精确命名为 `SKILL.md`
   - [ ] YAML frontmatter 有 `---` 分隔符
   - [ ] description 同时包含"做什么"和"何时用"
   - [ ] 无 XML 标签
5. **Claude.ai 手动测试** - 测试明显触发、改写触发、不相关查询不触发

### 阶段 2：质量迭代（1周）

1. **运行触发测试** - 10-20个测试查询，统计触发率
2. **运行功能测试** - 验证工作流完整执行，API 调用成功
3. **性能基准对比** - 对比有/无技能的 token 消耗和工具调用次数
4. **处理边缘案例** - 将失败案例带回 skill-creator 改进
5. **优化 SKILL.md 结构**：
   - 核心指令置顶
   - 详细文档移入 `references/`
   - SKILL.md 控制在 5000 词以内
   - 关键验证用脚本而非语言指令

### 阶段 3：分发与扩展（按需）

**个人/小团队分发**：
```
GitHub 公开仓库 → 清晰 README（仓库根目录）→ 截图示例 → 安装指南
```

**企业级分发**：
- 通过 Claude Console 管理技能版本
- 管理员工作区级统一部署
- 监控触发率和使用情况，持续迭代

**API/程序化使用**：
```python
# Messages API 中使用技能
response = anthropic.messages.create(
    model="claude-sonnet-4-6",
    container={"skills": ["your-skill-name"]},
    messages=[{"role": "user", "content": "..."}]
)
```

---

## 六、快速启动检查清单

### 开始前
- [ ] 已识别2-3个具体重复性用例
- [ ] 已明确所需工具（内置或 MCP）
- [ ] 已复习本指南和示例技能

### 开发中
- [ ] 文件夹名使用 kebab-case
- [ ] 文件精确命名为 `SKILL.md`（区分大小写）
- [ ] YAML frontmatter 有 `---` 开闭分隔符
- [ ] name 字段：kebab-case，无空格，无大写
- [ ] description 包含"做什么"和"何时用"
- [ ] 无 XML 标签（`< >`）
- [ ] 指令清晰可操作
- [ ] 包含错误处理
- [ ] 提供示例
- [ ] 参考资料有明确链接

### 上传前
- [ ] 相关任务触发测试通过
- [ ] 改写请求触发测试通过
- [ ] 无关主题不触发验证
- [ ] 功能测试通过
- [ ] 已压缩为 .zip

### 上传后
- [ ] 真实对话中测试
- [ ] 监控过/欠触发
- [ ] 收集用户反馈
- [ ] 根据反馈迭代 description 和 instructions
- [ ] metadata 中更新版本号

---

## 七、关键洞察总结

1. **描述字段是成败关键**：90% 的技能问题源于 description 写得不好。这是"Claude 决定是否加载技能"的唯一依据，必须同时包含功能描述和触发短语。

2. **渐进式披露是效率核心**：三层加载机制让技能在不增加 context 负担的前提下提供专业能力，这是与直接放入系统提示的根本区别。

3. **脚本优于语言指令**：对于关键验证逻辑，使用确定性脚本（Python/Bash）而非语言指令——"代码是确定的，语言解释不是"。

4. **MCP + Skills = 完整解决方案**：MCP 提供连接能力（"做什么"），Skills 提供知识层（"怎么做"）。仅有 MCP 的集成让用户困惑；加上 Skills 才能形成真正的产品级体验。

5. **技能是活文档**：预计要不断迭代。设计触发测试、功能测试、性能对比基准，建立持续改进机制。

6. **20-50个技能是使用上限**：同时启用过多技能会降低 Claude 性能，建议按场景组织"技能包"，而非无限叠加。

---

## 八、资源与参考

| 资源 | 说明 |
|------|------|
| [原始 PDF 指南](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) | 官方完整指南 |
| [anthropics/skills GitHub](https://github.com/anthropics/skills) | 官方示例技能仓库 |
| [Agent Skills 介绍博客](https://www.anthropic.com/news/agent-skills) | 官方发布公告 |
| [Skills API 快速入门-有的地区不支持](https://docs.anthropic.com/skills-api-quickstart) | API 使用文档 |
| [工程博客：利用Agent Skills为真实世界装备智能体](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills) | 深度技术解析 |
| skill-creator 技能 | Claude.ai 插件目录内置 / Claude Code 可下载 |
| 合作伙伴技能目录 | Sentry、Asana、Atlassian、Canva、Figma、Zapier 等 |
