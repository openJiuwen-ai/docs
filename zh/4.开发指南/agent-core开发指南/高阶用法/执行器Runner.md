Runner是openJiuwen执行所有核心组件（包括Workflow，Agent，AgentGroup）的统一入口和控制中心。它将复杂的执行逻辑抽象化，为开发者提供了一个简洁、一致且强大的编程接口。
Runner的主要功能包括：

- 提供Agent标准的异步调用（invoke）和异步流式调用（stream）两种执行入口。
- 提供Workflow标准的异步调用（invoke）和异步流式调用（stream）两种执行入口。
- 提供AgentGroup标准的异步调用（invoke）和异步流式调用（stream）两种执行入口。

## Agent执行

Runner支持所有Agent的单次输出执行和流式输出执行，包括ReActAgent、WorkflowAgent等内置Agent的执行，也包括用户自定义的Agent的执行。

下面以一个`WorkflowAgent`为例，介绍通过`Runner`执行`Agent`的过程。

首先，创建一个WorkflowAgent实例：

```python
from openjiuwen.core.common.constants.enums import ControllerType
from openjiuwen.core.single_agent.legacy import WorkflowSchema
from openjiuwen.core.application.workflow_agent import WorkflowAgentConfig, WorkflowAgent
from openjiuwen.core.workflow import End, Start, Workflow, WorkflowCard
from openjiuwen.core.workflow.workflow_config import WorkflowConfig
from openjiuwen.core.runner.runner import resource_mgr


def create_agent():
    # 创建工作流 flow，将 flow 注册到资源管理器
    flow = Workflow(workflow_config=WorkflowConfig(
        card=WorkflowCard(id="workflow_id", name="简单工作流", version="1",
                          description="this_is_a_demo")))
    flow.set_start_comp("start", Start(), inputs_schema={"query": "${query}"})
    flow.set_end_comp("end", End(), inputs_schema={"result": "${start.query}"})
    flow.add_connection("start", "end")

    resource_mgr.workflow().add_workflow("workflow_id_1", flow)

    # 创建Agent
    workflow_agent_config = WorkflowAgentConfig(id="agent_id", version="1", description="this_is_a_demo",
                                                workflows=[WorkflowSchema(
                                                    id="workflow_id",
                                                    version="1",
                                                    name="简单工作流",
                                                    description="this_is_a_demo",
                                                    inputs={"query": {"type": "string"}},
                                                )],
                                                controller_type=ControllerType.WorkflowController
                                                )
    agent = WorkflowAgent(agent_config=workflow_agent_config)
    return agent


agent = create_agent()
```

然后，创建一个 `Runner` 实例并调用 `run_agent` 接口直接运行 WorkflowAgent：

```python
import asyncio
from openjiuwen.core.runner.runner import Runner

runner = Runner()
print(asyncio.run(runner.run_agent(agent=agent, inputs={"conversion_id": "id1", "query": "哈哈"})))
```

执行结果：

```python
{'output': WorkflowOutput(result={'output': {'result': '哈哈'}},
                          state= < WorkflowExecutionState.COMPLETED: 'COMPLETED' >), 'result_type': 'answer'}
```

## Workflow执行

Runner支持Workflow的单次输出执行和流式输出执行。

下面通过构建一个简单工作流为例，介绍`Runner`执行`Workflow`的过程。

首先，创建一个Workflow:

```python
from openjiuwen.core.workflow import End, Start, Workflow, WorkflowCard
from openjiuwen.core.workflow.workflow_config import WorkflowConfig


def build_workflow(name, workflow_id, version):
    flow = Workflow(workflow_config=WorkflowConfig(
        card=WorkflowCard(id=workflow_id, name=name, version=version,
                          description="this_is_a_demo")))
    flow.set_start_comp("start", Start(), inputs_schema={"query": "${query}"})
    flow.set_end_comp("end", End(), inputs_schema={"result": "${start.query}"})
    flow.add_connection("start", "end")
    return flow


workflow = build_workflow("test_workflow", "test_workflow", "1")
```

然后，调用 `Runner` 的 `run_workflow` 直接运行 `Workflow`（无需再显式构造会话，内部会自动创建 `WorkflowSession`）：

```python
import asyncio
from openjiuwen.core.runner.runner import Runner

runner = Runner()
result = asyncio.run(runner.run_workflow(workflow=workflow, inputs={"query": "query workflow"}))
print(result)
```

执行结果：

```python
result = {'output': {'result': 'query workflow'}}
state = < WorkflowExecutionState.COMPLETED: 'COMPLETED' >
```


## AgentGroup执行

AgentGroup是用于管理多个智能体协作的核心组件。Runner支持AgentGroup的单次输出执行和流式输出执行。

下面以一个用于银行服务的`HierarchicalGroup`为例，介绍如何通过`Runner`执行`AgentGroup`。

首先，分别创建用于转账服务、余额查询和理财服务的三个`WorkflowAgent`，然后创建`HierarchicalGroup`，并将三个`WorkflowAgent`绑定到
`HierarchicalGroup`中。

