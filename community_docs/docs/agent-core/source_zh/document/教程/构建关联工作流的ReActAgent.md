本章节演示了如何基于openJiuwen构建一个关联工作流的`ReActAgent`样例，该样例支持通过ReAct规划模式引导大模型生成工具调用命令，通过多次执行工具调用命令对应的插件/工作流，并结合插件/工作流的执行结果生成最终答案。通过本示例，你将会了解到如下信息：

- 如何在`ReActAgent`中关联工作流。
- 如何创建和执行关联工作流的`ReActAgent`。

# 应用设计流程

在ReAct Agent中可以关联工作流。

1. ReActController调用LLM进行任务规划，解析 LLM 输出里是否包含工具执行指令。
2. 有工具执行指令 → 调工具，选择并调用合适的工具（如工作流、检索、数据库、包装第三方API的插件、代码执行等）执行具体的操作，本样例中设计的插件和工作流功能如下：

   - 查天气插件：能根据日期和地点查询天气。
   - 根据天气状态生成文本工作流：意图识别组件首先根据输入判断是否是晴天，如果是晴天，就生成一段建议出行的文本，如果不是晴天，就生成一段疗愈心灵的文本。

   无指令 → 把LLM输出作为最终答案。
3. ReActController把工具的执行结果，作为智能体对真实世界的观察（Observation）追加到对话历史，再调用LLM进行下一次任务规划。

![ReActWorkflowAgent](../images/ReActWorkflowAgent.png)

# 前提条件

Python的版本应高于或者等于Python 3.11版本，建议使用3.11.4版本，使用前请检查Python版本信息。

# 安装openJiuwen

执行如下命令安装openJiuwen：

```bash
pip install openjiuwen-x.y.z-py3-none-any.whl
```

# 创建提示词模板

以下系统提示词定义了人设、任务目标，引导大模型。示例代码如下：

```python
def _create_prompt_template():
    system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！"
    return [dict(role="system", content=system_prompt)]
```

# 创建插件对象及其描述信息

本示例创建了天气查询插件，并定义了其输入参数的结构和要求。首先通过`RestfulApi`接口将天气查询服务封装成可以被框架使用的工具类。示例代码如下：

