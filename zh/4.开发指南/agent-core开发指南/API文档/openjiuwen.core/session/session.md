# openjiuwen.core.session.session

## class openjiuwen.core.session.session.BaseSession

`BaseSession` 是底层会话实现的抽象基类，框架内部的 `WorkflowSession / NodeSession / SubWorkflowSession` 等会实现该接口。它负责承载并组织执行期的底座能力，包括：

- **配置**：`config()`（env、工作流配置、agent 配置等）
- **状态**：`state()`（组件私有状态、全局状态、工作流状态等）
- **观测**：`tracer()`（trace/span/埋点）
- **流式输出**：`stream_writer_manager()`（output/custom writer）
- **回调**：`callback_manager()`（执行过程回调/钩子）
- **中断恢复**：`checkpointer()`（保存/恢复执行状态）

> **说明**：一般业务开发不需要直接实现 `BaseSession`。\n+> 组件开发/工作流编排通常接触的是更高层 `Session`（下文）以及 `openjiuwen.core.session.node.Session` 这类适配器。

### config

```python
config(self) -> Config
```

获取会话配置对象，用于读取运行参数与已注册配置。

**返回**：

- **Config**：会话配置对象。\n  - 常见用途：读取超时、开关、循环次数上限等运行参数。\n  - 上层通常通过 `Session.get_env(key)`、`Session.get_workflow_config(...)`、`Session.get_agent_config()` 间接使用。

### state

```python
state(self) -> State
```

获取会话状态容器（底层状态存储与提交机制）。

**返回**：

- **State**：状态容器。
  - 组件状态/全局状态的读写最终会落到这里。
  - 工作流执行过程中可能会在关键节点进行 commit（用于中断恢复与一致性）。

### tracer

```python
tracer(self) -> Any
```

获取会话的 tracer 对象，用于埋点、链路追踪、异常定位等场景。

**返回**：

- **Any**：tracer 对象。
  - 常见用途：记录执行过程中的关键事件、耗时、错误信息等。
  - 上层通常通过 `Session.trace()`、`Session.trace_error()` 间接使用。

### stream_writer_manager

```python
stream_writer_manager(self) -> StreamWriterManager
```

获取流式写入管理器，用于输出结构化帧。

**返回**：

- **StreamWriterManager**：writer 管理器。\n  - 可获取 output writer（标准输出）与 custom writer（自定义输出）。\n  - 典型输出数据结构为 `OutputSchema`。

### callback_manager

```python
callback_manager(self) -> CallbackManager
```

获取回调管理器。

**返回**：

- **CallbackManager**：用于注册/触发回调。\n  - 典型用于观测、统计、调试扩展点。

### session_id

```python
session_id(self) -> str
```

获取会话 id。\n+会话 id 用于把一次执行过程关联到同一条会话上下文（状态、输出、trace 等），从而支持：

- 同一会话内多轮调用共享上下文
- 中断后恢复时定位到正确状态

### checkpointer

```python
checkpointer(self)
```

获取 checkpointer，用于保存/恢复执行状态。\n+当工作流发生交互中断或异常中断时，checkpointer 会参与持久化与恢复流程。

### actor_manager

```python
actor_manager(self) -> "ActorManager"
```

返回执行参与者管理器。\n+目前主要由框架内部使用，用于管理参与执行的各种“actor”（例如调度器、worker 等），一般业务开发者可以忽略。

### close

```python
async close(self) -> None
```

关闭底层会话资源。\n+通常由框架在执行结束时调用，用于释放连接、刷写缓冲区等；业务代码一般不需要显式调用。

## class openjiuwen.core.session.session.Session

`Session` 是面向组件开发者的会话抽象接口（更高层），提供统一的状态读写、流式输出、trace 与交互能力。\n+工作流组件的 `invoke/stream/collect/transform` 运行时会拿到一个 `Session`（例如 `openjiuwen.core.session.node.Session`），并通过它读写状态。

### executable_id

```python
executable_id(self) -> str
```

返回当前正在执行的“可执行单元”id（通常是组件/节点 id）。\n+常见用途：

- 作为交互输出 `InteractionOutput.id` 的来源（标识是哪一个节点发起交互）
- 作为 trace/span 的归属 id
- 作为节点状态命名空间的关键索引

### session_id

```python
session_id(self) -> str
```

返回会话 id（与 `BaseSession.session_id()` 含义一致）。

### update_state

```python
update_state(self, data: dict)
```

更新**组件私有状态**（仅当前节点/组件可见）。

**参数**：

- **data** (dict)：要更新的私有状态键值对。\n  - 建议用于保存组件内部计数、阶段性中间结果、组件缓存等。\n  - key 冲突时覆盖旧值。\n  - 写入后对当前组件后续执行可见；对其他组件不可见。

### get_state

```python
get_state(self, key: str | list | dict = None) -> Any
```

读取**组件私有状态**。

**参数**：

