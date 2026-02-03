# openjiuwen.core.workflow.components.llm.llm_comp

`openjiuwen.core.workflow.components.llm.llm_comp` 模块提供工作流中的大模型（LLM）组件，用于在工作流节点中调用大模型完成文本生成、结构化输出等任务。支持通过 [LLMCompConfig](llm_comp.md#class-llmcompconfig) 配置模型、提示词模板、响应格式（text / markdown / json）与输出字段；支持非流式 [invoke](llm_comp.md#invoke) 与流式 [stream](llm_comp.md#stream)。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import LLMComponent, LLMCompConfig` 导入。更多组件说明见 [components](../../components.README.md)。

## class LLMCompConfig

```python
@dataclass
class openjiuwen.core.workflow.components.llm.llm_comp.LLMCompConfig(ComponentConfig)
```

LLM 组件的配置数据类，继承自 [ComponentConfig](../components.md)。

### 字段

- **model_id**（str | None）：已注册模型 id，从 [Runner](../../../runner/runner.md) 资源管理器获取模型；与 `model_client_config` + `model_config` 二选一。
- **model_client_config**（ModelClientConfig | None）：模型客户端配置，与 `model_config` 配合使用，用于直接构造 [Model](../../../foundation/llm.README.md)。
- **model_config**（ModelRequestConfig | None）：模型请求配置（如温度、最大 token 等）。
- **template_content**（list[Any]）：提示词模板内容，为消息列表（含 `role`、`content`），须至少包含一条 `user` 消息，`system` 消息须在 `user` 之前。
- **system_prompt_template**（SystemMessage | None）：可选系统提示词模板。
- **user_prompt_template**（UserMessage | None）：可选用户提示词模板。
- **response_format**（dict[str, Any]）：响应格式，`type` 为 `"text"`、`"markdown"` 或 `"json"`；json 时可配 `jsonInstruction` 等。
- **output_config**（dict[str, Any]）：输出字段定义，与 `response_format` 配合；text/markdown 时仅允许一个字段，json 时为多字段及类型描述。
- **enable_history**（bool）：是否将上下文中对话历史拼入模型输入，默认 `False`。

---

## class LLMComponent

```python
class openjiuwen.core.workflow.components.llm.llm_comp.LLMComponent(ComponentComposable)
```

大模型组件，实现 [ComponentComposable](../components.md)。根据 [LLMCompConfig](llm_comp.md#class-llmcompconfig) 构造 [LLMExecutable](llm_comp.md#class-llmexecutable)，在 [to_executable](llm_comp.md#to_executable) 时校验模板、响应格式与输出配置，并创建可执行实例。

### \_\_init\_\_

```python
def __init__(self, component_config: Optional[LLMCompConfig] = None) -> None
```

**参数**：

- **component_config**（LLMCompConfig | None）：LLM 组件配置；为 `None` 时由可执行对象侧校验。

### to_executable

```python
def to_executable(self) -> LLMExecutable
```

返回 [LLMExecutable](llm_comp.md#class-llmexecutable) 实例。首次调用时会校验 `template_content`、`response_format`、`output_config`。

**返回**：

- **LLMExecutable**：本组件的可执行实现。

### executable

```python
@property
def executable(self) -> LLMExecutable
```

懒加载的可执行实例，等价于首次调用 [to_executable](llm_comp.md#to_executable) 的结果。

---

## class LLMExecutable

```python
class openjiuwen.core.workflow.components.llm.llm_comp.LLMExecutable(ComponentExecutable)
```

LLM 组件的可执行实现。根据配置获取或创建 [Model](../../../foundation/llm.README.md) 实例，将节点输入填充到提示词模板并可选加入对话历史，调用模型后按 `response_format` 与 `output_config` 格式化输出（支持 text、markdown、json）。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

非流式调用：准备模型输入（模板填充、历史拼接），调用 `_llm.invoke`，再按 `response_format` 与 `output_config` 格式化为输出。

**参数**：

- **inputs**（Input）：节点输入，用于填充提示词模板。
- **session**（Session）：工作流会话。
- **context**（ModelContext）：上下文，`enable_history` 为真时用于获取对话历史。

**返回**：

- **Output**：格式化后的输出，字段由 `output_config` 决定。

**异常**：

- **JiuWenBaseException**：模板/响应格式/输出配置校验失败、模型初始化失败、调用失败或输出解析失败时，错误码参见 [StatusCode](../../../common/exception/status_code.md)（如 `COMPONENT_LLM_TEMPLATE_CONFIG_ERROR`、`COMPONENT_LLM_CONFIG_ERROR`、`COMPONENT_LLM_RESPONSE_CONFIG_INVALID`、`COMPONENT_LLM_INIT_FAILED`、`COMPONENT_LLM_INVOKE_CALL_FAILED`、`COMPONENT_LLM_JSON_SCHEMA_OUTPUT_ERROR` 等）。

### stream

```python
async def stream(self, inputs: Input, session: Session, context: ModelContext) -> AsyncIterator[Output]
```

流式调用：`response_format.type` 为 `"json"` 时先完整调用再 yield 一次格式化结果；为 `"text"`/`"markdown"` 时调用 `_llm.stream` 并逐片段格式化 yield。

**异常**：

- **JiuWenBaseException**：与 [invoke](llm_comp.md#invoke) 相同的配置或调用类错误。

### config

```python
@property
def config(self) -> LLMCompConfig
```

只读属性，返回构造时传入的 [LLMCompConfig](llm_comp.md#class-llmcompconfig)。
