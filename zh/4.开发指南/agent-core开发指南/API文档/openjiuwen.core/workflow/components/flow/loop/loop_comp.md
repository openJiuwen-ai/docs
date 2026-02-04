# openjiuwen.core.workflow

`openjiuwen.core.workflow.components.flow.loop.loop_comp` 模块提供工作流中的循环能力，包括：用于定义循环体的 [LoopGroup](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopgroup)、作为循环节点的 [LoopComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopcomponent)、用于中途退出的 [LoopBreakComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopbreakcomponent)、以及用于在循环内写回变量的 [LoopSetVariableComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopsetvariablecomponent)。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import LoopComponent, LoopGroup, LoopBreakComponent, LoopSetVariableComponent` 导入。更多组件说明见 [components](../../../components.README.md)。

---

## class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopGroup

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopGroup()
```

循环体工作流，继承自 `BaseWorkflow`（工作流基类） 并实现 [Executable](../../../../graph/graph.md)。用于定义循环体内包含的节点、边、起始与结束节点；不支持嵌套 [LoopComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopcomponent)。通过 `start_nodes` / `start_comp` 与 `end_nodes` / `end_comp` 配置循环体的入口与出口。

### add_workflow_comp(comp_id: str, workflow_comp: ComponentComposable, *, wait_for_all: bool = None, inputs_schema: dict | Transformer = None, outputs_schema: dict | Transformer = None, stream_inputs_schema: dict | Transformer = None, stream_outputs_schema: dict | Transformer = None, comp_ability: list[ComponentAbility] = None) -> Self

向循环体内添加组件。若添加的是 [LoopBreakComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopbreakcomponent)，会记录到内部 break 列表；不支持添加 [LoopComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopcomponent)（嵌套循环）。

**参数**：

- **comp_id**（str）：工作流组件的唯一标识 ID，用于在工作流中区分不同组件，不可重复。
- **workflow_comp**（ComponentComposable）：要添加的工作流组件实例，需实现 ComponentComposable 接口。
- **wait_for_all**（bool = None）：是否等待所有前置组件完成后再执行当前组件，默认值为 None（使用工作流容器的默认配置）。
- **inputs_schema**（dict | Transformer = None）：组件的常规输入数据模式 / 转换器，用于规范和处理传入组件的非流式输入参数，默认值为 None（不进行额外输入处理）。
- **outputs_schema**（dict | Transformer = None）：组件的常规输出数据模式 / 转换器，用于规范和处理组件输出的非流式结果，默认值为 None（不进行额外输出处理）。
- **stream_inputs_schema**（dict | Transformer = None）：组件的流式输入数据模式 / 转换器，用于处理流式传输的输入参数，默认值为 None（不支持流式输入）。
- **stream_outputs_schema**（dict | Transformer = None）：组件的流式输出数据模式 / 转换器，用于处理流式传输的输出结果，默认值为 None（不支持流式输出）。
- **comp_ability**（list[ComponentAbility] = None）：当前组件具备的能力列表，用于标识组件的功能特性，默认值为 None（无额外能力声明）。

**异常**：

- **BaseError**：当 `workflow_comp` 为 `LoopComponent` 时，错误码为 `COMPONENT_LOOP_NOT_SUPPORT`（嵌套循环不支持）。

### start_nodes(nodes: list[str]) -> Self

批量设置循环体的起始节点。循环执行将从这些节点开始。

**参数**：

- **nodes**（list[str]）：起始节点 ID 列表，列表中的每个字符串对应一个工作流组件的唯一标识 ID。

---

### start_comp(start_comp_id: str) -> Self

设置单个起始节点。即使调用者直接使用 BaseWorkflow API，也会记录到内部起始节点列表中。

**参数**：

- **start_comp_id**（str）：起始组件的唯一标识 ID。

---

### end_nodes(nodes: list[str] | str) -> Self

批量设置循环体的结束节点。循环执行到这些节点后，会根据循环条件判断是否继续下一轮迭代。

**参数**：

- **nodes**（list[str] | str）：结束节点 ID，可以是单个字符串或字符串列表。若为字符串，则作为单个结束节点；若为列表，则设置多个结束节点。

---

### end_comp(end_comp_id: str) -> Self

设置单个结束节点。即使调用者直接使用 BaseWorkflow API，也会记录到内部结束节点列表中。

**参数**：

- **end_comp_id**（str）：结束组件的唯一标识 ID。

---

###  async on_invoke(inputs: Input, session: BaseSession, **kwargs) -> Output

循环组件的调用入口。自动完成能力补全，创建子工作流会话，编译并执行循环图。

**参数**：

- **inputs**（Input）：输入数据对象。
- **session**（BaseSession）：当前会话实例。
- **kwargs**：额外参数，支持传递 `context` 上下文信息。

**返回值**：

- **Output**：执行结果，当前实现返回 `None`。

---

### skip_trace() -> bool

是否跳过链路追踪。

**返回值**：

- **bool**：固定返回 `True`，表示跳过链路追踪。

---

### graph_invoker() -> bool

是否为图调用器。

**返回值**：

- **bool**：固定返回 `True`，表示该组件作为图调用器执行。

---

### check_validate()

