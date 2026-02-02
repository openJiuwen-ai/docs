# openjiuwen.core.context_engine.engine

## class openjiuwen.core.context_engine.engine.ContextEngine

```python
class ContextEngine(config: ContextEngineConfig | None = None)
```

`ContextEngine` 类用于管理模型输入上下文的信息，基于 `Session` 提供统一的上下文创建、缓存、保存与恢复能力。它为会话中的不同「上下文 ID」创建和维护对应的 `ModelContext`，并可选地与记忆引擎协作，从长期记忆加载/写入消息。

**参数：**

- **config** ([ContextEngineConfig](./config.md), 可选)：上下文引擎全局配置。
  - 控制默认窗口消息数、从记忆中恢复的消息条数等。
  - 省略时使用默认配置。

**示例：**

```python
from openjiuwen.core.context_engine import ContextEngine, ContextEngineConfig

# 创建上下文引擎实例
context_engine = ContextEngine(config=ContextEngineConfig())
```

### create_context

```python
async create_context(
    context_id: str = "default_context_id",
    session: Session | None = None,
    *,
    history_messages: list[BaseMessage] | None = None,
    token_counter: TokenCounter | None = None,
    mem_scope_id: str | None = None,
) -> ModelContext
```

为给定 `Session` 和 `context_id` 创建或获取一个 `ModelContext`。

**参数：**

- **context_id** (str)：上下文 ID，用于区分同一会话内的多个上下文（默认 `"default_context_id"`）。
- **session** (`Session` | None)：会话对象。
  - 若传入，会从 `session.get_session_id()` 读取会话 ID，用于隔离不同用户/会话。
  - 为空时使用 `"default_session_id"`。
- **history_messages** (list[`BaseMessage`] | None)：初始历史消息列表。
  - 若不传且提供 `mem_scope_id`，则会尝试从长期记忆中恢复历史。
  - 若两者都不提供，则从空上下文开始。
- **token_counter** (`TokenCounter` | None)：Token 统计策略。
  - 为空时使用默认实现（如 `TiktokenCounter`）。
  - 用于在构造窗口时统计消息与工具的 Token 消耗。
- **mem_scope_id** (str | None)：记忆作用域 ID。
  - 仅当 `history_messages` 为空时生效。
  - 配合记忆引擎从长期记忆中加载最近的若干条消息。

**返回：**

- **ModelContext**：会话 + 上下文 ID 唯一对应的上下文对象。

### get_context

```python
get_context(
    context_id: str = "default_context_id",
    session_id: str = "default_session_id",
) -> ModelContext | None
```

从内部缓存中获取已存在的 `ModelContext`。

**参数：**

- **context_id** (str)：上下文 ID。
- **session_id** (str)：会话 ID。

**返回：**

- 已存在的 `ModelContext`，若不存在则返回 `None`。

### clear_context

```python
clear_context(
    context_id: str | None = None,
    session_id: str | None = None,
) -> None
```

从内部池中移除上下文。

**行为：**

- 两个参数都为空：清空所有上下文。
- 只传 `session_id`：清空该会话下的所有上下文。
- 同时传 `session_id` 与 `context_id`：仅删除对应的单个上下文。

### save_contexts

```python
async save_contexts(
    context_ids: list[str],
    session: Session | None = None,
    *,
    mem_scope_id: str | None = None,
) -> None
```

批量保存多个上下文的运行时状态。

**行为：**

- 总是更新本地上下文缓存（调用每个上下文的 `on_save()`）。
- 若提供 `mem_scope_id`，则会将对应上下文的新增消息写入长期记忆，以便后续恢复。

**参数：**

- **context_ids** (list[str])：要保存的上下文 ID 列表。
- **session** (`Session` | None)：会话对象。
  - 若为空，使用 `"default_session_id"`。
- **mem_scope_id** (str | None)：记忆作用域 ID。
  - 提供时，会将新消息写入长期记忆。

**简要示例：**

```python
import asyncio
from openjiuwen.core.context_engine import ContextEngine
from openjiuwen.core.session.workflow import create_workflow_session
from openjiuwen.core.foundation.llm.messages import HumanMessage


async def main():
    engine = ContextEngine()
    session = create_workflow_session(session_id="s1")

    # 创建上下文并追加消息
    ctx = await engine.create_context(context_id="chat", session=session)
    await ctx.add_messages(HumanMessage(content="你好，请帮我查询天气"))

    # 构造用于模型推理的窗口
    window = await ctx.get_context_window()
    messages = window.get_messages()

    # 保存上下文（可选：同时写入长期记忆）
    await engine.save_contexts(["chat"], session=session, mem_scope_id="weather_scope")


asyncio.run(main())
```

# openjiuwen.core.context_engine.engine

## class openjiuwen.core.context_engine.engine.ContextEngine

```python
class ContextEngine(config: ContextEngineConfig | None = None)
```

`ContextEngine` 类用于管理模型输入上下文的信息，基于 `Session` 提供统一的上下文创建、缓存、保存与恢复能力。它为会话中的不同「上下文 ID」创建和维护对应的 `ModelContext`，并可选地与记忆引擎协作，从长期记忆加载/写入消息。

**参数**：

- **config** ([ContextEngineConfig](./config.md), 可选)：上下文引擎全局配置。
  - 控制默认窗口消息数、从记忆中恢复的消息条数等。
  - 省略时使用默认配置。

**示例**：

