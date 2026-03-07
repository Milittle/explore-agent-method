# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This is a **Claude Code skill project** for systematically exploring and analyzing AI agent methodologies. The core skill (`agent-methodology-explorer`) analyzes agent-related resources (GitHub repos, papers, blogs, documentation) and generates structured insight reports with reusable patterns, business application analysis, and implementation roadmaps.

## Core Architecture

```
.claude/skills/agent-methodology-explorer/
├── SKILL.md           # The skill definition - core workflow and templates
├── evals/evals.json   # Test cases for the skill
├── references/        # Shared reference materials (currently empty)
└── scripts/           # Utility scripts (currently empty)

docs/                          # All generated insight reports
├── index.md                   # Master index (MUST be updated after each exploration)
└── [insight-name]-[YYYY-MM-DD]/
    ├── [topic]-insight.md              # Deep-dive report (required)
    └── [topic]-quick-reference.md      # Quick reference guide (recommended)
```

## The Agent Methodology Explorer Skill

When users ask to "explore", "analyze", or "study" agent-related resources, use the `agent-methodology-explorer` skill. See `.claude/skills/agent-methodology-explorer/SKILL.md` for the complete workflow.

### Key Workflow Steps

1. **Source Acquisition** - Fetch content using appropriate tools (web reader for URLs, direct read for local files, etc.)
2. **Content Analysis** - Extract: problem definition, core concepts, architecture patterns, decision logic, interaction patterns, data flow
3. **Pattern Extraction** - Identify reusable positive patterns and anti-patterns to avoid
4. **Business Application Mapping** - Connect methodology to practical business use cases
5. **Implementation Roadmap** - Provide concrete, actionable steps

### CRITICAL: Index Update Requirement

After generating insight documents, you **MUST** update `docs/index.md`:

- Add entry to "探索报告列表" (exploration report list)
- Update "核心发现汇总" (core findings summary)
- Update "探索统计" (exploration statistics)
- Update the last modified date

This is verified in the quality checklist at the end of the skill workflow.

### Output Language

**Default: Chinese** (unless user explicitly requests English)

All reports, templates, and documentation should be generated in Chinese. Technical terms may retain English with Chinese explanations.

### Output Template Structure

Each insight generates a structured report following this format:

```markdown
# [Methodology/Paper/Framework Name] - 分析与应用指南

## 快速摘要
[2-3 sentence overview]

## 核心方法论
### 解决的问题
### 核心概念
### 架构概览
### 决策与交互模式

## 提取的模式
### 可复用模式 (table format)
### 需避免的反模式 (table format)

## 业务应用分析
### 直接适用性
### 适配需求
### 投入与回报

## 实施路线图
### 阶段 1：基础建设
### 阶段 2：核心实施
### 阶段 3：集成与优化

## 快速启动检查清单
## 关键资源
```

## Directory Naming Convention

Insight directories follow the pattern: `docs/[insight-name]-[YYYY-MM-DD]/`

- Use English names with hyphens
- Name should represent the core topic
- Examples: `context-engineering-insight-2026-03-06/`, `multi-agent-patterns-2026-03-10/`

## Permissions

The project has a local permissions setting allowing:
- `WebSearch`
- Specific `git clone` commands for analysis

See `.claude/settings.local.json` for details.

## Testing the Skill

Test cases are defined in `.claude/skills/agent-methodology-explorer/evals/evals.json`:

1. GitHub repo exploration
2. Blog/article analysis
3. Local document analysis

Use these as reference examples when testing skill functionality.
