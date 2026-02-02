# openjiuwen.dev_tools.tune.trainer

## class openjiuwen.dev_tools.tune.trainer.trainer.Trainer

`Trainer` 用于将 `Optimizer` 与 `Evaluator` 组合起来，完成一轮或多轮调优训练流程。

它的核心职责是：

- 驱动 agent 在训练集上执行（predict）
- 用 evaluator 在验证集上评估分数（evaluate）
- 用 optimizer 做参数更新（backward/update）
- 进行候选参数搜索与早停（early stop）

## 初始化

```python
Trainer(
    optimizer: BaseOptimizer,
    evaluator: BaseEvaluator,
    num_parallel: int = 1,
    early_stop_score: float = 1.0,
)
```

**参数**：

- **optimizer** (BaseOptimizer)：优化器。
  - 负责把 `EvaluatedCase` 反向成“文本梯度/候选修改”，并更新 prompt 参数。
  - 需支持上下文管理器协议（`with optimizer:`），Trainer 会用它包裹每轮优化以管理内部状态。
- **evaluator** (BaseEvaluator)：评估器。
  - 负责把 (Case, predict) 转成 `EvaluatedCase`。
  - Trainer 会按样本平均得到本轮 score。
- **num_parallel** (int)：并行数。
  - 用于控制 predict/evaluate 的并发 worker 数。
  - 会做范围校验（1~20），过大可能导致模型/工具调用被放大、成本升高。
- **early_stop_score** (float)：早停阈值。
  - 默认 1.0。
  - 当验证集分数达到/超过该阈值时，Trainer 会提前结束训练，避免无意义迭代。

## train

```python
train(
    self,
    agent: BaseAgent,
    train_cases: CaseLoader,
    val_cases: Optional[CaseLoader] = None,
    num_iterations: int = 3,
) -> Optional[BaseAgent]
```

执行训练流程并返回（可能被更新过的）agent。

**参数**：

- **agent**：待调优的 agent。
  - 当前实现主要支持 legacy 单 agent（`LegacyBaseAgent`）并要求 agent 实现 `get_llm_calls()`，用于暴露可调的 prompt 参数。
  - 若 agent 不满足可训练条件，会抛出异常（避免静默“训练无效”）。
- **train_cases** (CaseLoader)：训练集。
  - `Trainer.evaluate` 会先在该集合上跑 predict，然后交给 optimizer.backward/update。
  - 若为空集合，训练会退化成无更新。
- **val_cases** (CaseLoader, 可选)：验证集。
  - 不传时默认使用 train_cases。
  - 主要用于“选择更好的参数组/早停判断”。
- **num_iterations** (int)：最大迭代轮数。
  - 会做范围校验（1~20）。
  - 实际训练轮数可能因 early_stop 提前结束。

**返回**：

- **BaseAgent | None**：训练后的 agent（通常是原对象被原地更新）。
  - 若训练被跳过或出错，可能返回 None（取决于上层调用与异常处理）。

## evaluate

```python
evaluate(self, agent: BaseAgent, cases: CaseLoader) -> Tuple[float, List[EvaluatedCase]]
```

对给定数据集做预测与评估，并返回平均分与逐样本评估结果。

**返回**：

- **score**：平均分（0~1）。
  - 默认 evaluator 为二值 0/1，平均后得到通过率。
- **evaluated_cases**：逐样本 `EvaluatedCase` 列表。
  - 将作为 optimizer 的主要训练信号。

## predict

```python
predict(self, agent: BaseAgent, cases: CaseLoader) -> List[Dict]
```

并发执行 agent，得到模型输出列表（predicts）。

实现细节：

- 会把 `case.inputs` 展开传给 `agent.invoke`。
- 会额外注入 `conversation_id=case.case_id`，用于区分不同样本的会话。
- 若执行异常，会返回 `{"error": "..."} ` 结构，避免整个批次崩溃。

## set_callbacks

```python
set_callbacks(self, callbacks: Callbacks)
```

注入训练回调。

用于在 train 开始/每轮/结束等阶段收集日志、保存中间结果等（具体回调接口见 `openjiuwen.dev_tools.tune.trainer.base`）。

