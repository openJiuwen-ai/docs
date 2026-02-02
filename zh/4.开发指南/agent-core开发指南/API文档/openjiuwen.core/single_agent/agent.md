# openjiuwen.core.single_agent.agent

本模块提供单 Agent 基类与能力管理：`AbilityKit` 用于管理 Agent 的各类能力（工具、工作流、子 Agent、MCP 等），`BaseAgent` 为单 Agent 抽象基类，要求子类实现 `configure`、`invoke`、`stream`。

> **注意**：类通过 `openjiuwen.core.single_agent` 导出。建议使用 `from openjiuwen.core.single_agent import BaseAgent, AgentCard` 或 `from openjiuwen.core.single_agent.agent import BaseAgent, AbilityKit` 导入。

---

## class openjiuwen.core.single_agent.agent.AbilityKit

Agent 能力管理器：仅存储能力元数据（Card），不持有实例；提供增删查与转换为 `ToolInfo`、执行能力调用的接口；执行时通过 `Runner.resource_mgr` 按 Card 获取实例并调用。

### \_\_init\_\_

```python
__init__(self)
```

初始化空的能力集合（内部维护 tools / workflows / agents / mcp_servers 四类 Card 字典）。

### add

```python
add(self, ability: Ability) -> None
```

添加一项能力。

**参数**：

- **ability** (Ability)：能力 Card。支持类型：`ToolCard`、`WorkflowCard`、`AgentCard`、`McpServerConfig`。根据类型存入对应内部字典；`McpServerConfig` 以 `server_name` 为键。

### remove

```python
remove(self, name: str) -> Optional[Ability]
```

按名称移除一项能力。

**参数**：

- **name** (str)：要移除的能力名称（或 MCP 的 server_name）。

**返回**：

- **Optional[Ability]**：被移除的 Card，若不存在则返回 `None`。

### get

```python
get(self, name: str) -> Optional[Ability]
```

按名称获取一项能力 Card。

**参数**：

- **name** (str)：能力名称。

**返回**：

- **Optional[Ability]**：对应的 Card，若不存在则返回 `None`。

### list

```python
list(self) -> List[Ability]
```

列出当前所有能力 Card（合并 tools、workflows、agents、mcp_servers 四类）。

**返回**：

- **List[Ability]**：所有 Card 的列表。

### list_tool_info

```python
async list_tool_info(
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
async execute(
    self,
    tool_call: ToolCall,
    session: Session
) -> Tuple[Any, ToolMessage]
```

执行一次能力调用：根据 `tool_call.name` 解析参数，按能力类型从 `Runner.resource_mgr` 获取工具/工作流/Agent 实例并调用，将结果或错误信息封装为 `ToolMessage` 返回。

**参数**：

- **tool_call** (ToolCall)：LLM 返回的工具调用（含 name、arguments、id 等）。
- **session** (Session)：会话实例；调用工作流或子 Agent 时会传入。

**返回**：

- **Tuple[Any, ToolMessage]**：`(执行结果, ToolMessage)`。执行异常时 result 可为 `None`，ToolMessage 的 content 为错误信息。

---

## class openjiuwen.core.single_agent.agent.BaseAgent

单 Agent 抽象基类。设计原则：必须提供 `AgentCard`（定义“是什么”），配置可选（定义“如何运行”）；配置类方法均支持链式调用。

**属性**：

- **card** (AgentCard)：Agent 卡片（必填）。
- **ability_kit** (AbilityKit)：能力管理器，通过 `add_ability` / `remove_ability` 等使用。

### \_\_init\_\_

```python
__init__(self, card: AgentCard)
```

使用给定的 `AgentCard` 初始化 Agent，并创建空的 `AbilityKit`。

**参数**：

- **card** (AgentCard)：Agent 身份与输入输出描述，见 [schema](schema.md)。

### configure

```python
def configure(self, config) -> 'BaseAgent'
```

