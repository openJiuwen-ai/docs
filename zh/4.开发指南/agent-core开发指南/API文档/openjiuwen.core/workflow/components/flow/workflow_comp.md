# openjiuwen.core.workflow.components.flow.workflow_comp

`openjiuwen.core.workflow.components.flow.workflow_comp` 模块提供子工作流组件，用于将另一个 [Workflow](../components.md) 作为当前工作流中的一个节点执行。输入中的 `inputs` 会作为子工作流的输入，`config` 会传递给子工作流。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import SubWorkflowComponent` 导入。更多组件说明见 [components](../../components.README.md)。

## class SubWorkflowComponent

```python
class openjiuwen.core.workflow.components.flow.workflow_comp.SubWorkflowComponent(WorkflowComponent)
```

子工作流组件，继承自 [WorkflowComponent](../components.md#class-workflowcomponent)。封装一个 [Workflow](../components.md) 实例，在 `invoke` 和 `stream` 中调用该子工作流的执行接口，并标记为“图内调用”以便框架正确调度。

### \_\_init\_\_

```python
def __init__(self, sub_workflow: Workflow) -> None
```

**参数**：

- **sub_workflow**（Workflow）：要作为本节点执行的子工作流实例，不可为 `None`。


### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

使用输入中的 `inputs` 和可选 `config` 调用子工作流的 `invoke`，并以子工作流身份执行（`is_sub=True`）。

**参数**：

- **inputs**（Input）：应包含图约定键 `INPUTS_KEY`（子工作流输入）及可选的 `CONFIG_KEY`（子工作流配置）。
- **session**（Session）：当前工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：子工作流的执行结果。

### stream

```python
async def stream(self, inputs: Input, session: Session, context: ModelContext) -> AsyncIterator[Output]
```

流式执行子工作流，将 `inputs` 中的 `inputs` 与 `config` 传给子工作流的 `stream`，并逐项 yield 子工作流的流式输出。

### graph_invoker

```python
def graph_invoker(self) -> bool
```

返回 `True`，表示本组件由图的执行器调度（子图调用）。

**返回**：

- **bool**：`True`。

### component_type

```python
def component_type(self) -> str
```

返回组件类型标识，用于区分子工作流节点。

**返回**：

- **str**：常量 `"sub_workflow"`。

### sub_workflow

```python
@property
def sub_workflow(self) -> Workflow
```

只读属性，返回构造时传入的子工作流实例。

**返回**：

- **Workflow**：子工作流实例。
