# openjiuwen.core.utils.tool.tool

`@tool`装饰器用于将本地Python函数注册为可被LLM调用的Tool类，从而实现类似Function Calling的能力。

## tool

```python
tool(*, name: str, description: str, params: List[Param]) -> Tool
```

开发者配置工具名称、描述信息与params参数等信息用于描述本地函数。

**参数**：

* **name**(str)：工具名称，具有唯一性。
* **description**(str)：工具的描述信息，用于说明功能和用途。
* **params**(List[[Param](./param.md#class-openjiuwencoreutilstoolparamparam)])：请求参数列表，用于组成Json schema供大模型识别。

**返回**：

**Tool**，可被LLM调用的Tool类。

**样例**：

```python
>>> from openjiuwen.core.utils.tool.param import Param
>>> from openjiuwen.core.utils.tool.tool import tool
>>> 
>>> # 开发者实现add函数，直接使用tool装饰器进行封装，可供大模型识别调用
>>> @tool(name="Local_add",
...     description="本地加法插件",
...     params=[
...         Param(name="a", description="第一个参数", type="int", required=True),
...         Param(name="b", description="第二个参数", type="int", required=True),
...     ])
... def add(a: int, b: int) -> int:
...     return a + b
... 
>>> # 准备输入参数
>>> inputs = {
...     "a": 5,
...     "b": 1
... }
... 
>>> result = add.invoke(inputs=inputs)
>>> print(f"调用成功，返回结果: {result}")
调用成功，返回结果: 6
```
