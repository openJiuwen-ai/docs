# openjiuwen.core.utils.llm.messages

## class openjiuwen.core.utils.llm.messages.Parameters

```python
openjiuwen.core.utils.llm.messages.Parameters(type: str = "object", properties: Dict[str, Any] = {}, required: List[str] = [])
```

描述了工具入参的参数结构化定义，用于描述参数类型、约束与说明。
**参数**：

* ​**type**​(str, 可选)：工具入参的参数类型，指明该参数的整体类型，通常为对象类型，用于描述入参的结构。默认值：`"object"`。
* ​**properties**​(Dict[str, Any], 可选)：工具参数输入属性，键为参数名称，值为该参数的类型、约束、描述等。此项需要与工具函数的实际参数输入一一对应，默认值：`{}`。
* ​**required**​(List[str])：必选参数列表。若未提供这些参数，将触发验证错误并抛出异常。

**异常**：

* **ValidationError**：未提供必选参数，则抛出。

**样例**：

```python
>>> params = Parameters(
...  properties = {
...   "city": {"type": "string", "description": "城市名"},
...   "units": {"type": "string", "enum": ["c", "f"], "default": "c"}
...  },
...     required = ["city"]
... )
```

## class openjiuwen.core.utils.llm.messages.Function

```python
openjiuwen.core.utils.llm.messages.Function(name: str, description: str, parameters: Parameters)
```

可被调用的工具函数信息。
**参数**：

