# openjiuwen.core.single_agent

本模块提供单 Agent 相关核心组件：`AgentCard` 定义 Agent 身份与参数，`BaseAgent` 为单 Agent 抽象基类，`AbilityManager` 管理 Agent 的各类能力，`ReActAgent` 实现 ReAct 范式的 Agent，`Session` 与 `create_agent_session` 用于管理 Agent 会话。

> **注意**：所有类均通过 `openjiuwen.core.single_agent` 导出。建议使用 `from openjiuwen.core.single_agent import AgentCard, BaseAgent, ReActAgent` 等方式导入。

---

## class openjiuwen.core.single_agent.AgentCard

Agent 卡片数据类，继承自 `BaseCard`，用于描述 Agent 的身份、说明与输入输出参数。作为 `BaseAgent` 构造必填参数，也用于将 Agent 作为能力（Ability）注册到其他 Agent 的 `AbilityManager` 中。

### 继承字段（BaseCard）

- **id** (str)：可选唯一标识。
- **name** (str)：Agent 名称，用于展示与能力查找。
- **description** (str)：Agent 说明，供 LLM 或上层选择能力时使用。

### 字段

- **input_params** (Optional[dict[str, Any] | Type[BaseModel]])：输入参数 schema，支持 JSON Schema 字典或 Pydantic BaseModel 类型。用于生成 `ToolInfo.parameters` 及子 Agent 调用时的 schema。默认 `None`。
- **output_params** (Optional[dict[str, Any] | Type[BaseModel]])：输出参数 schema，描述 Agent 返回结果的结构，供上层或工作流编排使用。默认 `None`。

### tool_info

```python
def tool_info(self) -> ToolInfo
```

将当前 AgentCard 转为 `ToolInfo`，便于作为子 Agent 或能力被其他组件调用。

**返回**：

- **ToolInfo**：包含 name、description、parameters 的工具信息对象。

### 使用示例

```python
from openjiuwen.core.single_agent import AgentCard

card = AgentCard(
    name="assistant",
    description="通用助手 Agent",
    input_params={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "用户输入"}
        },
        "required": ["query"]
    },
    output_params={
        "type": "object",
        "properties": {
            "reply": {"type": "string", "description": "回复内容"}
        }
    }
)
```

---

## class openjiuwen.core.single_agent.BaseAgent

单 Agent 抽象基类。设计原则：必须提供 `AgentCard`（定义"是什么"），配置可选（定义"如何运行"）；配置类方法均支持链式调用。

**属性**：

- **card** (AgentCard)：Agent 卡片（必填）。
- **ability_manager** (AbilityManager)：能力管理器，用于管理 Agent 的工具、工作流、子 Agent 等能力。

### \_\_init\_\_

```python
__init__(self, card: AgentCard)
```

使用给定的 `AgentCard` 初始化 Agent，并创建空的 `AbilityManager`。

**参数**：

- **card** (AgentCard)：Agent 身份与输入输出描述。

### configure

```python
@abstractmethod
def configure(self, config) -> 'BaseAgent'
```

设置运行时配置（抽象方法）。子类需实现，通常接受如 `ReActAgentConfig` 等配置对象，并返回 `self` 以支持链式调用。

**返回**：

- **BaseAgent**：self。

### config

```python
@property
def config(self)
```

只读属性，返回当前 Agent 的配置对象。

### ability_manager

```python
@property
def ability_manager(self) -> AbilityManager
```

只读属性，返回当前 Agent 的 `AbilityManager` 实例。

### invoke

```python
@abstractmethod
async def invoke(
    self,
    inputs: Any,
    session: Optional[Session] = None
) -> Any
```

批量执行入口（抽象方法）。子类需实现。inputs 通常支持：

- **dict**：需包含 `user_input` 或 `query`、`session_id` 等，例如 `{"query": "xxx", "conversation_id": "session_123"}`。
- **str**：直接作为 user_input，此时需通过参数提供 session。

**参数**：

- **inputs** (Any)：Agent 输入，格式见上。
- **session** (Optional[Session])：可选会话；由实现按需使用。

**返回**：