```python
import os

from openjiuwen.core.application.workflow_agent import WorkflowAgentConfig, WorkflowAgent
from openjiuwen.core.workflow import (
    End, Start, Workflow, WorkflowCard,
    QuestionerComponent, QuestionerConfig, FieldInfo,
)
from openjiuwen.core.workflow.workflow_config import WorkflowConfig
from openjiuwen.core.foundation.llm import ModelConfig, BaseModelInfo
from openjiuwen.core.multi_agent import BaseGroup, GroupCard

API_BASE = os.getenv("API_BASE", "your api base")
API_KEY = os.getenv("API_KEY", "your api key")
MODEL_NAME = os.getenv("MODEL_NAME", "")
MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")


def _create_model_config() -> ModelConfig:
    """创建模型配置"""
    return ModelConfig(
        model_provider=MODEL_PROVIDER,
        model_info=BaseModelInfo(
            model=MODEL_NAME,
            api_base=API_BASE,
            api_key=API_KEY,
            temperature=0.7,
            top_p=0.9,
            timeout=120,
        ),
    )


def _create_start_component():
    """创建 Start 组件"""
    return Start({
        "inputs": [
            {
                "id": "query",
                "type": "String",
                "required": "true",
                "sourceType": "ref"
            }
        ]
    })


def _build_financial_workflow(
        workflow_id: str,
        workflow_name: str,
        workflow_desc: str,
        field_name: str,
        field_desc: str
) -> Workflow:
    """
    构建金融业务工作流（带中断节点）

    Args:
        workflow_id: 工作流ID
        workflow_name: 工作流名称
        workflow_desc: 工作流描述
        field_name: 提问字段名
        field_desc: 提问字段描述

    Returns:
        Workflow: 包含 start -> questioner -> end 的工作流
    """
    workflow_config = WorkflowConfig(
        card=WorkflowCard(
            id=workflow_id,
            name=workflow_name,
            version="1.0",
            description=workflow_desc,
        )
    )
    flow = Workflow(workflow_config=workflow_config)

    # 创建组件
    start = _create_start_component()

    # 创建提问器（中断节点）
    key_fields = [
        FieldInfo(
            field_name=field_name,
            description=field_desc,
            required=True
        ),
    ]
    model_config = _create_model_config()
    questioner_config = QuestionerConfig(
        model=model_config,
        question_content="",
        extract_fields_from_response=True,
        field_names=key_fields,
        with_chat_history=False,
    )
    questioner = QuestionerComponent(questioner_config)

    # End 组件
    end = End({"responseTemplate": f"{workflow_name}完成: {{{{{field_name}}}}}"})

    # 注册组件
    flow.set_start_comp("start", start, inputs_schema={"query": "${query}"})
    flow.add_workflow_comp(
        "questioner", questioner, inputs_schema={"query": "${start.query}"}
    )
    flow.set_end_comp(
        "end", end, inputs_schema={field_name: f"${{questioner.{field_name}}}"}
    )

    # 连接拓扑: start -> questioner -> end
    flow.add_connection("start", "questioner")
    flow.add_connection("questioner", "end")

    return flow


def _create_workflow_agent(
        agent_id: str,
        description: str,
        workflow: Workflow
) -> WorkflowAgent:
    """创建 WorkflowAgent"""
    config = WorkflowAgentConfig(
        id=agent_id,
        version="1.0",
        description=description,
        workflows=[],
        model=_create_model_config(),
    )
    agent = WorkflowAgent(config)
    agent.add_workflows([workflow])
    return agent


# 创建金融业务工作流
transfer_workflow = _build_financial_workflow(
    workflow_id="transfer_flow",
    workflow_name="转账服务",
    workflow_desc="处理用户转账请求，支持转账到指定账户",
    field_name="amount",
    field_desc="转账金额（数字）"
)

balance_workflow = _build_financial_workflow(
    workflow_id="balance_flow",
    workflow_name="余额查询",
    workflow_desc="查询用户账户余额信息",
    field_name="account",
    field_desc="账户号码"
)

invest_workflow = _build_financial_workflow(
    workflow_id="invest_flow",
    workflow_name="理财服务",
    workflow_desc="提供理财产品推荐和购买服务",
    field_name="product",
    field_desc="理财产品名称"
)

# 创建 WorkflowAgent
transfer_agent = _create_workflow_agent(
    agent_id="transfer_agent",
    description="转账服务，处理用户的转账请求",
    workflow=transfer_workflow
)

balance_agent = _create_workflow_agent(
    agent_id="balance_agent",
    description="余额查询服务，查询用户账户余额",
    workflow=balance_workflow
)

invest_agent = _create_workflow_agent(
    agent_id="invest_agent",
    description="理财服务，提供理财产品推荐和购买",
    workflow=invest_workflow
)

# 创建一个简单的AgentGroup实现

# 创建 GroupCard
group_card = GroupCard(
    name="financial_group",
    description="金融服务智能体组",
    version="1.0"
)

# 创建 BaseGroup 实例
hierarchical_group = BaseGroup(card=group_card)

# 添加所有 agent 到 group
hierarchical_group.add_agent(transfer_agent, agent_id="transfer_agent")
hierarchical_group.add_agent(balance_agent, agent_id="balance_agent")
hierarchical_group.add_agent(invest_agent, agent_id="invest_agent")
```

然后，启动 Runner，并执行 AgentGroup：

```python
import asyncio
from openjiuwen.core.runner.runner import Runner

runner = Runner()
asyncio.run(runner.start())

# 准备输入数据
inputs = {
    "conversation_id": "session_001",
    "content": "给张三转账"
}

# 执行 AgentGroup
result1 = asyncio.run(runner.run_agent_group(hierarchical_group, inputs))
print(result1)

asyncio.run(runner.stop())
```

最终获得输出：

```python
[OutputSchema(type='__interaction__', index=0,
              payload=InteractionOutput(id='questioner', value='请您提供转账金额（数字）相关的信息'))]
```
