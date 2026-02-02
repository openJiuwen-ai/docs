# tune

`openjiuwen.dev_tools.tune`提供了openJiuwen的 agent 调优功能。

## 基本概念

`tune` 把一次“可重复的调优实验”拆成三类角色：

- **Case / CaseLoader**：数据。
  - Case 是单条样本（inputs/label/tools）。
  - CaseLoader 负责管理样本集并提供 shuffle/split。
- **Evaluator**：评估。
  - 把 (Case, predict) 转成 EvaluatedCase，并提供 score/reason。
  - 默认提供 `DefaultEvaluator`（LLM 判分）。
- **Optimizer**：优化。
  - 根据 EvaluatedCase 生成“改进信号”，并更新 agent 的 prompt 参数。
  - 提供 Instruction/Example/Joint 三类优化器。
- **Trainer**：流程编排。
  - 把 Optimizer + Evaluator 串起来跑多轮训练，包含早停与候选搜索。

> 建议：先用小数据集跑通流程（几十条 Case），确认评估稳定后再扩大规模。
>
> 默认 evaluator 依赖 LLM 判分，成本会随样本数线性增长。

**Classes**：

| CLASS | DESCRIPTION |
|-------|-------------|
| [Case](./tune/base.md) | 用例类。 |
| [EvaluatedCase](./tune/base.md) | 评估后的用例类。 |
| [CaseLoader](./tune/dataset.md) | 用例加载器类。 |
| [DefaultEvaluator](./tune/evaluator.md) | 默认用例评估器类。 |
| [InstructionOptimizer](./tune/optimizer.md) | 提示词指令优化类。 |
| [ExampleOptimizer](./tune/optimizer.md) | 提示词示例优化类。 |
| [JointOptimizer](./tune/optimizer.md) | 指令-示例联合优化类。 |
| [Trainer](./tune/trainer.md) | Agent 训练器类。 |

