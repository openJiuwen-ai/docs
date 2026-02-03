# openjiuwen.core.workflow.components.flow.branch_comp

`openjiuwen.core.workflow.components.flow.branch_comp` 模块提供工作流分支组件，用于在工作流中根据条件将执行路由到不同下游节点。分支组件内部使用 [BranchRouter](branch_router.md#class-branchrouter) 管理多条分支，每条分支由条件与目标节点列表组成。条件支持字符串表达式、可调用对象或 [Condition](../condition/condition.md) 子类，详见 [ExpressionCondition](../condition/expression.md) 等。

组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import BranchComponent` 导入。更多组件说明见 [components](../../components.README.md)。

## class BranchComponent

```python
class openjiuwen.core.workflow.components.flow.branch_comp.BranchComponent(WorkflowComponent)
```

工作流分支组件，继承自 [WorkflowComponent](../base.md#class-openjiuwencoreworkflowcomponentsbaseworkflowcomponent)。将当前节点加入图时，会为该节点添加条件边；运行时由 [BranchRouter](../flow/branch_router.md#class-branchrouter) 根据当前会话状态依次求值各分支条件，并返回首个满足条件的分支对应的目标节点列表。

### \_\_init\_\_

```python
def __init__(self) -> None
```

构造分支组件。内部会创建一个 `report_trace=True` 的 [BranchRouter](branch_router.md#class-branchrouter)，用于在执行时上报分支信息。

### add_branch

```python
def add_branch(self, condition: Union[str, Callable[[], bool], Condition], target: Union[str, list[str]], branch_id: str = None) -> None
```

添加一条分支：当 `condition` 求值为真时，路由到 `target` 所指定的节点（或节点列表）。

**参数**：

- **condition**（`str | Callable[[], bool] | Condition`）：分支条件。可为字符串表达式（如 `"${intent.result} == '出行'"`）、无参可调用对象（返回 `bool`），或 [Condition](../condition/condition.md) 子类实例。字符串会由框架解析为 [ExpressionCondition](../condition/expression.md)。
- **target**（`str | list[str]`）：该分支对应的下游节点 id，或 id 列表。若为单个 id 则内部转为单元素列表。
- **branch_id**（`str | None`）：可选分支标识，用于追踪与调试，默认 `None`。

**异常**：

- **JiuWenBaseException**：当 `condition`、`target` 或其元素为 `None` 或空时，错误码参见 [StatusCode](../../../common/exception/status_code.md) 中的组件分支参数相关项。

**样例**：

```python
>>> from openjiuwen.core.workflow import BranchComponent
>>>
>>> comp = BranchComponent()
>>> comp.add_branch(condition="${intent.result} == '出行'", target=["travel"], branch_id="1")
>>> comp.add_branch(condition="${intent.result} == '餐饮'", target=["eat"], branch_id="2")
```

### router

```python
def router(self) -> Callable[..., Union[Hashable, list[Hashable]]]
```

返回内部使用的 [BranchRouter](branch_router.md#class-branchrouter) 实例，供图在添加条件边时使用。该可调用对象在执行时根据当前会话返回满足条件的分支目标节点 id 列表。

**返回**：

- **Callable**：即内部的 `BranchRouter` 实例，调用时返回 `list[str]`（目标节点 id 列表）。

### add_component

```python
def add_component(self, graph: Graph, node_id: str, wait_for_all: bool = False) -> None
```

将当前分支组件作为节点加入给定的 [Graph](../../../graph/base.md#class-graph)，并为该节点添加条件边，由 [router](branch_comp.md#router) 返回的路由器决定下一跳。

**参数**：

- **graph**（[Graph](../../../graph/base.md#class-graph)）：工作流图实例。
- **node_id**（str）：本组件在图中的唯一节点 id。
- **wait_for_all**（bool）：是否等待所有前驱节点执行完成再执行本节点，默认 `False`。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

分支组件本身不产生业务输出，仅将当前 `session` 设置到内部 [BranchRouter](branch_router.md#class-branchrouter)，由路由器在条件边求值时使用会话状态。返回空字典 `{}`。

**参数**：

- **inputs**（Input）：当前节点输入（本组件未使用）。
- **session**（Session）：工作流会话。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：`{}`。

### skip_trace

```python
def skip_trace(self) -> bool
```

是否跳过该组件的 trace 记录。分支组件返回 `True`，由 [BranchRouter](branch_router.md#class-branchrouter) 负责上报分支相关信息。

**返回**：

- **bool**：`True`。
