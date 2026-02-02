# openjiuwen.core.multi_agent.config

本模块提供多 Agent 组运行时配置类 `GroupConfig`，用于设置组内最大 Agent 数、并发消息数、消息超时等。对应源码 `openjiuwen.core.multi_agent.config`。

> **注意**：类通过 `openjiuwen.core.multi_agent` 导出。建议使用 `from openjiuwen.core.multi_agent import GroupConfig` 导入。

---

## class openjiuwen.core.multi_agent.config.GroupConfig

组运行时配置（Pydantic BaseModel）。与单 Agent 的 `ReActAgentConfig` 模式一致：可变运行时参数，配置方法支持链式调用。

### 字段

- **max_agents** (int)：组内允许的最大 Agent 数量，默认 `10`。
- **max_concurrent_messages** (int)：最大并发消息处理数，默认 `100`。
- **message_timeout** (float)：单条消息处理超时时间（秒），默认 `30.0`。

模型配置为 `extra="allow"`，允许额外字段。

### configure_max_agents

```python
configure_max_agents(self, max_agents: int) -> 'GroupConfig'
```

设置组内最大 Agent 数量。在 `BaseGroup.add_agent` 时，若当前成员数已达 `max_agents`，会抛出 `JiuWenBaseException`。

**参数**：

- **max_agents** (int)：最大 Agent 数量。

**返回**：

- **GroupConfig**：self，支持链式调用。

### configure_timeout

```python
configure_timeout(self, timeout: float) -> 'GroupConfig'
```

设置消息处理超时时间（秒）。

**参数**：

- **timeout** (float)：超时时间（秒）。

**返回**：

- **GroupConfig**：self，支持链式调用。

### configure_concurrency

```python
configure_concurrency(self, max_concurrent: int) -> 'GroupConfig'
```

设置最大并发消息数。

**参数**：

- **max_concurrent** (int)：最大并发消息数。

**返回**：

- **GroupConfig**：self，支持链式调用。

---

## 使用示例

```python
from openjiuwen.core.multi_agent import GroupConfig

config = (
    GroupConfig()
    .configure_max_agents(5)
    .configure_timeout(60.0)
    .configure_concurrency(50)
)

# 或直接构造
config = GroupConfig(
    max_agents=5,
    message_timeout=60.0,
    max_concurrent_messages=50
)
```
