# single_agent

`openjiuwen.core.single_agent`提供了openJiuwen的单Agent功能模块。

**Classes**：

| CLASS | DESCRIPTION |
|-------|-------------|
| [BaseAgent](./single_agent/agent.md) | Agent抽象基类。 |
| [ReActAgent](./single_agent/agent.md) | ReAct Agent实现类。 |
| [ReActAgentConfig](./single_agent/agent.md) | ReAct Agent配置类。 |
| [AgentCard](./single_agent/schema.md) | Agent卡片类。 |
| [Session](session/session.md) | Agent会话类。 |

**Functions**：

| FUNCTION | DESCRIPTION |
|----------|-------------|
| [create_agent_session](session/session.md) | 创建Agent会话。 |

**Legacy Classes** (兼容性，不推荐使用)：

| CLASS | DESCRIPTION |
|-------|-------------|
| [LegacyBaseAgent](./single_agent/agent.md) | 旧版Agent基类（已废弃）。 |

> **注意**：Legacy类已迁移到 `openjiuwen.core.single_agent.legacy` 模块，仅用于向后兼容。新代码请使用新的API。
