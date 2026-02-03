# openjiuwen.core.runtime.resources_manager.base


## aliases AgentProvider
```python
AgentProvider = Callable[['AgentCard'], Awaitable['BaseAgent']] | Callable[['AgentCard'], 'BaseAgent']
```
代理提供者类型定义。

一个可调用对象，接受AgentCard并返回异步BaseAgent实例。用于代理资源的延迟加载，避免在注册时立即创建。支持在实际需要代理时才进行初始化。

## aliases AgentGroupProvider

```python
AgentGroupProvider = Callable[['GroupCard'], Awaitable['BaseGroup']] | Callable[['GroupCard'], 'BaseGroup']
```

代理组提供者类型定义。

一个可调用对象，接受GroupCard并返回异步BaseGroup实例。用于代理组资源的延迟加载，适用于需要按需初始化的复杂代理组配置。

## aliases WorkflowProvider

```python
WorkflowProvider = Callable[[WorkflowCard], Awaitable[Workflow]] | Callable[[WorkflowCard], Workflow]
```

工作流提供者类型定义。

一个可调用对象，接受WorkflowCard并返回异步Workflow实例。用于工作流资源的延迟加载，支持复杂工作流配置和依赖的按需初始化。

## aliases ModelProvider

```python
ModelProvider = Callable[[...], Awaitable[BaseModel]] | Callable[[...], BaseModel]
```

模型提供者类型定义。

一个可调用对象，接受可变参数并返回异步Model实例。用于模型资源的延迟加载，支持灵活的配置参数。

## aliases Tag

```python
Tag = str
```

用于分类和过滤资源的标签类型定义。

标签是附加到资源上的字符串标识符，用于组织、分类和查询操作。标签应具有描述性，以清晰指示资源功能、状态或所有权。

## constants GLOBAL
```python
GLOBAL: Tag = "__global__"
```

没有显式标签资源的默认标签常量。

具有此标签的资源被视为公开可访问或未分类的公共资源。通常用于系统级别或不需要特定分类的公共资源。


## enum TagMatchStrategy



## enum TagUpdateStrategy


## class Ok
```python
class Ok(value: T)
```
表示成功的操作结果。

此类以类型安全的方式封装成功的返回值，遵循显式错误处理的结果模式。

参数：

value(T)：要封装的成功结果值。

### is_ok
```python
def is_ok(self) -> bool
```
检查结果是否表示成功。

返回：

bool：始终为True，因为这是Ok实例。

### is_err
```python
def is_err(self) -> bool
```
检查结果是否表示错误。

返回：

bool：始终为False，因为这是Ok实例。

### msg
```python
def msg(self) -> T
```
获取成功消息/值。

返回：

T：封装的成功值。

注意：此命名可能令人困惑 - 考虑重命名为value()以增加清晰度，除非"msg"是您代码库中的既定约定。

## class Error
```python
class Error(error: E = None)
```
表示失败的操作结果。

此类以类型安全的方式封装错误值，遵循显式错误处理的结果模式。

参数：

error(E，可选)：要封装的错误值。

### is_ok
```python
def is_ok(self) -> bool
```
检查结果是否表示成功。

返回：

bool：始终为False，因为这是Error实例。

### is_err
```python
def is_err(self) -> bool
```
检查结果是否表示错误。

返回：

bool：始终为True，因为这是Error实例。

### msg
```python
def msg(self) -> E
```
获取错误消息/值。

返回：

E：封装的错误值。

注意：此命名可能令人困惑 - 考虑重命名为error()以与实际错误性质保持一致。

### error
```python
def error(self) -> E
```
获取错误值（msg()的替代方法）。

返回：

E：封装的错误值。

注意：同时拥有返回相同内容的msg()和error()方法可能是冗余的。考虑合并为一个方法。

### alias Result
```python
Result: TypeAlias = Ok[T] | Error[E]
```
使用结果模式的操作结果类型别名。

表示成功结果（Ok[T]）或错误结果（Error[E]）。此模式提供显式错误处理而无需异常，使错误状态成为类型系统和API契约的一部分。

示例用法：

函数返回Result[str, int]意味着它返回Ok[str]或Error[int]

启用错误处理的编译时检查

强制显式处理成功和错误情况

样例：

```python
>>> from openjiuwen.core.runner.resources_manager.base import Ok, Error, Result
>>> 
>>> def divide(a: int, b: int) -> Result[float, str]:
...     if b == 0:
...         return Error("Division by zero")
...     return Ok(a / b)
... 
>>> result = divide(10, 2)
>>> if result.is_ok():
...     print(f"Result: {result.msg()}")
... else:
...     print(f"Error: {result.msg()}")

```