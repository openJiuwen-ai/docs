openJiuwen框架提供了`ReActAgent`预置智能体。开发者通过调用`create_react_agent`方法创建`ReActAgent`智能体对象。然后，再调用`ReActAgent`类的对外接口`invoke`、`stream`，分别实现`ReActAgent`智能体的批、流调用流程。同时，`ReActAgent`智能体基于框架的中断恢复功能，支持人机交互场景。具体流程包括：

- 创建与执行`ReActAgent`，执行批/流的调用流程，`ReActAgent`返回相应的批输出结果/流式输出数据帧。
- 一旦触发人机交互流程，`ReActAgent`自动记录中断时的状态；将用户反馈的信息作为输入，再次执行`ReActAgent`智能体对象时，将基于中断状态快照从中断点继续执行。

以下以`ReActAgent`智能体绑定一个向用户提问的工作流为例，帮助开发者理解如何实现批/流调用`ReActAgent`智能体，并在批流调用的同时实现人机交互场景。

## 创建关联工作流

创建一个基本工作流：由开始组件、提问器组件、结束组件先后串联构成，功能为根据用户输入、提取用户的姓名和年龄等信息。部分示例代码如下。

```python
def _create_workflow():
    # 工作流的元数据信息：名称、功能描述、输入参数定义
    inputs_schema_dict = {
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "用户输入信息",
                "required": True
            }
        }
    }
    workflow_inputs_schema = WorkflowInputsSchema(**inputs_schema_dict)
    workflow_config = WorkflowConfig(metadata=WorkflowMetadata(name="questioner_workflow", id="questioner_workflow", version="0.0.1"), workflow_inputs_schema=workflow_inputs_schema)
    workflow = Workflow(workflow_config=workflow_config)

    # 创建开始组件和结束组件对象
    start_component = Start({"inputs": [{"id": "query", "type": "String", "required": "true", "sourceType": "ref"}]})
    end_component = End({})

    # 提问器组件的配置信息：依赖的大模型对象，打开提取参数的开关，待提取参数的信息
    key_fields = [
        FieldInfo(field_name="name", description="姓名", required=True),
        FieldInfo(field_name="age", description="年龄", required=True)
    ]
    questioner_config = QuestionerConfig(
        model=model_config,
        extract_fields_from_response=True,
        field_names=key_fields
    )
    questioner_component = QuestionerComponent(questioner_comp_config=questioner_config)

    # 工作流的结构：开始组件、提问器组件、结束组件先后串联构成
    workflow.set_start_comp("s", start_component, inputs_schema={"query": "${query}"})
    workflow.set_end_comp("e", end_component,
                      inputs_schema={"name": "${questioner.name}", "age": "${questioner.age}"})
    workflow.add_workflow_comp("questioner", questioner_component, inputs_schema={"query": "${s.query}"})
    workflow.add_connection("s", "questioner")
    workflow.add_connection("questioner", "e")
    return workflow
```

描述该工作流功能、入参等元数据信息的示例代码如下：

```python
def _create_workflow_schema():
    workflow_info = WorkflowSchema(
        id="questioner_workflow",
        name="questioner_workflow",
        description="整理用户信息的工作流",
        version="0.0.1",
        inputs={
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "用户输入",
                    "required": True
                }
            }
        }
    )
    return [workflow_info]
```

## 创建ReActAgent

使用openJiuwen提供的`create_react_agent_config`方法快速创建`ReActAgentConfig`对象，涵盖`ReActAgent`相关的配置参数信息，如绑定工作流的元数据信息、系统提示词模板及大模型配置信息等，并且不绑定工具。示例代码如下：

```python
from jiuwen.agent.react_agent import create_react_agent_config

def _create_model():
    return ModelConfig(model_provider=MODEL_PROVIDER,
                        model_info=BaseModelInfo(
                            model=MODEL_NAME,
                            api_base=API_BASE,
                            api_key=API_KEY,
                            temperature=0.7,
                            top_p=0.9,
                            timeout=30
                        ))

def _create_prompt_template():
    return prompt_template = [
           dict(role="system", content="你是一个AI助手。请帮我调用工具解决问题。")
       ]

react_agent_config = create_react_agent_config(
    agent_id="react_agent_123",
    agent_version="0.0.1",
    description="AI助手",
    plugins=[],
    workflows=_create_workflow_schema(),        # 工作流描述信息
    model=_create_model_config(),               # 大模型配置信息
    prompt_template=_create_prompt_template()   # 自定义提示词
)
```

