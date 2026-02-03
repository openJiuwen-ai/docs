# openjiuwen.core.foundation.llm

`openjiuwen.core.foundation.llm` 是 openJiuwen 中统一的**大模型调用与管理模块**，负责：

- 提供 `Model` 统一调用入口，根据 `client_provider` 创建并委托给对应 ModelClient；
- 定义 `BaseModelClient` 抽象基类及 `OpenAIModelClient`、`SiliconFlowModelClient` 实现；
- 提供模型请求/客户端配置（`ModelRequestConfig`、`ModelClientConfig`）及消息、流式块、工具调用等 Schema；
- 提供输出解析器抽象（`BaseOutputParser`）及 `JsonOutputParser`、`MarkdownOutputParser` 实现。

对应源码：`openjiuwen.core.foundation.llm`。

## class Model

```python
class openjiuwen.core.foundation.llm.model.Model
```

统一的 LLM 调用入口：根据 `model_client_config.client_provider` 从注册表选择 ModelClient 实现并创建实例，将 `invoke`、`stream` 委托给该 ModelClient。

对应源码：`openjiuwen.core.foundation.llm.model.Model`。

### __init__

```python
def __init__(
    self,
    model_client_config: Optional[ModelClientConfig],
    model_config: ModelRequestConfig = None,
)
```

初始化 Model 实例，并创建对应的 ModelClient。

**参数**：

- `model_client_config: ModelClientConfig | None`：客户端配置（api_key、api_base、client_provider、client_id 等）。若为 `None`，抛出 `JiuWenBaseException(StatusCode.MODEL_SERVICE_CONFIG_ERROR, error_msg="model client config is none.")`。
- `model_config: ModelRequestConfig | None`：模型请求参数（model_name、temperature、top_p、max_tokens、stop 等）。默认值：`None`。

**行为**：

1. 保存 `model_config`、`model_client_config`，初始化 `_client: Optional[BaseModelClient] = None`；
2. 若 `model_client_config` 为 `None`，抛出 `JiuWenBaseException`（见上）；
3. 调用 `_create_model_client(model_client_config)`：校验 `client_provider`、`client_id` 非空，否则抛出异常；根据 `client_provider`（当前支持 `"OpenAI"`、`"SiliconFlow"`）从 `_CLIENT_TYPE_REGISTRY` 取实现类，用 `(self.model_config, client_config)` 构造实例并赋值给 `self._client`；不支持的 provider 抛出 `JiuWenBaseException` 并列出支持的类型。

### async invoke

```python
async def invoke(
    self,
    messages: Union[str, List[BaseMessage], List[dict]],
    *,
    tools: Union[List[ToolInfo], List[dict], None] = None,
    temperature: Optional[float] = None,
    top_p: Optional[float] = None,
    max_tokens: Optional[int] = None,
    stop: Union[Optional[str], None] = None,
    model: str = None,
    output_parser: Optional[BaseOutputParser] = None,
    timeout: float = None,
    **kwargs
) -> AssistantMessage
```

异步调用 LLM，返回完整助手消息。

**参数**：

- `messages: Union[str, List[BaseMessage], List[dict]]`：**必填**，输入内容；可为单字符串、BaseMessage 列表或已序列化的 dict 列表。
- `tools: Union[List[ToolInfo], List[dict], None]`：工具列表，可选。默认值：`None`。
- `temperature: Optional[float]`：采样温度，可选；未传时使用 model_config 中的值。默认值：`None`。
- `top_p: Optional[float]`：top_p 采样，可选。默认值：`None`。
- `max_tokens: Optional[int]`：最大生成 token 数，可选。默认值：`None`。
- `stop: Union[Optional[str], None]`：停止序列，可选。默认值：`None`。
- `model: str`：模型名覆盖，可选。默认值：`None`。
- `output_parser: Optional[BaseOutputParser]`：输出解析器，可选。默认值：`None`。
- `timeout: float`：请求超时（秒），可选。默认值：`None`。
- `**kwargs`：其他参数，透传给 ModelClient。

**返回**：

- `AssistantMessage`：助手回复消息，包含 content、tool_calls、usage_metadata 等。

**行为**：

1. 直接委托给 `self._client.invoke(messages=..., tools=..., temperature=..., ...)`，返回其返回值。

### async stream

```python
async def stream(
    self,
    messages: Union[str, List[BaseMessage], List[dict]],
    *,
    tools: Union[List[ToolInfo], List[dict], None] = None,
    temperature: Optional[float] = None,
    top_p: Optional[float] = None,
    max_tokens: Optional[int] = None,
    stop: Union[Optional[str], None] = None,
    model: str = None,
    output_parser: Optional[BaseOutputParser] = None,
    timeout: float = None,
    **kwargs
) -> AsyncIterator[AssistantMessageChunk]
```

异步流式调用 LLM，逐块返回助手消息。

**参数**：与 `invoke` 相同（无 `output_parser` 时流式块未经解析器处理）。