- **Any**：Agent 输出结果，具体类型由子类约定（如 dict、`AgentResult` 等）。

### stream

```python
@abstractmethod
async def stream(
    self,
    inputs: Any,
    session: Optional[Session] = None,
    stream_modes: Optional[List[StreamMode]] = None
) -> AsyncIterator[Any]
```

流式执行入口（抽象方法）。子类需实现。inputs 与 session 语义同 `invoke`；`stream_modes` 用于指定流式输出的模式（如仅最终答案、含中间步骤等）。

**参数**：

- **inputs** (Any)：同 `invoke`。
- **session** (Optional[Session])：可选会话。
- **stream_modes** (Optional[List[StreamMode]])：可选的流式模式列表。

**返回**：

- **AsyncIterator[Any]**：异步迭代器，迭代得到流式输出帧。

---

## class openjiuwen.core.single_agent.AbilityManager

Agent 能力管理器：仅存储能力元数据（Card），不持有实例；提供增删查与转换为 `ToolInfo`、执行能力调用的接口；执行时通过 `Runner.resource_mgr` 按 Card 获取实例并调用。

### \_\_init\_\_

```python
__init__(self)
```

初始化空的能力集合（内部维护 tools / workflows / agents / mcp_servers 四类 Card 字典）。

### add

```python
def add(self, ability: Union[Ability, List[Ability]]) -> None
```

添加一项或多项能力。

**参数**：

- **ability** (Union[Ability, List[Ability]])：能力 Card 或 Card 列表。支持类型：`ToolCard`、`WorkflowCard`、`AgentCard`、`McpServerConfig`。根据类型存入对应内部字典；`McpServerConfig` 以 `server_name` 为键。

### remove

```python
def remove(self, name: Union[str, List[str]]) -> Union[None, Ability, List[Ability]]
```

按名称移除一项或多项能力。

**参数**：

- **name** (Union[str, List[str]])：要移除的能力名称或名称列表（MCP 使用 server_name）。

**返回**：

- **Union[None, Ability, List[Ability]]**：被移除的 Card 或 Card 列表，若不存在则返回 `None`。

### get

```python
def get(self, name: str) -> Optional[Ability]
```

按名称获取一项能力 Card。

**参数**：

- **name** (str)：能力名称。

**返回**：

- **Optional[Ability]**：对应的 Card，若不存在则返回 `None`。

### list

```python
def list(self) -> List[Ability]
```

列出当前所有能力 Card（合并 tools、workflows、agents、mcp_servers 四类）。

**返回**：

- **List[Ability]**：所有 Card 的列表。

### list_tool_info

```python
async def list_tool_info(
    self,
    names: Optional[List[str]] = None,
    mcp_server_name: Optional[str] = None
) -> List[ToolInfo]
```

将当前能力转换为供 LLM 使用的 `ToolInfo` 列表。Tool/Workflow/Agent 类能力按 Card 的 name、description、input_params 等生成 `ToolInfo`；MCP 类能力会通过 `Runner.resource_mgr.get_mcp_tool_infos` 拉取并合并。

**参数**：

- **names** (Optional[List[str]])：仅包含这些名称的能力；为 `None` 时包含全部。
- **mcp_server_name** (Optional[str])：仅包含该 MCP 服务器的工具；为 `None` 时不做此过滤。

**返回**：

- **List[ToolInfo]**：供 LLM 使用的工具信息列表。

### execute

```python
async def execute(
    self,
    tool_call: Union[ToolCall, List[ToolCall]],
    session: Session
) -> List[Tuple[Any, ToolMessage]]
```

执行一次或多次能力调用：根据 `tool_call.name` 解析参数，按能力类型从 `Runner.resource_mgr` 获取工具/工作流/Agent 实例并调用，将结果或错误信息封装为 `ToolMessage` 返回。支持并行执行多个 tool_call。

**参数**：

- **tool_call** (Union[ToolCall, List[ToolCall]])：LLM 返回的工具调用（含 name、arguments、id 等），支持单个或列表。
- **session** (Session)：会话实例；调用工作流或子 Agent 时会传入。

