# openjiuwen.core.utils.prompt.template

## class openjiuwen.core.utils.prompt.template.template.Template

```python
class openjiuwen.core.utils.prompt.template.template.Template.Template(content, name="", filters={})
```

`Template`类为提示词模板，提供格式转换、提示词填充等功能。

**参数**：

- **content**(Union[List[Dict], List[[BaseMessage]()], str])：模板内容。如包含待填充的占位符，可使用双花括号`{{}}`来表示，例如`{{role}}`。支持以下三种类型：
  - List[Dict]：支持`BaseMessage`格式的字典列表。例如：`[{"role": "system", "content": "你是一个{{role}}助手"}]`
  - List[BaseMessage]：支持`BaseMessage`的列表，例如：`[SystemMessage(content="你是一个{{role}}助手")]`。
  - str：例如`你是一个{{role}}助手`。
- **name**(str, 可选)：模板名称，默认值：`""`。
- **filters**(Dict[str, str], 可选)：提示词模板的标签，作为TemplateManager的过滤条件。默认值：`{}`。如该提示词模板属于某个模型，filters可设置为`{"model_name": "model_A"}`。

> **说明**
>
> 注册提示词模板到`TemplateManager`时，name需非空，且name和filters需组成唯一标识，避免重复。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template import Template
>>> 
>>> # 创建字符串类型提示词模板
>>> str_template = Template(content="12345{{num}}")
>>> # 创建BaseMessage消息列表类型提示词模板
>>> messages_template = Template(content=[AIMessage(content="12345{{num}}")])
>>> # 创建字典消息列表类型提示词模板
>>> dict_template = Template(content=[{"role": "user", "content": "12345{{num}}"}])
>>> 
>>> print("string-template:", str_template.content)
输出：string-template: 12345{{num}}
>>> print("messages-template:", messages_template.content)
输出：messages-template: [AIMessage(role='assistant', content='12345{{num}}', name=None, tool_calls=None, usage_metadata=None, raw_content=None, reason_content=None)]
>>> print("dict-template:", dict_template.content)
输出：dict-template: [{'role': 'user', 'content': '12345{{num}}'}]
```

### format

```python
format(self, keywords: Dict[str, str]=None) -> Template
```

通过键值对方式填充提示词模板。

**参数**：

- **keywords**(Dict[str, str])：待填充的占位符。键为待填充的占位符名称，值为要填充的占位符具体内容。

**返回**：

**Template**，返回一个新的填充后的提示词模板对象。

**异常**：

- **JiuWenBaseException**：openJiuwen异常基类，具体详细信息和解决方法，参见[StatusCode](../../common/exception/status_code.md))。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template import Template
>>> 
>>> str_template = Template(content="12345{{num}}")
>>> str_template = str_template.format({"num": "67890"})
>>> # 创建BaseMessage消息列表类型提示词模板并填充
>>> messages_template = Template(content=[AIMessage(content="12345{{num}}")])
>>> messages_template = messages_template.format({"num": "67890"})
>>> # 创建字典消息列表类型提示词模板并填充
>>> dict_template = Template(content=[{"role": "user", "content": "12345{{num}}"}])
>>> dict_template = dict_template.format({"num": "67890"})
>>> 
>>> # 支持链式填充
>>> chain_template = Template(content="12345{{num_1}}{{num_2}}{{num_3}}{{num_4}}{{num_5}}")
>>> chain_template = chain_template.format({"num_1": "6"}) \
...                                .format({"num_2": "7"}) \
...                                .format({"num_3": "8"}) \
...                                .format({"num_4": "9"}) \
...                                .format({"num_5": "0"})
... 
>>> print("string-template:", str_template.content)
输出：string-template: 1234567890
>>> print("messages-template:", messages_template.content)
输出：messages-template: [AIMessage(role='assistant', content='1234567890', name=None, tool_calls=None, usage_metadata=None, raw_content=None, reason_content=None)]
>>> print("dict-template:", dict_template.content)
输出：dict-template: [{'role': 'user', 'content': '1234567890'}]
>>> print("chain-template:", chain_template.content)
输出：chain-template: 1234567890
```

### to_messages

```python
to_messages(self) -> List[BaseMessage]
```

转换提示词模板到`BaseMessage`消息列表类型。

**返回**：

**List[BaseMessage]**，转换后的消息列表。

