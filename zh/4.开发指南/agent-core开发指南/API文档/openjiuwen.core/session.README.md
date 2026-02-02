# session

`openjiuwen.core.session`是openJiuwen框架的会话管理模块，提供Agent和工作流的会话上下文管理、状态管理、流式输出和交互能力。

**Classes**：

| CLASS | DESCRIPTION |
|-------|-------------|
| [Session](./session/session.md) | 会话基类。 |
| [BaseSession](./session/session.md) | 会话抽象基类。 |
| [WorkflowSession](./session/workflow.md) | 工作流会话类。 |
| [NodeSession](./session/workflow.md) | 节点会话类。 |
| [SubWorkflowSession](./session/workflow.md) | 子工作流会话类。 |
| [RouterSession](./session/wrapper.md) | 路由会话类。 |
| [StaticAgentSession](./session/agent.md) | 静态Agent会话类。 |
| [WrappedSession](./session/wrapper.md) | 包装会话类。 |
| [ProxySession](./session/session.md) | 代理会话类。 |
| [InteractiveInput](./session/interaction.md) | 交互输入类。 |
| [InteractionOutput](./session/interaction.md) | 交互输出类。 |
| [Checkpointer](./session/checkpointer.md) | 检查点管理器类。 |
| [InMemoryCheckpointer](./session/checkpointer.md) | 内存检查点管理器实现。 |
| [Config](./session/config.md) | 会话配置类。 |
| [BaseStreamMode](./session/stream/base.md) | 流式输出模式枚举。 |
| [OutputSchema](./session/stream/base.md) | 标准流式输出数据格式。 |
| [TraceSchema](./session/stream/base.md) | 调试信息流式输出数据格式。 |
| [CustomSchema](./session/stream/base.md) | 自定义流式输出数据格式。 |

**Functions**：

| FUNCTION | DESCRIPTION |
|----------|-------------|
| [create_workflow_session](./session/workflow.md) | 创建工作流会话。 |
| [create_agent_session](./session/agent.md) | 创建Agent会话。 |
| [get_default_inmemory_checkpointer](./session/checkpointer.md) | 获取默认内存检查点管理器。 |
