# openjiuwen.core.runner.runner

## class Runner

```python
class openjiuwen.core.runner.runner.Runner
```

`Runner` 是 openJiuwen 中统一的**执行入口与资源管理中心**，负责：

- 执行 Workflow（工作流）、Agent（单智能体）、AgentGroup（多智能体协作）；
*- 管理工作流、Agent、工具、大模型、系统操作等资源（通过 `ResourceMgr`）；
*- 在分布式模式下，启动和管理消息队列以及系统级订阅。

对应源码：`openjiuwen.core.runner.runner.Runner`。

### __init__

```python
def __init__(self, runner_id: str = "global", config: RunnerConfig | None = None)
```

**参数**：

- `runner_id: str`：Runner 的唯一 id，默认 `"global"`；
- `config: RunnerConfig | None`：Runner 全局配置，不传时使用 `DEFAULT_RUNNER_CONFIG`（本地模式、fake MQ 等）。

构造时会：

- 创建资源管理器 `ResourceMgr()`；
- 创建本地消息队列 `LocalMessageQueue()`；
- 调用 `set_runner_config(config or DEFAULT_RUNNER_CONFIG)` 初始化 Runner 全局配置。

### 属性 resource_mgr / pubsub / dist_pubsub

```python
@property
def resource_mgr(self) -> ResourceMgr

@property
def pubsub(self)

@property
def dist_pubsub(self)
```

- `resource_mgr`：返回 `ResourceMgr` 实例，用于统一管理工作流、Agent、AgentGroup、Tool、Model、Prompt、SysOperation 等资源；
- `pubsub`：本地消息队列（`LocalMessageQueue` 实例），用于进程内发布/订阅；
- `dist_pubsub`：分布式消息队列（仅在 `distributed_mode=True` 时初始化），用于跨进程通信。

### set_config / get_config

```python
def set_config(self, config: RunnerConfig) -> None
def get_config(self) -> RunnerConfig
```

- `set_config`：设置 Runner 全局配置（内部调用 `set_runner_config(config)`）；
- `get_config`：获取当前全局配置（内部调用 `get_runner_config()`）。

## 生命周期管理

### async start

```python
async def start(self) -> bool
```

启动 Runner 及其相关组件。

**行为**：

- 打印启动日志；
- 若 `RunnerConfig.distributed_mode` 为 `True`：
  - 通过 `MessageQueueFactory.create(...)` 创建分布式消息队列实例并启动；
  - 创建 `ReplyTopicSubscription` 实例用于处理系统级回复，并启动；
  - 启动本地消息队列 `self._message_queue.start()`；
- 若全部成功，返回 `True`，否则返回 `False` 并输出错误日志。

### async stop

```python
async def stop(self) -> bool
```

停止 Runner 并释放资源。

**行为**：

- 若当前处于分布式模式：
  - 关闭并清理 `ReplyTopicSubscription`；
  - 停止分布式消息队列；
- 停止本地消息队列；
- 始终调用 `self._resource_manager.release()` 释放所有资源；
- 打印停止日志，并返回停止结果（`True/False`）。

## 执行工作流

### async run_workflow

```python
async def run_workflow(
    self,
    workflow: str | Workflow,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    envs: dict[str, Any] | None = None,
) -> Any
```

执行单个 Workflow，并返回其执行结果。

**参数**：

- `workflow: str | Workflow`：
  - 若为字符串：表示 workflow id，Runner 会从 `resource_mgr.workflow()` 中获取对应的 Workflow 实例；
  - 若为 `Workflow` 实例：直接使用该实例执行。
- `inputs: Any`：工作流输入数据。
- `session: str | Session | None`：
  - 可以传入已有的工作流会话 id（字符串）或 `Session` 实例；
  - 若不传，则内部自动创建新的工作流会话（`create_workflow_session()`）。
- `context: ModelContext | None`：模型上下文（如历史消息、token 统计等），可选。
- `envs: dict[str, Any] | None`：执行环境变量/配置覆盖项（当前实现中预留）。

**行为**：

