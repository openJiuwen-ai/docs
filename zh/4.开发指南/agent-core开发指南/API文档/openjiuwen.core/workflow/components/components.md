# openjiuwen.core.workflow.components
## class ComponentExecutable

## class ComponentComposable

> **注意**：组件类通过 `openjiuwen.core.workflow` 模块导出。建议使用 `from openjiuwen.core.workflow import ComponentComposable` 导入。

`ComponentComposable`是自定义工作流组件的抽象基类。所有自定义组件需继承该类，并实现将自身加入图以及转换为可执行单元（`ComponentExecutable`）的逻辑。

### add_component

```python
def add_component(self, graph: Graph, node_id: str, wait_for_all: bool = False) -> None:
```

定义如何将当前组件加入到指定`graph`。

**参数**：

- **graph**([Graph](../../graph/graph.md#class-graph))：`workflow`的`graph`实例。
- **node_id**(str)：本组件在`graph`中唯一的`node_id`。
- **wait_for_all**(bool)：本组件是否需要等待前置组件全部执行完成，`True`表示等待，`False`表示不等待。默认值：`False`。

**样例**：

```python
>>> import asyncio
>>> import random
>>> 
>>> from openjiuwen.core.context_engine import ModelContext
>>> from openjiuwen.core.workflow import ComponentComposable, BranchRouter, End, Start, Workflow, Input, Output
>>> from openjiuwen.core.session import Session
>>> from openjiuwen.core.session.workflow import create_workflow_session
>>> from openjiuwen.core.graph import Graph
>>> 
>>> 
>>> # 自定义组件`CustomIntentComponent`实现了`add_component`接口，用于为本组件定制内置的条件边，并自定义实现了接口`add_branch`用于绑定内置条件边的目标节点。
>>> # 创建自定义组件，并实现add_component接口
>>> class CustomIntentComponent(ComponentComposable):
...     def __init__(self, default_intents):
...         super().__init__()
...         self._intents = default_intents
...         self._router = BranchRouter()
... 
...     def add_component(self, graph: Graph, node_id: str, wait_for_all: bool = False) -> None:
...         graph.add_node(node_id, self.to_executable(), wait_for_all=wait_for_all)
...         graph.add_conditional_edges(node_id, self._router)
... 
...     def add_branch(self, condition: str, target: list[str], branch_id:str):
...         self._router.add_branch(condition=condition, target=target, branch_id=branch_id)
... 
...     async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output:
...         self._router.set_session(session)
...         return {'result': self._intents[random.randint(0, len(self._intents) - 1)]}
>>> 
>>> 
>>> class TravelComponent(ComponentComposable):
...     def __init__(self, node_id):
...         super().__init__()
...         self.node_id = node_id
... 
...     async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output:
...         print(f'[{self.node_id}] inputs = {inputs}')
...         return inputs
>>> 
>>> 
>>> class EatComponent(ComponentComposable):
...     def __init__(self, node_id):
...         super().__init__()
...         self.node_id = node_id
... 
...     async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output:
...         print(f'[{self.node_id}] inputs = {inputs}')
...         return inputs
>>> 
>>> 
>>> async def run_workflow():
...     # 创建工作流
...     flow = Workflow()
...     flow.set_start_comp("start", Start(), inputs_schema={"query": "${user_inputs.query}"})
... 
...     intent_component = CustomIntentComponent(["出行", "餐饮"])
...     intent_component.add_branch(condition="${intent.result} == '出行'", target=['travel'], branch_id='1')
...     intent_component.add_branch(condition="${intent.result} == '餐饮'", target=['eat'], branch_id='2')
...     flow.add_workflow_comp("intent", intent_component, inputs_schema={"query": "${start.query}"})
...     flow.add_workflow_comp("eat", EatComponent("eat"), inputs_schema={"intent": '${intent.result}'})
...     flow.add_workflow_comp("travel", TravelComponent("travel"), inputs_schema={"intent": '${intent.result}'})
...     flow.set_end_comp("end", End(), inputs_schema={"eat": "${eat.intent}", "travel": "${travel.intent}"})
...     flow.add_connection("start", "intent")
...     flow.add_connection("eat", "end")
...     flow.add_connection("travel", "end")
... 
...     session = create_workflow_session()
...     output = await flow.invoke({"user_inputs": {"query": "去新疆"}}, session)
...     print(output.result)
>>> 
>>> if __name__ == '__main__':
...     asyncio.get_event_loop().run_until_complete(run_workflow())
[eat] inputs = {'intent': '餐饮'}
{'output': {'eat': '餐饮'}}
```

### to_executable

```python
def to_executable(self) -> Executable
```

返回该组件对应的可执行实例，openJiuwen支持用户自定义实现`ComponentExecutable`，作为本接口的返回，用于运行时调度执行。

**返回**：

**[Executable](../../graph/graph.md)**，本组件的执行实例。

**样例**：

```python

>>> import asyncio
>>>
>>> from openjiuwen.core.context_engine import ModelContext 
>>> from openjiuwen.core.graph.executable import Executable
>>> from openjiuwen.core.session import Session
>>> from openjiuwen.core.session.workflow import create_workflow_session
>>> from openjiuwen.core.workflow import ComponentExecutable, End, Input, Output, Start, Workflow, ComponentComposable
>>> 
>>> # `CalculateComponent`继承`ComponentComposable`，是用于数学运算的组件，实现`to_executable`方法提供了实现计算逻辑的`ComputeExecutor`实例，实现`add_component`方法将`CalculateComponent`节点添加到工作流中。
>>> # 创建自定义组件，并实现to_executable接口
>>> class CalculateComponent(ComponentComposable):
...     def to_executable(self) -> Executable:
...         return ComputeExecutor()
>>> 
>>> class ComputeExecutor(ComponentExecutable):
...     async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output:
...         return {'result': self.__calculate__(data=inputs.get('data'))}
... 
...     def __calculate__(self, data):
...         operator = data.get('operator')
...         a = data.get('a')
...         b = data.get('b')
...         if operator == 'add':
...             return a + b
...         if operator == 'multiply':
...             return a * b
...         if operator == 'substract':
...             return a - b
...         return a / b
>>> 
>>> async def run_workflow():
...     # 创建工作流，并执行
...     flow = Workflow()
...     flow.set_start_comp("start", Start(), inputs_schema={"a": "${user_inputs.a}", "b": "${user_inputs.b}", "operator": "${user_inputs.operator}"})
...     flow.add_workflow_comp("compute", CalculateComponent(), inputs_schema={"data": "${start}"})
...     flow.set_end_comp("end", End(), inputs_schema={"result": "${compute.result}"})
...     flow.add_connection("start", "compute")
...     flow.add_connection("compute", "end")
...     session = create_workflow_session()
...     output = await flow.invoke({"user_inputs": {"a": 1, "b": 2, "operator": "add"}}, session)
...     print(output.result)
>>> 
>>> 
>>> if __name__ == '__main__':
...     asyncio.run(run_workflow())
{'output': {'result': 3}}
```


## class WorkflowComponent