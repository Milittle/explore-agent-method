# AutoHarness - 快速参考指南

> **一句话总结**：让LLM自动生成任务特定的代码护栏，将非法操作率降为零，同时让小模型超越大模型。

---

## 核心原则速查表

| 原则 | 记忆口诀 |
|------|----------|
| 代码约束 > 语言约束 | "写规则，不求人" |
| 失败即训练信号 | "错了就改，改到完美" |
| 小+专 > 大+通用 | "专才胜通才" |
| 一次生成，永久使用 | "编译优化思维" |
| 合法是门槛，奖励是排名 | "先合法，再优秀" |

---

## 三种Harness类型选择

```
你的任务是什么？
│
├─ 合法动作可枚举（棋类、有限状态）
│   └─ ✅ Action-Filter（枚举合法集，LLM排序选择）
│
├─ LLM输出需灵活但需验证
│   └─ ✅ Action-Verifier（LLM生成 → 验证 → 失败重试）
│
└─ 规则稳定、高频执行、成本敏感
    └─ ✅ Harness-as-Policy（纯代码策略，零LLM调用）
```

---

## AutoHarness 实施检查清单

### 前提条件
- [ ] 任务有明确的"合法/非法"输出定义
- [ ] 可以实现 `is_legal(context, output) -> bool` 函数
- [ ] 有可运行的任务环境（可采集失败案例）
- [ ] 能运行LLM来做代码精化（Refiner）

### 核心实施步骤
- [ ] 实现验证器（`is_legal_action`）
- [ ] 实现动作提议接口（`propose_action`）
- [ ] 构建失败收集机制（每次失败保存 context + output + error）
- [ ] 实现Refiner调用（失败 → 提示 → 新代码）
- [ ] 设置迭代终止条件（连续N次100%合法）
- [ ] 生产化：Harness版本管理 + 监控告警

---

## Thompson采样启发式值

```python
def heuristic(is_legal: bool, reward: float) -> float:
    if not is_legal:
        return 0.0
    return 0.5 + 0.5 * reward  # reward ∈ [0, 1]

# 含义：
# H=0.0 → 非法动作，直接淘汰
# H=0.5 → 合法但零奖励（至少合格）
# H=1.0 → 完美（合法且最高奖励）
```

---

## Refiner提示词模板

```
你是一个代码精化专家。以下是当前Harness代码和失败案例，
请生成改进版本以消除这些失败。

# 当前代码
{current_harness_code}

# 失败案例（最多5个）
{failure_examples}

# 任务
修复上述代码使其能正确处理失败案例，同时保持对已有测试的正确性。
只返回完整的Python代码，不要解释。
```

---

## 性能数据速查

| 对比项 | Flash（无Harness） | Flash+Harness | Pro | GPT-5.2-High |
|--------|-------------------|---------------|-----|--------------|
| 合法移动率 | ~22%（棋类） | **100%** | 100%* | 100%* |
| 双人游戏胜率 | - | **56.3%** | 38.2% | - |
| 单人游戏奖励 | 0.673 | 0.745 | 0.707 | 0.844 |
| **Policy模式奖励** | - | **0.870** | 0.707 | 0.844 |
| 推理成本 | 中 | 中 | 高 | ~$640/批 |
| **Policy推理成本** | - | **≈$0** | 高 | 极高 |

*需人工设计约束

---

## 常见问题快速修复

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| Harness生成后仍有非法输出 | 验证器覆盖不全 | 增加边界测试用例，重新触发Refiner |
| 迭代不收敛 | 搜索树节点选择不当 | 检查Thompson采样实现，增加探索系数 |
| Harness代码过于复杂 | Refiner倾向添加而非重构 | 在提示词中要求"保持代码简洁" |
| Policy模式正确率下降 | 未覆盖新的边界情况 | 在新失败案例上重新触发生成流程 |
| 不同任务Harness无法复用 | 任务特定逻辑耦合 | 抽象通用验证接口，特定逻辑插件化 |

---

## 与其他方法的对比

| 方法 | 成本 | 合法率保证 | 可解释性 | 适应速度 |
|------|------|------------|----------|----------|
| 提示词约束 | 低 | ❌ 无保证 | 低 | 快 |
| 微调 | 极高 | ⚠️ 部分 | 低 | 慢 |
| 人工Harness | 中 | ✅ 有保证 | 高 | 很慢 |
| **AutoHarness** | **低（一次性）** | **✅ 100%** | **高** | **快** |
| RAG约束检索 | 中 | ❌ 无保证 | 中 | 快 |

---

## 快速实施模板（Python伪代码）

```python
class AutoHarnessGenerator:
    def __init__(self, env, refiner_llm, max_iter=20):
        self.env = env
        self.refiner = refiner_llm
        self.harness_tree = []

    def generate(self, initial_harness: str) -> str:
        self.harness_tree = [{"code": initial_harness, "score": 0}]

        for _ in range(self.max_iter):
            # 1. Thompson采样选节点
            node = self._thompson_sample()

            # 2. 收集失败案例
            failures = self._collect_failures(node["code"], n=10)
            if not failures:
                return node["code"]  # 收敛！

            # 3. Refiner精化
            new_code = self.refiner.refine(node["code"], failures)

            # 4. 评估新代码
            score = self._eval_legality(new_code)
            self.harness_tree.append({"code": new_code, "score": score})

        return max(self.harness_tree, key=lambda x: x["score"])["code"]

    def _thompson_sample(self):
        # 基于 H = 0 or 0.5 + 0.5*reward 采样
        scores = [node["score"] for node in self.harness_tree]
        samples = [np.random.beta(s+1, 1-s+1) for s in scores]
        return self.harness_tree[np.argmax(samples)]
```

---

*参考原文：https://arxiv.org/abs/2603.03329 | 更新日期：2026-03-13*