1. 通过 `_prepare_workflow(workflow, session)` 获取 `(workflow_instance, workflow_session)`：
   - 字符串 id 会从资源管理器中查找；
   - 会话参数会根据类型创建或转换为 WorkflowSession。
2. 调用：

   ```python
   return await workflow_instance.invoke(inputs, session=workflow_session, context=context)
   ```

### async run_workflow_streaming

```python
async def run_workflow_streaming(
    self,
    workflow: str | Workflow,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    stream_modes: list[BaseStreamMode] | None = None,
    envs: dict[str, Any] | None = None,
)
```

以流式方式执行 Workflow，返回一个异步迭代器，每次迭代产生一个 `WorkflowChunk`（通常是 `OutputSchema/CustomSchema/TraceSchema`）。

**行为**：

1. 与 `run_workflow` 相同，通过 `_prepare_workflow` 获取实例和会话；
2. 调用：

   ```python
   async for chunk in workflow_instance.stream(
       inputs,
       session=workflow_session,
       stream_modes=stream_modes,
       context=context,
   ):
       yield chunk
   ```

## 执行 Agent

### async run_agent

```python
async def run_agent(
    self,
    agent: str | BaseAgent | LegacyBaseAgent,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    envs: dict[str, Any] | None = None,
) -> Any
```

执行单个 Agent，并返回其执行结果。

**行为（简要）**：

- 通过 `_prepare_agent(agent, inputs)` 获取 `agent_instance` 与 `agent_session`；
- 若 `agent_instance` 是 `RemoteAgent`：直接调用 `agent_instance.invoke(inputs)`；
- 若是 `LegacyBaseAgent`：调用 `agent_instance.invoke(inputs, session=None)`（由控制器内部管理会话）；
- 否则：调用 `agent_instance.invoke(inputs, agent_session)`，并在结束后执行 `agent_session._inner.post_run()`。

### async run_agent_streaming

```python
async def run_agent_streaming(
    self,
    agent: str | BaseAgent | LegacyBaseAgent,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    stream_modes: list[BaseStreamMode] | None = None,
    envs: dict[str, Any] | None = None,
)
```

流式执行单个 Agent。

**行为（简要）**：

- 通过 `_prepare_agent(agent, inputs, session)` 获取实例与会话；
- 若为 `RemoteAgent`：`async for chunk in agent_instance.stream(inputs): yield chunk`；
- 若为 `LegacyBaseAgent`：`agent_instance.stream(inputs, session=None)`；
- 否则：`agent_instance.stream(inputs, session=agent_session)`，流结束后调用 `agent_session._inner.post_run()`。

## 执行 AgentGroup

### async run_agent_group

```python
async def run_agent_group(
    self,
    agent_group: str | BaseGroup,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    envs: dict[str, Any] | None = None,
) -> Any
```

执行一个 AgentGroup，并返回其执行结果。

**行为**：

- 通过 `_prepare_agent_group(agent_group)` 获取 `agent_group_instance`（字符串 id 会从资源管理器中加载）；
- 调用 `await agent_group_instance.invoke(inputs)` 返回结果。

### async run_agent_group_streaming

```python
async def run_agent_group_streaming(
    self,
    agent_group: str | BaseGroup,
    inputs: Any,
    *,
    session: str | Session | None = None,
    context: ModelContext | None = None,
    stream_modes: list[BaseStreamMode] | None = None,
    envs: dict[str, Any] | None = None,
)
```

流式执行 AgentGroup。

**行为**：

- 通过 `_prepare_agent_group(agent_group)` 获取实例；
- 调用 `async for chunk in agent_group_instance.stream(inputs): yield chunk`。

## 释放会话资源

### async release

```python
async def release(self, session_id: str) -> None
```

根据会话 id 释放与之关联的内存 checkpoint 等资源。

**行为**：

- 调用 `get_default_inmemory_checkpointer().release(session_id)` 删除对应会话的保存状态。

> **注意**：在大多数场景下，Runner 和 Session 的生命周期由框架自动管理；只有在需要精细化控制会话资源释放时，才需要显式调用该方法。