接着使用openJiuwen提供的`create_react_agent`方法快速创建`ReActAgent`对象，包括`ReActAgent`配置、关联的工作流对象，示例代码如下：

```python
from jiuwen.agent.react_agent import create_react_agent, ReActAgent

react_agent: ReActAgent = create_react_agent(
    agent_config=react_agent_config,          # ReActAgent配置信息
    workflows=[_create_workflow()],             # ReActAgent关联的工作流对象
    tools=[]                                  # ReactAgent不关联工具对象
)
```

## 运行ReActAgent

通过调用`ReActAgent`类的对外接口`invoke`和`stream`，分别实现`ReActAgent`智能体的批、流执行流程。

### 批执行

调用`ReActAgent`的`invoke`接口，即可一键式触发`ReActAgent`的批执行流程。

```python
response = await react_agent.invoke({"conversation_id": "12345", "query": "请帮我收集和整理用户信息。这名用户的姓名是张三，年龄是25岁。"})
print(f"ReActAgent 批执行的输出结果 >>> {response}")
```

当用户请求为`请帮我收集和整理用户信息。这名用户的姓名是张三，年龄是25岁`时，`ReActAgent`智能体对象将先调用`name="questioner_workflow"`的工作流，工作流能够从用户请求中一次性提取完整所有待提取参数信息，`ReActAgent`智能体根据工作流的执行结果总结生成最终答案。批执行流程返回的结果示例如下：

```python
ReActAgent 批执行的输出结果 >>> {'output': '用户信息整理完成，姓名为张三，年龄为25岁。', 'result_type': 'answer'}
```

`ReActAgent`智能体对象批输出的数据结构是一个字典，其中具体的字段信息包括：

- **result_type**(string)：`ReActAgent`智能体返回结果的类型，本例中批执行结果的类型为`"answer"`，表示`ReActAgent`智能体返回了最终结果。
- **output**(string)：`ReActAgent`智能体的最终返回结果，本例中为针对用户请求的最终答案，即`用户信息整理完成，姓名为张三，年龄为25岁`。

### 流式执行

调用`ReActAgent`的`stream`接口，即可一键式触发`ReActAgent`的流式执行流程。

```python
async for chunk in react_agent.stream({"conversation_id": "12345", "query": "请帮我收集和整理用户信息。这名用户的姓名是张三，年龄是25岁。"}):
    print(f"ReActAgent 流式输出的结果 >>> {chunk}")
```

当用户请求为`请帮我收集和整理用户信息。这名用户的姓名是张三，年龄是25岁`时，`ReActAgent`智能体对象将先调用`name="questioner_workflow"`的工作流，工作流能够从用户请求中一次性提取完整所有待提取参数信息，`ReActAgent`智能体根据工作流的执行结果总结生成最终答案。流式调用执行流程返回的数据帧示例如下：

```python
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 49, 45, 349522), 'endTime': None, 'inputs': {'query': '张三，25岁'}, 'outputs': None, 'error': None, 'invokeId': 's', 'parentInvokeId': None, 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 's', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 49, 45, 349522), 'endTime': datetime.datetime(2025, 10, 27, 13, 49, 45, 359243), 'inputs': {'query': '张三，25岁'}, 'outputs': {'query': '张三，25岁'}, 'error': None, 'invokeId': 's', 'parentInvokeId': None, 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 's', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 49, 45, 474902), 'endTime': None, 'inputs': {'query': '张三，25岁'}, 'outputs': None, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': 's', 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 49, 45, 474902), 'endTime': datetime.datetime(2025, 10, 27, 13, 50, 43, 391185), 'inputs': {'query': '张三，25岁'}, 'outputs': {'name': '张三', 'age': 25}, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': 's', 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [{'on_invoke_data': 'extra trace data'}], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 50, 43, 506907), 'endTime': None, 'inputs': {'name': '张三', 'age': 25}, 'outputs': None, 'error': None, 'invokeId': 'e', 'parentInvokeId': 'questioner', 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'e', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='tracer_workflow' payload={'traceId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'startTime': datetime.datetime(2025, 10, 27, 13, 50, 43, 506907), 'endTime': datetime.datetime(2025, 10, 27, 13, 50, 43, 511545), 'inputs': {'name': '张三', 'age': 25}, 'outputs': {'responseContent': '', 'output': {'name': '张三', 'age': 25}}, 'error': None, 'invokeId': 'e', 'parentInvokeId': 'questioner', 'executionId': '3df30997-e20f-4cc6-873c-fcb1f4375923', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'e', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 流式输出的结果 >>> type='workflow_final' index=0 payload={'responseContent': '', 'output': {'name': '张三', 'age': 25}}
ReActAgent 流式输出的结果 >>> type='answer' index=0 payload={'output': '用户信息整理完成，姓名为张三，年龄为25岁。', 'result_type': 'answer'}
```

