# openjiuwen.core.workflow.start_comp

## class openjiuwen.core.workflow.components.flow.start_comp.Start

```python
class openjiuwen.core.workflow.components.flow.start_comp.Start()
```

`Start`用于定义工作流入口节点。该节点不执行业务逻辑，仅将上游传入的输入原样向下游传递。组件通过 openjiuwen.core.workflow 导出，建议使用 from openjiuwen.core.workflow import Start 导入。


**样例**：

```python
>>> import asyncio
>>> from openjiuwen.core.workflow import Start
>>> from openjiuwen.core.workflow import create_workflow_session
>>> from openjiuwen.core.workflow.base import Workflow
>>> 
>>> # 演示Start组件的基本用法和工作流创建流程
>>> async def demo_start_component():
...     # 1. 创建工作流实例
...     workflow = Workflow()
.
...     
...     # 3. 创建并设置开始组件
...     start_comp = Start()
...     workflow.set_start_comp("start", start_comp,
...                            inputs_schema={"query": "${user_inputs.query}", "content": "${user_inputs.content}"})
...     
...     # 4. 设置结束组件捕获输出
...     workflow.set_end_comp("end", Start(),
...                          inputs_schema={"query": "${user_inputs.query}", "content": "${user_inputs.content}"})
...     
...     # 5. 添加组件间连接
...     workflow.add_connection("start", "end")
...     
...     # 6. 执行工作流
...     result = await workflow.invoke(
...        {"user_inputs": {"query": "你好", "content": "杭州"}}, create_workflow_session()
...     )
... 
...     print(f"{result}")
>>> 
>>> def main():
...     asyncio.run(demo_start_component())
>>> 
>>> if __name__ == "__main__":
...     main()
>>> # `Start`组件会根据`conf`对于用户的`input`进行格式化，传递给下一个组件，格式化完的结果为：
result={'output': {'param1': '你好', 'param2': '杭州'}} state=<WorkflowExecutionState.COMPLETED: 'COMPLETED'>
```
