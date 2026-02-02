# openjiuwen.core.workflow.components.flow.branch_router

`openjiuwen.core.workflow.components.flow.branch_router` 模块提供分支路由器及其单条分支的抽象，用于在工作流条件边中根据当前会话状态选择下一跳节点。 [BranchComponent](branch_comp.md#class-branchcomponent) 内部使用本模块的 [BranchRouter](branch_router.md#class-branchrouter) 管理多条 [Branch](branch_router.md#class-branch)；条件支持字符串表达式、可调用对象或 [Condition](condition/condition.md) 子类，表达式语法见 [ExpressionCondition](condition/expression.md)。

类通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import BranchRouter, Branch` 导入。

## class Branch

```python
class openjiuwen.core.workflow.components.flow.branch_router.Branch
```

表示单条分支：一个条件与一个目标节点 id 列表。用于在 [BranchRouter](branch_router.md#class-branchrouter) 内按顺序求值并选择首个满足条件的分支。

### \_\_init\_\_

```python
def __init__(self, condition: Union[str, Callable[[], bool], Condition], target: list[str], branch_id: str = None) -> None
```

构造一条分支。

**参数**：

- **condition**（`str | Callable[[], bool] | Condition`）：分支条件。若为 `str` 会封装为 [ExpressionCondition](condition/expression.md)；若为可调用对象则封装为 [FuncCondition](condition/condition.md#class-openjiuwencoreworkflowcomponentsconditionconditionfunccondition)；若为 [Condition](condition/condition.md) 子类则直接使用。其他类型会抛出异常。
- **target**（list[str]）：该分支对应的下游节点 id 列表。
- **branch_id**（str | None）：可选分支标识，默认 `None`。

**异常**：

- **JiuWenBaseException**：当条件类型不符合要求时，错误码参见 [StatusCode](../../common/exception/status_code.md) 中组件分支相关项。

### evaluate

```python
def evaluate(self, session: BaseSession) -> bool
```

在当前会话下求值本分支条件。

**参数**：

- **session**（BaseSession）：当前会话，用于解析表达式中的占位符等。

**返回**：

- **bool**：条件是否成立。

### trace_info

```python
def trace_info(self, session: BaseSession) -> str
```

返回当前分支条件的可读描述，用于追踪与调试。

**参数**：

- **session**（BaseSession）：当前会话。

**返回**：

- **str**：条件的 trace 信息（如表达式字符串或函数名）。

---

## class BranchRouter

```python
class openjiuwen.core.workflow.components.flow.branch_router.BranchRouter
```

分支路由器，用于在工作流条件边中根据当前会话状态从多条 [Branch](branch_router.md#class-branch) 中选出首个满足条件的分支，并返回其目标节点 id 列表。可由 [BranchComponent](branch_comp.md#class-branchcomponent) 使用，也可在自定义组件中单独使用。

### \_\_init\_\_

```python
def __init__(self, report_trace: bool = False) -> None
```

构造分支路由器。

**参数**：

- **report_trace**（bool）：是否在执行时上报分支信息（如分支 id、条件描述等）用于追踪，默认 `False`。

### add_branch

```python
def add_branch(self, condition: Union[str, Callable[[], bool], Condition], target: Union[str, list[str]], branch_id: str = None) -> None
```

添加一条分支。语义与 [BranchComponent.add_branch](branch_comp.md#add_branch) 一致：`condition` 为真时路由到 `target`。

**参数**：

- **condition**（`str | Callable[[], bool] | Condition`）：分支条件，类型要求同上文 Branch 构造参数。
- **target**（`str | list[str]`）：下游节点 id 或 id 列表；若为单个 id 则内部转为单元素列表。
- **branch_id**（str | None）：可选分支标识，默认 `None`。

**异常**：

- **JiuWenBaseException**：当 `condition` 或 `target` 为 `None` 时，错误码参见 [StatusCode](../../common/exception/status_code.md)。

### set_session

```python
def set_session(self, session: Union[Session, BaseSession]) -> None
```

设置当前会话，供条件边执行时在 [\_\_call\_\_](branch_router.md#__call__) 中求值各分支条件使用。

**参数**：

- **session**（Session | BaseSession）：工作流或 Agent 的会话实例。若为 [Session](../../session/session.md)，内部会使用其 `_inner` 会话。

**异常**：

- **JiuWenBaseException**：当 `session` 类型不是 `Session` 或 `BaseSession` 时，错误码参见 [StatusCode](../../common/exception/status_code.md)。

### \_\_call\_\_

```python
async def __call__(self, *args, **kwargs) -> list[str]
```

作为条件边的路由函数被图调用：按添加顺序依次求值各分支条件，返回首个满足条件的分支的 `target` 节点 id 列表。若 `report_trace` 为 `True`，会先上报各分支信息，再在命中时上报对应 `branch_id`。

**返回**：

- **list[str]**：命中的分支对应的目标节点 id 列表。

**异常**：

- **JiuWenBaseException**：当没有任何分支条件满足时，错误码参见 [StatusCode](../../common/exception/status_code.md) 中的分支执行相关项（如“未找到满足条件的分支”）。

### get_drawable_branch_router

```python
def get_drawable_branch_router(self)
```

当环境变量 `WORKFLOW_DRAWABLE=true` 时，在构造路由器时会创建用于可视化的分支路由信息。本方法返回该可绘制对象，供工作流可视化使用；未开启绘制时为 `None`。

**返回**：

- 可绘制分支路由器实例或 `None`。
