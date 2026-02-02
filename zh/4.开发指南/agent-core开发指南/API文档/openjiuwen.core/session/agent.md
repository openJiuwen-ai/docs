# openjiuwen.core.session.agent

## class openjiuwen.core.session.agent.Session

`openjiuwen.core.session.agent.Session` 是单 Agent（`openjiuwen.core.single_agent`）运行期的会话入口，内部持有 `TaskSession`，提供：

- 会话 id / envs
- 流式输出写入（`write_stream` / `write_custom_stream`）
- stream iterator（用于消费流式结果）
- 创建 workflow session（`create_workflow_session`）
- 交互（`interact`）

### write_stream

```python
async write_stream(self, data: dict | OutputSchema)
```

向输出流写入一帧（通常用于流式输出）。

**参数**：

- **data** (dict | OutputSchema)：要写入的输出帧。
  - 传 `OutputSchema` 时：建议明确 `type/index/payload`，便于上层稳定解析。
  - 传 dict 时：会话实现会按内部规则封装为结构化帧（建议在业务侧统一约定字段）。

**使用建议**：

- 输出帧应尽量小且可序列化，避免写入大对象。
- 若需要展示“中间过程 + 最终答案”，建议用不同的 `type` 或在 payload 中区分阶段。

### stream_iterator

```python
stream_iterator(self) -> AsyncIterator[Any]
```

获取流式输出迭代器。

**返回**：

- **AsyncIterator[Any]**：异步迭代器。
  - 上层可 `async for frame in session.stream_iterator(): ...` 消费流式结果。
  - frame 的具体结构通常为 `OutputSchema` 或可被上层直接展示/解析的对象。

### create_workflow_session

```python
create_workflow_session(self) -> openjiuwen.core.session.workflow.Session
```

从 Agent 会话派生一个工作流会话（共享 session_id/envs）。

**返回**：

- **openjiuwen.core.session.workflow.Session**：工作流会话入口。
  - 用于在 Agent 执行过程中调用 workflow，确保 session_id/envs 一致。
  - 典型用法：`flow.invoke(inputs, session.create_workflow_session())`。

## function openjiuwen.core.session.agent.create_agent_session

```python
create_agent_session(session_id: str = None, envs: dict[str, Any] = None, card: \"AgentCard\" = None) -> Session
```

创建 Agent 会话对象。

**参数**：

- **session_id** (str, 可选)：显式指定会话 id。
  - 不传时会自动生成。
  - 需要中断恢复时应复用同一个 session_id（由上层/调用方保证）。
- **envs** (dict[str, Any], 可选)：执行期环境变量。
  - 用于传递运行参数与外部配置。
  - 会与内置默认参数合并；相同 key 将覆盖默认值。
- **card** (\"AgentCard\", 可选)：Agent card 元信息。
  - 用于标识 agent 的 id/name/描述/输入输出 schema 等。
  - 部分执行链路会使用 card 做观测标记与归档。

**返回**：

- **Session**：Agent 会话入口。

