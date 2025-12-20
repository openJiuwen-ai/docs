# openjiuwen.agent.react_agent

## create_react_agent_config

```python
create_react_agent_config(agent_id: str, agent_version: str, description: str, workflows: List[WorkflowSchema], plugins: List[PluginSchema], model: ModelConfig, prompt_template: List[Dict])
```

创建ReActAgent配置对象。

**参数**：

- **agent_id**(str)：agent的唯一标识符。
- **agent_version**(str)：agent的版本号。
- **description**(str)：agent的描述信息。
- **workflows**(List[[WorkflowSchema](common/schema.md#class-openjiuwenagentcommonschemaworkflowschema)])：agent支持的工作流描述信息列表。
- **plugins**(List[[PluginSchema](common/schema.md#class-openjiuwenagentcommonschemapluginschema)])：agent支持的插件描述信息列表。
- **model**([ModelConfig](../openjiuwen.core/common/configs/model_config.md#class-openjiuwencorecomponentcommonconfigsmodel_configmodelconfig))：agent使用的模型配置。
- **prompt_template**(List[Dict])：提示词模板列表。dict格式为`dict(role="system", content="你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！")`，role的取值可为`system`或`user`，为`system`时表示是系统提示词，为`user`时表示是用户提示词。默认值：[]。

**返回**：

**[ReActAgentConfig](config/react_config.md#class-openjiuwenagentconfigreact_configreactagentconfig)**，一个新的ReActAgentConfig配置对象。

**样例**：

```python
>>> import os 
>>> from datetime import datetime
>>> from openjiuwen.agent.common.schema import PluginSchema
>>> from openjiuwen.agent.react_agent import create_react_agent_config
>>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
>>> from openjiuwen.core.utils.llm.base import BaseModelInfo
>>> 
>>> API_BASE = os.getenv("API_BASE", "")
>>> API_KEY = os.getenv("API_KEY", "")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def build_current_date():
...     current_datetime = datetime.now()
...     return current_datetime.strftime("%Y-%m-%d")
>>> 
>>> def _create_model():
...     return ModelConfig(model_provider=MODEL_PROVIDER,
...                        model_info=BaseModelInfo(
...                            model=MODEL_NAME,
...                            api_base=API_BASE,
...                            api_key=API_KEY,
...                            temperature=0.7,
...                            top_p=0.9,
...                            timeout=30
...                        ))
>>> 
>>> def _create_tool_schema():
...     tool_info = PluginSchema(
...         name='WeatherReporter',
...         description='天气查询插件',
...         inputs={
...             "type": "object",
...             "properties": {
...                 "location": {
...                     "type": "string",
...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
...                     "required": True
...                 },
...                 "date": {
...                     "type": "string",
...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
...                     "required": True
...                 }
...             }
...         }
...     )
...     return tool_info
>>> 
>>> def _create_prompt_template():
...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
...     return [
...         dict(role="system", content=system_prompt.format(build_current_date()))
...     ]
>>> 
>>> tools_schema = [_create_tool_schema()]
>>> model_config = _create_model()
>>> prompt_template = _create_prompt_template()
>>> 
>>> react_agent_config = create_react_agent_config(
...     agent_id="react_agent_123",
...     agent_version="0.0.1",
...     description="AI助手",
...     plugins=tools_schema,
...     workflows=[],
...     model=model_config,
...     prompt_template=prompt_template
... )
```

## create_react_agent

```python
create_react_agent(agent_config: ReActAgentConfig, workflows: List[Workflow] = None, tools: List[Tool] = None)
```

创建ReActAgent对象。

**参数**：

- **agent_config**([ReActAgentConfig](config/react_config.md#class-openjiuwenagentconfigreact_configreactagentconfig))：ReActAgent的配置信息。
- **workflows**(List[[Workflow](../openjiuwen.core/workflow/workflow_config.md#openjiuwencoreworkflowworkflow_config)], 可选)：需要注册到ReActAgent中的工作流。
- **tools**(List[[Tool](../openjiuwen.core/utils/tool/tool.md#openjiuwencoreutilstooltool)], 可选)：需要注册到ReActAgent中的工具。

> **说明**
> 
> - `tools`是工具的实际可执行实例，`tools`中的每个工具实例，必须与`agent_config.plugins`中的某一个`PluginSchema`元数据匹配，通过工具名称关联。
> - `workflows`是工作流的实际可执行实例，必须与`agent_config.workflows`中的某一个`WorkflowSchema`元数据匹配，通过工作流名称关联。

**返回**：

**[ReActAgent](react_agent.md#class-openjiuwenagentreact_agentreactagent)**，一个新的ReActAgent对象。

**样例**：

```python
>>> import os
>>> import asyncio
>>> from datetime import datetime
>>> from openjiuwen.agent.common.schema import PluginSchema
>>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
>>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
>>> from openjiuwen.core.utils.llm.base import BaseModelInfo
>>> from openjiuwen.core.utils.tool.param import Param
>>> from openjiuwen.core.utils.tool.service_api.restful_api import RestfulApi
>>> 
>>> API_BASE = os.getenv("API_BASE", "")
>>> API_KEY = os.getenv("API_KEY", "")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def build_current_date():
...     current_datetime = datetime.now()
...     return current_datetime.strftime("%Y-%m-%d")
>>> 
>>> def _create_model():
...     return ModelConfig(model_provider=MODEL_PROVIDER,
...                        model_info=BaseModelInfo(
...                            model=MODEL_NAME,
...                            api_base=API_BASE,
...                            api_key=API_KEY,
...                            temperature=0.7,
...                            top_p=0.9,
...                            timeout=30
...                        ))
>>> 
>>> def _create_tool():
...     weather_plugin = RestfulApi(
...         name="WeatherReporter",
...         description="天气查询插件",
...         params=[
...             Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
...             Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
...         ],
...         path="your path to weather service",
...         headers={},
...         method="GET",
...         response=[],
...     )
...     return weather_plugin
>>> 
>>> def _create_tool_schema():
...     tool_info = PluginSchema(
...         name='WeatherReporter',
...         description='天气查询插件',
...         inputs={
...             "type": "object",
...             "properties": {
...                 "location": {
...                     "type": "string",
...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
...                     "required": True
...                 },
...                 "date": {
...                     "type": "string",
...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
...                     "required": True
...                 }
...             }
...         }
...     )
...     return tool_info
>>> 
>>> def _create_prompt_template():
...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
...     return [
...         dict(role="system", content=system_prompt.format(build_current_date()))
...     ]
>>> 
>>> tools_schema = [_create_tool_schema()]
>>> model_config = _create_model()
>>> prompt_template = _create_prompt_template()
>>> 
>>> react_agent_config = create_react_agent_config(
...     agent_id="react_agent_123",
...     agent_version="0.0.1",
...     description="AI助手",
...     plugins=tools_schema,
...     workflows=[],
...     model=model_config,
...     prompt_template=prompt_template
... )
>>> 
>>> react_agent: ReActAgent = create_react_agent(
...     agent_config=react_agent_config,
...     workflows=[],
...     tools=[_create_tool()]
... )
>>> 
>>> async def run_test():
...     result = await react_agent.invoke({"query": "查询杭州的天气"})
...     print(f"ReActAgent 最终输出结果：{result}")
ReActAgent 最终输出结果：{'output': '杭州今天的天气情况如下：\n\n- 天气：晴\n- 温度：31.53°C\n- 体感温度：33.52°C\n- 湿度：50%\n- 风速：3.99 m/s\n\n请注意防晒和补水！', 'result_type': 'answer'}
>>> 
>>> 
>>> if __name__ == "__main__":
...     asyncio.run(run_test())
```

## class openjiuwen.agent.react_agent.ReActAgent

```python
openjiuwen.agent.react_agent.ReActAgent(agent_config: ReActAgentConfig)
```

ReActAgent是一种遵循ReAct（Reasoning + Action）规划模式的Agent，通过 “思考（Thought）→ 行动（Action）→ 观察（Observe）”的循环迭代完成用户任务。首先，模型进行逻辑推理进行规划，选择并调用合适的工具（如检索、数据库、第三方 API、代码执行等）或工作流执行具体的操作，接着根据执行结果观察与反馈，不断调整策略，优化推理路径，直至达成任务目标或获得最终答案。其强大的多轮推理与自我修正能力，使ReActAgent具备动态决策能力且能够灵活应对环境变化，适用于需要复杂推理和策略调整的多样化任务场景。

**参数**：

- **agent_config**([ReActAgentConfig](config/react_config.md#class-openjiuwenagentconfigreact_configreactagentconfig))：ReActAgent配置。

**样例**：

```python
>>> import os
>>> from datetime import datetime
>>> from openjiuwen.agent.common.schema import PluginSchema
>>> from openjiuwen.agent.react_agent import create_react_agent_config, ReActAgent
>>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
>>> from openjiuwen.core.utils.llm.base import BaseModelInfo
>>> 
>>> API_BASE = os.getenv("API_BASE", "")
>>> API_KEY = os.getenv("API_KEY", "")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def build_current_date():
...     current_datetime = datetime.now()
...     return current_datetime.strftime("%Y-%m-%d")
>>> 
>>> def _create_model():
...     return ModelConfig(model_provider=MODEL_PROVIDER,
...                        model_info=BaseModelInfo(
...                            model=MODEL_NAME,
...                            api_base=API_BASE,
...                            api_key=API_KEY,
...                            temperature=0.7,
...                            top_p=0.9,
...                            timeout=30
...                        ))
>>> 
>>> def _create_tool_schema():
...     tool_info = PluginSchema(
...         name='WeatherReporter',
...         description='天气查询插件',
...         inputs={
...             "type": "object",
...             "properties": {
...                 "location": {
...                     "type": "string",
...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
...                     "required": True
...                 },
...                 "date": {
...                     "type": "string",
...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
...                     "required": True
...                 }
...             }
...         }
...     )
...     return tool_info
>>> 
>>> def _create_prompt_template():
...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
...     return [
...         dict(role="system", content=system_prompt.format(build_current_date()))
...     ]
>>> 
>>> tools_schema = [_create_tool_schema()]
>>> model_config = _create_model()
>>> prompt_template = _create_prompt_template()
>>> 
>>> react_agent_config = create_react_agent_config(
...     agent_id="react_agent_123",
...     agent_version="0.0.1",
...     description="AI助手",
...     plugins=tools_schema,
...     workflows=[],
...     model=model_config,
...     prompt_template=prompt_template
... )
>>> 
>>> react_agent = ReActAgent(agent_config=react_agent_config)
```

### async invoke

```python
async invoke(self, inputs: Dict) -> Dict
```

运行ReActAgent。

**参数**：

- **inputs**(Dict)：ReActAgent的输入数据。支持的键包括：
  - **query**(str)：用户查询内容。
  - **conversation_id**(str, 可选)：会话ID，用于区分不同会话。默认值：default_session。

**返回**：

**Dict**，ReActAgent最终输出结果。

**样例**：

- 样例一：
  
  ```python
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import PluginSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.utils.tool.param import Param
  >>> from openjiuwen.core.utils.tool.service_api.restful_api import RestfulApi
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_model():
  ...     return ModelConfig(model_provider=MODEL_PROVIDER,
  ...                        model_info=BaseModelInfo(
  ...                            model=MODEL_NAME,
  ...                            api_base=API_BASE,
  ...                            api_key=API_KEY,
  ...                            temperature=0.7,
  ...                            top_p=0.9,
  ...                            timeout=30
  ...                        ))
  >>> 
  >>> def _create_tool():
  ...     weather_plugin = RestfulApi(
  ...         name="WeatherReporter",
  ...         description="天气查询插件",
  ...         params=[
  ...             Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
  ...             Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
  ...         ],
  ...         path="your path to weather service",
  ...         headers={},
  ...         method="GET",
  ...         response=[],
  ...     )
  ...     return weather_plugin
  >>> 
  >>> def _create_tool_schema():
  ...     tool_info = PluginSchema(
  ...         name='WeatherReporter',
  ...         description='天气查询插件',
  ...         inputs={
  ...             "type": "object",
  ...             "properties": {
  ...                 "location": {
  ...                     "type": "string",
  ...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
  ...                     "required": True
  ...                 },
  ...                 "date": {
  ...                     "type": "string",
  ...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
  ...                     "required": True
  ...                 }
  ...             }
  ...         }
  ...     )
  ...     return tool_info
  >>> 
  >>> def _create_prompt_template():
  ...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
  ...     return [
  ...         dict(role="system", content=system_prompt.format(build_current_date()))
  ...     ]
  >>> 
  >>> tools_schema = [_create_tool_schema()]
  >>> model_config = _create_model()
  >>> prompt_template = _create_prompt_template()
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_123",
  ...     agent_version="0.0.1",
  ...     description="AI助手",
  ...     plugins=tools_schema,
  ...     workflows=[],
  ...     model=model_config,
  ...     prompt_template=prompt_template
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[],
  ...     tools=[_create_tool()]
  ... )
  >>> 
  >>> async def run_test():
  ...     result = await react_agent.invoke({"query": "查询杭州的天气"})
  ...     print(f"ReActAgent 最终输出结果：{result}")
  ReActAgent 最终输出结果：{'output': '杭州今天的天气情况如下：\n\n- 天气：晴\n- 温度：32.6°C\n- 体感温度：34.48°C\n- 湿度：46%\n- 风速：3.21 m/s\n\n天气晴朗，但温度较高，请注意防晒和补水！', 'result_type': 'answer'}
  >>> 
  >>> asyncio.run(run_test())
  ```
- 样例二：
  
  ```python
  >>> # 中断恢复样例：当agent执行过程中遇到需要用户交互的场景（如追问），会返回交互请求。用户可以提供交互输入后继续执行。
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from typing import List
  >>> from openjiuwen.agent.common.schema import WorkflowSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.component.end_comp import End
  >>> from openjiuwen.core.component.start_comp import Start
  >>> from openjiuwen.core.component.questioner_comp import QuestionerComponent, QuestionerConfig, FieldInfo
  >>> from openjiuwen.core.runtime.interaction.interactive_input import InteractiveInput
  >>> from openjiuwen.core.stream.writer import OutputSchema
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata, WorkflowInputsSchema
  >>> from openjiuwen.core.workflow.base import Workflow
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_prompt_template():
  ...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
  ...     return [
  ...         dict(role="system", content=system_prompt.format(build_current_date()))
  ...     ]
  >>> 
  >>> # 创建包含追问组件的工作流
  >>> questioner_workflow_config = WorkflowConfig(
  ...     metadata=WorkflowMetadata(
  ...         name="questioner",
  ...         id="questioner_workflow",
  ...         version="1.0",
  ...         description="用户输入"
  ...     ),
  ...     workflow_inputs_schema = WorkflowInputsSchema(
  ...         type="object",
  ...         properties={
  ...             "query": {
  ...                 "type": "string",
  ...                 "description": "用户输入",
  ...                 "required": True
  ...             }
  ...         },
  ...         required=['query']
  ...     )
  ... )
  >>> 
  >>> flow = Workflow(workflow_config=questioner_workflow_config)
  >>> 
  >>> key_fields = [
  ...     FieldInfo(field_name="location", description="地点", required=True),
  ...     FieldInfo(field_name="time", description="时间", required=True, default_value="today")
  ... ]
  >>> 
  >>> start_component = Start(
  ...     {
  ...         "inputs": [
  ...             {"id": "query", "type": "String", "required": "true", "sourceType": "ref"}
  ...         ]
  ...     }
  ... )
  >>> end_component = End({"responseTemplate": "{{location}} | {{time}}"})
  >>> 
  >>> model_config = ModelConfig(model_provider=MODEL_PROVIDER,
  ...                            model_info=BaseModelInfo(
  ...                                model=MODEL_NAME,
  ...                                api_base=API_BASE,
  ...                                api_key=API_KEY,
  ...                                temperature=0.7,
  ...                                top_p=0.9,
  ...                                timeout=30
  ...                            ))
  >>> questioner_config = QuestionerConfig(
  ...     model=model_config,
  ...     question_content="",
  ...     extract_fields_from_response=True,
  ...     field_names=key_fields,
  ...     with_chat_history=False
  ... )
  >>> questioner_component = QuestionerComponent(questioner_comp_config=questioner_config)
  >>> 
  >>> flow.set_start_comp("s", start_component, inputs_schema={"query": "${query}"})
  >>> flow.set_end_comp("e", end_component,
  ...                   inputs_schema={"location": "${questioner.location}", "time": "${questioner.time}"})
  >>> flow.add_workflow_comp("questioner", questioner_component, inputs_schema={"query": "${s.query}"})
  >>> 
  >>> flow.add_connection("s", "questioner")
  >>> flow.add_connection("questioner", "e")
  >>> 
  >>> workflow_schema = WorkflowSchema(
  ...     id=flow.config().metadata.id,
  ...     name=flow.config().metadata.name,
  ...     version=flow.config().metadata.version,
  ...     description="追问器工作流",
  ...     inputs={
  ...         "type": "object",
  ...         "properties": {
  ...             "query": {
  ...                 "type": "string",
  ...                 "description": "用户输入",
  ...                 "required": True
  ...             }
  ...         }
  ...     }
  ... )
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_123",
  ...     agent_version="0.0.1",
  ...     description="AI助手",
  ...     plugins=[],
  ...     workflows=[workflow_schema],
  ...     model=model_config,
  ...     prompt_template=_create_prompt_template()
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[flow],
  ...     tools=[]
  ... )
  >>> 
  >>> async def run_test():
  ...     # 第一次调用，可能会触发中断返回交互请求
  ...     result = await react_agent.invoke({"conversation_id": "12345", "query": "查询今天天气"})
  ...     print(f"ReActAgent 第一次输出结果：{result}")
  ReActAgent 第一次输出结果：[OutputSchema(type='__interaction__', index=0, payload=InteractionOutput(id='questioner', value='请您提供地点相关的信息'))]
  ... 
  ...     # 如果返回的是交互请求，则提供用户输入后继续执行
  ...     if isinstance(result, List) and isinstance(result[0], OutputSchema) and result[0].type == '__interaction__':
  ...         interactive_input = InteractiveInput()
  ...         interactive_input.update("questioner", "杭州")
  ...         result = await react_agent.invoke({"conversation_id": "12345", "query": interactive_input})
  ...         print(f"ReActAgent 第二次输出结果：{result}")
  ReActAgent 第二次输出结果：{'output': '目前我无法直接获取杭州2025年10月10日的天气信息。您可以通过天气应用或网站（如中国天气网、AccuWeather等）查询具体信息。需要我帮您 搜索相关内容吗？', 'result_type': 'answer'}
  >>> 
  >>> if __name__ == "__main__":
  ...     asyncio.run(run_test())
  ```

### async stream

```python
async stream(self, inputs: Dict) -> AsyncIterator[Any]
```

以流式方式运行ReActAgent，逐步返回中间结果。

**参数**：

- **inputs**(Dict)：ReActAgent的输入数据。支持的键包括：
  - **query**(str)：用户查询内容。
  - **conversation_id**(str, 可选)：会话ID，用于区分不同会话。默认值：`default_session`。

**返回**：

**AsyncIterator[Any]**，异步迭代器，返回数据流的迭代器，用于支持流式输出，包括`BaseMessageChunk`等类型的数据流。

**样例**：

- 样例一：
  
  ```python
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import PluginSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.utils.tool.param import Param
  >>> from openjiuwen.core.utils.tool.service_api.restful_api import RestfulApi
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_model():
  ...     return ModelConfig(model_provider=MODEL_PROVIDER,
  ...                        model_info=BaseModelInfo(
  ...                            model=MODEL_NAME,
  ...                            api_base=API_BASE,
  ...                            api_key=API_KEY,
  ...                            temperature=0.7,
  ...                            top_p=0.9,
  ...                            timeout=30
  ...                        ))
  >>> 
  >>> def _create_tool():
  ...     weather_plugin = RestfulApi(
  ...         name="WeatherReporter",
  ...         description="天气查询插件",
  ...         params=[
  ...             Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
  ...             Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
  ...         ],
  ...         path="your path to weather service",
  ...         headers={},
  ...         method="GET",
  ...         response=[],
  ...     )
  ...     return weather_plugin
  >>> 
  >>> def _create_tool_schema():
  ...     tool_info = PluginSchema(
  ...         name='WeatherReporter',
  ...         description='天气查询插件',
  ...         inputs={
  ...             "type": "object",
  ...             "properties": {
  ...                 "location": {
  ...                     "type": "string",
  ...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
  ...                     "required": True
  ...                 },
  ...                 "date": {
  ...                     "type": "string",
  ...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
  ...                     "required": True
  ...                 }
  ...             }
  ...         }
  ...     )
  ...     return tool_info
  >>> 
  >>> def _create_prompt_template():
  ...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
  ...     return [
  ...         dict(role="system", content=system_prompt.format(build_current_date()))
  ...     ]
  >>> 
  >>> tools_schema = [_create_tool_schema()]
  >>> model_config = _create_model()
  >>> prompt_template = _create_prompt_template()
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_123",
  ...     agent_version="0.0.1",
  ...     description="AI助手",
  ...     plugins=tools_schema,
  ...     workflows=[],
  ...     model=model_config,
  ...     prompt_template=prompt_template
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[],
  ...     tools=[_create_tool()]
  ... )
  >>> 
  >>> async def run_test():
  ...     async for chunk in react_agent.stream({"query": "查询杭州的天气"}):
  ...         print(f"流式输出: {chunk}")
  流式输出: type='tracer_agent' payload={'traceId': '95230980-45e2-4a1d-b58f-b81435933463', 'startTime': datetime.datetime(2025, 10, 10, 15, 3, 56, 280611), 'endTime': None, 'inputs': {'inputs': {}}, 'outputs': None, 'error': None, 'invokeId': 'f8e1225a-24e2-49eb-a56b-c3bc9c3d46cb', 'parentInvokeId': 'df49b119-a0a5-41f8-b605-dc9456cf7ff3', 'childInvokes': [], 'invokeType': 'plugin', 'name': 'RestfulApi', 'elapsedTime': None, 'metaData': {'class_name': 'RestfulApi', 'type': 'tool'}}
  流式输出: type='tracer_agent' payload={'traceId': '95230980-45e2-4a1d-b58f-b81435933463', 'startTime': datetime.datetime(2025, 10, 10, 15, 3, 56, 280611), 'endTime': datetime.datetime(2025, 10, 10, 15, 3, 56, 283458), 'inputs': {'inputs': {}}, 'outputs': {'outputs': {'errCode': 0, 'errMessage': 'success', 'data': {'city': 'Hangzhou', 'country': 'CN', 'feels_like': 35.6, 'humidity': 48, 'temperature': 32.93, 'weather': '晴', 'wind_speed': 3.21}}}, 'error': None, 'invokeId': 'f8e1225a-24e2-49eb-a56b-c3bc9c3d46cb', 'parentInvokeId': 'df49b119-a0a5-41f8-b605-dc9456cf7ff3', 'childInvokes': [], 'invokeType': 'plugin', 'name': 'RestfulApi', 'elapsedTime': '3ms', 'metaData': {'class_name': 'RestfulApi', 'type': 'tool'}}
  流式输出: type='answer' index=0 payload={'output': '杭州今天的天气情况如下：\n\n- **天气**: 晴\n- **温度**: 32.93°C\n- **体感温度**: 35.6°C\n- **湿度**: 48%\n- **风速**: 3.21 m/s\n\n天气晴朗，温度较高，请注意防晒和补水！', 'result_type': 'answer'}
  >>> 
  >>> if __name__ == "__main__":
  ...     asyncio.run(run_test())
  ```
- 样例二：
  
  ```python
  >>> # 中断恢复样例：当agent执行过程中遇到需要用户交互的场景（如追问），流式输出会返回交互请求。用户可以提供交互输入后继续执行。
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import WorkflowSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.component.end_comp import End
  >>> from openjiuwen.core.component.start_comp import Start
  >>> from openjiuwen.core.component.questioner_comp import QuestionerComponent, QuestionerConfig, FieldInfo
  >>> from openjiuwen.core.runtime.interaction.interactive_input import InteractiveInput
  >>> from openjiuwen.core.stream.writer import OutputSchema
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata, WorkflowInputsSchema
  >>> from openjiuwen.core.workflow.base import Workflow
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_prompt_template():
  ...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
  ...     return [
  ...         dict(role="system", content=system_prompt.format(build_current_date()))
  ...     ]
  >>> 
  >>> # 创建包含追问组件的工作流
  >>> questioner_workflow_config = WorkflowConfig(
  ...     metadata=WorkflowMetadata(
  ...         name="questioner",
  ...         id="questioner_workflow",
  ...         version="1.0",
  ...         description="用户输入"
  ...     ),
  ...     workflow_inputs_schema = WorkflowInputsSchema(
  ...         type="object",
  ...         properties={
  ...             "query": {
  ...                 "type": "string",
  ...                 "description": "用户输入",
  ...                 "required": True
  ...             }
  ...         },
  ...         required=['query']
  ...     )
  ... )
  >>> 
  >>> flow = Workflow(workflow_config=questioner_workflow_config)
  >>> 
  >>> key_fields = [
  ...     FieldInfo(field_name="location", description="地点", required=True),
  ...     FieldInfo(field_name="time", description="时间", required=True, default_value="today")
  ... ]
  >>> 
  >>> start_component = Start(
  ...     {
  ...         "inputs": [
  ...             {"id": "query", "type": "String", "required": "true", "sourceType": "ref"}
  ...         ]
  ...     }
  ... )
  >>> end_component = End({"responseTemplate": "{{location}} | {{time}}"})
  >>> 
  >>> model_config = ModelConfig(model_provider=MODEL_PROVIDER,
  ...                            model_info=BaseModelInfo(
  ...                                model=MODEL_NAME,
  ...                                api_base=API_BASE,
  ...                                api_key=API_KEY,
  ...                                temperature=0.7,
  ...                                top_p=0.9,
  ...                                timeout=30
  ...                            ))
  >>> questioner_config = QuestionerConfig(
  ...     model=model_config,
  ...     question_content="",
  ...     extract_fields_from_response=True,
  ...     field_names=key_fields,
  ...     with_chat_history=False
  ... )
  >>> questioner_component = QuestionerComponent(questioner_comp_config=questioner_config)
  >>> 
  >>> flow.set_start_comp("s", start_component, inputs_schema={"query": "${query}"})
  >>> flow.set_end_comp("e", end_component,
  ...                   inputs_schema={"location": "${questioner.location}", "time": "${questioner.time}"})
  >>> flow.add_workflow_comp("questioner", questioner_component, inputs_schema={"query": "${s.query}"})
  >>> 
  >>> flow.add_connection("s", "questioner")
  >>> flow.add_connection("questioner", "e")
  >>> 
  >>> workflow_schema = WorkflowSchema(
  ...     id=flow.config().metadata.id,
  ...     name=flow.config().metadata.name,
  ...     version=flow.config().metadata.version,
  ...     description="追问器工作流",
  ...     inputs={
  ...         "type": "object",
  ...         "properties": {
  ...             "query": {
  ...                 "type": "string",
  ...                 "description": "用户输入",
  ...                 "required": True
  ...             }
  ...         }
  ...     }
  ... )
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_123",
  ...     agent_version="0.0.1",
  ...     description="AI助手",
  ...     plugins=[],
  ...     workflows=[workflow_schema],
  ...     model=model_config,
  ...     prompt_template=_create_prompt_template()
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[flow],
  ...     tools=[]
  ... )
  >>> 
  >>> async def run_test():
  ...     # 第一次流式调用，收集可能的交互请求
  ...     interaction_output_schema = []
  ...     async for chunk in react_agent.stream({"conversation_id": "12345", "query": "查询今天天气"}):
  ...         print(f"ReActAgent 第一次输出结果 >>>  {chunk}")
  ReActAgent 第一次输出结果 >>> type='__interaction__' index=0 payload=InteractionOutput(id='questioner', value='请您提供地点相关的信息')
  ...         if isinstance(chunk, OutputSchema) and chunk.type == "__interaction__":
  ...             interaction_output_schema.append(chunk)
  ... 
  ...     # 如果有交互请求，则提供用户输入后继续流式执行
  ...     if interaction_output_schema:
  ...         user_input = InteractiveInput()
  ...         for item in interaction_output_schema:
  ...             component_id = item.payload.id
  ...             user_input.update(component_id, "杭州")
  ...         async for chunk in react_agent.stream({"conversation_id": "12345", "query": user_input}):
  ...             print(f"ReActAgent 第二次输出结果 >>>  {chunk}")
  ReActAgent 第二次输出结果 >>> type='answer' index=0 payload={'output': '您提到的“杭州”是想查询杭州今天的天气吗？如果是，请告诉我具体需要哪方面的天气信息，例如温度、湿 度、风力等。', 'result_type': 'answer'}
  ... 
  >>> if __name__ == "__main__":
  ...     asyncio.run(run_test())
  ```

### bind_workflows

```python
bind_workflows(self, workflows: List[Workflow])
```

将工作流绑定到ReActAgent中。

**参数**：

- **workflows**(List[[Workflow](../openjiuwen.core/workflow/workflow_config.md#openjiuwencoreworkflowworkflow_config)])：需要绑定到ReActAgent的工作流列表。

**样例**：

```python
>>> import os
>>> from datetime import datetime
>>> from openjiuwen.agent.common.schema import PluginSchema, WorkflowSchema
>>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
>>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
>>> from openjiuwen.core.component.end_comp import End
>>> from openjiuwen.core.component.start_comp import Start
>>> from openjiuwen.core.component.questioner_comp import QuestionerComponent, QuestionerConfig, FieldInfo
>>> from openjiuwen.core.utils.llm.base import BaseModelInfo
>>> from openjiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata
>>> from openjiuwen.core.workflow.base import Workflow
>>> 
>>> API_BASE = os.getenv("API_BASE", "")
>>> API_KEY = os.getenv("API_KEY", "")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def build_current_date():
...     current_datetime = datetime.now()
...     return current_datetime.strftime("%Y-%m-%d")
>>> 
>>> def _create_model():
...     return ModelConfig(model_provider=MODEL_PROVIDER,
...                        model_info=BaseModelInfo(
...                            model=MODEL_NAME,
...                            api_base=API_BASE,
...                            api_key=API_KEY,
...                            temperature=0.7,
...                            top_p=0.9,
...                            timeout=30
...                        ))
>>> 
>>> # 创建工作流
>>> questioner_workflow_config = WorkflowConfig(
...     metadata=WorkflowMetadata(
...         name="questioner",
...         id="questioner_workflow",
...         version="1.0",
...     )
... )
>>> 
>>> flow = Workflow(workflow_config=questioner_workflow_config)
>>> 
>>> key_fields = [
...     FieldInfo(field_name="location", description="地点", required=True),
...     FieldInfo(field_name="time", description="时间", required=True, default_value="today")
... ]
>>> 
>>> start_component = Start(
...     {
...         "inputs": [
...             {"id": "query", "type": "String", "required": "true", "sourceType": "ref"}
...         ]
...     }
... )
>>> end_component = End({"responseTemplate": "{{location}} | {{time}}"})
>>> 
>>> model_config = _create_model()
>>> questioner_config = QuestionerConfig(
...     model=model_config,
...     question_content="",
...     extract_fields_from_response=True,
...     field_names=key_fields,
...     with_chat_history=False
... )
>>> questioner_component = QuestionerComponent(questioner_comp_config=questioner_config)
>>> 
>>> flow.set_start_comp("s", start_component, inputs_schema={"query": "${query}"})
>>> flow.set_end_comp("e", end_component,
...                   inputs_schema={"location": "${questioner.location}", "time": "${questioner.time}"})
>>> flow.add_workflow_comp("questioner", questioner_component, inputs_schema={"query": "${s.query}"})
>>> 
>>> flow.add_connection("s", "questioner")
>>> flow.add_connection("questioner", "e")
>>> 
>>> workflow_schema = WorkflowSchema(
...     id=flow.config().metadata.id,
...     name=flow.config().metadata.name,
...     version=flow.config().metadata.version,
...     description="追问器工作流",
...     inputs={
...         "type": "object",
...         "properties": {
...             "query": {
...                 "type": "string",
...                 "description": "用户输入",
...                 "required": True
...             }
...         }
...     }
... )
>>> 
>>> react_agent_config = create_react_agent_config(
...     agent_id="react_agent_123",
...     agent_version="0.0.1",
...     description="AI助手",
...     plugins=[],
...     workflows=[workflow_schema],
...     model=model_config,
...     prompt_template=[]
... )
>>> 
>>> react_agent: ReActAgent = create_react_agent(
...     agent_config=react_agent_config,
...     workflows=[],
...     tools=[]
... )
>>> 
>>> # 绑定工作流
>>> react_agent.bind_workflows([flow])
```

### bind_tools

```python
bind_tools(self, tools: List[Tool])
```

将工具绑定到ReActAgent中。

**参数**：

- **tools**(List[[Tool](../openjiuwen.core/utils/tool/tool.md#openjiuwencoreutilstooltool)])：需要绑定到ReActAgent的工具列表。

**样例**：

- 样例一：
  
  ```python
  >>> import os
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import PluginSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.utils.tool.param import Param
  >>> from openjiuwen.core.utils.tool.service_api.restful_api import RestfulApi
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_model():
  ...     return ModelConfig(model_provider=MODEL_PROVIDER,
  ...                        model_info=BaseModelInfo(
  ...                            model=MODEL_NAME,
  ...                            api_base=API_BASE,
  ...                            api_key=API_KEY,
  ...                            temperature=0.7,
  ...                            top_p=0.9,
  ...                            timeout=30
  ...                        ))
  >>> 
  >>> def _create_tool():
  ...     weather_plugin = RestfulApi(
  ...         name="WeatherReporter",
  ...         description="天气查询插件",
  ...         params=[
  ...             Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
  ...             Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
  ...         ],
  ...         path="your path to weather service",
  ...         headers={},
  ...         method="GET",
  ...         response=[],
  ...     )
  ...     return weather_plugin
  >>> 
  >>> def _create_tool_schema():
  ...     tool_info = PluginSchema(
  ...         name='WeatherReporter',
  ...         description='天气查询插件',
  ...         inputs={
  ...             "type": "object",
  ...             "properties": {
  ...                 "location": {
  ...                     "type": "string",
  ...                     "description": "天气查询的地点。\n注意：地点名称必须为英文",
  ...                     "required": True
  ...                 },
  ...                 "date": {
  ...                     "type": "string",
  ...                     "description": "天气查询的时间，格式为YYYY-MM-DD",
  ...                     "required": True
  ...                 }
  ...             }
  ...         }
  ...     )
  ...     return tool_info
  >>> 
  >>> def _create_prompt_template():
  ...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
  ...     return [
  ...         dict(role="system", content=system_prompt.format(build_current_date()))
  ...     ]
  >>> 
  >>> tools_schema = [_create_tool_schema()]
  >>> model_config = _create_model()
  >>> prompt_template = _create_prompt_template()
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_123",
  ...     agent_version="0.0.1",
  ...     description="AI助手",
  ...     plugins=tools_schema,
  ...     workflows=[],
  ...     model=model_config,
  ...     prompt_template=prompt_template
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[],
  ...     tools=[]
  ... )
  >>> 
  >>> # 绑定工具
  >>> react_agent.bind_tools([_create_tool()])
  ```
- 样例二：
  
  ```python
  >>> # 使用LocalFunction本地函数工具的样例：除了RestfulApi，还可以使用LocalFunction将Python函数注册为工具。
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import PluginSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.utils.tool.param import Param
  >>> from openjiuwen.core.utils.tool.function.function import LocalFunction
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_model():
  ...     return ModelConfig(model_provider=MODEL_PROVIDER,
  ...                        model_info=BaseModelInfo(
  ...                            model=MODEL_NAME,
  ...                            api_base=API_BASE,
  ...                            api_key=API_KEY,
  ...                            temperature=0.7,
  ...                            top_p=0.9,
  ...                            timeout=30
  ...                        ))
  >>> 
  >>> def _create_function_tool():
  ...     """创建本地函数工具"""
  ...     add_tool = LocalFunction(
  ...         name="add",
  ...         description="加法",
  ...         params=[
  ...             Param(name="a", description="加数", type="number", required=True),
  ...             Param(name="b", description="被加数", type="number", required=True),
  ...         ],
  ...         func=lambda a, b: a + b
  ...     )
  ...     return add_tool
  >>> 
  >>> def _create_function_tool_schema():
  ...     tool_info = PluginSchema(
  ...         name='add',
  ...         description='加法',
  ...         inputs={
  ...             "type": "object",
  ...             "properties": {
  ...                 "a": {
  ...                     "type": "number",
  ...                     "description": "加数",
  ...                     "required": True
  ...                 },
  ...                 "b": {
  ...                     "type": "number",
  ...                     "description": "被加数",
  ...                     "required": True
  ...                 }
  ...             }
  ...         }
  ...     )
  ...     return tool_info
  >>> 
  >>> def _create_function_prompt_template():
  ...     system_prompt = "你是一个数学计算专家。"
  ...     return [
  ...         dict(role="system", content=system_prompt)
  ...     ]
  >>> 
  >>> tools_schema = [_create_function_tool_schema()]
  >>> model_config = _create_model()
  >>> prompt_template = _create_function_prompt_template()
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_1234",
  ...     agent_version="0.0.2",
  ...     description="AI计算助手",
  ...     plugins=tools_schema,
  ...     workflows=[],
  ...     model=model_config,
  ...     prompt_template=prompt_template
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[],
  ...     tools=[]
  ... )
  >>> 
  >>> # 绑定本地函数工具
  >>> react_agent.bind_tools([_create_function_tool()])
  >>> async def run_test():
  ...     result = await react_agent.invoke({"query": "计算1+2"})
  ...     print(f"ReActAgent 最终输出结果：{result}")
  ReActAgent 最终输出结果：{'output': '计算结果：1 + 2 = 12', 'result_type': 'answer'}
  >>> 
  >>> if __name__ == "__main__":
  ...     asyncio.run(run_test())
  ```
- 样例三：
  
  ```python
  >>> # 使用@tool装饰器的样例：使用@tool装饰器可以更简洁地将Python函数注册为工具。
  >>> import os
  >>> import asyncio
  >>> from datetime import datetime
  >>> from openjiuwen.agent.common.schema import PluginSchema
  >>> from openjiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
  >>> from openjiuwen.core.component.common.configs.model_config import ModelConfig
  >>> from openjiuwen.core.utils.llm.base import BaseModelInfo
  >>> from openjiuwen.core.utils.tool.param import Param
  >>> from openjiuwen.core.utils.tool.tool import tool
  >>> 
  >>> API_BASE = os.getenv("API_BASE", "")
  >>> API_KEY = os.getenv("API_KEY", "")
  >>> MODEL_NAME = os.getenv("MODEL_NAME", "")
  >>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
  >>> 
  >>> def build_current_date():
  ...     current_datetime = datetime.now()
  ...     return current_datetime.strftime("%Y-%m-%d")
  >>> 
  >>> def _create_model():
  ...     return ModelConfig(model_provider=MODEL_PROVIDER,
  ...                        model_info=BaseModelInfo(
  ...                            model=MODEL_NAME,
  ...                            api_base=API_BASE,
  ...                            api_key=API_KEY,
  ...                            temperature=0.7,
  ...                            top_p=0.9,
  ...                            timeout=30
  ...                        ))
  >>> 
  >>> # 使用@tool装饰器定义工具函数
  >>> @tool(
  ...     name="add",
  ...     descripton="加法",
  ...     params=[
  ...         Param(name="a", description="加数", type="number", required=True),
  ...         Param(name="b", description="被加数", type="number", required=True),
  ...     ]
  ... )
  >>> def add_function(a, b):
  ...     """加法函数，使用tool注解装饰"""
  ...     return a + b
  >>> 
  >>> def _create_function_tool_schema():
  ...     tool_info = PluginSchema(
  ...         name='add',
  ...         description='加法',
  ...         inputs={
  ...             "type": "object",
  ...             "properties": {
  ...                 "a": {
  ...                     "type": "number",
  ...                     "description": "加数",
  ...                     "required": True
  ...                 },
  ...                 "b": {
  ...                     "type": "number",
  ...                     "description": "被加数",
  ...                     "required": True
  ...                 }
  ...             }
  ...         }
  ...     )
  ...     return tool_info
  >>> 
  >>> def _create_function_prompt_template():
  ...     system_prompt = "你是一个数学计算专家。"
  ...     return [
  ...         dict(role="system", content=system_prompt)
  ...     ]
  >>> 
  >>> tools_schema = [_create_function_tool_schema()]
  >>> model_config = _create_model()
  >>> prompt_template = _create_function_prompt_template()
  >>> 
  >>> react_agent_config = create_react_agent_config(
  ...     agent_id="react_agent_1235",
  ...     agent_version="0.0.3",
  ...     description="AI计算助手（使用注解）",
  ...     workflows=[],
  ...     model=model_config,
  ...     prompt_template=prompt_template
  ... )
  >>> 
  >>> react_agent: ReActAgent = create_react_agent(
  ...     agent_config=react_agent_config,
  ...     workflows=[],
  ...     tools=[]
  ... )
  >>> 
  >>> # 绑定使用@tool装饰器创建的工具
  >>> react_agent.bind_tools([add_function])
  >>> 
  >>> async def run_test():
  ...     result = await react_agent.invoke({"query": "计算1+2"})
  ...     print(f"ReActAgent 使用注解工具最终输出结果：{result}")
  ReActAgent 使用注解工具最终输出结果：{'output': '1 + 2 的计算结果是 3。', 'result_type': 'answer'}
  >>> 
  >>> if __name__ == "__main__":
  ...     asyncio.run(run_test())
  ```
