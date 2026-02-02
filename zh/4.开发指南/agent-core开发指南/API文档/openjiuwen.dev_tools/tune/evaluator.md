# openjiuwen.dev_tools.tune.evaluator

## class openjiuwen.dev_tools.tune.evaluator.evaluator.DefaultEvaluator

`DefaultEvaluator` 为默认用例评估器，用于对 `Case` 进行评测并产出 `EvaluatedCase`。

它的实现思路是：使用一个“LLM 评估 prompt”来判断模型回答与标准答案是否一致，并输出结构化 JSON 结果（含 reason）。

## DefaultEvaluator 初始化

```python
DefaultEvaluator(
    model_config: ModelRequestConfig,
    model_client_config: ModelClientConfig,
    metric: str = "",
)
```

**参数**：

- **model_config/model_client_config**：用于驱动“评估任务”的模型调用配置。
  - 注意：这不是被评估的 agent 使用的模型配置，而是 evaluator 自己用来做“判分”的模型配置。
- **metric** (str)：自定义评估规则。
  - 会被注入到评估模板的 `user_metrics` 中。
  - 用于覆盖/补充默认的一致性判断规则（例如强调格式、字段、工具调用约束等）。
  - 建议写得具体可执行，避免“过于主观”导致评分不稳定。

## evaluate

```python
evaluate(self, case: Case, predict: Dict[str, Any]) -> EvaluatedCase
```

评估单条样本。

默认行为：

- `score=1.0` 表示通过（回答与标准答案一致）
- `score=0.0` 表示不通过
- `reason` 提供简要解释（为什么一致/不一致）

**参数**：

- **case**：样本（包含 inputs/label）。
  - evaluator 会把 `case.inputs` 当作 question（字符串化）写入模板。
  - evaluator 会把 `case.label` 当作 expected_answer（字符串化）写入模板。
- **predict**：模型输出。
  - evaluator 会把 predict（字符串化）写入模板。
  - 若 predict 包含 error 字段，通常会导致评估失败并返回低分（具体取决于输出内容与模板判断）。

## batch_evaluate（继承自 BaseEvaluator）

```python
batch_evaluate(
    self,
    cases: List[Case] | CaseLoader,
    predicts: List[Dict[str, Any]],
    num_parallel: int = 1,
) -> List[EvaluatedCase]
```

批量评估。

会校验 `len(cases) == len(predicts)`，并使用线程池并发执行 `evaluate`。

**参数**：

- **num_parallel**：并发 worker 数。
  - 有范围校验（1~20）。
  - 设置过大可能导致“评估模型调用”被放大，成本显著上升。