设置运行时配置（抽象方法）。子类需实现，通常接受如 `ReActAgentConfig` 等配置对象，并返回 `self` 以支持链式调用。

**返回**：

- **BaseAgent**：self。

### ability_kit

```python
@property
ability_kit(self) -> AbilityKit
```

只读属性，返回当前 Agent 的 `AbilityKit` 实例。

### add_ability

```python
add_ability(self, ability: Union[Ability, List[Ability]]) -> 'BaseAgent'
```

添加一项或多项能力。支持传入单个 `ToolCard`/`WorkflowCard`/`AgentCard`/`McpServerConfig` 或列表。

**参数**：

- **ability** (Union[Ability, List[Ability]])：要添加的能力 Card 或 Card 列表。

**返回**：

- **BaseAgent**：self，支持链式调用。

### remove_ability

```python
remove_ability(self, name: Union[str, List[str]]) -> 'BaseAgent'
```

按名称移除一项或多项能力。

**参数**：

- **name** (Union[str, List[str]])：要移除的能力名称（或名称列表）。

**返回**：

- **BaseAgent**：self，支持链式调用。

### get_ability

```python
get_ability(self, name: str) -> Optional[Ability]
```

按名称获取一项能力 Card。

**参数**：

- **name** (str)：能力名称。

**返回**：

- **Optional[Ability]**：对应的 Card，不存在则返回 `None`。

### list_abilities

```python
list_abilities(self) -> List[Ability]
```

列出当前所有能力 Card。

**返回**：

- **List[Ability]**：所有 Card 的列表。

### list_tool_info

```python
async list_tool_info(self, names: Optional[List[str]] = None) -> List[ToolInfo]
```

获取供 LLM 使用的 `ToolInfo` 列表，内部委托给 `ability_kit.list_tool_info`。

**参数**：

- **names** (Optional[List[str]])：仅包含这些名称的能力；为 `None` 时包含全部。

**返回**：

- **List[ToolInfo]**：ToolInfo 列表。

### get_tool_info

```python
get_tool_info(self) -> ToolInfo
```

将当前 Agent 转为 `ToolInfo`，便于作为子 Agent 被其他 Agent 或工作流调用。根据 `card` 的 name、description、input_params 构建参数 schema。

**返回**：

- **ToolInfo**：表示本 Agent 的工具信息。

### invoke

```python
async invoke(
    self,
    inputs: Any,
    session: Optional[Session] = None
) -> Any
```

批量执行入口（抽象方法）。子类需实现。inputs 通常支持：

- **dict**：需包含 `user_input`、`session_id` 等，例如 `{"user_input": "xxx", "session_id": "session_123"}`。
- **str**：直接作为 user_input，此时需通过参数或 inputs 提供 session_id。

若未传入 `session`，实现中会根据 inputs 中的 session_id 创建或获取会话。

**参数**：

- **inputs** (Any)：Agent 输入，格式见上。
- **session** (Optional[Session])：可选会话；不传时由实现按 inputs 创建或获取。

**返回**：

- **Any**：Agent 输出结果，具体类型由子类约定（如 `AgentResult` 或 dict）。

### stream

```python
async stream(
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

## 使用示例

```python
from openjiuwen.core.single_agent import BaseAgent, AgentCard, ReActAgent, ReActAgentConfig
from openjiuwen.core.session.agent import create_agent_session

# 使用 AgentCard 定义 Agent 身份
card = AgentCard(
    name="my_agent",
    description="示例单 Agent",
    input_params=[],
    output_params=[]
)

# 使用 ReActAgent（BaseAgent 子类）并链式配置能力与配置
agent = (
    ReActAgent(card=card)
    .configure(ReActAgentConfig())
    .add_ability(some_tool_card)
)

session = create_agent_session()
result = await agent.invoke({"user_input": "你好", "session_id": session.session_id}, session=session)
```
