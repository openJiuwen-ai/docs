# openjiuwen.core.foundation.tool

`openjiuwen.core.foundation.tool` 是 openJiuwen 的**工具模块**，负责：

- 定义工具抽象基类 `Tool` 与工具卡片 `ToolCard`，供 LLM 或工作流调用；
- 提供 `tool` 装饰器，将普通函数快速封装为 `LocalFunction` 工具；
- 支持本地函数（`LocalFunction`）、RESTful API（`RestfulApi`）、MCP（`MCPTool`）等实现；
- 提供 `ToolInfo` Schema，供 LLM 工具调用转换为 OpenAI 等 API 的 tools 格式。


## class ToolCard
```python
class openjiuwen.core.foundation.tool.ToolCard(BaseCard)
```
工具的元信息数据类，用于描述工具的基本属性。

* **id**(str)：工具的ID，唯一标识。默认为随机的uuid值。
* **name**(str)：工具的名字。默认值：`""`。
* **description**(str)：工具描述信息。默认值：`""`。
* **input_params**(Dict[str,Any] | Type[BaseModel])：工具的入参描述，为标准JsonSchema格式。

## class Tool
```python
class openjiuwen.core.foundation.tool.Tool(card: ToolCard)
```
Tool基类，定义的抽象接口实现了Function Calling的能力。开发者实现自定义工具时，需要继承Tool基类。

对应源码：`openjiuwen.core.foundation.tool.base`。


**参数**：

- `card: ToolCard`：工具卡片，不可为 `None`，且须有 `card.id`。默认值：无。
    - 若 `card` 为 `None`，抛出 `build_error(StatusCode.TOOL_CARD_NOT_SUPPORTED)`； 
    - 若 `not card.id`，抛出 `build_error(StatusCode.TOOL_CARD_ID_NOT_SUPPORTED, card=card)`；

**样例**：

```python

```

### card

```python
@property
def card(self) -> ToolCard
```

返回构造时传入的 ToolCard。

### invoke

```python
async def invoke(self, inputs: Input, **kwargs) -> Output
```
调用工具的异步接口。

**参数**：

* **inputs**(Input)：工具接口的输入参数对象，包含接口调用所需的具体参数值。
* **kwargs**：可变关键字参数，用于传递额外的请求配置或参数。

**返回**：

**Output**，工具调用的响应结果。

### stream

```python
async def stream(self, inputs: Input, **kwargs) -> AsyncIterator[Output]
```

调用工具的流式异步接口。

**参数**：

* **inputs**(Input)：工具接口的输入参数对象，包含接口调用所需的具体参数值。
* **kwargs**：可变关键字参数，用于传递额外的请求配置或参数。

**返回**：

**AsyncIterator[Output]**，工具异步调用的流式结果。




## class LocalFunction

```python
class openjiuwen.core.foundation.tool.LocalFunction(card: ToolCard, func: Callable)
```

将可调用对象封装为 Tool：持有 `ToolCard` 与 `func`，实现 `invoke`、`stream`；invoke 时支持同步/异步函数，stream 时要求 func 为 generator 或 async generator。

对应源码：`openjiuwen.core.foundation.tool.function.function.LocalFunction`。

**参数**：

- `card: ToolCard`：工具卡片。
- `func: Callable`：可调用对象，不可为 `None`；若为 `None` 抛出异常。


### invoke

```python
async def invoke(self, inputs: Input, **kwargs) -> Output
```

使用 inputs 执行工具，返回完整结果。

**参数**：

- `inputs: Input`：输入，通常为 dict 或与 input_params 兼容的结构；若 card 有 input_params，会经 `SchemaUtils.format_with_schema` 格式化。
- `**kwargs`：可传 `skip_none_value`、`skip_inputs_validate` 等。

**返回**：

- `Output`：`func(**inputs)` 的返回值（若 func 为协程则 `await self._func(**inputs)`）。


### stream

```python
async def stream(self, inputs: Input, **kwargs) -> AsyncIterator[Output]
```

使用 inputs 流式执行工具，逐块返回结果。

**参数**：同 `invoke`。

**返回**：

- `AsyncIterator[Output]`：逐块 yield 的结果。


## @tool

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


## class RestfulApiCard
```python
class openjiuwen.core.foundation.tool.RestfulApiCard(ToolCard)
```
RestfulApi工具的元信息数据类。

* **id**(str)：工具的ID，唯一标识。默认为随机的uuid值。
* **name**(str)：工具的名字。默认值：`""`。
* **description**(str)：工具描述信息。默认值：`""`。
* **input_params**(Dict[str,Any] | Type[BaseModel])：工具的入参描述，为标准JsonSchema格式。
* todo


## class RestfulApi
```python
class openjiuwen.core.foundation.tool.RestualApi(card: RestfulApiCard)
```


### invoke

### stream


## class McpToolCard
```python
class openjiuwen.core.foundation.tool.McpToolCard(card: ToolCard)
```


## class MCPTool




