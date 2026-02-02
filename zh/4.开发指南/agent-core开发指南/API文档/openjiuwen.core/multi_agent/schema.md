# openjiuwen.core.multi_agent.schema

本模块提供多 Agent 组身份与事件驱动扩展的数据结构：`GroupCard` 定义组身份与成员元信息，`EventDrivenGroupCard` 在 `GroupCard` 基础上增加订阅信息，用于事件驱动消息路由。对应源码 `openjiuwen.core.multi_agent.schema.group_card`。

> **注意**：类通过 `openjiuwen.core.multi_agent` 导出。建议使用 `from openjiuwen.core.multi_agent import GroupCard, EventDrivenGroupCard` 导入。

---

## class openjiuwen.core.multi_agent.schema.group_card.GroupCard

组身份卡片（Pydantic，继承自 [BaseCard](../common.README.md)）。描述组的不可变身份信息：id、name、description、成员 Agent 的 Card 列表、主题、版本、标签等。

### 继承字段（BaseCard）

- **id** (str)：可选唯一标识。
- **name** (str)：组名称，用于展示与查找。
- **description** (str)：组说明。

### 字段

- **agent_cards** (List[AgentCard])：组内成员的 Agent 卡片列表（仅元数据，非实例），默认空列表。在 `BaseGroup.add_agent` 时，若 Agent 有 `card`，会自动追加到 `agent_cards`。
- **topic** (str)：组的主要主题或领域，默认空字符串。
- **version** (str)：组版本字符串，默认 `'1.0.0'`。
- **tags** (List[str])：可选标签，用于分类，默认空列表。

### 使用说明

- 创建 `BaseGroup` 时必须传入 `GroupCard`。
- `agent_cards` 与组内 `agents` 字典对应：注册 Agent 时会把其 `card` 加入 `agent_cards`，移除时从 `agent_cards` 中剔除。

**样例**：

```python
from openjiuwen.core.multi_agent import GroupCard
from openjiuwen.core.single_agent import AgentCard

card = GroupCard(
    name="support_group",
    description="客服多 Agent 组",
    agent_cards=[],
    topic="customer_service",
    version="1.0.0",
    tags=["support", "multi_agent"]
)
```

---

## class openjiuwen.core.multi_agent.schema.group_card.EventDrivenGroupCard

事件驱动组卡片，继承自 `GroupCard`。在组身份基础上增加 `subscriptions` 字段，用于描述“哪个 Agent 订阅哪些主题”，便于事件驱动场景下的消息路由。

### 继承字段

- 继承 `GroupCard` 全部字段（id、name、description、agent_cards、topic、version、tags）。

### 字段

- **subscriptions** (Dict[str, List[str]])：订阅映射，格式为 `{agent_id: [topic1, topic2, ...]}`，表示该 Agent 订阅的主题列表，默认空字典。路由逻辑可根据消息主题与 `subscriptions` 决定将消息发给哪些 Agent。

### 使用说明

- 适用于事件驱动多 Agent 场景：消息按主题分发，仅将消息发给订阅了该主题的 Agent。
- 与 `GroupCard` 一样作为 `BaseGroup` 的 `card` 使用；子类可根据 `card` 是否为 `EventDrivenGroupCard` 决定是否启用事件驱动路由。

**样例**：

```python
from openjiuwen.core.multi_agent import EventDrivenGroupCard

card = EventDrivenGroupCard(
    name="event_group",
    description="事件驱动组",
    agent_cards=[],
    topic="events",
    version="1.0.0",
    tags=[],
    subscriptions={
        "agent_a": ["order", "payment"],
        "agent_b": ["notification"]
    }
)
```
