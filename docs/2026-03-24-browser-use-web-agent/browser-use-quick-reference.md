# Browser-Use - 快速参考指南

> 一句话总结：**让 LLM 像人类一样操作浏览器的生产级智能体框架——DOM 管道将网页转化为 LLM 可理解格式，认知循环 Schema 保障任务连贯性，循环检测+Fallback LLM 提供生产健壮性。**

---

## 核心概念速查表

| 概念 | 一句话 | 关键参数/文件 |
|------|--------|--------------|
| Agent | 任务编排器 | `agent/service.py` |
| AgentOutput | LLM 结构化输出 Schema | `thinking + evaluation + memory + next_goal + action` |
| DomService | DOM→LLM 四阶段管道 | `dom/service.py` |
| BrowserSession | CDP 浏览器连接 | Chromium + CDP |
| ActionLoopDetector | 重复行为检测 | 阈值：5/8/12 步 |
| MessageCompaction | 历史压缩 | token 阈值触发 |
| Fallback LLM | 备用模型切换 | 速率限制时自动切换 |

---

## 快速启动（5分钟）

```bash
# 1. 安装
uv init my-agent && cd my-agent
uv add browser-use && uv sync
playwright install chromium

# 2. 设置 API Key
export OPENAI_API_KEY=sk-...

# 3. 运行第一个任务
python -c "
import asyncio
from browser_use import Agent
from langchain_openai import ChatOpenAI

async def main():
    agent = Agent(
        task='Go to github.com/browser-use/browser-use and tell me the star count',
        llm=ChatOpenAI(model='gpt-4o'),
    )
    result = await agent.run()
    print(result)

asyncio.run(main())
"
```

---

## 自定义工具模板

```python
from browser_use import Agent, Controller

controller = Controller()

@controller.action("描述这个动作做什么")
async def my_custom_action(param: str) -> str:
    # 你的业务逻辑
    return f"执行结果: {param}"

agent = Agent(
    task="执行你的任务",
    llm=llm,
    controller=controller,
)
```

---

## 生产配置检查清单

### 安全配置
- [ ] 配置 `allowed_domains` 白名单（凭证注入域名）
- [ ] 不在 task 描述中直接写入密码（使用 sensitive_data 参数）
- [ ] 启用域名验证日志

### 可靠性配置
- [ ] 配置 Fallback LLM（主备 LLM 切换）
- [ ] 设置 `max_steps` 防止无限运行
- [ ] 设置 `max_actions_per_step` 控制单步动作数
- [ ] 配置 `consecutive_failures` 阈值

### 成本控制
- [ ] 启用 MessageCompaction（长任务必须）
- [ ] 根据需要关闭视觉模式（`use_vision=False`）节省 token
- [ ] 监控每任务平均 token 消耗

### 可观测性
- [ ] 接入 telemetry（token 使用、成功率、动作序列）
- [ ] 保存 AgentHistory 用于调试
- [ ] 配置步骤级日志

---

## 常见问题快速修复

| 问题 | 原因 | 修复 |
|------|------|------|
| Agent 卡在同一页面循环 | 任务描述不清晰或页面无法完成任务 | 检查 LoopDetector 日志；优化任务描述 |
| Token 超出限制 | 长任务消息历史膨胀 | 启用 MessageCompaction |
| CDP 连接断开 | 网络超时或浏览器崩溃 | Agent 自动重连；增大超时配置 |
| 元素点击失败 | 元素不可见/被遮挡 | DomService 可见性过滤；检查 iframe 层级 |
| LLM 速率限制 | 高频调用超出配额 | 配置 Fallback LLM；增加重试延迟 |
| 凭证注入警告 | 域名不在白名单 | 添加目标域名到 `allowed_domains` |

---

## 架构选择决策树

```
需要 Web 自动化？
│
├─ 有规范 API？ → 优先用 API，不用 Browser-Use（更快更稳）
│
└─ 无 API 或需 UI 操作？
   │
   ├─ 任务规则固定、高频（>1000次/天）？
   │   → Playwright/Selenium 脚本（更快，Browser-Use 推理延迟高）
   │
   ├─ 任务描述不确定、需理解语义？
   │   → Browser-Use（LLM 理解能力强）
   │
   ├─ 需要处理 CAPTCHA/反爬？
   │   → Browser-Use Cloud 版本
   │
   └─ 企业内网/高安全需求？
       → 自托管 + 配置 allowed_domains + 禁用 telemetry
```

---

## 与其他方法论的结合点

| 结合点 | Browser-Use 功能 | 配套方法论 |
|--------|-----------------|----------|
| MCP 集成 | `browser_use/mcp/` 暴露为 MCP 工具 | Claude Skills + MCP（洞察05） |
| Skills 复用 | `browser_use/skills/` 封装常用操作序列 | Claude Skills 构建方法论（洞察05） |
| 上下文优化 | MessageCompaction 压缩历史 | 上下文工程（洞察01） |
| 代码护栏 | 自定义 Controller 过滤非法动作 | AutoHarness 模式（洞察04） |
| 工程纪律 | AgentHistory 提供完整审计 | Superpowers TDD 约束（洞察06） |

---

## 关键数字

- **Python 版本要求**：3.11+
- **基础模型推荐**：GPT-4o / Claude 3.5 Sonnet / Gemini 1.5 Pro
- **循环检测阈值**：5→8→12 步（三级干预）
- **最大并发**：理论无限，实践受 LLM 速率限制约束
- **DOM 处理 iframe 深度上限**：可配置（防止递归爆栈）
- **ChatBrowserUse 定价**：$0.20/M 输入，$2.00/M 输出

---

*快速参考指南 - Browser-Use Web 智能体 | 2026-03-24*