- **key** (str | list | dict, 可选)：要读取的状态 key。\n  - 传 `str` 时读取单个 key。\n  - 传 `list/dict` 时可用于描述嵌套路径（是否支持取决于底层实现）。\n  - 不传时通常表示读取全部私有状态（是否返回完整 dict 取决于实现）。

**返回**：

- **Any**：对应 key 的值；不存在时通常为 `None`。

### update_global_state

```python
update_global_state(self, data: dict)
```

更新**全局状态**（工作流内共享）。

**参数**：

- **data** (dict)：要更新的全局状态键值对。\n  - 适合多个组件需要共享的数据，例如调试轨迹、全局累加器、任务上下文等。\n  - key 冲突时覆盖旧值。\n  - 写入后对后续组件可见（取决于执行与 commit 时机）。

### get_global_state

```python
get_global_state(self, key: str | list | dict = None) -> Any
```

读取**全局状态**（工作流内共享）。

**参数**：

- **key** (str | list | dict, 可选)：要读取的 key。\n  - 传 `str` 时读取单个 key。\n  - 传 `list/dict` 时可用于描述嵌套路径（是否支持取决于底层实现）。\n  - 不传时通常表示读取全部全局状态（是否返回完整 dict 取决于实现）。

**返回**：

- **Any**：对应 key 的值；不存在时通常为 `None`。

### write_stream / write_custom_stream

```python
async write_stream(self, data: dict | OutputSchema)
async write_custom_stream(self, data: dict)
```

向输出流写入结构化帧。\n+用于把组件执行过程中的中间信息实时推送到上层（例如 UI、日志、外部消费端）。

**参数**：

- **data**：\n  - `write_stream`：建议传 `OutputSchema`（标准化结构化帧）。如果传 dict，具体封装规则取决于实现。\n  - `write_custom_stream`：传 dict，用于自定义结构化输出（上层可按业务协议解析与展示）。

### trace / trace_error

```python
async trace(self, data: dict)
async trace_error(self, error: Exception)
```

记录 trace 信息与错误信息。\n+用于埋点、链路追踪、异常定位等场景。

**参数**：

- **data** (dict)：trace 数据。\n  - 建议包含事件名、阶段、耗时、关键输入摘要等信息。\n  - 避免写入敏感信息（如密钥、隐私数据）。
- **error** (Exception)：异常对象。\n  - 建议在捕获异常后调用，让 tracer 记录完整错误上下文。

### interact

```python
async interact(self, value)
```

发起交互并等待用户输入（由上层恢复执行时提供）。\n+常用于“缺少关键信息需要用户补全”的场景。

**参数**：

- **value**：交互提示内容。\n  - 可以是字符串（最常见），用于向用户提问。\n  - 也可以是结构化对象（例如 dict），用于携带更多上下文（由上层 UI 决定如何展示）。

**返回**：

- 用户输入（类型取决于上层交互输入，可能是 str/dict/任意对象）。

### user_id

```python
user_id(self) -> str
```

返回当前会话关联的用户 id。\n+默认实现返回空字符串；如需区分不同终端用户，可在上层注入自定义实现。

### stream_writer / custom_writer

```python
stream_writer(self) -> Optional[StreamWriter]
custom_writer(self) -> Optional[StreamWriter]
```

获取标准输出流 writer 与自定义输出流 writer。\n+通常不直接使用，而是通过 `write_stream` / `write_custom_stream` 进行写入；少数高级场景可直接拿到 writer 做更细粒度控制。

### get_workflow_config

```python
get_workflow_config(self, workflow_id)
```

按 workflow_id 读取工作流配置。\n+底层会委托给 `BaseSession.config().get_workflow_config(workflow_id)`。

**参数**：

- **workflow_id**：工作流 id。\n  - 必填；为空会在底层触发异常。

### get_agent_config

```python
get_agent_config(self)
```

获取当前会话绑定的 agent 配置对象。\n+常用于在组件内部按配置调整行为（如启用某些调试/观测开关）。

### get_env

```python
get_env(self, key) -> Optional[Any]
```

读取指定环境变量。\n+等价于从 `Config` 中读取对应 key。

**参数**：

- **key**：环境变量名称。\n  - 常用于读取执行超时、循环次数上限、功能开关等运行参数。

### base

```python
base(self) -> BaseSession
```

返回底层 `BaseSession` 对象。\n+仅在需要访问更底层能力（例如 tracer、原始 checkpointer、原生状态容器）时使用，普通组件开发一般不需要调用。

### pre_run / post_run / release

```python
async pre_run(self, **kwargs) -> None
async post_run(self) -> None
async release(self, session_id: str) -> None
```

执行前置/后置钩子与会话释放逻辑。\n+这些方法由框架在合适时机调用用于资源准备与清理，业务代码通常不需要直接调用；只有在自定义复杂执行框架时才需要关心。

## class openjiuwen.core.session.session.ProxySession

`ProxySession` 是 `BaseSession` 的代理实现。\n+它允许在运行期替换内部 stub，并将接口调用透传给真实 `BaseSession`，常用于“先占位、后注入真实会话实现”的场景。