```python
from openjiuwen.core.context_engine import ContextEngine, ContextEngineConfig

# 创建上下文引擎实例
context_engine = ContextEngine(config=ContextEngineConfig())
```

### get_agent_context
```python
get_agent_context(session_id: str) -> AgentContext
```
获取agent上下文AgentContext实例。实例化过程依赖agent的创建(`agent_id`)。通过AgentContext可以对agent_id的上下文进行管理与操作。

**参数**：

- **session_id**(str)：当前会话id，用于隔离不同上下文内容，必填参数。

**返回**：

**AgentContext**，返回对应id的当前agent上下文实例，可访问和处理当前agent下的上下文信息。

**样例**：

```python
>>> import os
>>> os.environ['LLM_SSL_VERIFY'] = 'false'
>>> from datetime import datetime
>>> from openjiuwen.core.single_agent.legacy import create_react_agent_config
>>> from openjiuwen.core.single_agent import ReActAgent
>>> from openjiuwen.core.foundation.llm import ModelConfig, BaseModelInfo
>>> from openjiuwen.core.foundation.tool.param import Param
>>> from openjiuwen.core.foundation.tool.service_api.restful_api import RestfulApi
>>> 
>>> API_BASE = os.getenv("API_BASE", "your api base")
>>> API_KEY = os.getenv("API_KEY", "your api key")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def build_current_date():
...     current_datetime = datetime.now()
...     return current_datetime.strftime("%Y-%m-%d")
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
... 
>>> def _create_tool():
...     weather_plugin = RestfulApi(
...         name="WeatherReporter",
...         description="天气查询插件",
...         params=[
...             Param(name="location", description="天气查询的地点，必须为英文", type="string", required=True),
...             Param(name="date", description="天气查询的时间，格式为YYYY-MM-DD", type="string", required=True),
...         ],
...         path="your weather search api url",
...         headers={},
...         method="GET",
...         response=[],
...     )
...     return weather_plugin
... 
... 
>>> def _create_prompt_template():
...     system_prompt = "你是一个AI助手，在适当的时候调用合适的工具，帮助我完成任务！今天的日期为：{}\n注意：1. 如果用户请求中未指定具体时间，则默认为今天。"
...     return [
...         dict(role="system", content=system_prompt.format(build_current_date()))
...     ]
...
>>> agent_id="react_agent_123"
>>> model_config = _create_model()
>>> prompt_template = _create_prompt_template()
>>> 
>>> react_agent_config = create_react_agent_config(
...     agent_id=agent_id,
...     agent_version="0.0.1",
...     description="AI助手",
...     model=model_config,
...     prompt_template=prompt_template
... )
... 
>>> react_agent: ReActAgent = ReActAgent(react_agent_config)
>>> react_agent.add_tools([_create_tool()])
... 
>>> async def run_test():
...     # 生成会话id
...     session_id = "your_session_id"
...     # 调用agent
...     result = await react_agent.invoke({"query": "查询杭州的天气", "conversation_id": session_id})
...     print(f"ReActAgent 最终输出结果：{result}")
...     # 通过session_id获取agent中上下文实例
...     agent_context = react_agent.context_engine.get_agent_context(session_id=session_id)
... 
```

### get_workflow_context
```python
get_workflow_context(workflow_id: str, session_id: str) -> WorkflowContext
```
获取workflow上下文WorkflowContext实例。实例化过程依赖agent和workflow的创建(`agent_id`和`workflow_id`)。通过WorkflowContext可以对绑定在agent_id上的特定workflow的上下文进行管理与操作。

**参数**：

- **workflow_id**(str)：工作流上下文的workflow id，必填参数。
- **session_id**(str)：当前会话id，必填参数。

**返回**：

**WorkflowContext**，返回对应id的workflow上下文实例，可访问和处理当前workflow下的上下文信息。

**样例**：

```python
>>> from openjiuwen.core.single_agent.legacy import WorkflowSchema
>>> from openjiuwen.core.application.workflow_agent import WorkflowAgentConfig, WorkflowAgent
>>> from openjiuwen.core.workflow import Workflow, WorkflowCard
>>> from openjiuwen.core.workflow.workflow_config import WorkflowConfig
>>> 
>>> agent_id = "weather_agent_test"
>>> workflow_id = "weather_agent_flow_id_test"
>>> workflow_name = "weather"
>>> workflow_version = "1.0"
>>> 
>>> workflow_config = WorkflowConfig(
...     card=WorkflowCard(
...         id=workflow_id,
...         name=workflow_name,
...         version=workflow_version,
...     )
... )
... 
>>> flow = Workflow(workflow_config=workflow_config)
>>> 
>>> schema = WorkflowSchema(
...     id=workflow_id,
...     name=workflow_name,
...     description="天气查询工作流",
...     version=workflow_version,
...     inputs={"query": {"type": "string"}}
... )
... 
>>> config = WorkflowAgentConfig(
...     id=agent_id,
...     version="0.1.0",
...     description="测试用天气 agent",
...     workflows=[schema]
... )
... 
>>> workflow_agent = WorkflowAgent(config)
>>> workflow_agent.bind_workflows([flow])
>>> async def run_test():
...     # 生成会话id
...     session_id = "your_session_id"
...     result = await workflow_agent.invoke({"query": "查询上海的天气", "conversation_id": session_id})
...     # 通过workflow_id和session_id获取工作流中上下文实例
...     workflow_context = workflow_agent.context_engine.get_workflow_context(workflow_id, session_id)
... 
```