* ​**name**​(str)：工具函数名，指定工具调用时要调用的函数名称，用于标识具体功能。
* ​**description**​(str)：函数功能描述，用于说明函数的用途和功能，提供对函数的简要说明。
* ​**parameters**([Parameters](messages.md#class-openjiuwencoreutilsllmmessagesparameters))：工具入参参数的结构化定义，用于描述参数类型、约束与说明。

**样例**：

```python
>>> func = Function(name="get_weather", description="按城市查询天气", parameters=params)
```

## class openjiuwen.core.utils.llm.messages.ToolInfo

```python
openjiuwen.core.utils.llm.messages.ToolInfo(type: str = "function", function: Function)
```

描述工具的信息，封装了一个函数，通常用于注册工具以供模型调用。

**参数**：

* ​**type**​(str)：工具类型，固定为 `"function"`，表示该工具是一个函数类型工具，用于执行具体的操作。
* ​**function**([Function](messages.md#class-openjiuwencoreutilsllmmessagesfunction)​)：工具对应的函数定义，`Function` 类的实例。此项定义了工具的功能及其输入参数。

**样例**：

```python
>>> tool = ToolInfo(function=func)
```

## class openjiuwen.core.utils.llm.messages.FunctionInfo

```python
openjiuwen.core.utils.llm.messages.FunctionInfo(name: str, arguments: str)
```

表示工具函数的基本信息，包含函数名称及其参数的原始字符串。

**参数**：

* ​**name**​(str)：被调用的函数名。
* ​**arguments**​(str)：函数的原始参数字符串，描述了函数的输入参数。

**样例**：

```python
>>> funcInfo = FunctionInfo(name="get_weather", arguments='{"city":"上海","units":"c"}')
```

## class openjiuwen.core.utils.llm.messages.ToolCall

```python
openjiuwen.core.utils.llm.messages.ToolCall(args: Dict[str, Any] = {}, id: Optional[str] = None, index: int = 0, type: str, function: FunctionInfo)
```

表示一次工具调用的记录，包含了调用的函数信息以及所使用的参数。

**参数**：

* ​**args**​(Dict[str, Any], 可选)：函数调用时传递的参数字典，记录工具调用时传递给函数的所有参数，默认值：`{}`。
* ​**id**​(Optional[str])：工具调用的唯一标识符，用于唯一标识一次工具调用。如果多个工具调用有重复调用的可能，可帮助区分每次调用，不能为空。
* ​**index**​(int, 可选)：在本次回复中的调用序号（从 0 开始），表示其在当前响应中的位置，默认值：`0`。
* ​**type**​(str)：工具调用的类型，包括`function`、`function_call`。
* ​**function**​([FunctionInfo](messages.md#class-openjiuwencoreutilsllmmessagesfunctioninfo))：实际被调用的函数信息。

**样例**：

```python
>>> toolCall = ToolCall(id="test", type="function", function=funcInfo, index=0)
```

## class openjiuwen.core.utils.llm.messages.BaseMessage

```python
openjiuwen.core.utils.llm.messages.BaseMessage(role: str, content: Union[str, List[Union[str, Dict]]] = "", name: Optional[str] = None)
```

通用消息基类，表示消息的基本结构，包括角色、内容以及名称。

**参数**：

* ​**role**​(str)：消息的角色，通常为 `"user"`、`"assistant"`、`"system"`、`"tool"`，表示发送者的角色。
* ​**content**​(Union[str, List[Union[str, Dict]]], 可选)：消息内容，可以是字符串，也可以是包含字符串或字典的列表，默认值：`""`。
* ​**name**​(Optional[str], 可选)：发送消息的角色名称标识，默认值：`None`。
* **样例**：

```python
>>> base_msg = BaseMessage(role="user", content="你好")
```

## class openjiuwen.core.utils.llm.messages.UsageMetadata

```python
openjiuwen.core.utils.llm.messages.UsageMetadata(code: int = 0, errmsg: str = "", prompt: str = "", task_id: str = "", model_name: str = "", finish_reason: str = "", total_latency: float = 0.0, model_stats: dict = {}, first_token_time: str = "", request_start_time: str = "")
```

表示一次模型调用的元数据，包括状态码、延迟、模型统计信息等。

**参数**：

* ​**code**​(int, 可选)：业务状态码，表示调用的状态，`0` 为成功，非 `0` 表示失败，默认值：`0`。
* ​**errmsg**​(str, 可选)：错误信息，当模型处理过程中出现问题时，提供错误信息。默认值：`""`。
* ​**prompt**​(str, 可选)：实际发送给模型的提示词，描述用户输入或系统提示词的内容。默认值：`""`。
* ​**task_id**​(str, 可选)：任务的唯一标识符，用于唯一标识一个任务，便于追踪和管理任务，默认值：`""`。
* ​**model_name**​(str, 可选)：使用的模型名称，默认值：`""`。
* ​**finish_reason**​(str, 可选)：生成结束原因，可能为`"stop"`、`"tool_calls"`，默认值：`""`。
* ​**total_latency**​(float, 可选)：端到端耗时（秒），表示从任务开始到结束的总时间，默认值：`0.0`。
* ​**model_stats**​(dict, 可选)：模型侧统计数据（如 token 用量），默认值：`{}`。
* ​**first_token_time**​(str, 可选)：首次生成token的时间戳，记录首次生成 token 的时间，通常用于分析生成的延迟，默认值：`""`。
* ​**request_start_time**​(str, 可选)：请求开始的时间戳，记录模型请求开始的时间，默认值：`""`。

**样例**：

```python
>>> usage = UsageMetadata(code=0, model_name="qwen-plus", finish_reason="stop", total_latency=0.123)
```

## class openjiuwen.core.utils.llm.messages.AIMessage

```python
openjiuwen.core.utils.llm.messages.AIMessage(role: str = "assistant", content: Union[str, List[Union[str, Dict]]] = "", name: Optional[str] = None, tool_calls: Optional[List[ToolCall]] = None, usage_metadata: Optional[UsageMetadata] = None, raw_content: Optional[str] = None, reason_content: Optional[str] = None)
```

表示助手回复的消息，继承自 `BaseMessage`，用于承载模型回复与工具调用。

**参数**：

* ​**role**​(str)：固定为 `"assistant"`，默认值：`"assistant"`。
* ​**content**​(Union[str, List[Union[str, Dict]]], 可选)：助手的回复内容，默认值：`""`。
* ​**name**​(Optional[str], 可选)：可选名称，标识该消息的来源或其他附加信息，默认值：`None`。
* ​**tool_calls**​(Optional[List[[ToolCall](messages.md#class-openjiuwencoreutilsllmmessagestoolcall)], 可选)：本次回复内的工具调用列表，默认值：`None`。
* ​**usage_metadata**​(Optional[[UsageMetadata](messages.md#class-openjiuwencoreutilsllmmessagesusagemetadata)], 可选)：本次回复的统计元数据，默认值：`None`。
* ​**raw_content**​(Optional[str], 可选)：原始未处理内容，默认值：`None`。
* ​**reason_content**​(Optional[str], 可选)：思考或解释内容，默认值：`None`。

**样例**：

```python
>>> ai_msg = AIMessage(content="天气已查询完成。", tool_calls=[tcall], usage_metadata=usage)
```

## class openjiuwen.core.utils.llm.messages.HumanMessage

```python
openjiuwen.core.utils.llm.messages.HumanMessage(role: str = "user", content: Union[str, List[Union[str, Dict]]] = "", name: Optional[str] = None)
```

表示用户发送的消息，继承自 `BaseMessage`。

**参数**：

* ​**role**​(str)：固定为 `"user"`，默认值：`"user"`。
* ​**content**​(Union[str, List[Union[str, Dict]]], 可选)：用户输入的消息内容，默认值：`""`。
* ​**name**​(Optional[str], 可选)：可选名称，标识该消息的来源或其他附加信息，默认值：`None`。

**样例**：

```python
>>> human_msg = HumanMessage(content="帮我查下今天上海的温度")
```

## class openjiuwen.core.utils.llm.messages.SystemMessage

```python
openjiuwen.core.utils.llm.messages.SystemMessage(role: str = "system", content: Union[str, List[Union[str, Dict]]] = "", name: Optional[str] = None)
```

表示系统消息，继承自 `BaseMessage`，用于设置对话系统级指令。

**参数**：

* ​**role**​(str)：固定为 `"system"`，默认值：`"system"`。
* ​**content**​(Union[str, List[Union[str, Dict]]], 可选)：系统提示或指令，默认值：`""`。
* ​**name**​(Optional[str], 可选)：可选名称，标识该消息的来源或其他附加信息，默认值：`None`。

**样例**：

```python
>>> sys_msg = SystemMessage(content="你是一名助手。")
```

## class openjiuwen.core.utils.llm.messages.ToolMessage

```python
openjiuwen.core.utils.llm.messages.ToolMessage(role: str = "tool", content: Union[str, List[Union[str, Dict]]] = "", name: Optional[str] = None, tool_call_id: str)
```

表示工具返回消息，继承自 `BaseMessage`，展示工具执行的结果。

**参数**：

* ​**role**​(str)：固定为 `"tool"`，默认值：`"tool"`。
* ​**content**​(Union[str, List[Union[str, Dict]]])：工具的输出内容，默认值：`""`。
* ​**name**​(Optional[str])：可选名称，标识该消息的来源或其他附加信息，默认值：`None`。
* ​**tool_call_id**​(str)：对应工具调用的唯一标识符。

**样例**：

```python
>>> tool_msg = ToolMessage(tool_call_id=toolCall.id or "call_001", content='{"temp": 23, "units": "c"}')
```
