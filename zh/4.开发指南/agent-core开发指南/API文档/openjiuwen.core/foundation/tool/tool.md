# openjiuwen.core.foundation.tool

`openjiuwen.core.foundation.tool` 是 openJiuwen 的**工具模块**，负责：

- 定义工具抽象基类 `Tool` 与工具卡片 `ToolCard`，供 LLM 或工作流调用；
- 提供 `tool` 装饰器，将普通函数快速封装为 `LocalFunction` 工具；
- 支持本地函数（`LocalFunction`）、RESTful API（`RestfulApi`）、MCP（`MCPTool`）等实现；
- 提供 `ToolInfo` Schema，供 LLM 工具调用转换为 OpenAI 等 API 的 tools 格式。

对应源码：`openjiuwen.core.foundation.tool`。

## class Tool / ToolCard

```python
class openjiuwen.core.foundation.tool.base.Tool(ABC)
class openjiuwen.core.foundation.tool.base.ToolCard(BaseCard)
```

- **Tool**：工具抽象基类，定义 `invoke`、`stream` 两个抽象方法，供 LLM 或工作流调用；构造时接收 `ToolCard`，通过 `card` 属性暴露。
- **ToolCard**：工具卡片，继承 `BaseCard`（含 name、description、id），增加 **input_params**（Dict[str, Any] 或 Pydantic Model），用于描述工具名称、说明与参数 schema；`tool_info()` 返回 `ToolInfo`。
- **Input** / **Output**：类型变量别名，用于泛型标注。

对应源码：`openjiuwen.core.foundation.tool.base`。

### Tool.__init__

```python
def __init__(self, card: ToolCard)
```

构造工具实例，校验 card 非空且含 id。

**参数**：

- `card: ToolCard`：工具卡片，不可为 `None`，且须有 `card.id`。默认值：无。

**行为**：

1. 若 `card` 为 `None`，抛出 `build_error(StatusCode.TOOL_CARD_NOT_SUPPORTED)`；
2. 若 `not card.id`，抛出 `build_error(StatusCode.TOOL_CARD_ID_NOT_SUPPORTED, card=card)`；
3. 将 `card` 保存为 `self._card`。

### Tool.card

```python
@property
def card(self) -> ToolCard
```

返回构造时传入的 ToolCard。

### abstractmethod async invoke

```python
async def invoke(self, inputs: Input, **kwargs) -> Output
```

使用给定 inputs 执行工具，返回完整结果。子类实现。

### abstractmethod async stream

```python
async def stream(self, inputs: Input, **kwargs) -> AsyncIterator[Output]
```

使用给定 inputs 流式执行工具，逐块返回结果。子类实现。

---

## tool 装饰器

```python
def tool(
    func: Optional[Callable] = None,
    *,
    name: Optional[str] = None,
    description: Optional[str] = None,
    input_params: Optional[Union[Dict[str, Any], Type[BaseModel]]] = None,
    card: Optional[ToolCard] = None,
    auto_extract: bool = True,
) -> LocalFunction | Callable[[Callable], LocalFunction]
```

将普通函数装饰为 `LocalFunction` 工具；支持无参 `@tool`、带参 `@tool(name="xxx")`、预构建 `ToolCard`、非装饰器用法 `tool(my_func)` 等。

**参数**：

- `func: Callable | None`：被装饰函数；省略时返回装饰器函数。默认值：`None`。
- `name: Optional[str]`：覆盖工具名称，未传时使用函数名。默认值：`None`。
- `description: Optional[str]`：覆盖工具描述；未传时优先从自动提取、docstring 获取。默认值：`None`。
- `input_params: Optional[Union[Dict[str, Any], Type[BaseModel]]]`：参数 schema（jsonschema 或 Pydantic 模型）；未传且 `auto_extract=True` 时从函数签名自动提取。默认值：`None`。
- `card: Optional[ToolCard]`：预构建的 ToolCard；若提供，可再通过 name/description/input_params 覆盖部分字段。默认值：`None`。
- `auto_extract: bool`：是否从函数签名自动提取描述与参数 schema。默认值：`True`。

**返回**：

- 若传入 `func`：返回 `LocalFunction` 实例；
- 若未传 `func`：返回装饰器函数，对被装饰函数应用后得到 `LocalFunction` 实例。

**行为**：

