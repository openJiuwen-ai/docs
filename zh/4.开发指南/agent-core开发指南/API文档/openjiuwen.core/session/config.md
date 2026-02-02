# openjiuwen.core.session.config

## class openjiuwen.core.session.config.base.Config

`Config` 用于会话级配置管理（envs、工作流配置、agent 配置等）。`Workflow.invoke()` / `Runner` 等执行链路会通过 `Config` 读取环境变量与运行参数。

### __init__

```python
Config()
```

初始化时会加载一组**内置运行参数默认值**，并尝试从环境变量与会话变量中覆盖（例如工作流超时、流式超时、循环上限等）。

### set_envs

```python
set_envs(self, envs: dict[str, Any]) -> None
```

设置（合并）环境变量字典。

**参数**：

- **envs** (dict[str, Any])：需要写入的环境变量键值对。\n  - 传入的对象必须是 dict；非 dict 会被忽略。\n  - 会与已有 env 合并：相同 key 将覆盖旧值，不同 key 将追加。

### get_env

```python
get_env(self, key: str, default: Any = None) -> Optional[Any]
```

读取指定 key 的环境变量。

**参数**：

- **key** (str)：要读取的环境变量名称。\n  - 一般用于读取执行期参数（例如各类 timeout、开关、循环上限等）。\n  - key 不存在时不会抛异常，会返回 default。
- **default** (Any, 可选)：当 key 不存在时返回的默认值。\n  - 如果不传，默认返回 `None`。

**返回**：

- **Optional[Any]**：key 对应的值，或 default。

### add_workflow_config

```python
add_workflow_config(self, workflow_id, workflow_config)
```

注册某个 workflow 的配置对象，供执行期通过 `get_workflow_config(workflow_id)` 读取。

**参数**：

- **workflow_id**：工作流 id。\n  - 不能为空；为空会抛出 `JiuWenBaseException`。\n  - 建议使用工作流 card 的 id，确保全局唯一。
- **workflow_config**：工作流配置对象。\n  - 不能为空；为空会抛出 `JiuWenBaseException`。

### get_workflow_config

```python
get_workflow_config(self, workflow_id)
```

按 workflow_id 获取已注册的 workflow 配置对象。

**参数**：

- **workflow_id**：工作流 id。\n  - 不能为空；为空会抛出 `JiuWenBaseException`。

**返回**：

- workflow_config：若未注册可能返回 `None`（取决于上层如何使用）。

### set_agent_config / get_agent_config

```python
set_agent_config(self, agent_config)
get_agent_config(self)
```

设置/获取 agent 配置对象。

## variable openjiuwen.core.session.config.base.workflow_session_vars

`workflow_session_vars` 是会话变量集合（用于 workflow session 环境变量管理）。它基于 `contextvars` 实现，允许在同一进程/协程上下文中覆盖 env 配置（例如超时/开关），并在创建 `Config` 时被读取。

