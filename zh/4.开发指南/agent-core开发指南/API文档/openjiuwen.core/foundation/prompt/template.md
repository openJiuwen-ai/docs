# openjiuwen.core.foundation.prompt.template

`openjiuwen.core.foundation.prompt` 提供可插值提示词模板，支持占位符替换与转换为消息列表：

- 支持模板内容为字符串或 `BaseMessage` 列表；
- 占位符由可配置的前后缀标识（默认 `{{`、`}}`），通过 `format(keywords)` 替换；
- 通过 `to_messages()` 将模板内容转换为 `BaseMessage` 列表，供 LLM 或上下文引擎使用。

对应源码：`openjiuwen.core.foundation.prompt.template`。

## class PromptTemplate

```python
class openjiuwen.core.foundation.prompt.template.PromptTemplate(BaseModel)
```

可插值提示词模板：内容可为字符串或 `BaseMessage` 列表，占位符由可配置的前后缀标识，通过 `format(keywords)` 替换后得到新模板实例，通过 `to_messages()` 转为消息列表。

对应源码：`openjiuwen.core.foundation.prompt.template.PromptTemplate`。

### 字段

- `name: str`：模板名称，默认 `""`。
- `content: str | List[BaseMessage]`：模板内容，可为一段文本或消息列表；默认 `""`。
- `placeholder_prefix: str`：占位符左边界，默认 `"{{"`。
- `placeholder_suffix: str`：占位符右边界，默认 `"}}"`。

占位符格式为 `{{ 变量名 }}`，在 `format(keywords)` 时用 `keywords` 中同名键的值替换；解析与替换由 `PromptAssembler` 完成。

### to_messages

```python
def to_messages(self) -> List[BaseMessage]
```

将当前模板内容转换为 `BaseMessage` 列表。

**返回**：

- `List[BaseMessage]`：消息列表；若 content 为空则返回空列表。

**行为**：

1. 若 `content` 为空，返回 `[]`；
2. 若 `content` 为 `str`，包装为单个 `UserMessage(content=self.content)` 并返回单元素列表；
3. 若 `content` 为 `List[BaseMessage]`，校验每项为 `BaseMessage` 类型；若存在非 `BaseMessage` 项，抛出 `build_error(StatusCode.PROMPT_TEMPLATE_INVALID, error_msg="prompt template type must be in str or list[BaseMessage]")`；
4. 对 `content` 中每条消息深拷贝后返回 `[copy.deepcopy(msg) for msg in self.content]`。

### format

```python
def format(self, keywords: dict = None) -> PromptTemplate
```

用 `keywords` 替换模板内容中的占位符，返回新的 `PromptTemplate` 实例；不修改当前实例。

**参数**：

- `keywords: dict | None`：变量名到替换值的映射。默认值：`None`；若为 `None` 或空字典，则返回当前模板的深拷贝（未做替换）。

**返回**：

- `PromptTemplate`：新实例，`name`、`placeholder_prefix`、`placeholder_suffix` 与当前实例相同，`content` 为替换后的内容。

**行为**：

1. 若 `keywords` 为 `None` 或空，返回 `copy.deepcopy(self)`；
2. 创建 `PromptAssembler(prompt_template_content=copy.deepcopy(self.content), placeholder_prefix=..., placeholder_suffix=...)`；
3. 从 assembler 获取 `input_keys`，从 `keywords` 中仅保留 `input_keys` 中存在的键，得到 `valid_keywords`；
4. 调用 `assembler.prompt_assemble(**valid_keywords)` 得到替换后的 `content`；
5. 构造并返回 `PromptTemplate(name=self.name, content=content, placeholder_prefix=..., placeholder_suffix=...)`。

> **说明**：占位符解析与替换由 `openjiuwen.core.foundation.prompt.assemble.assembler.PromptAssembler` 完成；字符串内容按 `{{ key }}` 形式替换，消息列表中每条消息的 `content` 若为字符串也会参与替换。

---

## 典型使用流程示例

```python
from openjiuwen.core.foundation.prompt import PromptTemplate
from openjiuwen.core.foundation.llm import UserMessage, SystemMessage


# 1. 字符串模板 + 占位符替换
template = PromptTemplate(
    name="greeting",
    content="你好，{{ user_name }}！今天是 {{ date }}。",
)
filled = template.format(keywords={"user_name": "张三", "date": "2025-02-02"})
messages = filled.to_messages()
# messages 为 [UserMessage(content="你好，张三！今天是 2025-02-02。")]

# 2. 消息列表模板（占位符在消息 content 中）
template2 = PromptTemplate(
    name="multi_turn",
    content=[
        SystemMessage(content="你是助手。用户叫 {{ user_name }}。"),
        UserMessage(content="请介绍一下{{ topic }}。"),
    ],
)
filled2 = template2.format(keywords={"user_name": "李四", "topic": "Python"})
messages2 = filled2.to_messages()
# 得到两条消息，content 中占位符已替换

# 3. 仅转消息、不替换
template3 = PromptTemplate(name="simple", content="直接使用这段文字。")
messages3 = template3.to_messages()
# [UserMessage(content="直接使用这段文字。")]
```
