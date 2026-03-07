# 文件系统抽象 - 快速参考指南

## 一句话总结

将Unix"一切皆文件"哲学应用于GenAI上下文管理，通过文件系统抽象实现可追溯、可治理、可验证的上下文工程基础设施。

---

## 核心原则速查表

| 原则 | 实践 |
|------|------|
| **统一抽象** | 所有上下文源都挂载为文件系统节点 |
| **分层生命周期** | History（永久）→ Memory（可变）→ Scratchpad（临时） |
| **显式治理** | 每个操作都有事务日志和元数据 |
| **人类参与** | 人类输入作为一等公民存入/context/human/ |
| **Token约束** | 通过Constructor-Updater-Evaluator管道管理 |

---

## Token优化检查清单

### Context Constructor（选择与压缩）
- [ ] 按时间/相关性/优先级排序选择
- [ ] 应用压缩/摘要策略
- [ ] 计算Token预算
- [ ] 生成Context Manifest

### Context Updater（加载与刷新）
- [ ] 选择加载模式：静态快照 | 增量流式 | 自适应刷新
- [ ] 监控Token使用
- [ ] 动态替换过期上下文

### Context Evaluator（验证与更新）
- [ ] 检测幻觉和矛盾
- [ ] 验证后写入Memory
- [ ] 触发人类审核（低置信度）

---

## 常见问题快速修复

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Token超限 | 选择策略过于宽松 | 调整Constructor的优先级权重 |
| 上下文陈旧 | 未启用刷新机制 | 使用Updater的自适应刷新模式 |
| 无法追溯决策 | 未启用事务日志 | 启用AFS的transaction logging |
| 多Agent冲突 | 缺乏访问控制 | 实施文件系统级别的权限治理 |
| 幻觉频发 | 缺少评估循环 | 启用Evaluator的人类验证 |
| 存储膨胀 | 无老化策略 | 实施Memory的自动压缩和归档 |

---

## 架构选择决策树

```
需要管理Agent上下文？
│
├─ 只需要简单对话记忆？
│   └─ 使用基础Memory模块（episodic）
│
├─ 需要访问多种数据源？
│   └─ 实施AFS + Resolvers
│
├─ 需要审计合规？
│   └─ 必须启用History + 事务日志
│
├─ 多Agent协作？
│   └─ 实施访问控制 + 共享命名空间
│
└─ 需要人类验证？
    └─ 实施完整的CEP（包括Evaluator）
```

---

## 技能推荐学习顺序

```
1. AIGNE框架基础
   ↓
2. AFS挂载和Resolver编写
   ↓
3. Memory类型选择（episodic/fact/semantic）
   ↓
4. Context Constructor实现
   ↓
5. Context Updater加载模式
   ↓
6. Context Evaluator验证循环
   ↓
7. 权限治理和审计
```

---

## 实际案例效果

### 场景：企业知识Agent

| 指标 | 传统方法 | 文件系统抽象 | 改进 |
|------|---------|-------------|------|
| 上下文可追溯性 | 0% | 100% | ∞ |
| 跨会话连贯性 | 20% | 80% | +300% |
| 审计合规 | 需手动记录 | 自动日志 | 节省90%时间 |
| 数据源集成 | 每个需编码 | 即插即用 | 节省70%代码 |

---

## 快速实施模板

### 最小可行实现（1周）

```typescript
import { AIAgent } from "@aigne/core";
import { AFS } from "@aigne/afs";
import { AFSHistory } from "@aigne/afs-history";

// 1. 创建AFS并挂载History
const afs = new AFS().mount(new AFSHistory({
  storage: { url: "file:./memory.sqlite3" }
}));

// 2. 创建Agent
const agent = AIAgent.from({
  instructions: "You are a helpful assistant",
  inputKey: "message",
  afs  // 挂载的文件系统
});
```

### 完整CEP实现（4-6周）

```typescript
// 1. 持久化上下文存储库
const afs = new AFS()
  .mount(new History({ storage: db }))
  .mount(new Memory({ type: "episodic", storage: db }))
  .mount(new Memory({ type: "fact", storage: db }))
  .mount(new Scratchpad({ storage: db }));

// 2. Context Engineering Pipeline
const constructor = new ContextConstructor({
  selectors: [recency, relevance, priority],
  compressor: new Summarizer(),
  budget: { tokens: 100000 }
});

const updater = new ContextUpdater({
  mode: LoadMode.ADAPTIVE_REFRESH,
  monitor: new TokenMonitor()
});

const evaluator = new ContextEvaluator({
  validators: [hallucination, consistency],
  humanLoop: new HumanVerifier()
});

// 3. 组合使用
const context = await constructor.build(afs);
const updated = await updater.load(context);
const result = await model.reason(updated);
await evaluator.validate(result, afs);
```

---

## 关键指标监控

| 指标 | 健康阈值 | 告警级别 |
|------|---------|---------|
| Token使用率 | < 80% | Warning > 80%, Critical > 95% |
| 上下文命中相关性 | > 0.7 | Warning < 0.5 |
| 人类审核率 | < 30% | Review策略 > 50% |
| 存储增长率 | < 10%/周 | Review老化策略 |
| 响应延迟 | < 2s | Review检索优化 |

---

## 相关资源

- **深度洞察报告**: `file-system-context-engineering-insight.md`
- **AIGNE框架**: https://github.com/AIGNE-io/aigne-framework
- **论文原文**: https://arxiv.org/html/2512.05470v1
- **MCP协议**: https://modelcontextprotocol.io