> 说明：
>
> 1. 转换`string`类型模板到消息列表时，默认返回的消息类型为`HumanMessage`。
> 2. 转换`dict`类型模板到消息列表时，根据role信息实例化对应的消息类型，如`user`对应`HumanMessage`、`assistant`对应`AIMessage`、`system`对应`SystemMessage`、`tool`对应`ToolMessage`。
> 3. 转换`BaseMessage`类型模板到消息列表时，返回其原始类型。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template import Template
>>> 
>>> # 创建字符串类型提示词模板
>>> str_template = Template(content="12345{{num}}")
>>> # 创建BaseMessage消息列表类型提示词模板
>>> messages_template = Template(content=[AIMessage(content="12345{{num}}")])
>>> # 创建字典消息列表类型提示词模板
>>> dict_template = Template(content=[{"role": "user", "content": "12345{{num}}"}])
>>> 
>>> print("string-template:", str_template.to_messages())
输出：string-template: [HumanMessage(role='user', content='12345{{num}}', name=None)]
>>> print("messages-template:", messages_template.to_messages())
输出：messages-template: [AIMessage(role='assistant', content='12345{{num}}', name=None, tool_calls=None, usage_metadata=None, raw_content=None, reason_content=None)]
>>> print("dict-template:", dict_template.to_messages())
输出：dict-template: [HumanMessage(role='user', content='12345{{num}}', name=None)]
```

## class openjiuwen.core.utils.prompt.template.template_manager.TemplateManager

```python
class openjiuwen.core.utils.prompt.template.template_manager.TemplateManager()
```

`TemplateManager`类为提示词模板管理单例，提供全局的提示词模板注册、获取等功能。

### register

```python
register(self, template: Template, force: bool=False) -> bool
```

注册提示词模板到全局模板管理器，注册的模板需要在创建时指定提示词模板的名称（不可为空）以及过滤条件。

**参数**：

- **template**(Template)：提示词模板。
- **force**(bool，可选)：是否强制覆盖已经存在的提示词，True表示会覆盖已有名字和过滤条件相同的提示词模板，False表示不会覆盖。默认值：False。

**返回**：

**bool**，是否注册成功。True表示注册成功，Flase表示注册失败。

**异常**：

- **JiuWenBaseException**：openJiuwen异常基类，具体详细信息和解决方法，参见[StatusCode](../../common/exception/status_code.md))。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template import Template
>>> from openjiuwen.core.utils.prompt.template.template_manager import TemplateManager
>>> 
>>> # 创建提示词模板对象
>>> template = Template(name="system_prompt", content="你是一个聊天助手"}])
>>> print(TemplateManager().register(template))
True, 成功注册提示词模板
>>> 
>>> # 创建带过滤条件的提示词模板对象
>>> template_with_filrers = Template(name="system_prompt", content="你是一个聊天助手"}], filters={"agent_id": "agent-1"})
>>> print(TemplateManager().register(template_with_filrers))
True, 成功注册提示词模板
```

### get

```python
get(self, name: str, filters: dict=None) -> Optional[Template]
```

根据模板名称和过滤条件，获取提示词模板。

**参数**：

- **name**(str)：检索提示词模板名称。
- **filters**(dict, 可选)：过滤条件。默认值：`None` 。

**返回**：

**Optional[Template]**，查询返回的提示词模板；如果模板不存在返回`None`。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template_manager import TemplateManager
>>> 
>>> # 创建&获取提示词模板对象
>>> template = Template(name="system_prompt", content="你是一个聊天助手")
>>> TemplateManager().register(template, force=True)
>>> template = TemplateManager().get(name="system_prompt")
>>> print(template)
Template(name="system_prompt", content="你是一个聊天助手")
>>>
>>> # 创建&获取带过滤条件的提示词模板对象
>>> template = Template(name="system_prompt", content="你是一个agent-1的聊天助手", , filters={"agent_id": "agent-1"})
>>> TemplateManager().register(template, force=True)
>>> template = TemplateManager().get(name="system_prompt",  filters={"agent_id": "agent-1"})
>>> print(template)
Template(name="system_prompt", content="你是一个agent-1的聊天助手",  filters={"agent_id": "agent-1"})
```

### delete

```python
delete(self, name: str, filters: dict=None) -> bool
```

根据模板名称和过滤条件，删除提示词模板。

**参数**：

- **name**(str)：检索提示词模板名称。
- **filters**(dict, 可选)：过滤条件。默认值：`None` 。

**返回**：

**bool**，是否删除成功。True表示删除成功，Flase表示删除失败。

**异常**：

- **JiuWenBaseException**：openJiuwen异常基类，具体详细信息和解决方法，参见[StatusCode](../../common/exception/status_code.md))。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template_manager import TemplateManager
>>> 
>>> # 删除提示词模板
>>> template = Template(name="system_prompt", content="你是一个agent-1的聊天助手")
>>> TemplateManager().register(template, force=True)
>>> TemplateManager().delete(name="system_prompt")
True，删除提示词模板成功
>>> TemplateManager().delete(name="system_prompt")
False，删除提示词模板失败
```

### register_in_bulk

```python
register_in_bulk(self, dir_path:str) -> bool
```

批量注册提示词模板，注册后模板名称为文件名，内容为文件内容。

**参数**：

- **dir_path**(str)：提示词模板文件路径，注册路径下所有`.pr`后缀的提示词模板文件，

**返回**：

**bool**，是否批量注册成功。True表示注册成功；Flase表示注册失败。

**异常**：

- **JiuWenBaseException**：openJiuwen异常基类，具体详细信息和解决方法，参见[StatusCode](../../common/exception/status_code.md))。

**样例**：

```python
>>> from openjiuwen.core.utils.prompt.template.template_manager import TemplateManager
>>> 
>>> # dir_path为自定义提示词模板文件路径。假设模板存放的路径为./my_template_dir，该路径下包含如下文件：
>>> # math.pr，内容为"你是一个数学小助手，用户问题是{{question}}"
>>> # code.pr，内容为"你是一个编码专家，请基于用户问题问题生成代码：{{question}}"
>>> dir_path = "./my_template_dir"
>>> 
>>> # 批量注册提示词模板
>>> TemplateManager().register_in_bulk(dir_path=dir_path)
>>> print(TemplateManager().get("math"))
Template(name="math", content="你是一个数学小助手，用户问题是{{question}}")
>>> print(TemplateManager().get("code"))
Template(name="code", content="你是一个编码专家，请基于用户问题问题生成代码：{{question}}")
```