**返回**：

- `AsyncIterator[AssistantMessageChunk]`：异步迭代器，每次 yield 一个 `AssistantMessageChunk`。

**行为**：

1. 委托给 `self._client.stream(...)`，`async for chunk in self._client.stream(...): yield chunk`。

---

## class BaseModelClient

```python
class openjiuwen.core.foundation.llm.model_clients.base_model_client.BaseModelClient(ABC)
```

大模型客户端抽象基类，所有 ModelClient 实现需继承此类并实现 `invoke`、`stream`；提供消息/工具转 dict、请求参数构建等辅助方法。

对应源码：`openjiuwen.core.foundation.llm.model_clients.base_model_client.BaseModelClient`。

### __init__

```python
def __init__(self, model_config: ModelRequestConfig, model_client_config: ModelClientConfig)
```

初始化 ModelClient，并校验配置。

**参数**：

- `model_config: ModelRequestConfig`：模型请求参数（temperature、top_p、model_name 等）。
- `model_client_config: ModelClientConfig`：客户端配置（api_key、api_base、timeout、verify_ssl、ssl_cert 等）。

**行为**：

1. 保存 `model_config`、`model_client_config`；
2. 调用 `_validate_config()`：校验 `api_key`、`api_base` 非空；校验 `verify_ssl` 为 bool；若 `verify_ssl` 为 `True` 则校验 `ssl_cert` 非空；不通过则抛出 `JiuWenBaseException(StatusCode.MODEL_SERVICE_CONFIG_ERROR, ...)`。

### abstractmethod async invoke

```python
async def invoke(
    self,
    messages: Union[str, List[BaseMessage], List[dict]],
    *,
    tools: Union[List[ToolInfo], List[dict], None] = None,
    temperature: Optional[float] = None,
    top_p: Optional[float] = None,
    model: str = None,
    max_tokens: Optional[int] = None,
    stop: Union[Optional[str], None] = None,
    output_parser: Optional[BaseOutputParser] = None,
    timeout: float = None,
    **kwargs
) -> AssistantMessage
```

异步调用 LLM，返回单条完整助手消息。子类实现。

### abstractmethod async stream

```python
async def stream(
    self,
    messages: Union[str, List[BaseMessage], List[dict]],
    *,
    tools: Union[List[ToolInfo], List[dict], None] = None,
    temperature: Optional[float] = None,
    top_p: Optional[float] = None,
    model: str = None,
    max_tokens: Optional[int] = None,
    stop: Union[Optional[str], None] = None,
    output_parser: Optional[BaseOutputParser] = None,
    timeout: float = None,
    **kwargs
) -> AsyncIterator[AssistantMessageChunk]
```

异步流式调用 LLM，逐块返回助手消息。子类实现。

### 辅助方法（供子类复用）

- `_convert_messages_to_dict(messages)`：将 str 或 `List[BaseMessage]`/`List[dict]` 转为 OpenAI 风格 `[{"role": "...", "content": "..."}]`；支持 `AssistantMessage.tool_calls`、`ToolMessage.tool_call_id`。
- `_convert_tools_to_dict(tools)`：将 `List[ToolInfo]` 或 `List[dict]` 转为 `[{"type": "function", "function": {...}}]`。
- `_build_request_params(..., messages, tools, temperature, model, stop, max_tokens, stream, **kwargs)`：组装请求参数字典（model、messages、stream、temperature、top_p、max_tokens、stop、tools、tool_choice 等），优先使用方法参数，否则使用 `model_config`；内部会调用上述 convert 方法并记录 LLM 调用日志。

---

## OpenAIModelClient / SiliconFlowModelClient

- **OpenAIModelClient**：继承 `BaseModelClient`，对接 OpenAI 兼容 API（通过 `openai.AsyncOpenAI`），使用 `ModelClientConfig` 的 api_key、api_base、timeout、verify_ssl、ssl_cert 等；实现 `invoke`、`stream`，支持 tool_calls、output_parser。对应源码：`openjiuwen.core.foundation.llm.model_clients.openai_model_client.OpenAIModelClient`。
- **SiliconFlowModelClient**：继承 `BaseModelClient`，对接 SiliconFlow 服务，配置方式与 OpenAI 类似。对应源码：`openjiuwen.core.foundation.llm.model_clients.siliconflow_model_client.SiliconFlowModelClient`。

`Model` 构造时根据 `model_client_config.client_provider` 在 `"OpenAI"`、`"SiliconFlow"` 中选择对应实现。

---

## 辅助类型

### ModelClientConfig

```python
class openjiuwen.core.foundation.llm.schema.config.ModelClientConfig(BaseModel)
```

客户端配置。

**字段**：