流式调用`ReActAgent`智能体的返回结果和批调用的区别在于：流式调用返回若干数据帧，而批调用返回单个输出结果。本示例涉及的流式数据帧主要包含三种：

- type="tracer_workflow"：调测信息的数据帧。记录了工作流执行流程中涉及的各个组件的输入输出信息、执行开始/结束的时间等调测信息，供开发者做具体的业务分析。由于本例中的工作流包含开始组件、提问器组件、结束组件3个组件，每个组件开始执行和结束执行又各产生一帧调测信息，故此类数据帧共有6帧。
- type="workflow_final"：工作流执行结果的数据帧。`payload`字段包含工作流最终的执行结果。本例中，最终输出结束组件的输出变量`output`字段对应的key-value键值对。
- type="answer"：智能体最终答复的数据帧。`payload`字段包含智能体的最终答案。本例中由于工作流只输出key-value键值对，未输出完整文本，`ReActAgent`智能体需要总结工作流的执行结果，并将总结答案作为智能体的最终答复。

## 人机交互

ReActAgent智能体触发人机交互流程需满足以下条件：

1. 首先，智能体需绑定并调用一个特定的工作流。
2. 其次，该工作流中必须包含支持中断恢复的组件（如预置的提问器组件）。
3. 最后，该组件在执行过程中触发了追问操作。

本小节将以提问器组件为例进行说明。

### 批执行触发

当用户请求为：请帮我收集和整理用户信息。这名用户的姓名是张三。调用`ReActAgent`的`invoke`接口：

```python
response = await react_agent.invoke({"conversation_id": "12345", "query": "请帮我收集和整理用户信息。这名用户的姓名是张三。"})
print(f"ReActAgent 批调用的输出结果 >>> {response}")
```

`ReActAgent`智能体对象将先调用`name="questioner_workflow"`的向用户提问的工作流。在工作流执行到提问器组件时，由于用户只提供了姓名信息，而年龄作为必需信息仍然无法提取其值，因此提问器触发了向用户追问机制，工作流返回结果中包含中断信息，`ReActAgent`智能体汇总中断信息后作为输出信息透传。此次调用`ReActAgent`返回的结果如下：

```python
ReActAgent 批调用的输出结果 >>> [OutputSchema(type='__interaction__', index=0, payload=InteractionOutput(id='questioner', value='请您提供用户年龄相关的信息'))]
```

`ReActAgent`智能体批调用的输出结果为若干OutputSchema对象汇总的列表，当`type='__interaction__'`，即表示中断信息，其`payload`字段包含了中断的具体组件名称（本例中为`"questioner"`）和用户追问的问题（本例中为`请您提供用户年龄相关的信息`）。

因此，开发者根据`ReActAgent`智能体的返回结果判断是否触发了人机交互场景。也就是当智能体输出结果为若干`OutputSchema`对象汇总的列表，且其中包含`type='__interaction__'`的对象。

```python
if isinstance(response, List):
    interactive_input = InteractiveInput()
    for item in response:
        if isinstance(item, OutputSchema) and item.type == '__interaction__':
            component_id = item.payload.id
            if component_id == "questioner":
                # 针对提问器组件的用户反馈
                interactive_input.update(component_id, "用户的年龄是25岁")
    result = await react_agent.invoke({"conversation_id": "12345", "query": interactive_input})
    print(f"ReActAgent 用户反馈后的批调用输出结果 >>> {result}")
```

用户反馈年龄信息后，基于同个会话id（本例中为`conversation_id=12345`）再次批调用`ReActAgent`智能体。框架支持中断恢复，即`ReActAgent`智能体再次调用刚刚执行中断的工作流，且工作流基于上次执行记录的checkpointer断点继续执行，直接从提问器组件开始执行，而不是从开始组件从头执行。`ReActAgent`智能体最终输出的答案为：

```python
ReActAgent 用户反馈后的批调用输出结果 >>> {'output': '用户信息已整理如下：\n\n- 姓名：张三\n- 年龄：25岁', 'result_type': 'answer'}
```