1. 若 `func` 非空，调用内部 `decorator(func)` 并返回；
2. 若 `func` 为空，返回 `decorator`；
3. 在 `decorator` 内：若 `card` 非空，调用 `_handle_prebuilt_card(func_, card, ...)`，根据 name/description/input_params 覆盖 card 字段（若有）后构造新 ToolCard，再构造 `LocalFunction(card=..., func=func)` 并返回；
4. 若 `card` 为空，调用 `_create_new_tool_card(func_, final_name, description, input_params, auto_extract)`：最终描述优先取 `description`，否则 `auto_extract` 时从 `CallableSchemaExtractor.extract_function_description(func)` 获取，否则取 docstring，否则 `f"Function {func.__name__}"`；最终 input_params 优先取 `input_params`，否则 `auto_extract` 时从 `CallableSchemaExtractor.generate_schema(func)` 获取，否则 `{"type": "object", "properties": {}}`；构造 `ToolCard` 与 `LocalFunction(card=new_card, func=func)` 并返回。

对应源码：`openjiuwen.core.foundation.tool.tool.tool`。

---

## class LocalFunction

```python
class openjiuwen.core.foundation.tool.function.function.LocalFunction(Tool)
```

将可调用对象封装为 Tool：持有 `ToolCard` 与 `func`，实现 `invoke`、`stream`；invoke 时支持同步/异步函数，stream 时要求 func 为 generator 或 async generator。

对应源码：`openjiuwen.core.foundation.tool.function.function.LocalFunction`。

### __init__

```python
def __init__(self, card: ToolCard, func: Callable)
```

**参数**：

- `card: ToolCard`：工具卡片。
- `func: Callable`：可调用对象，不可为 `None`；若为 `None` 抛出 `build_error(StatusCode.TOOL_LOCAL_FUNCTION_FUNC_NOT_SUPPORTED, card=self._card)`。

**行为**：

1. 调用 `super().__init__(card)`；
2. 若 `func` 为 `None`，抛出上述异常；
3. 保存 `self._func = func`。

### async invoke

```python
async def invoke(self, inputs: Input, **kwargs) -> Output
```

使用 inputs 执行工具，返回完整结果。

**参数**：

- `inputs: Input`：输入，通常为 dict 或与 input_params 兼容的结构；若 card 有 input_params，会经 `SchemaUtils.format_with_schema` 格式化。
- `**kwargs`：可传 `skip_none_value`、`skip_inputs_validate` 等。

**返回**：

- `Output`：`func(**inputs)` 的返回值（若 func 为协程则 `await self._func(**inputs)`）。

**行为**：

1. 若 `self.card.input_params` 非空，调用 `SchemaUtils.format_with_schema(inputs, self._card.input_params, skip_none_value=kwargs.get("skip_none_value", False), skip_validate=kwargs.get("skip_inputs_validate", False))` 得到格式化后的 `inputs`；
2. 若 `func` 为 generator 或 async generator，抛出 `build_error(StatusCode.TOOL_LOCAL_FUNCTION_EXECUTION_ERROR, interface="invoke", reason="func can not be generator", card=self._card)`；
3. 若 `func` 为协程函数，`res = await self._func(**inputs)`；否则 `res = self._func(**inputs)`；
4. 返回 `res`。

### async stream

```python
async def stream(self, inputs: Input, **kwargs) -> AsyncIterator[Output]
```

使用 inputs 流式执行工具，逐块返回结果。

**参数**：同 `invoke`。

**返回**：

- `AsyncIterator[Output]`：逐块 yield 的结果。

**行为**：

1. 同 invoke，先根据 `input_params` 格式化 `inputs`；
2. 若 `func` 为 async generator，`async for item in self._func(**inputs): yield item`；
3. 若 `func` 为同步 generator，`for item in self._func(**inputs): yield item`；
4. 否则抛出 `build_error(StatusCode.TOOL_LOCAL_FUNCTION_EXECUTION_ERROR, interface="stream", reason="func is not generator", card=self._card)`。

---

## RestfulApi / RestfulApiCard