> **注意**
> 本地测试请求服务（http 开头），可以通过配置相关的环境变量关闭SSL校验，详情请见相关[SSL校验开关配置](../产品规格和约束.md#ssl校验开关配置)。但禁用SSL会跳过证书验证，可能遭遇数据篡改、中间人攻击，导致敏感信息泄露，仅允许测试环境临时使用，生产环境务必启用SSL校验保障安全。

```python
import os
from jiuwen.core.utils.tool.param import Param
from jiuwen.core.utils.tool.service_api.restful_api import RestfulApi

os.environ["LLM_SSL_VERIFY"] = "false"  # 关闭SSL校验仅用于本地调试，生产环境请务必打开
os.environ["RESTFUL_SSL_VERIFY"] = "false"  # 关闭SSL校验仅用于本地调试，生产环境请务必打开

def _create_tool():
    weather_plugin = RestfulApi(
        name="WeatherReporter",
        description="天气查询插件",
        params=[
            Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
            Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
        ],
        path="your path to weather service", # 天气查询服务部署地址
        headers={},
        method="GET",
        response=[],
    )
    return weather_plugin
```

同时通过`PluginSchema`接口创建了插件描述信息，该描述信息后续将成为大模型输入的一部分，引导大模型生成工具调用命令。示例代码如下：

```python
from jiuwen.agent.common.schema import PluginSchema

def _create_tool_schema():
    tool_info = PluginSchema(
        name='WeatherReporter',
        description='天气查询插件',
        inputs={
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "天气查询的地点。\n注意：地点名称必须为英文",
                    "required": True
                },
                "date": {
                    "type": "string",
                    "description": "天气查询的时间，格式为YYYY-MM-DD",
                    "required": True
                }
            }
        }
    )
    return tool_info
```

# 创建工作流

## 创建工作流对象及其描述信息

本示例中`ReActAgent`关联了一个工作流，该工作流具有如下功能：

- 当输入的天气为晴天时，生成一段建议出行的文本。
- 当输入的天气不为晴天时，生成一段疗愈心灵的文本。

首先创建工作流对象，指定`workflow_config`配置参数，示例代码如下：

```python
from jiuwen.core.workflow.base import Workflow
from jiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata, WorkflowInputsSchema

# 初始化工作流
id = "weather_generation_text_workflow"
version = "1.0"
name = "weather_generation_text"
workflow_config = WorkflowConfig(
    metadata=WorkflowMetadata(
        name=name,
        id=id,
        version=version,
        description="根据天气生成对应文本"        # 模型基于description判断是否规划执行当前workflow
    ),
    workflow_inputs_schema = WorkflowInputsSchema(
        type="object",
        properties={
            "query": {
                "type": "string",
                "description": "用户输入",
                "required": True
            }
        },
        required=['query']
    )
)

# 创建工作流
flow = Workflow(workflow_config=workflow_config)
```

然后通过`WorkflowSchema`接口创建工作流描述信息，该描述信息后续将成为大模型输入的一部分，引导大模型生成工作流调用命令。示例代码如下：

```python
def _create_workflow_schema():
    workflow_info = WorkflowSchema(
        id="weather_generation_text_workflow",
        name='weather_generation_text',
        version='1.0',
        description='根据天气生成不同风格文本的工作流',
        inputs={
            "type": "object",
            "properties": {
                "weather_condition": {
                    "type": "string",
                    "description": "工作流输入：天气查询结果",
                    "required": True
                }
            }
        }
    )

    return workflow_info
```

## 注册组件到工作流

根据任务需求创建核心组件实例，配置各组件的输入/输出规则，并将组件注册到工作流。本教程主要使用以下openJiuwen的预置组件（包括开始组件、结束组件、意图识别组件、大模型组件）。

### 开始组件

通过`Start`创建开始组件对象。开始组件作为工作流的开端，定义了工作流的输入参数规范。在本例中，开始组件的入参为固定输入参数`query`，类型为字符串，并且参数值引用自工作流的输入。示例代码如下：

```python
from jiuwen.core.component.start_comp import Start

def _create_start_component():
    """创建开始组件"""
    return Start({"inputs": [{"id": "query", "type": "String", "required": "true", "sourceType": "ref"}]})
```

> **说明**
> 此处定义了开始组件的固定输入参数`query`，该参数的值引用自Workflow Agent的输入，即用户在调用Workflow Agent执行接口invoke、stream时，输入中必须带有`query`字段。例如`react_agent.invoke({"query": "你好"})`时，则开始组件接收到的输入为`{"query": "你好"}`。

### 结束组件

通过`End`创建结束组件对象。结束组件作为工作流的终止，定义了工作流的输出结果格式。在本示例中，输出结果的格式为输出文本。示例代码如下：

```python
from jiuwen.core.component.end_comp import End

def _create_end_component():
    """创建结束组件"""
    return End({"responseTemplate": "最终结果为：{{good_weather_output}} {{bad_weather_output}}"})
```

> **说明**
> 此处结束组件配置了输出文本的模板`responseTemplate`，因此最终结果会按照输出变量`output`字段的值，拼接得到字符串，作为结束组件的`responseContent`字段输出。

### 意图识别组件

通过`IntentDetectionComponent`构造意图识别组件，用于判断用户意图。意图识别组件提供了`add_branch`方法用于定义不同意图下路由到相应的分支链路。代码如下：

```python
from jiuwen.core.component.intent_detection_comp import IntentDetectionComponent, IntentDetectionCompConfig

def _create_intent_detection_component() -> IntentDetectionComponent:
    """创建意图识别组件。"""
    user_prompt = """
         {{user_prompt}}

         当前可供选择的功能分类如下：
         {{category_info}}

         用户与助手的对话历史：
         {{chat_history}}

         当前输入：
         {{input}}

         请根据当前输入和对话历史分析并输出最适合的功能分类。输出格式为 JSON：
         {"class": "分类xx"}
         如果没有合适的分类，请输出 {{default_class}}。
     """

    config = IntentDetectionCompConfig(
        user_prompt="请判断用户意图",
        category_name_list=["天气是晴天", "天气不是晴天"],
        model=_create_model_config(),
    )

    component = IntentDetectionComponent(config)
    # 定义不同意图下路由到相应的分支链路，如果天气是晴天，则路由到`llm_1`组件所在的分支进行处理，否则路由到`llm_2`组件所在的分支执行非晴天场景的流程
    component.add_branch("${intent.classification_id} == 0", ["end"], "默认分支")
    component.add_branch("${intent.classification_id} == 1", ["llm_1"], "晴天分支")
    component.add_branch("${intent.classification_id} == 2", ["llm_2"], "非晴天分支")

    return component
```

### 大模型组件

通过`LLMComponent`构造大模型组件对象，通过设置提示词引导大模型输出预定义格式的结果。本示例用于对用户原始输入的query进行改写。示例代码如下：

```python
from jiuwen.core.component.llm_comp import LLMComponent, LLMCompConfig

def _create_llm_component(user_prompt) -> LLMComponent:
    """创建大模型组件"""
    config = LLMCompConfig(
        model=_create_model_config(),
        template_content=[{"role": "system", "content": "你是一个AI助手。"},
                          {"role": "user", "content": "user_prompt"}],
        response_format={"type": "text"},
        output_config={
            "output": {"type": "string", "description": "大模型生成的文本", "required": True}
        },
    )
    return LLMComponent(config)
```

上述部分介绍了各组件的实例化过程，下面统一介绍将组件注册到工作流：

```python
# 实例化各组件
start = _create_start_component()
intent = _create_intent_detection_component()
llm_1 = _create_llm_component("天气很好，请帮我生成一段建议出行的文本")
llm_2 = _create_llm_component("天气不好，请帮我生成一段疗愈心灵的文本")
end = _create_end_component()

# 注册组件到workflow
flow.set_start_comp("start", start, inputs_schema={"query": "${query}"})
flow.add_workflow_comp("intent", intent, inputs_schema={"query": "${start.query}"})
flow.add_workflow_comp("llm_1", llm_1, inputs_schema={"query": "${start.query}"})
flow.add_workflow_comp("llm_2", llm_2, inputs_schema={"query": "${start.query}"})
flow.set_end_comp("end", end,
                  inputs_schema={
                      "good_weather_output": "${llm_1.output}",
                      "bad_weather_output": "${llm_2.output}"
                  })
```

### 连接组件

通过`flow.add_connection`方法设置组件间的连接关系，完成工作流对象的创建：

```python
# 连接组件
flow.add_connection("start", "intent")
flow.add_connection("llm_1", "end")
flow.add_connection("llm_2", "end")
```

# 创建关联工作流的ReActAgent

首先使用`openJiuwen`提供的`create_react_agent_config`方法快速创建`ReActAgentConfig`对象，涵盖`ReActAgent`相关的配置参数信息，如天气插件的元数据信息、根据天气生成不同风格文本工作流的元数据信息、提示词定义及大模型配置信息等。示例代码如下：

```python
from jiuwen.agent.react_agent import create_react_agent_config

react_agent_config = create_react_agent_config(
    agent_id="react_agent_123",
    agent_version="0.0.1",
    description="AI助手",
    plugins=_create_tool_schema(),              # 插件描述信息
    workflows=[_create_workflow_schema()],      # 工作流描述信息
    model=_create_model_config(),               # 大模型配置信息
    prompt_template=_create_prompt_template()   # 自定义提示词
)
```

接着使用openJiuwen提供的`create_react_agent`方法快速创建`ReActAgent`对象，包括`ReActAgent`配置、关联的插件对象以及关联的工作流对象，示例代码如下：

```python
from jiuwen.agent.react_agent import create_react_agent, ReActAgent
react_agent: ReActAgent = create_react_agent(
    agent_config=react_agent_config,            # ReActAgent配置信息
    workflows=[_create_workflow()],             # ReActAgent关联的工作流对象
    tools=[_create_tool()]                      # ReactAgent关联的插件对象
)
```

# 运行ReActAgent

创建完`ReActAgent`对象后，可以调用`invoke`方法，获取用户query的回复。示例代码如下：

```python
result = await react_agent.invoke({"query": "今天上海天气晴朗，温度适宜，请生成一段文本"})
print(f"ReActAgent 最终输出结果：{result.get("output")}")
```

查询成功后，会得到如下的结果：

```text
ReActAgent 最终输出结果：【出行小贴士】  
阳光正好，微风不燥，正是出门走走的绝佳时机！  

🌿 **自然爱好者**：郊外踏青、山野徒步、湖边野餐，呼吸新鲜空气，感受秋日暖阳～  
🌆 **城市探索者**：漫步老街、打卡咖啡馆、骑行公园，用镜头记录城市温柔的一面。  
👨👩👧 **亲子时光**：带上孩子去动物园看小动物，或在草坪上放风筝，留下欢声笑语。  

✨ **温馨提示**：  
- 出行前查好实时天气，备好防晒霜和遮阳帽  
- 轻便穿搭更舒适，记得带瓶水补充能量  
- 选择人少时段出行，享受更宁静的风景  

别让好天气白白溜走，背上行囊，出发吧！
```

# 完整示例代码

```python
import asyncio
import os
from datetime import datetime

from jiuwen.agent.common.schema import PluginSchema
from jiuwen.agent.react_agent import create_react_agent_config, create_react_agent, ReActAgent
from jiuwen.core.component.common.configs.model_config import ModelConfig
from jiuwen.core.component.start_comp import Start
from jiuwen.core.component.end_comp import End
from jiuwen.core.workflow.base import Workflow
from jiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata, WorkflowInputsSchema
from jiuwen.agent.common.schema import WorkflowSchema
from jiuwen.core.component.intent_detection_comp import IntentDetectionComponent, IntentDetectionCompConfig
from jiuwen.core.component.llm_comp import LLMComponent, LLMCompConfig
from jiuwen.core.utils.llm.base import BaseModelInfo
from jiuwen.core.utils.tool.param import Param
from jiuwen.core.utils.tool.service_api.restful_api import RestfulApi

API_BASE = os.getenv("API_BASE", "")
API_KEY = os.getenv("API_KEY", "")
MODEL_NAME = os.getenv("MODEL_NAME", "")
MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
os.environ.setdefault("LLM_SSL_VERIFY", "false")  # 关闭SSL校验仅用于本地调试，生产环境请务必打开
os.environ.setdefault("RESTFUL_SSL_VERIFY", "false")  # 关闭SSL校验仅用于本地调试，生产环境请务必打开

def build_current_date():
    current_datetime = datetime.now()
    return current_datetime.strftime("%Y-%m-%d")

class ReactAgent:
    @staticmethod
    def _create_start_component():
        return Start({"inputs": [{"id": "query", "type": "String", "required": "true", "sourceType": "ref"}]})

    @staticmethod
    def _create_end_component():
        return End({"responseTemplate": "最终结果为：{{good_weather_output}} {{bad_weather_output}}"})

    @staticmethod
    def _create_model_config():
        return ModelConfig(model_provider=MODEL_PROVIDER,
                           model_info=BaseModelInfo(
                               model=MODEL_NAME,
                               api_base=API_BASE,
                               api_key=API_KEY,
                               temperature=0.7,
                               top_p=0.9,
                               timeout=30
                           ))

    @staticmethod
    def _create_intent_detection_component() -> IntentDetectionComponent:
        """创建意图识别组件。"""
        config = IntentDetectionCompConfig(
            user_prompt="请判断用户意图",
            category_name_list=["天气是晴天", "天气不是晴天"],
            model=ReactAgent._create_model_config(),
        )

        component = IntentDetectionComponent(config)
        # 定义不同意图下路由到相应的分支链路，如果天气是晴天，则路由到`llm_1`组件所在的分支进行处理，否则路由到`llm_2`组件所在的分支执行非晴天场景的流程
        component.add_branch("${intent.classification_id} == 0", ["end"], "默认分支")
        component.add_branch("${intent.classification_id} == 1", ["llm_1"], "晴天分支")
        component.add_branch("${intent.classification_id} == 2", ["llm_2"], "非晴天分支")

        return component

    def _create_llm_component(self, user_prompt) -> LLMComponent:
        config = LLMCompConfig(
            model=ReactAgent._create_model_config(),
            template_content=[{"role": "system", "content": "你是一个AI助手。"},
                              {"role": "user", "content": user_prompt}],
            response_format={"type": "text"},
            output_config={
                "output": {"type": "string", "description": "大模型生成的文本", "required": True}
            },
        )
        return LLMComponent(config)

    def _create_workflow(self):
        # 实例化各组件
        start = self._create_start_component()
        intent = self._create_intent_detection_component()
        llm_1 = self._create_llm_component("天气很好，请帮我生成一段建议出行的文本")
        llm_2 = self._create_llm_component("天气不好，请帮我生成一段疗愈心灵的文本")
        end = self._create_end_component()

        # 注册组件到工作流
        id = "weather_generation_text_workflow"
        version = "1.0"
        name = "weather_generation_text"
        workflow_config = WorkflowConfig(
            metadata=WorkflowMetadata(
                name=name,
                id=id,
                version=version,
                description="根据天气生成对应文本"
            ),
            workflow_inputs_schema=WorkflowInputsSchema(
                type="object",
                properties={
                    "query": {
                        "type": "string",
                        "description": "用户输入",
                        "required": True
                    }
                },
                required=['query']
            )
        )
        flow = Workflow(workflow_config=workflow_config)
        flow.set_start_comp("start", start, inputs_schema={"query": "${query}"})
        flow.add_workflow_comp("intent", intent, inputs_schema={"query": "${start.query}"})
        flow.add_workflow_comp("llm_1", llm_1, inputs_schema={"query": "${start.query}"})
        flow.add_workflow_comp("llm_2", llm_2, inputs_schema={"query": "${start.query}"})
        flow.set_end_comp("end", end,
                          inputs_schema={
                              "good_weather_output": "${llm_1.output}",
                              "bad_weather_output": "${llm_2.output}"
                          })

        # 连接组件
        flow.add_connection("start", "intent")
        flow.add_connection("llm_1", "end")
        flow.add_connection("llm_2", "end")

        return flow

    @staticmethod
    def _create_workflow_schema():
        workflow_info = WorkflowSchema(
            id="weather_generation_text_workflow",
            name='weather_generation_text',
            version='1.0',
            description='根据天气生成不同风格文本的工作流',
            inputs={
                "type": "object",
                "properties": {
                    "weather_condition": {
                        "type": "string",
                        "description": "工作流输入：天气查询结果",
                        "required": True
                    }
                }
            }
        )

        return workflow_info

    @staticmethod
    def _create_tool():
        weather_plugin = RestfulApi(
            name="WeatherReporter",
            description="天气查询插件",
            params=[
                Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
                Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
            ],
            path="your path to weather service",
            headers={},
            method="GET",
            response=[],
        )
        return weather_plugin

    @staticmethod
    def _create_tool_schema():
        tool_info = PluginSchema(
            name='WeatherReporter',
            description='天气查询插件',
            inputs={
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "天气查询的地点。\n注意：地点名称必须为英文",
                        "required": True
                    },
                    "date": {
                        "type": "string",
                        "description": "天气查询的时间，格式为YYYY-MM-DD",
                        "required": True
                    }
                }
            }
        )
        return tool_info

    @staticmethod
    def _create_prompt_template():
        system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
        return [
            dict(role="system", content=system_prompt.format(build_current_date()))
        ]
async def main():
    tools_schema = [ReactAgent._create_tool_schema()]
    model_config = ReactAgent._create_model_config()
    prompt_template = ReactAgent._create_prompt_template()

    react_agent_config = create_react_agent_config(
        agent_id="react_agent_123",
        agent_version="0.0.1",
        description="AI助手",
        plugins=tools_schema,
        workflows=[ReactAgent._create_workflow_schema()],
        model=model_config,
        prompt_template=prompt_template
    )
    react_agent: ReActAgent = create_react_agent(
        agent_config=react_agent_config,
        workflows=[ReactAgent._create_workflow()],
        tools=[ReactAgent._create_tool()]
    )
    result = await react_agent.invoke({"query": "今天上海天气晴朗，温度适宜，请生成一段文本"})
    print(f"ReActAgent 最终输出结果：{result.get('output')}")

if __name__ == "__main__":
    asyncio.run(main())
```

最终输出结果为：

```python
ReActAgent 最终输出结果：【出行小贴士】  
阳光正好，微风不燥，正是出门走走的绝佳时机！  

🌿 **自然爱好者**：郊外踏青、山野徒步、湖边野餐，呼吸新鲜空气，感受秋日暖阳～  
🌆 **城市探索者**：漫步老街、打卡咖啡馆、骑行公园，用镜头记录城市温柔的一面。  
👨👩👧 **亲子时光**：带上孩子去动物园看小动物，或在草坪上放风筝，留下欢声笑语。  

✨ **温馨提示**：  
- 出行前查好实时天气，备好防晒霜和遮阳帽  
- 轻便穿搭更舒适，记得带瓶水补充能量  
- 选择人少时段出行，享受更宁静的风景  

别让好天气白白溜走，背上行囊，出发吧！
```
