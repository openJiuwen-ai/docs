# openjiuwen.core.session.wrapper

## class openjiuwen.core.session.internal.wrapper.WrappedSession

`WrappedSession` 为 `Session` 的包装基类，用于把底层 `BaseSession` 适配为更高层 `Session` 接口（对外暴露状态/流式/trace/交互等统一能力）。

### __init__

```python
WrappedSession(inner: BaseSession)
```

创建包装会话。

**参数**：

- **inner** (BaseSession)：底层会话实现。\n  - `WrappedSession` 会把 env/workflow_config/agent_config 的读取委托给 `inner.config()`。\n  - 状态/流式/trace/交互等能力则由具体子类决定如何适配。

### get_workflow_config

```python
get_workflow_config(self, workflow_id)
```

获取某个 workflow 的配置对象（透传到底层 `Config`）。

**参数**：

- **workflow_id**：工作流 id。\n  - 不能为空；为空时底层会抛出 `JiuWenBaseException`。

### get_agent_config

```python
get_agent_config(self)
```

获取 agent 配置对象（透传到底层 `Config`）。

### get_env

```python
get_env(self, key) -> Optional[Any]
```

读取执行期环境变量（透传到底层 `Config.get_env`）。

**参数**：

- **key**：环境变量 key。\n  - 常用于读取超时、开关、循环上限等运行参数。

**返回**：

- **Optional[Any]**：key 对应的值，或 `None`。

### base

```python
base(self) -> BaseSession
```

返回底层会话对象，便于在需要时访问底层能力（例如 tracer、checkpointer、resource_manager 等）。

## class openjiuwen.core.session.internal.wrapper.StaticWrappedSession

`StaticWrappedSession` 是静态包装会话的抽象基类。

### 设计意图

`StaticWrappedSession` 作为抽象基类，定义了 `Session` 的核心接口形态（executable_id、state/global_state、stream、trace、interact 等），但并不提供具体实现。\n+框架内部会按不同执行场景提供具体子类（例如节点会话、路由会话、task 会话等）。

## class openjiuwen.core.session.internal.wrapper.RouterSession

`RouterSession` 是路由节点会话的包装实现。

### 典型用途

路由节点往往不直接执行业务逻辑，而是根据条件选择下游分支。\n+`RouterSession` 的职责是：

- 在路由节点场景下提供最小必要的 `Session` 能力\n  - 例如 trace 错误上报与必要的状态读写\n- 避免在路由节点上发生不必要的 stream/state 写入（具体取决于实现）