**返回**：

- **List[Tuple[Any, ToolMessage]]**：`[(执行结果, ToolMessage), ...]`。执行异常时 result 可为 `None`，ToolMessage 的 content 为错误信息。

---

## class openjiuwen.core.single_agent.ReActAgent

ReAct（Reasoning + Acting）范式 Agent 实现，继承自 `BaseAgent`。

**工作原理**：ReAct 循环 - Reasoning（推理）-> Acting（行动）-> Observation（观察）-> 重复，直到得出最终答案或达到最大迭代次数。

**输入格式**：

- **dict**：`{"query": "用户问题", "conversation_id": "session_123"}`
- **str**：直接作为 query 使用

**输出格式**：

- **invoke**：`{"output": "回复内容", "result_type": "answer|error"}`
- **stream**：yields `OutputSchema` 对象

### \_\_init\_\_

```python
__init__(self, card: AgentCard)
```

初始化 ReActAgent。

**参数**：

- **card** (AgentCard)：Agent 卡片（必填）。

### configure

```python
def configure(self, config: ReActAgentConfig) -> 'BaseAgent'
```

设置 ReActAgent 配置。

**参数**：

- **config** (ReActAgentConfig)：ReActAgent 配置对象。

**返回**：

- **BaseAgent**：self，支持链式调用。

**说明**：配置更新后，context_engine 和 memory_scope 会相应更新。

### register_skill

```python
async def register_skill(self, skill_path: Union[str, List[str]])
```

注册技能。

**参数**：

- **skill_path** (Union[str, List[str]])：技能路径或路径列表。

### invoke

```python
async def invoke(
    self,
    inputs: Any,
    session: Optional[Session] = None
) -> Dict[str, Any]
```

执行 ReAct 流程。

**参数**：

- **inputs** (Any)：用户输入，支持 `{"query": "..."}` 或 `str`。
- **session** (Optional[Session])：会话对象（工具执行需要）。

**返回**：

- **Dict[str, Any]**：包含 `output` 和 `result_type` 的字典。

### stream

```python
async def stream(
    self,
    inputs: Any,
    session: Optional[Session] = None,
    stream_modes: Optional[List[StreamMode]] = None
) -> AsyncIterator[Any]
```

流式执行 ReAct 流程。

**参数**：

- **inputs** (Any)：用户输入。
- **session** (Optional[Session])：会话对象。
- **stream_modes** (Optional[List[StreamMode]])：流式输出模式。

**Yields**：

- **OutputSchema**：流式输出对象。

---

## class openjiuwen.core.single_agent.ReActAgentConfig

ReActAgent 配置类，继承自 `pydantic.BaseModel`，提供链式配置方法。

### 字段

- **mem_scope_id** (str)：记忆作用域 ID，默认 `""`。
- **model_name** (str)：模型名称，默认 `""`。
- **model_provider** (str)：模型提供商，默认 `"openai"`。
- **api_key** (str)：API 密钥，默认 `""`。
- **api_base** (str)：API 基础 URL，默认 `""`。
- **prompt_template_name** (str)：Prompt 模板名称，默认 `""`。
- **prompt_template** (List[Dict])：Prompt 模板列表，格式如 `[{"role": "system", "content": "..."}]`，默认空列表。
- **max_iterations** (int)：ReAct 循环最大迭代次数，默认 `5`。
- **model_client_config** (Optional[ModelClientConfig])：模型客户端配置，默认 `None`。
- **model_config_obj** (Optional[ModelRequestConfig])：模型请求配置，默认 `None`。
- **sys_operation_id** (Optional[str])：系统操作 ID，默认 `None`。
- **context_engine_config** (ContextEngineConfig)：上下文引擎配置，默认 `max_context_message_num=200, default_window_round_num=10`。
- **context_processors** (List[Tuple[str, BaseModel]])：上下文处理器配置，默认 `None`。

### configure_model

```python
def configure_model(self, model_name: str) -> 'ReActAgentConfig'
```

配置模型名称。

**参数**：

- **model_name** (str)：模型名称。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_model_provider