`ReActAgent`智能体对象批输出的数据结构是一个字典，其中具体的字段信息包括：

- **result_type**(string)：`ReActAgent`智能体返回结果的类型，本例中批执行结果的类型为`"answer"`，表示`ReActAgent`智能体返回了最终结果。
- **output**(string)：`ReActAgent`智能体的最终返回结果，本例中为针对用户请求的最终答案——`用户信息已整理如下：\n\n- 姓名：张三\n- 年龄：25岁`。

### 流式执行触发

当用户请求为：请帮我收集和整理用户信息。这名用户的姓名是张三。调用`ReActAgent`的`stream`接口：

```python
interaction_output_schema = []
async for chunk in react_agent.stream({"conversation_id": "12345", "query": "请帮我收集和整理用户信息。这名用户的姓名是张三"}):  # 首次调用ReActAgent智能体对象
    print(f"ReActAgent 首次调用的输出结果 >>> {chunk}")
    if isinstance(chunk, OutputSchema) and chunk.type == "__interaction__":
        interaction_output_schema.append(chunk)
```

`ReActAgent`智能体对象将先调用`name="questioner_workflow"`的向用户提问的工作流。在工作流执行到提问器组件时，由于用户只提供了姓名信息，而年龄作为必需信息仍然无法提取其值，因此提问器触发了向用户追问机制，工作流返回中断数据帧，`ReActAgent`智能体将中断数据帧作为本地调用的流式输出数据帧。此次调用返回的数据帧示例如下：

```python
ReActAgent 首次调用的输出结果 >>> type='tracer_workflow' payload={'traceId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'startTime': datetime.datetime(2025, 10, 26, 14, 9, 57, 898820), 'endTime': None, 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': None, 'error': None, 'invokeId': 's', 'parentInvokeId': None, 'executionId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 's', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 首次调用的输出结果 >>> type='tracer_workflow' payload={'traceId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'startTime': datetime.datetime(2025, 10, 26, 14, 9, 57, 898820), 'endTime': datetime.datetime(2025, 10, 26, 14, 9, 57, 904140), 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': {'query': '这名用户的姓名是张三'}, 'error': None, 'invokeId': 's', 'parentInvokeId': None, 'executionId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 's', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 首次调用的输出结果 >>> type='tracer_workflow' payload={'traceId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'startTime': datetime.datetime(2025, 10, 26, 14, 9, 57, 957560), 'endTime': None, 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': None, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': 's', 'executionId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 首次调用的输出结果 >>> type='tracer_workflow' payload={'traceId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'startTime': datetime.datetime(2025, 10, 26, 14, 9, 57, 957560), 'endTime': None, 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': None, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': 's', 'executionId': '7b6dcdfa-d88a-4664-86b0-60c8ce386444', 'onInvokeData': [{'on_invoke_data': 'extra trace data'}], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'running', 'parentNodeId': ''}
ReActAgent 首次调用的输出结果 >>> type='__interaction__' index=0 payload=InteractionOutput(id='questioner', value='请您提供用户年龄相关的信息')
```

本示例涉及的流式数据帧主要包含两种：

- type="tracer_workflow"：调测信息的数据帧。记录了工作流执行流程中涉及的各个组件的输入输出信息、执行开始/结束的时间等调测信息，供开发者做具体的业务分析。本例中，由于工作流顺序执行到第二个组件（名为`questioner`的提问器组件），且每个组件开始执行和结束执行又各产生一帧调测信息，此类调测信息数据帧共有4帧。此外，值得注意的是：由于提问器组件触发追问流程，并未真正执行完，提问器第二帧（总计第四帧）调测信息中的`status`字段值为`"running"`，而不是执行完成的状态`"finish"`。
- type="__interaction__"：中断数据帧。`payload`字段中包含了中断的数据结构`InteractionOutput`，其中包含了中断组件的名称和向用户追问的具体内容。而本例中，工作流执行到名为`questioner`的提问器组件触发了中断流程，并且向用户追问年龄信息。

因此开发者就根据调用`ReActAgent`智能体对象返回结果的type字段，判断是否触发了人机交互流程，并且能够获取中断的具体信息，从而提供相应的用户反馈。示例代码如下：