验证循环组配置的合法性。检查起始节点、结束节点是否设置，以及循环体内是否包含至少一个节点。

**异常**：

- **BaseError**：当配置不合法时抛出，错误码为 `COMPONENT_LOOP_GROUP_PARAM_INVALID`，可能的原因包括：
  - 缺少起始节点（`missing start_nodes in loop group`）
  - 缺少结束节点（`missing end_nodes in loop group`）
  - 循环体为空（`loop group is empty (contains no nodes)`）

## class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopComponent(loop_group: LoopGroup, output_schema: dict)
```

循环组件，继承自 [WorkflowComponent](../../components.md#class-openjiuwencoreworkflowcomponentscomponentworkflowcomponent)。根据输入中的 `loop_type`、`loop_array` / `loop_number` / `bool_expression` 等构建条件，驱动 [LoopGroup](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopgroup) 重复执行，并支持 [LoopBreakComponent](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopbreakcomponent) 与中间变量回写。

**参数**：

- **loop_group**（LoopGroup）：循环体工作流，不可为空。
- **output_schema**（dict）：循环节点输出结构定义，用于聚合循环结果。

**异常**：

- **BaseError**：当对 `loop_group` 校验失败时抛出异常。

### async invoke(inputs: Input, session: Session, context: ModelContext) -> Output:

执行循环：从 `inputs` 中解析 `INPUTS_KEY` 得到 LoopInput（含 `loop_type`、`loop_array`、`loop_number`、`bool_expression`、`intermediate_var` 等），根据类型构建 [Condition](../../condition/condition.md)（如 [ArrayConditionInSession](../../condition/array.md)、[NumberConditionInSession](../../condition/number.md)、[AlwaysTrue](../../condition/condition.md)、[ExpressionCondition](../../condition/expression.md)），驱动 [LoopGroup](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopgroup) 执行直至条件不满足或遇到 break。

**参数**：

- **inputs**（Input）：须为 dict，且包含键 `INPUTS_KEY`，其值符合 `LoopInput` 约定。
- **session**（Session）：工作流会话，用于获取循环次数上限等环境变量。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：循环结束后聚合的输出，结构由 `output_schema` 与内部回调决定。

**异常**：

- **BaseError**：输入无效、缺少 `INPUTS_KEY`、`loop_type` 非法、`loop_number` 超限或循环体为空等，错误码参见 [StatusCode](../../../../common/exception/status_code.md)（如 `COMPONENT_LOOP_INPUT_INVALID`、`NUMBER_CONDITION_ERROR`、`COMPONENT_LOOP_EXECUTION_ERROR`）。若子图抛出 GraphInterrupt，会原样上抛。

### graph_invoker() -> bool

**返回**：

- **bool**：表示本组件由图的执行器调度（子图调用）是否成功。

### property loop_group() -> LoopGroup

**返回**：

- **LoopGroup**：返回构造时传入的 [LoopGroup](loop_comp.md#class-openjiuwencoreworkflowcomponentsflowlooploop_comploopgroup) 实例。

---

## class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopBreakComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopBreakComponent()
```

循环中断组件。在循环体内作为节点加入后，执行时会调用当前循环控制器的 `break_loop` 方法，使当前循环在当轮结束后退出。需由循环执行逻辑注入 LoopController（通常由循环执行逻辑在内部自动完成）。

### set_controller(loop_controller: LoopController)

**参数**：

- **loop_controller**（LoopController）：为循环中断组件配置循环控制器。

### async invoke(inputs: Input, session: Session, context: ModelContext) -> Output


调用控制器的 `break_loop` 方法标记循环中断，并返回空字典 `{}`。

**参数**：

- **inputs**（Input）：需要组件处理的输入数据对象。
- **session**（Session）：当前会话实例，用于获取组件相关信息。
- **context**（ModelContext）：模型上下文对象。

**返回值**：

- **Output**：组件响应的输出对象。

**异常**：

- **BaseError**：控制器未设置时，错误码为 `COMPONENT_BREAK_EXECUTION_ERROR`。

---

## class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopSetVariableComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopSetVariableComponent(variable_mapping: dict[str, Any])
```

循环内设置变量组件，继承自 [WorkflowComponent](../../components.md#class-workflowcomponent)。根据 `variable_mapping` 将右侧表达式的值（可为引用路径）写入会话状态中左侧路径对应的节点输出，用于在循环体内更新变量供后续节点或下一轮使用。

**参数**：

- **variable_mapping**（dict[str, Any]）：键为写入目标路径（支持 `${node_id.key}` 形式），值为字面量或引用路径；不可为空。

**异常**：

- **BaseError**：当 `variable_mapping` 为 `None` 或空时，错误码为 `COMPONENT_SET_VAR_INIT_FAILED`。

### async invoke(inputs: Input, session: Session, context: ModelContext) -> Output

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

**参数**：

- **inputs**（Input）：需要本组件处理的输入数据对象。
- **session**（Session）：当前会话实例，用于获取组件相关信息。
- **context**（ModelContext）：模型上下文对象。

**返回值**：

- **Output**：组件响应的输出对象。

**异常**：

- **BaseError**：左侧 key 格式不支持时，错误码为 `COMPONENT_SET_VAR_INPUT_PARAM_ERROR`。