```python
def configure_model_provider(
    self,
    provider: str,
    api_key: str,
    api_base: str
) -> 'ReActAgentConfig'
```

配置模型提供商详情。

**参数**：

- **provider** (str)：模型提供商名称（如 `"openai"`）。
- **api_key** (str)：API 密钥。
- **api_base** (str)：API 基础 URL。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_prompt

```python
def configure_prompt(self, prompt_name: str) -> 'ReActAgentConfig'
```

配置 Prompt 模板名称。

**参数**：

- **prompt_name** (str)：Prompt 模板名称。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_prompt_template

```python
def configure_prompt_template(
    self,
    prompt_template: List[Dict]
) -> 'ReActAgentConfig'
```

直接配置 Prompt 模板。

**参数**：

- **prompt_template** (List[Dict])：Prompt 模板列表，格式如 `[{"role": "system", "content": "..."}]`。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_context_engine

```python
def configure_context_engine(
    self,
    max_context_message_num: Optional[int] = 200,
    default_window_round_num: Optional[int] = 10,
    enable_reload: bool = False
) -> 'ReActAgentConfig'
```

配置上下文引擎参数。

**参数**：

- **max_context_message_num** (int)：上下文窗口保留的最大消息数，默认 `200`。
- **default_window_round_num** (int)：保留的最近对话轮数（一轮 = 用户消息 → 最终助手回复），默认 `10`。
- **enable_reload** (bool)：是否允许自动重新加载之前卸载的消息，默认 `False`。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_mem_scope

```python
def configure_mem_scope(self, mem_scope_id: str) -> 'ReActAgentConfig'
```

配置记忆作用域 ID。

**参数**：

- **mem_scope_id** (str)：记忆作用域 ID。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_max_iterations

```python
def configure_max_iterations(self, max_iterations: int) -> 'ReActAgentConfig'
```

配置最大迭代次数。

**参数**：

- **max_iterations** (int)：ReAct 循环最大迭代次数。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_model_client

```python
def configure_model_client(
    self,
    provider: str,
    api_key: str,
    api_base: str,
    model_name: str,
    verify_ssl: bool = False
) -> 'ReActAgentConfig'
```

配置模型客户端，用于 LLM 初始化。此方法会同时创建 `ModelClientConfig` 和 `ModelRequestConfig`。

**参数**：

- **provider** (str)：模型提供商名称（如 `"OpenAI"`、`"SiliconFlow"`）。
- **api_key** (str)：API 密钥。
- **api_base** (str)：API 基础 URL。
- **model_name** (str)：模型名称。
- **verify_ssl** (bool)：是否验证 SSL，默认 `False`。

**返回**：

- **ReActAgentConfig**：self，支持链式调用。

### configure_context_processors

```python
def configure_context_processors(
    self,
    processors: List[Tuple[str, BaseModel]]
)
```

配置上下文处理器。

**参数**：

- **processors** (List[Tuple[str, BaseModel]])：处理器配置列表。

---

## class openjiuwen.core.single_agent.Session

Agent 会话类，封装底层 `TaskSession`，提供会话 ID、环境变量、流式输出等管理功能。

### \_\_init\_\_

```python
__init__(
    self,
    session_id: str = None,
    envs: dict[str, Any] = None,
    card: "AgentCard" = None
)
```

初始化会话。

**参数**：

- **session_id** (str)：会话 ID，若为 `None` 则自动生成 UUID。
- **envs** (dict[str, Any])：环境变量字典，可选。
- **card** (AgentCard)：Agent 卡片，可选。

### get_session_id

```python
def get_session_id(self) -> str
```

获取会话 ID。

**返回**：

- **str**：会话 ID。

### get_envs

```python
def get_envs(self)
```

获取环境变量。

**返回**：

- 环境变量字典。

### get_agent_id

```python
def get_agent_id(self)
```

获取关联的 Agent ID。

**返回**：

- Agent 的 id。

### get_agent_name

```python
def get_agent_name(self)
```

获取关联的 Agent 名称。

**返回**：

- Agent 的 name。

### get_agent_description

```python
def get_agent_description(self)
```

获取关联的 Agent 描述。

