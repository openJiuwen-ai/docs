# openjiuwen.core.workflow.components.tool.tool_comp

`openjiuwen.core.workflow.components.tool.tool_comp` 模块提供工作流中的工具（插件）组件，用于绑定并执行通过 [Runner](../../../runner/runner.md) 注册的工具。组件根据配置的 `tool_id` 从资源管理器获取 [Tool](../../../foundation/tool.README.md) 实例，将节点输入校验后传给工具执行，并将结果封装为统一输出格式。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import ToolComponent, ToolComponentConfig` 导入。更多组件说明见 [components](../../components.README.md)。

## class ToolComponentConfig

```python
@dataclass
class openjiuwen.core.workflow.components.tool.tool_comp.ToolComponentConfig(ComponentConfig)
```

工具组件的配置数据类，继承自 [ComponentConfig](../base.md)。当前主要用于指定要绑定的工具 id，其余为预留扩展。

### 字段

- **tool_id**（str | None）：要绑定的工具 id，对应 [Runner](../../../runner/runner.md) 资源管理器中注册的工具。构造 [ToolComponent](tool_comp.md#class-toolcomponent) 时必填，否则初始化会抛异常。

---

## class ToolComponent

```python
class openjiuwen.core.workflow.components.tool.tool_comp.ToolComponent(ComponentComposable)
```

工具组件，实现 [ComponentComposable](../base.md)。在构造时根据 [ToolComponentConfig](tool_comp.md#class-toolcomponentconfig) 的 `tool_id` 从 `Runner.resource_mgr` 获取工具实例；[to_executable](tool_comp.md#to_executable) 返回 [ToolExecutable](tool_comp.md#class-toolexecutable) 实例，由该可执行对象在运行时校验输入并调用工具。

### \_\_init\_\_

```python
def __init__(self, config: ToolComponentConfig) -> None
```

**参数**：

- **config**（ToolComponentConfig）：工具组件配置，其中 `tool_id` 必填且必须在 Runner 中已注册。

**异常**：

- **JiuWenBaseException**：当 `tool_id` 为空或 Runner 中找不到对应工具时，错误码参见 [StatusCode](../../../common/exception/status_code.md) 中的 `COMPONENT_TOOL_INIT_FAILED`。

### to_executable

```python
def to_executable(self) -> Executable
```

返回本组件对应的可执行实例 [ToolExecutable](tool_comp.md#class-toolexecutable)，该实例内部已绑定从 Runner 获取的工具。

**返回**：

- **Executable**：即 `ToolExecutable(self._config).set_tool(self._tool)`。

---

## class ToolExecutable

```python
class openjiuwen.core.workflow.components.tool.tool_comp.ToolExecutable(ComponentExecutable)
```

工具组件的可执行实现。使用 [ToolComponentConfig](tool_comp.md#class-toolcomponentconfig) 构造，由 [ToolComponent](tool_comp.md#class-toolcomponent) 通过 `set_tool` 注入已获取的 [Tool](../../../foundation/tool.README.md) 实例；`invoke` 时校验输入并调用工具，将结果封装为统一结构（含 `errCode`、`errMessage`、`data` 等）。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

校验输入（通过 `ToolComponentInput`），调用已绑定的工具执行，并对工具返回结果做后处理与统一封装。

**参数**：

- **inputs**（Input）：节点输入，需符合工具组件的输入约定。
- **session**（Session）：工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：统一格式的输出（如 `ToolComponentOutput` 的 `model_dump`），包含 `data` 及可选的 `errCode`、`errMessage`。

**异常**：

- **JiuWenBaseException**：输入校验失败（StatusCode：`COMPONENT_TOOL_INPUT_PARAM_ERROR`）或工具未初始化（`COMPONENT_TOOL_EXECUTION_ERROR`）。工具执行过程中的异常会被捕获并转为输出中的错误信息。

### set_tool

```python
def set_tool(self, tool: Tool) -> Self
```

设置要执行的 [Tool](../../../foundation/tool.README.md) 实例，通常由 [ToolComponent](tool_comp.md#class-toolcomponent) 在构造可执行对象后调用。

**参数**：

- **tool**（Tool）：工具实例。

**返回**：

- **Self**：当前实例，便于链式调用。
