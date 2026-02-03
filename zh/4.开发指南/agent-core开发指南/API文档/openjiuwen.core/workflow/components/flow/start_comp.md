# openjiuwen.core.workflow.components.flow.start_comp

`openjiuwen.core.workflow.components.flow.start_comp` 模块提供工作流开始组件，用于定义工作流入口节点。该节点不执行业务逻辑，仅将上游传入的输入原样向下游传递。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import Start` 导入。更多组件说明见 [components](../../components.README.md)。

## class Start

```python
class openjiuwen.core.workflow.components.flow.start_comp.Start(WorkflowComponent)
```

工作流开始组件，继承自 [WorkflowComponent](../base.md#class-openjiuwencoreworkflowcomponentsbaseworkflowcomponent)。作为工作流的入口，其输入通常由框架根据 `inputs_schema` 从用户输入或上下文填充，本组件在 `invoke` 中直接返回该输入。

### \_\_init\_\_

```python
def __init__(self) -> None
```

无参构造，无配置项。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

将当前节点接收到的输入原样返回，供下游节点使用。

**参数**：

- **inputs**（Input）：工作流传入的入口输入（通常由 `inputs_schema` 映射得到）。
- **session**（Session）：工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：与 `inputs` 相同，即本节点的输出即入口输入。
