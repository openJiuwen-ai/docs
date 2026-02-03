# openjiuwen.core.foundation.tool

`openjiuwen.core.foundation.tool` 模块提供了工具（Tool）的抽象定义及多种实现，支持本地函数、RESTful API 和 MCP（Model Context Protocol）工具。工具是 Agent 与外部世界交互的桥梁。

## alias Input

```python
Input = TypeVar('Input', contravariant=True)
```

工具输入的泛型类型变量。

## alias Output

```python
Output = TypeVar('Output', contravariant=True)
```

工具输出的泛型类型变量。

## class openjiuwen.core.foundation.tool.base.ToolCard

```python
class ToolCard(BaseCard)
```

工具卡片类，用于定义工具的元数据。

继承自 `BaseCard`，包含工具的基本信息和参数定义。

**参数**：

* **name**(str, 可选)：工具名称。
* **description**(str, 可选)：工具描述。
* **id**(str, 可选)：工具的唯一标识符。
* **input_params**(Dict[str, Any] | Type[BaseModel], 可选)：工具的输入参数模式（Schema）。可以是 JSON Schema 字典或 Pydantic 模型类。默认为空字典。

### tool_info() -> ToolInfo

将工具卡片转换为 `ToolInfo` 对象，供 LLM 调用使用。

**返回**：

**ToolInfo**，包含工具名称、描述和参数的 ToolInfo 对象。

## class openjiuwen.core.foundation.tool.base.Tool

```python
class Tool(ABC)
```

工具的抽象基类。

定义了工具的基本行为，所有具体工具实现都应继承此类。

**参数**：

* **card**(ToolCard)：工具卡片，定义了工具的行为和参数。不可为 None，且必须包含 id。

### card() -> ToolCard

获取工具对应的卡片。

**返回**：

**ToolCard**，工具的配置卡片。

### invoke(inputs: Input, **kwargs) -> Output

执行工具并返回完整结果。

这是一个抽象方法，子类必须实现具体的执行逻辑。

**参数**：

* **inputs**(Input)：符合工具输入模式的结构化输入数据。
* **\*\*kwargs**：额外的执行参数，如超时、重试策略或特定于工具的选项。

**返回**：

**Output**，工具执行的完整结果。

### stream(inputs: Input, **kwargs) -> AsyncIterator[Output]

执行工具并流式返回结果。

这是一个抽象方法，支持长运行操作的部分结果返回。

**参数**：

* **inputs**(Input)：符合工具输入模式的结构化输入数据。
* **\*\*kwargs**：额外的流式执行参数。

**返回**：

**AsyncIterator[Output]**，工具执行过程中的增量结果迭代器。

## class openjiuwen.core.foundation.tool.function.function.LocalFunction

```python
class LocalFunction(Tool)
```

本地函数工具类。

将本地的可调用对象（函数、协程等）封装为 `Tool`。

**参数**：

* **card**(ToolCard)：工具卡片。
* **func**(Callable)：要封装的本地可调用对象。不可为 None。

### invoke(inputs: Input, **kwargs) -> Output

执行本地函数。

支持同步函数和异步协程函数。如果在 `kwargs` 中指定了 `skip_inputs_validate=False`（默认），会先根据 `input_params` 格式化输入。

**参数**：

* **inputs**(Input)：输入数据。
* **\*\*kwargs**：可选参数，支持 `skip_none_value` (bool) 和 `skip_inputs_validate` (bool)。

**返回**：

**Output**，函数的返回值。

### stream(inputs: Input, **kwargs) -> AsyncIterator[Output]

流式执行本地函数。

仅支持生成器函数（Generator）或异步生成器函数（AsyncGenerator）。

**参数**：

* **inputs**(Input)：输入数据。
* **\*\*kwargs**：可选参数，同 `invoke`。

**返回**：

**AsyncIterator[Output]**，函数 yield 的值的异步迭代器。

## method openjiuwen.core.foundation.tool.tool.tool

```python
def tool(
    func: Optional[Callable] = None,
    *,
    name: Optional[str] = None,
    description: Optional[str] = None,
    input_params: Optional[Union[Dict[str, Any], Type[BaseModel]]] = None,
    card: Optional[ToolCard] = None,
    auto_extract: bool = True
) -> LocalFunction | Callable[[Callable], LocalFunction]
```

通用装饰器，用于将函数转换为 `LocalFunction` 工具。

支持多种用法：作为无参装饰器、带参装饰器或直接函数调用。

**参数**：

* **func**(Callable, 可选)：要装饰的函数。如果作为装饰器使用，此参数由 Python 自动传递。
* **name**(str, 可选)：覆盖函数的名称。如果未提供，默认使用函数名。
* **description**(str, 可选)：覆盖函数的描述。如果未提供，尝试从文档字符串或自动提取中获取。
* **input_params**(Dict[str, Any] | Type[BaseModel], 可选)：自定义参数模式。
* **card**(ToolCard, 可选)：预构建的 `ToolCard`。
* **auto_extract**(bool, 可选)：是否从函数签名自动提取参数模式。默认为 True。

**返回**：

**LocalFunction | Callable[[Callable], LocalFunction]**，返回 `LocalFunction` 实例（如果是直接调用或无参装饰器）或装饰器函数。

**样例**：

