# openjiuwen.core.operator.llm_call.base

## class LLMCall

```python
class openjiuwen.core.operator.llm_call.base.LLMCall
```

`LLMCall` 是对大模型调用进行封装的算子类，用于：

- 将一个 `Model` 实例与固定的 `model_name`、系统提示词（system prompt）和用户提示词（user prompt）绑定；
*- 在调用时将输入变量渲染进提示词，组合历史消息，形成完整的消息列表；
*- 支持普通调用 (`invoke`) 与流式调用 (`stream`)，并在需要时回调优化器函数记录执行结果用于后续优化。

对应源码：`openjiuwen.core.operator.llm_call.base.LLMCall`。

### __init__

```python
def __init__(
    self,
    model_name: str,
    llm: Model,
    system_prompt: str | list[BaseMessage] | list[dict],
    user_prompt: str | list[BaseMessage] | list[dict],
    freeze_system_prompt: bool = False,
    freeze_user_prompt: bool = True,
    llm_call_id: str = "llm_call",
) -> None
```

**参数**：

- `model_name: str`：要调用的大模型名称（例如 `"gpt-4o"`），在调用 `llm.invoke/llm.stream` 时作为 `model` 传入；
- `llm: Model`：`openjiuwen.core.foundation.llm.Model` 实例，已经配置好 `ModelClientConfig` / `ModelRequestConfig` 等参数；
- `system_prompt`：系统提示词模版内容：
  - 可以是字符串；
  - 也可以是 `BaseMessage` 列表或等价的字典列表；
  - 内部会包装成 `PromptTemplate`。
- `user_prompt`：用户提示词模版内容，类型与 `system_prompt` 一致；
- `freeze_system_prompt: bool`：是否冻结系统提示词模板，若为 `True`，后续调用 `update_system_prompt` 将不再生效；
- `freeze_user_prompt: bool`：是否冻结用户提示词模板，若为 `True`，后续调用 `update_user_prompt` 将不再生效；
- `llm_call_id: str`：该 `LLMCall` 的标识，用于在优化器回调中区分不同的调用算子。

### async invoke

```python
async def invoke(
    self,
    inputs: dict[str, Any],
    session: Session,
    history: list[BaseMessage] | None = None,
    tools: list[ToolInfo] | None = None,
) -> BaseMessage:
```

异步调用大模型，返回完整的单条响应消息。

**参数**：

- `inputs: dict[str, Any]`：用于渲染系统提示词和用户提示词模版的变量字典（例如 `{"query": "..."}`）；
- `session: Session`：会话对象，用于在触发优化器回调时传递上下文；
- `history: list[BaseMessage] | None`：历史对话消息列表，可为 `None`；
- `tools: list[ToolInfo] | None`：可选的工具列表，在需要工具调用时传入。

**行为**：

1. 调用内部 `_format_llm_input(inputs, history)` 构造消息列表：
   - 使用系统提示词模版渲染出一组 `SystemMessage`；
   - 拼接历史消息 `history`；
   - 使用用户提示词模版渲染出用户消息；
2. 调用底层模型：

   ```python
   response = await self._llm.invoke(model=self._model_name, messages=messages, tools=tools)
   ```

3. 若已通过 `set_optimizer_callback` 设置优化器回调，则在得到响应后调用：

   ```python
   await self._optimizer_callback(self._llm_call_id, inputs, response, session)
   ```

4. 返回 `response`（`BaseMessage` 实例）。

### async stream

```python
async def stream(
    self,
    inputs: dict[str, Any],
    session: Session,
    history: list[BaseMessage] | None = None,
    tools: list[ToolInfo] | None = None,
) -> AsyncIterator:
```

异步流式调用大模型，逐块返回消息片段，并在结束时触发优化器回调。

**行为**：

1. 同样通过 `_format_llm_input` 构造消息列表；
2. 调用底层模型的 `stream` 接口：

   ```python
   async for chunk in self._llm.stream(model=self._model_name, messages=messages, tools=tools):
       ...
       yield chunk
   ```

3. 将所有片段的内容累积成一个字符串 `response`，在流完成后：

   ```python
   await self._optimizer_callback(self._llm_call_id, inputs, response, session)
   ```

> 注意：`stream` 方法本身不会返回最终聚合的 `BaseMessage`，而是通过迭代器逐块返回底层模型的流式输出。

### 提示词相关接口

```python
def get_system_prompt(self) -> PromptTemplate
def get_user_prompt(self) -> PromptTemplate

def update_system_prompt(self, system_prompt: str | list[BaseMessage] | list[dict]) -> None
def update_user_prompt(self, user_prompt: str | list[BaseMessage] | list[dict]) -> None

def set_freeze_system_prompt(self, switch: bool) -> None
def set_freeze_user_prompt(self, switch: bool) -> None

def get_freeze_system_prompt(self) -> bool
def get_freeze_user_prompt(self) -> bool
```

- `get_system_prompt / get_user_prompt`：返回当前使用的系统/用户提示词模板对象（`PromptTemplate` 实例）；
- `update_system_prompt / update_user_prompt`：在未冻结的情况下更新提示词模板内容；
- `set_freeze_system_prompt / set_freeze_user_prompt`：设置是否冻结对应的提示词模板；
- `get_freeze_system_prompt / get_freeze_user_prompt`：查询当前冻结状态。

### 优化器回调接口

```python
def get_optimizer_callback(self) -> Optional[Callable]
def set_optimizer_callback(self, callback: Optional[Callable]) -> None
```

- `set_optimizer_callback`：注册一个回调函数，签名大致为：

  ```python
  async def optimizer_callback(
      llm_call_id: str,
      inputs: dict[str, Any],
      output: Any,
      session: Session,
  ) -> None:
      ...
  ```

- `get_optimizer_callback`：获取当前注册的回调函数。

当回调被设置后：

- 在 `invoke` 成功得到 `response` 后调用一次；
- 在 `stream` 完成并聚合所有片段后调用一次。  
这为后续的提示词/模型调用优化提供了完整的输入输出与会话上下文信息。

