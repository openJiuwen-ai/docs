# openjiuwen.core.workflow.base

`openjiuwen.core.workflow.base` 模块定义了工作流的核心元数据结构与运行结果类型，包括：

- 用于描述工作流元信息的 `WorkflowCard`；
- 表示流式执行过程中不同数据片段类型的 `WorkflowChunkType`；
- 表示工作流整体执行状态的 `WorkflowExecutionState`；
- 工作流流式输出中使用的联合类型 `WorkflowChunk`；
- 封装最终输出和执行状态的 `WorkflowOutput`；
- 生成工作流全局 key 的工具函数 `generate_workflow_key`。

## class WorkflowCard

```python
class openjiuwen.core.workflow.base.WorkflowCard(BaseCard)
```

工作流的元数据卡片，继承自 `BaseCard`，用于描述工作流的基础信息与输入参数定义。

在继承自 `BaseCard` 的基础上，增加了：

- `version: str = ""`：工作流版本号；
- `input_params: dict[str, Any] | type[BaseModel] | None`：
  - 工作流输入参数的定义；
  - 可以是 JSON Schema 风格的 `dict`，也可以是 Pydantic `BaseModel` 子类类型，或 `None`。

### tool_info

```python
def tool_info(self) -> ToolInfo
```

将 `WorkflowCard` 转换为一个 `ToolInfo` 对象，便于将工作流作为工具在其他组件/Agent 中调用。

返回的 `ToolInfo` 字段：

- `name`：来自 `self.name`；
- `description`：来自 `self.description`；
- `parameters`：若 `input_params` 不为空则使用它，否则使用空 `dict`。

## class WorkflowChunkType

```python
class openjiuwen.core.workflow.base.WorkflowChunkType(str, Enum)
```

表示工作流执行过程中生成的数据分片类型，用于对流式输出进行分类：

- `INTERACTION`：交互流（例如中断提问时的交互输出）；
- `OUTPUT`：最终输出节点产生的内容流；
- `ERROR`：错误信息流。

## class WorkflowExecutionState

```python
class openjiuwen.core.workflow.base.WorkflowExecutionState(str, Enum)
```

表示工作流执行结束时的整体状态：

- `COMPLETED`：执行成功完成；
- `INPUT_REQUIRED`：等待用户输入（例如中断提问）；
- `ERROR`：执行过程中出现错误。

## type alias WorkflowChunk

```python
WorkflowChunk = Union[OutputSchema, CustomSchema, TraceSchema]
```

工作流执行过程中的单个流式输出片段类型别名，可以是：

- `OutputSchema`：标准输出流片段；
- `CustomSchema`：自定义输出流片段；
- `TraceSchema`：调试/trace 信息流片段。

## class WorkflowOutput

```python
class openjiuwen.core.workflow.base.WorkflowOutput(BaseModel)
```

封装工作流执行的最终结果和状态的 Pydantic 模型。

### 字段

- `result: Any`：
  - 工作流的输出结果；
  - 对于非流式场景，通常是一个 `dict`；
  - 对于流式场景，可以是分片列表或其他聚合结构，具体由上层约定。
- `state: WorkflowExecutionState`：
  - 工作流最终执行状态（`COMPLETED/INPUT_REQUIRED/ERROR`）。

## function generate_workflow_key

```python
def generate_workflow_key(workflow_id: str, workflow_version: str) -> str
```

根据工作流 id 和版本号生成一个全局唯一的 key，用于在资源管理器等地方标识不同版本的工作流。

**参数**：

- `workflow_id: str`：工作流 id；
- `workflow_version: str`：工作流版本号。

**返回值**：

- `str`：形如 `"{workflow_id}_{workflow_version}"` 的字符串，例如：

  ```python
  generate_workflow_key("my_workflow", "v1")  # 返回 "my_workflow_v1"
  ```

