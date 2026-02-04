# openjiuwen.core.workflow

`openjiuwen.core.workflow.components.flow.workflow_comp` 模块提供子工作流组件，用于将另一个 [Workflow](../components.md) 作为当前工作流中的一个节点执行。输入中的 `inputs` 会作为子工作流的输入，`config` 会传递给子工作流。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import SubWorkflowComponent` 导入。更多组件说明见 [components](../../components.README.md)。

## class openjiuwen.core.workflow.components.flow.workflow_comp.SubWorkflowComponent

```python
class openjiuwen.core.workflow.components.flow.workflow_comp.SubWorkflowComponent(sub_workflow: Workflow)
```

子工作流组件，继承自 [WorkflowComponent](../components.md#class-workflowcomponent)。封装一个 [Workflow](../components.md) 实例，在 `invoke` 和 `stream` 中调用该子工作流的执行接口，并标记为“图内调用”以便框架正确调度。

**参数**：

- **sub_workflow**（Workflow）：要作为本节点执行的子工作流实例，不可为 `None`。

**异常**：

- **BaseError**：当 `sub_workflow` 为 `None` 时，错误码参见 [StatusCode](../../../common/exception/status_code.md) 中的 `COMPONENT_SUB_WORKFLOW_INIT_FAILED`。

### async invoke(inputs: Input, session: Session, context: ModelContext) -> Output

使用输入中的 `inputs` 和可选 `config` 调用子工作流的 `invoke`，并以子工作流身份执行（`is_sub=True`）。

**参数**：

- **inputs**（Input）：应包含图约定键 `INPUTS_KEY`（子工作流输入）及可选的 `CONFIG_KEY`（子工作流配置）。
- **session**（Session）：当前工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：子工作流的执行结果。

### async stream(inputs: Input, session: Session, context: ModelContext) -> AsyncIterator[Output]

**参数**：

- **inputs**（Input）：应包含图约定键 `INPUTS_KEY`（子工作流输入）及可选的 `CONFIG_KEY`（子工作流配置）。
- **session**（Session）：当前工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **AsyncIterator[Output]**：异步迭代器，逐帧输出子工作流处理结果。

### graph_invoker() -> bool

**返回**：

- **bool**：`True`,表示本组件由图的执行器调度（子图调用）。

### component_type() -> str

**返回**：

- **str**：常量 `"sub_workflow"`，即组件类型标识，用于区分子工作流节点

### property sub_workflow() -> Workflow

**返回**：

- **Workflow**：构造时传入的子工作流实例。