```python
>>> import asyncio
>>> from openjiuwen.core.foundation.tool import tool
>>> 
>>> # 1. 基本用法
>>> @tool
>>> def add(a: int, b: int) -> int:
>>>     """两数相加"""
>>>     return a + b
>>> 
>>> # 2. 自定义元数据
>>> @tool(name="multiply", description="Multiply two numbers")
>>> def mul(x: int, y: int) -> int:
>>>     return x * y
>>> 
>>> # 3. 异步流式工具
>>> @tool
>>> async def count_up(n: int):
>>>     for i in range(n):
>>>         yield i
>>> 
>>> async def main():
>>>     print(await add.invoke({"a": 1, "b": 2}))
>>>     print(await mul.invoke({"x": 3, "y": 4}))
>>>     async for i in count_up.stream({"n": 3}):
>>>         print(i, end=" ")
>>> 
>>> # 运行样例（实际使用时请在 async 环境中运行）
>>> # asyncio.run(main())
```

## class openjiuwen.core.foundation.tool.service_api.restful_api.RestfulApiCard

```python
class RestfulApiCard(ToolCard)
```

RESTful API 工具卡片配置。

**参数**：

* **url**(str)：API 的路径，例如 `/api/v1/users`。必须是有效的 URL。
* **method**(Literal["POST", "GET"])：HTTP 方法，仅支持 POST 或 GET。默认为 "POST"。
* **headers**(Dict[str, Any], 可选)：请求头。
* **queries**(Dict[str, Any], 可选)：请求查询参数。
* **paths**(Dict[str, Any], 可选)：URL 路径参数。
* **timeout**(float, 可选)：请求超时时间（秒）。默认为 60.0。
* **max_response_byte_size**(int, 可选)：最大响应字节数。默认为 10MB。
* **name**(str, 可选)：工具名称。
* **description**(str, 可选)：工具描述。
* **input_params**(Dict[str, Any] | Type[BaseModel], 可选)：输入参数模式。

## class openjiuwen.core.foundation.tool.service_api.restful_api.RestfulApi

```python
class RestfulApi(Tool)
```

RESTful API 工具实现。

通过 HTTP 请求调用远程 API。不支持 `stream` 方法。

**参数**：

* **card**(RestfulApiCard)：API 配置卡片。

### invoke(inputs: Input, **kwargs) -> Output

发送 HTTP 请求并返回响应结果。

**参数**：

* **inputs**(Input)：输入参数，将根据配置映射到 Body、Query、Path 或 Header。
* **\*\*kwargs**：支持 `timeout` (float) 覆盖默认超时。

**返回**：

**Output**，API 响应的 JSON 数据。

**样例**：

```python
>>> from openjiuwen.core.foundation.tool.service_api.restful_api import RestfulApi, RestfulApiCard
>>> 
>>> card = RestfulApiCard(
>>>     name="get_user",
>>>     description="Get user info",
>>>     url="https://api.example.com/users/{user_id}",
>>>     method="GET",
>>>     paths={"user_id": 123}, # 默认路径参数
>>>     input_params={"type": "object", "properties": {"user_id": {"type": "integer"}}}
>>> )
>>> 
>>> api_tool = RestfulApi(card)
>>> # await api_tool.invoke({"user_id": 456})
```

## class openjiuwen.core.foundation.tool.mcp.base.McpToolCard

```python
class McpToolCard(ToolCard)
```

MCP 工具卡片配置。

**参数**：

* **server_name**(str)：MCP 服务器名称。
* **server_id**(str, 可选)：服务器 ID，默认为空字符串。
* **name**(str, 可选)：工具名称。
* **description**(str, 可选)：工具描述。
* **input_params**(Dict[str, Any] | Type[BaseModel], 可选)：输入参数模式。

## class openjiuwen.core.foundation.tool.mcp.base.MCPTool

```python
class MCPTool(Tool)
```

MCP（Model Context Protocol）工具实现。

包装 MCP 客户端以调用远程 MCP 工具。不支持 `stream` 方法。

**参数**：

* **mcp_client**(Any)：MCP 客户端实例（如 `McpClient` 子类）。
* **tool_info**(McpToolCard)：MCP 工具配置卡片。

### invoke(inputs: Input, **kwargs) -> Output

调用 MCP 工具。

**参数**：

* **inputs**(Input)：输入参数。
* **\*\*kwargs**：额外参数。

**返回**：

**Output**，包含结果的字典 `{"result": ...}`。

**样例**：

```python
>>> from openjiuwen.core.foundation.tool.mcp.base import MCPTool, McpToolCard
>>> from unittest.mock import AsyncMock
>>> 
>>> # 模拟 MCP Client
>>> mock_client = AsyncMock()
>>> mock_client.call_tool.return_value = "Result"
>>> 
>>> card = McpToolCard(
>>>     name="mcp_tool",
>>>     server_name="test_server"
>>> )
>>> 
>>> mcp_tool = MCPTool(mcp_client=mock_client, tool_info=card)
>>> # await mcp_tool.invoke({})
```

## class openjiuwen.core.foundation.tool.mcp.base.McpServerConfig

```python
class McpServerConfig(BaseModel)
```

MCP 服务器配置类。

**参数**：

* **server_name**(str)：服务器名称。
* **server_path**(str)：服务器路径或命令。
* **client_type**(str, 可选)：客户端类型，如 'sse'。默认为 'sse'。
* **server_id**(str, 可选)：服务器 ID。默认生成 UUID。
* **params**(Dict[str, Any], 可选)：额外参数。
* **auth_headers**(dict, 可选)：认证头。
* **auth_query_params**(Dict[str, str], 可选)：认证查询参数。

## class openjiuwen.core.foundation.tool.schema.ToolInfo

```python
class ToolInfo(BaseModel)
```

工具信息模型，用于 LLM API 交互。

**参数**：

* **type**(str, 可选)：工具类型，默认为 "function"。
* **name**(str, 可选)：工具名称。
* **description**(str, 可选)：工具描述。
* **parameters**(Union[Dict[str, Any], Type[BaseModel]], 可选)：参数模式。
