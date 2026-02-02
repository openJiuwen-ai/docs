# openjiuwen.dev_tools.tune.optimizer

## class openjiuwen.dev_tools.tune.optimizer.instruction_optimizer.InstructionOptimizer

提示词**指令（instruction）**优化器。

它会从 `EvaluatedCase` 中抽取失败样本，生成“文本梯度”（对提示词问题的反思与改进建议），再据此改写 system/user prompt。

### 初始化

```python
InstructionOptimizer(
    model_config: ModelRequestConfig,
    model_client_config: ModelClientConfig,
    parameters: Optional[Dict[str, LLMCall]] = None,
)
```

**参数**：

- **model_config/model_client_config**：用于驱动“优化任务”的模型调用配置。
- **parameters**：待优化的参数集合。
  - key 通常是节点/llm_call 名称。
  - value 是 `LLMCall`，内部包含 system/user prompt。
  - 训练时通常由 `Trainer` 在 `bind_parameter(agent.get_llm_calls())` 后注入。

### 行为要点

- 会尊重 `LLMCall` 的 freeze 配置：
  - `get_freeze_system_prompt() == True` 时不改 system prompt。
  - `get_freeze_user_prompt() == True` 时不改 user prompt。
- 对需要保留的占位符（placeholders）会做恢复/补全校验，避免优化后丢失关键变量（具体逻辑见源码）。

## class openjiuwen.dev_tools.tune.optimizer.example_optimizer.ExampleOptimizer

提示词**示例（few-shot examples）**优化器。

它的思路是：从用例集中挑选“代表性示例”（优先坏例或代表性样本），将示例以 few-shot 形式拼接进 prompt，从而提升模型在相似输入上的稳定性。

### 初始化

```python
ExampleOptimizer(
    model_config: ModelRequestConfig,
    model_client_config: ModelClientConfig,
    parameters: Optional[Dict[str, LLMCall]] = None,
    num_examples: int = 1,
)
```

**参数**：

- **num_examples**：示例数量。
  - 允许为 0（表示禁用示例优化）。
  - 有范围限制（0~20）；越大提示词越长，成本越高，收益不一定线性增长。

### 关键方法（语义）

- `init_examples(evaluated_cases)`：在每轮优化前先选一批初始示例，确保后续优化有可用信号。
- `format_prompt(prompt, gradient)`：把示例文本拼接到原 prompt 后面（以换行分隔）。

## class openjiuwen.dev_tools.tune.optimizer.joint_optimizer.JointOptimizer

指令-示例**联合优化器**。

它组合了：

- `InstructionOptimizer`：做指令/约束层面的修复与改写
- `ExampleOptimizer`：做 few-shot 示例补强

### 初始化

```python
JointOptimizer(
    model_config: ModelRequestConfig,
    model_client_config: ModelClientConfig,
    parameters: Optional[Dict[str, LLMCall]] = None,
    num_examples: int = 1,
)
```

### 行为要点

- 每轮会在“指令优化”与“示例优化”之间选择策略（当 `num_examples > 0` 时可能随机选择）。
- 更新阶段会先应用指令优化结果，再根据 freeze 配置决定把示例拼到 system 或 user prompt 上。
- 适合你不确定“问题在指令还是示例”时，用联合策略做自动探索。

