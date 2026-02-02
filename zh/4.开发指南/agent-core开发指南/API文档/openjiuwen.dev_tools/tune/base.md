# openjiuwen.dev_tools.tune.base

## class openjiuwen.dev_tools.tune.base.Case

`Case` 表示一条用于调优/评测的用例数据（inputs/label/tools 等）。

在 tune 体系中，`Case` 的定位是“训练样本/评测样本”的最小单元。

### 字段

- **inputs** (Dict[str, Any])：输入。
  - 必填，且至少包含 1 个字段。
  - 建议与 Agent/Workflow 的输入协议保持一致（例如 `{ "query": "...", "context": ... }`）。
  - `Trainer.predict` 会把 `case.inputs` 展开传给 agent.invoke，并附带 `conversation_id`（用于区分不同样本的会话）。
- **label** (Dict[str, Any])：标准答案/目标输出。
  - 必填，且至少包含 1 个字段。
  - `DefaultEvaluator` 会把 label 与模型输出 predict 做一致性对比。
  - label 建议尽量结构化（字段稳定），避免仅用长文本导致评测不稳定。
- **tools** (Optional[List[ToolInfo]])：该样本允许使用的工具列表。
  - 可选。
  - 用于在优化与评测时提供工具上下文（部分 Optimizer/Evaluator 会用到工具描述）。
- **case_id** (str)：样本 id。
  - 默认自动生成 uuid。
  - `CaseLoader` 会在加载后重写成稳定序号（例如 `case_0`、`case_1`），以便日志与对齐。

### 示例

```python
from openjiuwen.dev_tools.tune import Case

case = Case(
    inputs={"query": "北京今天的天气怎么样？"},
    label={"answer": "……标准答案……"},
)
```

## class openjiuwen.dev_tools.tune.base.EvaluatedCase

`EvaluatedCase` 表示带评测结果的用例数据（通常由 Evaluator 产出，供 Optimizer/Trainer 使用）。

它把“样本 + 模型输出 + 评分/原因”绑定在一起，作为 Optimizer 反向更新的主要信号来源。

### 字段

- **case** (Case)：原始样本。
  - 内含 inputs/label/tools/case_id。
- **answer** (Dict[str, Any] | None)：模型输出。
  - 通常来自 `Trainer.predict`。
  - 若模型执行异常，可能是 `{"error": "..."} ` 结构（具体取决于 predict 的错误处理）。
- **score** (float)：评分，范围 0~1。
  - `DefaultEvaluator` 默认输出 1.0（通过）或 0.0（不通过）。
  - 你也可以实现自定义 evaluator 输出更细粒度评分。
- **reason** (str)：评分原因/解释。
  - 建议写明“不一致点/失败原因”，以便 Optimizer 能聚焦修复。
  - `BadCasePromptBuilder` 等会把 reason 作为优化信号的一部分。

### 属性快捷访问

- **inputs/label/tools/case_id**：`EvaluatedCase` 提供了 property 直接透传到 `case.*`，便于在优化器里访问。

