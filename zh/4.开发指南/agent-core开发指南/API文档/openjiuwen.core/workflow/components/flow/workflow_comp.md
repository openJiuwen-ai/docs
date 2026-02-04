# openjiuwen.core.workflow.workflow_comp

`openjiuwen.core.workflow.components.flow.workflow_comp` 模块提供子工作流组件，用于将另一个 [Workflow](../components.md) 作为当前工作流中的一个节点执行。输入中的 `inputs` 会作为子工作流的输入，`config` 会传递给子工作流。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import SubWorkflowComponent` 导入。更多组件说明见 [components](../../components.README.md)。

## class SubWorkflowComponent

```python
class SubWorkflowComponent(WorkflowComponent)
```

子工作流组件，继承自 [WorkflowComponent](../components.md#class-workflowcomponent)。封装一个 [Workflow](../components.md) 实例，在 `invoke` 和 `stream` 中调用该子工作流的执行接口，并标记为“图内调用”以便框架正确调度。


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
