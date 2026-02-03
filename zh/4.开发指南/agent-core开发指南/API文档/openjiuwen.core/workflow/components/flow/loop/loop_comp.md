# openjiuwen.core.workflow.components.flow.loop.loop_comp

`openjiuwen.core.workflow.components.flow.loop.loop_comp` 模块提供工作流中的循环能力，包括：用于定义循环体的 [LoopGroup](loop_comp.md#class-loopgroup)、作为循环节点的 [LoopComponent](loop_comp.md#class-loopcomponent)、用于中途退出的 [LoopBreakComponent](loop_comp.md#class-loopbreakcomponent)、以及用于在循环内写回变量的 [LoopSetVariableComponent](loop_comp.md#class-loopsetvariablecomponent)。循环类型由 [LoopType](loop_comp.md#class-looptype) 枚举表示。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import LoopComponent, LoopGroup, LoopBreakComponent, LoopSetVariableComponent` 导入。更多组件说明见 [components](../../../components.README.md)。

## class LoopType

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopType(str, Enum)
```

循环类型枚举，用于 [LoopComponent](loop_comp.md#class-loopcomponent) 的输入 `loop_type`：

- **Array**（`"array"`）：按数组迭代，输入需提供 `loop_array`。
- **Number**（`"number"`）：按次数循环，输入需提供 `loop_number`，且不超过会话环境中的最大循环次数限制。
- **AlwaysTrue**（`"always_true"`）：条件恒真，需通过 [LoopBreakComponent](loop_comp.md#class-loopbreakcomponent) 等中断。
- **Expression**（`"expression"`）：按表达式或布尔值判断，输入需提供 `bool_expression`。

---

## class LoopGroup

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopGroup(BaseWorkflow, Executable)
```

循环体工作流，继承自 `BaseWorkflow`（工作流基类） 并实现 [Executable](../../../../graph/graph.md)。用于定义循环体内包含的节点、边、起始与结束节点；不支持嵌套 [LoopComponent](loop_comp.md#class-loopcomponent)。通过 `start_nodes` / `start_comp` 与 `end_nodes` / `end_comp` 配置循环体的入口与出口。

### add_workflow_comp

```python
def add_workflow_comp(self, comp_id: str, workflow_comp: ComponentComposable, *, wait_for_all: bool = None, inputs_schema: dict | Transformer = None, outputs_schema: dict | Transformer = None, stream_inputs_schema: dict | Transformer = None, stream_outputs_schema: dict | Transformer = None, comp_ability: list[ComponentAbility] = None) -> Self
```

向循环体内添加组件。若添加的是 [LoopBreakComponent](loop_comp.md#class-loopbreakcomponent)，会记录到内部 break 列表；不支持添加 [LoopComponent](loop_comp.md#class-loopcomponent)（嵌套循环）。

**异常**：

- **JiuWenBaseException**：当 `workflow_comp` 为 `LoopComponent` 时，错误码为 `COMPONENT_LOOP_NOT_SUPPORT`（嵌套循环不支持）。

### start_nodes / start_comp

```python
def start_nodes(self, nodes: list[str]) -> Self
def start_comp(self, start_comp_id: str) -> Self
```

指定循环体的起始节点（可多个）。`start_comp` 单次指定一个并记录到 `_start_nodes`。

### end_nodes / end_comp

```python
def end_nodes(self, nodes: list[str]) -> Self
def end_comp(self, end_comp_id: str) -> Self
```

指定循环体的结束节点（可多个）。`end_comp` 单次指定一个并记录到 `_end_nodes`。

### on_invoke

```python
async def on_invoke(self, inputs: Input, session: BaseSession, **kwargs) -> Output
```

执行循环体子图：要求已配置 `start_nodes` 与 `end_nodes`，编译子图并在子会话中执行，返回由子图产生的输出（当前实现可为 `None`，由上层聚合）。

**异常**：

- **JiuWenBaseException**：未配置起始或结束节点时，错误码为 `COMPONENT_LOOP_CONFIG_ERROR`。

### break_components

```python
@property
def break_components(self) -> list[LoopBreakComponent]
```

返回本循环体内注册的 [LoopBreakComponent](loop_comp.md#class-loopbreakcomponent) 列表。

### is_empty

```python
@property
def is_empty(self) -> bool
```

判断循环体是否未包含任何节点（空图）。

---

## class LoopComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopComponent(WorkflowComponent)
```

循环组件，继承自 [WorkflowComponent](../../components.md#class-workflowcomponent)。根据输入中的 `loop_type`、`loop_array` / `loop_number` / `bool_expression` 等构建条件，驱动 [LoopGroup](loop_comp.md#class-loopgroup) 重复执行，并支持 [LoopBreakComponent](loop_comp.md#class-loopbreakcomponent) 与中间变量回写。

### \_\_init\_\_

```python
def __init__(self, loop_group: LoopGroup, output_schema: dict) -> None
```

**参数**：

- **loop_group**（LoopGroup）：循环体工作流，不可为空且 [is_empty](loop_comp.md#is_empty) 为 `False`。
- **output_schema**（dict）：循环节点输出结构定义，用于聚合循环结果。

**异常**：

- **JiuWenBaseException**：当 `loop_group.is_empty` 为 `True` 时，错误码为 `COMPONENT_LOOP_EXECUTION_ERROR`。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

执行循环：从 `inputs` 中解析 `INPUTS_KEY` 得到 LoopInput（含 `loop_type`、`loop_array`、`loop_number`、`bool_expression`、`intermediate_var` 等），根据类型构建 [Condition](../../condition/condition.md)（如 [ArrayConditionInSession](../../condition/array.md)、[NumberConditionInSession](../../condition/number.md)、[AlwaysTrue](../../condition/condition.md)、[ExpressionCondition](../../condition/expression.md)），驱动 [LoopGroup](loop_comp.md#class-loopgroup) 执行直至条件不满足或遇到 break。

**参数**：

- **inputs**（Input）：须为 dict，且包含键 `INPUTS_KEY`，其值符合 `LoopInput` 约定。
- **session**（Session）：工作流会话，用于获取循环次数上限等环境变量。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：循环结束后聚合的输出，结构由 `output_schema` 与内部回调决定。

**异常**：

- **JiuWenBaseException**：输入无效、缺少 `INPUTS_KEY`、`loop_type` 非法、`loop_number` 超限或循环体为空等，错误码参见 [StatusCode](../../../../common/exception/status_code.md)（如 `COMPONENT_LOOP_INPUT_INVALID`、`NUMBER_CONDITION_ERROR`、`COMPONENT_LOOP_EXECUTION_ERROR`）。若子图抛出 GraphInterrupt，会原样上抛。

### graph_invoker

```python
def graph_invoker(self) -> bool
```

返回 `True`，表示本组件由图的执行器调度（子图调用）。

### loop_group

```python
@property
def loop_group(self) -> LoopGroup
```

返回构造时传入的 [LoopGroup](loop_comp.md#class-loopgroup) 实例。

---

## class LoopBreakComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopBreakComponent(ComponentComposable, Executable)
```

循环中断组件。在循环体内作为节点加入后，执行时会调用当前循环控制器的 `break_loop` 方法，使当前循环在当轮结束后退出。需由循环执行逻辑注入 LoopController（通常由循环执行逻辑在内部自动完成）。

### \_\_init\_\_

```python
def __init__(self) -> None
```

无参构造。控制器通过 [set_controller](loop_comp.md#set_controller) 由框架注入。

### set_controller

```python
def set_controller(self, loop_controller: LoopController) -> None
```

设置循环控制器，执行 `on_invoke` 时会调用其 `break_loop` 方法。

### on_invoke

```python
async def on_invoke(self, inputs: Input, session: BaseSession, **kwargs) -> Output
```

调用控制器的 `break_loop` 方法标记循环中断，并返回空字典 `{}`。

**异常**：

- **JiuWenBaseException**：控制器未设置时，错误码为 `COMPONENT_BREAK_EXECUTION_ERROR`。

---

## class LoopSetVariableComponent

```python
class openjiuwen.core.workflow.components.flow.loop.loop_comp.LoopSetVariableComponent(WorkflowComponent)
```

循环内设置变量组件，继承自 [WorkflowComponent](../../components.md#class-workflowcomponent)。根据 `variable_mapping` 将右侧表达式的值（可为引用路径）写入会话状态中左侧路径对应的节点输出，用于在循环体内更新变量供后续节点或下一轮使用。

### \_\_init\_\_

```python
def __init__(self, variable_mapping: dict[str, Any]) -> None
```

**参数**：

- **variable_mapping**（dict[str, Any]）：键为写入目标路径（支持 `${node_id.key}` 形式），值为字面量或引用路径；不可为空。

**异常**：

- **JiuWenBaseException**：当 `variable_mapping` 为 `None` 或空时，错误码为 `COMPONENT_SET_VAR_INIT_FAILED`。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

根据 `variable_mapping` 解析右侧值（引用则从会话全局状态读取），并写入左侧路径对应的节点会话输出中，返回 `None`。

**异常**：

- **JiuWenBaseException**：左侧 key 格式不支持时，错误码为 `COMPONENT_SET_VAR_INPUT_PARAM_ERROR`。