- **RestfulApiCard**：继承 `ToolCard`，增加 `url: str`、`method: Literal["POST", "GET"]`、`headers`、`queries`、`paths`、`timeout`、`max_response_byte_size` 等 HTTP 配置；校验 method 仅支持 POST/GET、url 合法。
- **RestfulApi**：继承 `Tool`，接收 `RestfulApiCard`；`invoke` 时用 `APIParamMapper` 将 input_params 映射到 body/query/path/header，通过 aiohttp 发起 HTTP 请求并解析 JSON 响应；**stream 不支持**，调用会抛出 `build_error(StatusCode.TOOL_STREAM_NOT_SUPPORTED, card=self._card)`。

对应源码：`openjiuwen.core.foundation.tool.service_api.restful_api`。

---

## MCPTool / McpToolCard / McpServerConfig

- **McpServerConfig**：MCP 服务配置：`server_id`、`server_name`、`server_path`、`client_type`（如 `'sse'`）、`params`、`auth_headers`、`auth_query_params`。
- **McpToolCard**：继承 `ToolCard`，增加 `server_name: str`、`server_id: str`，用于标识 MCP 服务与工具。
- **MCPTool**：继承 `Tool`，包装 `McpClient` 与 `McpToolCard`；**invoke** 时将 inputs 作为 arguments 调用 `mcp_client.call_tool(tool_name=self._card.name, arguments=arguments)`，返回 `{"result": result}`；**stream 不支持**，会抛出 `TOOL_STREAM_NOT_SUPPORTED`。

对应源码：`openjiuwen.core.foundation.tool.mcp.base`。

---

## McpClient 及实现

- **McpClient**：抽象基类，定义 `async connect`、`async disconnect`、`async list_tools`、`async call_tool`、`async get_tool_info` 等抽象方法。
- **StdioClient**：基于 stdio 传输，通过 `StdioServerParameters`（command、args、env 等）连接本地进程。
- **SseClient**：基于 SSE（Server-Sent Events），通过 server_path（URL）连接，支持 auth_headers、auth_query_params。
- **PlaywrightClient**：基于 Playwright 浏览器会话，可根据 server_path 类型选择 stdio 或 SSE 传输。

对应源码：`openjiuwen.core.foundation.tool.mcp.client.mcp_client`、`stdio_client`、`sse_client`、`playwright_client`。

---

## 辅助类型

### ToolInfo

```python
class openjiuwen.core.foundation.tool.schema.ToolInfo(BaseModel)
```

供 LLM 工具调用使用的工具信息，由 `ToolCard.tool_info()` 返回，用于转换为 OpenAI 等 API 的 tools 格式。

**字段**：

- `type: str`：工具类型，默认 `"function"`。
- `name: str`：工具名称，默认 `""`。
- `description: str`：工具描述，默认 `""`。
- `parameters: Union[Dict[str, Any], Type[BaseModel]]`：参数 schema（dict 或 Pydantic Model），默认 `{}`。

---

## 典型使用流程示例

```python
import asyncio
from openjiuwen.core.foundation.tool import tool, ToolInfo, LocalFunction
from openjiuwen.core.foundation.tool.base import ToolCard


# 1. 使用 @tool 装饰器（无参）
@tool
def add(a: int, b: int) -> int:
    """两数相加"""
    return a + b


# 2. 使用 @tool 带参数
@tool(name="multiply", description="两数相乘")
def mul(x: int, y: int) -> int:
    return x * y


# 3. 流式工具（generator）
@tool
def count_up(n: int):
    """从 0 数到 n-1"""
    for i in range(n):
        yield i


async def demo_tools():
    # invoke 同步工具
    result = await add.invoke({"a": 1, "b": 2})
    print(result)  # 3

    result2 = await mul.invoke({"x": 3, "y": 4})
    print(result2)  # 12

    # 获取 ToolInfo 供 LLM 使用
    info: ToolInfo = add.card.tool_info()
    print(info.name, info.description, info.parameters)

    # 流式调用
    async for num in count_up.stream({"n": 3}):
        print(num)  # 0, 1, 2


asyncio.run(demo_tools())
```

> **说明**：`LocalFunction` 的 `invoke` 不支持 generator/async generator 函数，仅 `stream` 支持；`RestfulApi`、`MCPTool` 不支持 `stream`，调用会抛出 `TOOL_STREAM_NOT_SUPPORTED`。使用 `tool` 装饰器时，若未传 `name`/`description`/`input_params` 且 `auto_extract=True`，会从函数签名与 docstring 自动提取。