```python
if interaction_output_schema:  # 若存在中断数据帧，则触发人机交互流程
    user_input = InteractiveInput()
    for item in interaction_output_schema:
        component_id = item.payload.id
        if component_id == "questioner":
            user_input.update(component_id, "用户的年龄是25岁")  # 针对提问器组件的用户反馈信息
    async for chunk in react_agent.stream({"conversation_id": "12345", "query": user_input}):  # 再次调用ReActAgent智能体对象，输入为包含用户反馈信息的InteractiveInput对象
        print(f"ReActAgent 用户反馈后的输出结果 >>> {chunk}")
```

用户反馈年龄信息后，基于同个会话id（本例中为`conversation_id=12345`）再次流调用`ReActAgent`智能体。框架支持中断恢复，即`ReActAgent`智能体再次调用刚刚执行中断的工作流，且工作流基于上次执行记录的checkpointer断点继续执行，直接从提问器组件开始执行，而不是从开始组件从头执行。最终输出结果为：

```python
ReActAgent 用户反馈后的输出结果 >>> type='tracer_workflow' payload={'traceId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'startTime': datetime.datetime(2025, 10, 26, 14, 10, 41, 296359), 'endTime': None, 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': None, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': None, 'executionId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 用户反馈后的输出结果 >>> type='tracer_workflow' payload={'traceId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'startTime': datetime.datetime(2025, 10, 26, 14, 10, 41, 296359), 'endTime': None, 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': None, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': None, 'executionId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'onInvokeData': [{'on_invoke_data': 'extra trace data'}], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'running', 'parentNodeId': ''}
ReActAgent 用户反馈后的输出结果 >>> type='tracer_workflow' payload={'traceId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'startTime': datetime.datetime(2025, 10, 26, 14, 10, 41, 296359), 'endTime': datetime.datetime(2025, 10, 26, 14, 10, 58, 283978), 'inputs': {'query': '这名用户的姓名是张三'}, 'outputs': {'age': 25, 'name': '张三'}, 'error': None, 'invokeId': 'questioner', 'parentInvokeId': None, 'executionId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'onInvokeData': [{'on_invoke_data': 'extra trace data'}], 'componentId': '', 'componentName': '', 'componentType': 'questioner', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 用户反馈后的输出结果 >>> type='tracer_workflow' payload={'traceId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'startTime': datetime.datetime(2025, 10, 26, 14, 10, 58, 394324), 'endTime': None, 'inputs': {'name': '张三', 'age': 25}, 'outputs': None, 'error': None, 'invokeId': 'e', 'parentInvokeId': 'questioner', 'executionId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'e', 'loopNodeId': None, 'loopIndex': None, 'status': 'start', 'parentNodeId': ''}
ReActAgent 用户反馈后的输出结果 >>> type='tracer_workflow' payload={'traceId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'startTime': datetime.datetime(2025, 10, 26, 14, 10, 58, 394324), 'endTime': datetime.datetime(2025, 10, 26, 14, 10, 58, 397371), 'inputs': {'name': '张三', 'age': 25}, 'outputs': {'responseContent': '', 'output': {'name': '张三', 'age': 25}}, 'error': None, 'invokeId': 'e', 'parentInvokeId': 'questioner', 'executionId': '9fcd107e-2f48-4080-9dce-cdb9cf62f191', 'onInvokeData': [], 'componentId': '', 'componentName': '', 'componentType': 'e', 'loopNodeId': None, 'loopIndex': None, 'status': 'finish', 'parentNodeId': ''}
ReActAgent 用户反馈后的输出结果 >>> type='workflow_final' index=0 payload={'responseContent': '', 'output': {'name': '张三', 'age': 25}}
ReActAgent 用户反馈后的输出结果 >>> type='answer' index=0 payload={'output': '用户信息已整理，当前信息如下：\n\n- 姓名：张三\n- 年龄：25岁', 'result_type': 'answer'}
```

本示例涉及的流式数据帧主要包含三种：

- type="tracer_workflow"：调测信息的数据帧。记录了工作流执行流程中涉及的各个组件的输入输出信息、执行开始/结束的时间等调测信息。由于本例中的工作流从上次执行的断点继续执行，且每个组件开始执行和结束执行又各产生一帧调测信息，故此类数据帧共有4帧。
- type="workflow_final"：工作流执行结果的数据帧。`payload`字段包含工作流最终的执行结果。本例中，由于结束组件未配置输出变量，最终输出结束组件的输出变量`output`字段对应的key-value键值对。
- type="answer"：智能体最终答复的数据帧。`payload`字段包含智能体的最终答案。本例中由于工作流只输出key-value键值对，未输出完整文本，`ReActAgent`智能体需要总结工作流的执行结果，并将总结答案作为智能体的最终答复。
