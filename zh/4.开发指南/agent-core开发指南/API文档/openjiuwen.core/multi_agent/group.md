# openjiuwen.core.multi_agent.group

本模块提供多 Agent 组抽象基类 `BaseGroup`。设计原则与单 Agent 一致：必须提供 `GroupCard`（定义“是什么”），配置可选（`GroupConfig` 定义“如何运行”）；配置与注册方法均支持链式调用。

> **注意**：类通过 `openjiuwen.core.multi_agent` 导出。建议使用 `from openjiuwen.core.multi_agent import BaseGroup, GroupCard, GroupConfig` 导入。

---

## class openjiuwen.core.multi_agent.group.BaseGroup

Agent 组抽象基类。内部维护 `card`、`config` 与 `agents` 字典（agent_id -> Agent 实例）；子类需实现 `invoke`、`stream`。

**属性**：

- **card** (GroupCard)：组身份卡片（必填），见 [schema](schema.md)。
- **config** (GroupConfig)：运行时配置，见 [config](config.md)。
- **group_id** (str)：组 id，默认与 `card.name` 一致。
- **agents** (Dict[str, BaseAgent])：已注册的 Agent 实例，键为 agent_id。

### \_\_init\_\_

```python
__init__(
    self,
    card: GroupCard,
    config: Optional[GroupConfig] = None
)
```

使用给定的 `GroupCard` 初始化组；若未传入 `config`，则使用 `_create_default_config()` 得到的默认 `GroupConfig`。

**参数**：

- **card** (GroupCard)：组身份与成员元信息（如 agent_cards、topic、version 等）。
- **config** (Optional[GroupConfig])：可选运行时配置；为 `None` 时使用默认配置。

### _create_default_config

```python
_create_default_config(self) -> GroupConfig
```

创建默认的 `GroupConfig` 实例。子类可重写以返回自定义默认配置。

**返回**：

- **GroupConfig**：默认配置对象。

### configure

```python
configure(self, config: GroupConfig) -> 'BaseGroup'
```

设置运行时配置。

**参数**：

- **config** (GroupConfig)：组配置对象（如 max_agents、message_timeout、max_concurrent_messages 等）。

**返回**：

- **BaseGroup**：self，支持链式调用。

### add_agent

```python
add_agent(
    self,
    agent: BaseAgent,
    agent_id: Optional[str] = None
) -> 'BaseGroup'
```

向组内注册一个 Agent。若未指定 `agent_id`，则使用 `agent.card.name`；若 Agent 已有 `controller` 且支持 `set_group`，会自动注入当前组引用。

**参数**：

- **agent** (BaseAgent)：要注册的 Agent 实例，需具备 `card.name`（或显式传入 agent_id）。
- **agent_id** (Optional[str])：自定义注册 id；为 `None` 时使用 `agent.card.name`。

**返回**：

- **BaseGroup**：self，支持链式调用。

**异常**：

- **JiuWenBaseException**：当 agent 无法得到 agent_id、agent_id 已存在、或当前成员数已达 `config.max_agents` 时抛出（StatusCode.AGENT_GROUP_ADD_FAILED）。

**样例**：

```python
group.add_agent(agent1).add_agent(agent2)
group.add_agent(agent1, agent_id="custom_id")
```

### remove_agent

```python
remove_agent(
    self,
    agent_id: Union[str, BaseAgent]
) -> 'BaseGroup'
```

从组内移除一个 Agent。可传入 agent_id 字符串或 Agent 实例（此时用其 `card.name` 作为 id）。

**参数**：

- **agent_id** (Union[str, BaseAgent])：要移除的 Agent 的 id，或 Agent 实例。

**返回**：

- **BaseGroup**：self，支持链式调用。

### get_agent

```python
get_agent(self, agent_id: str) -> Optional[BaseAgent]
```

按 id 获取组内 Agent 实例。

**参数**：

- **agent_id** (str)：Agent 注册 id。

**返回**：

- **Optional[BaseAgent]**：对应的 Agent 实例，不存在则返回 `None`。

### get_agent_count

```python
get_agent_count(self) -> int
```

获取当前组内 Agent 数量。

**返回**：

- **int**：Agent 数量。

### list_agents

```python
list_agents(self) -> List[str]
```

列出当前组内所有 Agent 的 id。

**返回**：

- **List[str]**：Agent id 列表。

### invoke

```python
async invoke(
    self,
    message,
    session: Optional[Session] = None
) -> Any
```

对组执行一次同步调用（抽象方法）。子类需实现：根据 message 与 session 决定路由与执行逻辑，并返回组的整体输出。

**参数**：

- **message**：消息对象或 dict，具体由子类约定。
- **session** (Optional[Session])：Agent 组会话，见 `openjiuwen.core.session.agent_group.Session`。

**返回**：

- **Any**：组的集体输出，类型由子类约定。

### stream

```python
async stream(
    self,
    message,
    session: Optional[Session] = None
) -> AsyncIterator[Any]
```

对组执行流式调用（抽象方法）。子类需实现：根据 message 与 session 产生流式输出。

**参数**：

- **message**：同 `invoke`。
- **session** (Optional[Session])：同 `invoke`。

**返回**：

- **AsyncIterator[Any]**：异步迭代器，迭代得到流式输出帧。

---

## 使用示例

```python
from openjiuwen.core.multi_agent import BaseGroup, GroupCard, GroupConfig
from openjiuwen.core.single_agent import AgentCard, ReActAgent

card = GroupCard(
    name="my_group",
    description="示例多 Agent 组",
    agent_cards=[],
    topic="demo",
    version="1.0.0",
    tags=[]
)
config = GroupConfig().configure_max_agents(5).configure_timeout(60.0)

# 子类需实现 invoke / stream；此处仅展示注册方式
class MyGroup(BaseGroup):
    async def invoke(self, message, session=None):
        return {"result": "from group"}
    async def stream(self, message, session=None):
        yield {"chunk": "from group"}

group = MyGroup(card=card, config=config)
group.add_agent(agent1).add_agent(agent2)
result = await group.invoke({"content": "hello"}, session=group_session)
```
