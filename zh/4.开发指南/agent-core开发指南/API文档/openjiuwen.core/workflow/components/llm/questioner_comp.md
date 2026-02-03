# openjiuwen.core.workflow.components.llm.questioner_comp

`openjiuwen.core.workflow.components.llm.questioner_comp` 模块提供工作流中的提问器组件，用于根据配置的字段信息通过大模型从用户输入或对话历史中抽取参数，并在缺失必填项时通过人机交互（[session.interact](../../../session/interaction.md)）向用户追问，直至收齐或达到最大追问次数。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import QuestionerComponent, QuestionerConfig, FieldInfo` 导入。更多组件说明见 [components](../../components.README.md)。

## class FieldInfo

```python
class openjiuwen.core.workflow.components.llm.questioner_comp.FieldInfo(BaseModel)
```

单个待抽取/追问字段的描述，用于 [QuestionerConfig](questioner_comp.md#class-questionerconfig) 的 `field_names`。

### 字段

- **field_name**（str）：字段名，与 LLM 输出及状态中的 key 对应。
- **description**（str）：字段说明，用于构造提示词。
- **type**（Literal["string", "integer", "number", "boolean"]）：字段类型，默认 `"string"`，用于抽取结果的校验与转换。
- **cn_field_name**（str）：中文展示名，用于追问提示，默认 `""`。
- **required**（bool）：是否必填，默认 `False`；必填且未抽取到时会产生追问。
- **default_value**（Any）：默认值，默认 `""`；当未抽取到且配置了默认值时写入输出。

---

## class QuestionerConfig

```python
@dataclass
class openjiuwen.core.workflow.components.llm.questioner_comp.QuestionerConfig(ComponentConfig)
```

提问器组件的配置数据类。

### 字段

- **model_id**（str | None）：已注册模型 id，与 `model_client_config` + `model_config` 二选一。
- **model_client_config**（ModelClientConfig | None）：模型客户端配置。
- **model_config**（ModelRequestConfig | None）：模型请求配置。
- **response_type**（str）：响应类型，当前支持 `ResponseType.ReplyDirectly`（`"reply_directly"`），默认 `"reply_directly"`。
- **question_content**（str）：固定提问内容模板（可选），若配置则优先使用，支持 `{{key}}` 占位符；与 `field_names` 抽取二选一或组合使用。
- **extract_fields_from_response**（bool）：是否从用户/模型回复中抽取 `field_names` 中的字段，默认 `True`。
- **field_names**（list[FieldInfo]）：待抽取字段列表，见 [FieldInfo](questioner_comp.md#class-fieldinfo)；抽取模式下至少配置一个且 `field_name` 非空。
- **max_response**（int）：最大追问轮数，须大于 0；超过后若仍有必填未填会抛错。
- **with_chat_history**（bool）：是否使用上下文中对话历史参与抽取，默认 `False`。
- **chat_history_max_rounds**（int）：参与抽取的对话历史最大轮数，默认 5。
- **extra_prompt_for_fields_extraction**（str）：抽取时的额外约束说明，默认 `""`。
- **example_content**（str）：示例内容，用于提示词，默认 `""`。

---

## class QuestionerComponent

```python
class openjiuwen.core.workflow.components.llm.questioner_comp.QuestionerComponent(ComponentComposable)
```

提问器组件，实现 [ComponentComposable](../base.md)。根据 [QuestionerConfig](questioner_comp.md#class-questionerconfig) 构造 QuestionerExecutable；可执行对象在 `invoke` 时加载/恢复会话中的提问器状态，按状态与配置决定是直接提问、从历史抽取还是结束并输出，必要时调用 [session.interact](../../../session/interaction.md) 进行人机交互。

### \_\_init\_\_

```python
def __init__(self, questioner_comp_config: QuestionerConfig = None) -> None
```

**参数**：

- **questioner_comp_config**（QuestionerConfig | None）：提问器配置；为 `None` 时由可执行对象侧校验（会要求必填项如 `field_names`、`max_response` 等）。

### to_executable

```python
def to_executable(self) -> Executable
```

返回 QuestionerExecutable 实例，内部会校验 `response_type`、`extract_fields_from_response` 与 `field_names`、`max_response`。

**返回**：

- **Executable**：即 `QuestionerExecutable(self._questioner_config).state(QuestionerState())`。

---

## QuestionerExecutable 与状态机

可执行实现内部使用状态机（**QuestionerState**、**QuestionerStartState**、**QuestionerInteractState**、**QuestionerEndState**）与事件（**QuestionerEvent**：START_EVENT、USER_INTERACT_EVENT、END_EVENT）管理“开始 → 追问/抽取 → 结束”的流程；状态会序列化到 [session](../../../session/session.md) 的 `questioner_state` 中以便跨轮恢复。

- **invoke**：从 session 恢复或初始化状态，根据 `response_type` 调用直接回复处理器；若当前处于“正在交互”状态，会在返回前调用 `session.interact(question)` 向用户展示问题并等待输入。
- 输出：通常包含 **user_response**、**question** 以及已抽取的字段（与 [FieldInfo](questioner_comp.md#class-fieldinfo) 的 `field_name` 对应）；结束时为最终收集到的参数。
- **异常**：配置错误（如 `max_response <= 0`、抽取模式下 `field_names` 为空、`response_type` 非法）错误码为 `COMPONENT_QUESTIONER_CONFIG_ERROR`；输入校验失败为 `COMPONENT_QUESTIONER_INPUT_PARAM_ERROR`；运行时错误（如达到 `max_response` 仍未收齐必填项）为 `COMPONENT_QUESTIONER_RUNTIME_ERROR`；其他参见 [StatusCode](../../../common/exception/status_code.md) 中的提问器相关项。