**返回**：

- Agent 的 description。

### write_stream

```python
async def write_stream(self, data: Union[dict, OutputSchema])
```

写入流式输出数据。

**参数**：

- **data** (Union[dict, OutputSchema])：要写入的数据。

### write_custom_stream

```python
async def write_custom_stream(self, data: dict)
```

写入自定义流式输出数据。

**参数**：

- **data** (dict)：要写入的自定义数据。

### stream_iterator

```python
def stream_iterator(self) -> AsyncIterator[Any]
```

获取流式输出迭代器。

**返回**：

- **AsyncIterator[Any]**：异步迭代器。

### post_run

```python
async def post_run(self)
```

执行后处理，关闭流式输出。

### create_workflow_session

```python
def create_workflow_session(self) -> WorkflowSession
```

创建工作流子会话。

**返回**：

- **WorkflowSession**：工作流会话实例。

### interact

```python
async def interact(self, value)
```

向会话发送交互数据。

**参数**：

- **value**：交互数据。

---

## function openjiuwen.core.single_agent.create_agent_session

```python
def create_agent_session(
    session_id: str = None,
    envs: dict[str, Any] = None,
    card: "AgentCard" = None
) -> Session
```

创建 Agent 会话的工厂函数。

**参数**：

- **session_id** (str)：会话 ID，若为 `None` 则自动生成 UUID。
- **envs** (dict[str, Any])：环境变量字典，可选。
- **card** (AgentCard)：Agent 卡片，可选。

**返回**：

- **Session**：创建的会话实例。

---

## 类型定义

### Ability

```python
Ability = Union[ToolCard, WorkflowCard, AgentCard, McpServerConfig]
```

能力类型定义，表示 Agent 可使用的能力 Card 类型，包括：

- **ToolCard**：工具卡片
- **WorkflowCard**：工作流卡片
- **AgentCard**：Agent 卡片（子 Agent）
- **McpServerConfig**：MCP 服务器配置

---

## 使用示例

### 基本使用

```python
from openjiuwen.core.single_agent import (
    AgentCard,
    ReActAgent,
    ReActAgentConfig,
    create_agent_session
)

# 1. 定义 Agent 身份
card = AgentCard(
    name="my_assistant",
    description="智能助手 Agent",
    input_params={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "用户问题"}
        },
        "required": ["query"]
    }
)

# 2. 创建并配置 ReActAgent
config = (
    ReActAgentConfig()
    .configure_model_client(
        provider="OpenAI",
        api_key="your-api-key",
        api_base="https://api.openai.com/v1",
        model_name="gpt-4"
    )
    .configure_prompt_template([
        {"role": "system", "content": "你是一个智能助手。"}
    ])
    .configure_max_iterations(5)
)

agent = ReActAgent(card=card).configure(config)

# 3. 创建会话并执行
session = create_agent_session(card=card)
result = await agent.invoke({"query": "你好，请介绍一下自己"}, session=session)
print(result)  # {"output": "...", "result_type": "answer"}
```

### 添加工具能力

```python
from openjiuwen.core.foundation.tool import ToolCard

# 定义工具卡片
search_tool = ToolCard(
    name="web_search",
    description="搜索网页内容",
    input_params={
        "type": "object",
        "properties": {
            "keyword": {"type": "string", "description": "搜索关键词"}
        },
        "required": ["keyword"]
    }
)

# 添加工具到 Agent
agent.ability_manager.add(search_tool)

# 列出所有能力
abilities = agent.ability_manager.list()
```

### 流式输出

```python
session = create_agent_session(card=card)

async for chunk in agent.stream({"query": "帮我写一首诗"}, session=session):
    print(chunk)
```

### 链式配置

```python
agent = (
    ReActAgent(card=card)
    .configure(
        ReActAgentConfig()
        .configure_model_client(
            provider="OpenAI",
            api_key="your-api-key",
            api_base="https://api.openai.com/v1",
            model_name="gpt-4"
        )
        .configure_max_iterations(10)
        .configure_context_engine(
            max_context_message_num=100,
            default_window_round_num=5
        )
    )
)
```

