# openjiuwen.core.session.interaction

## class openjiuwen.core.session.interaction.interactive_input.InteractiveInput

`InteractiveInput` 表示“需要用户继续输入”的交互输入结构，常用于工作流执行进入 `INPUT_REQUIRED` 状态时，携带需要用户补充的信息。

### 字段

- **user_inputs** (Dict[str, Any])：按交互 id（通常是 node_id）聚合的用户输入。\n  - key：交互 id（例如节点 id）。\n  - value：该 id 对应的用户输入值（类型由业务决定）。
- **raw_inputs** (Any)：未绑定到任何 id 的输入。\n  - 常用于“第一轮交互”，即还没有明确 node_id 的场景。\n  - 当 `raw_inputs` 不为 `None` 时，不允许再向 `user_inputs` 写入（见 `update` 的约束）。

### __init__

```python
InteractiveInput(raw_inputs: Any = _sentinel)
```

创建交互输入对象。

**参数**：

- **raw_inputs** (Any, 可选)：首轮输入。\n  - 显式传 `None` 会被认为是非法输入并抛出异常。\n  - 不传该参数时，`raw_inputs` 会被置为 `None`（表示“还未收到首轮输入”）。

**异常**：

- **JiuWenBaseException**：当 `raw_inputs` 显式为 `None` 时抛出（输入非法）。

### update

```python
update(self, node_id: str, value: Any)
```

向 `user_inputs` 写入某个节点/交互 id 的输入值。

**参数**：

- **node_id** (str)：交互 id（通常为节点 id）。\n  - 不能为空；为空会抛出异常。\n  - 建议与 `Session.executable_id()` 保持一致，便于恢复与路由。
- **value** (Any)：用户输入值。\n  - 不能为空；为空会抛出异常。\n  - 类型由业务决定（例如 str/dict/结构化表单数据）。

**行为约束**：

- 若 `raw_inputs` 已存在（不为 `None`），则禁止调用 `update`，否则会抛出异常。\n  - 该约束用于避免“首轮 raw 输入”与“按节点输入”混用导致语义不清。

**异常**：

- **JiuWenBaseException**：\n  - 当 `raw_inputs` 已存在时（更新非法）。\n  - 当 `node_id` 或 `value` 为空时（输入非法）。

## class openjiuwen.core.session.interaction.interaction.InteractionOutput

`InteractionOutput` 表示工作流/组件向用户发起交互时输出的数据结构。

### 字段

- **id** (str)：交互 id（通常为节点/组件 id）。\n  - 用于让上层 UI/调用方知道“是谁在等待输入”。\n  - 恢复执行时通常会把用户输入回填到同一个 id。
- **value** (Any)：交互提示内容。\n  - 通常是字符串提示，也可以是结构化对象（例如需要用户填写的字段描述）。

## class openjiuwen.core.session.interaction.base.AgentInterrupt

`AgentInterrupt` 表示交互中断信号（用于控制交互流程）。

### 语义说明

当 Agent/Workflow 需要等待用户输入时，会抛出中断信号，使执行进入可恢复状态。\n+上层在收到中断后可：

- 将 `InteractionOutput` 推送到 UI 或外部系统
- 收集用户输入后，通过 session/checkpointer 恢复继续执行