- `client_id: str`：客户端唯一标识，默认由 `uuid.uuid4()` 生成，用于在 Runner 中注册。
- `client_provider: str`：服务商标识，枚举值：`OpenAI`、`SiliconFlow`。
- `api_key: str`：API 密钥。
- `api_base: str`：API 基础 URL。
- `timeout: float`：请求超时（秒），默认 60.0。
- `max_retries: int`：最大重试次数，默认 3。
- `verify_ssl: bool`：是否验证 SSL，默认 True。
- `ssl_cert: Optional[str]`：SSL 证书路径，默认 None。

### ModelRequestConfig

```python
class openjiuwen.core.foundation.llm.schema.config.ModelRequestConfig(BaseModel)
```

单次请求的模型参数。

**字段**：

- `model_name: str`：模型名，别名 `model`，默认 `""`。
- `temperature: float`：温度，默认 0.95。
- `top_p: float`：top_p，默认 0.1。
- `max_tokens: Optional[int]`：最大 token 数，默认 None。
- `stop: Union[Optional[str], None]`：停止序列，默认 None。
- 支持 `model_config = {"extra": "allow"}` 以透传额外字段。

### BaseMessage / UserMessage / SystemMessage / AssistantMessage / ToolMessage

- **BaseMessage**：`role: str`、`content: Union[str, List[Union[str, dict]]]`、`name: Optional[str]`。
- **UserMessage**：role 固定为 `"user"`。
- **SystemMessage**：role 固定为 `"system"`。
- **AssistantMessage**：role 为 `"assistant"`；`tool_calls: Optional[List[ToolCall]]`、`usage_metadata: Optional[UsageMetadata]`、`finish_reason: str`、`parser_content`、`reasoning_content`；可从 OpenAI 返回的嵌套 tool_calls 格式自动转成扁平 ToolCall。
- **ToolMessage**：role 为 `"tool"`；必填 `tool_call_id: str`。

### UsageMetadata

```python
class openjiuwen.core.foundation.llm.schema.message.UsageMetadata(BaseModel)
```

用量与元数据：`input_tokens`、`output_tokens`、`total_tokens`、`cache_tokens`、`total_latency`、`model_name`、`request_start_time`、`first_token_time` 等。

### AssistantMessageChunk / BaseMessageChunk

- **BaseMessageChunk**：继承 BaseMessage，支持 `__add__` 合并内容（字符串或列表拼接）。
- **AssistantMessageChunk**：助手消息的流式块，支持与另一 AssistantMessageChunk 合并（content、tool_calls 按 id 合并片段，usage_metadata、finish_reason 等取后者或合并）。

### ToolCall

```python
class openjiuwen.core.foundation.llm.schema.tool_call.ToolCall(BaseModel)
```

单次工具调用：`id: Optional[str]`、`type: str`、`name: str`、`arguments: str`、`index: Optional[int]`。

### BaseOutputParser / JsonOutputParser / MarkdownOutputParser

- **BaseOutputParser**：抽象基类，定义 `async def parse(self, inputs) -> Any`、`async def stream_parse(self, streaming_inputs: AsyncIterator) -> AsyncIterator[Any]`。
- **JsonOutputParser**：从 AssistantMessage 或字符串中提取 `` ```json ... ``` `` 代码块并解析为 JSON；支持流式解析。
- **MarkdownOutputParser**：将 Markdown 文本解析为结构化内容（如 MarkdownContent、MarkdownElement 等），支持流式解析。

---

## 典型使用流程示例

```python
import os
import asyncio
from openjiuwen.core.foundation.llm import (
    Model,
    ModelRequestConfig,
    ModelClientConfig,
    UserMessage,
    AssistantMessage,
)


async def demo_model():
    # 1. 构造配置与 Model
    model_config = ModelRequestConfig(
        model_name="<model_name>",
        temperature=0.7,
        max_tokens=1024,
    )
    client_config = ModelClientConfig(
        client_id="my_llm",
        client_provider="OpenAI",
        api_key=os.getenv("OPENAI_API_KEY"),
        api_base=os.getenv("OPENAI_API_BASE", "https://api.openai.com/v1"),
    )
    model = Model(model_client_config=client_config, model_config=model_config)

    # 2. 同步调用（字符串输入）
    response: AssistantMessage = await model.invoke("你好，请用一句话介绍你自己。")
    print(response.content)

    # 3. 多轮消息
    messages = [
        UserMessage(content="我叫小明。"),
        UserMessage(content="请记住我的名字，并说一声你好。"),
    ]
    response2 = await model.invoke(messages)
    print(response2.content)

    # 4. 流式调用
    async for chunk in model.stream("写一首短诗。"):
        if chunk.content:
            print(chunk.content, end="", flush=True)
    print()


asyncio.run(demo_model())
```

> **说明**：`model_client_config` 不可为 `None`；`client_provider` 当前仅支持 `"OpenAI"`、`"SiliconFlow"`，其他值会抛出异常并提示支持的类型。未传的 `temperature`、`top_p`、`max_tokens`、`stop` 等将使用 `model_config` 中的默认值。
