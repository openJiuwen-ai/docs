# openjiuwen.core.session.workflow

## class openjiuwen.core.session.workflow.Session

`openjiuwen.core.session.workflow.Session` 是工作流调用 `Workflow.invoke()` 时使用的会话对象（“工作流会话入口”），负责：

- **生成/保存 `session_id`**：用于把一次执行过程与状态/输出/trace 关联起来。
- **维护 envs**：用于把运行参数传递到执行链路（例如超时、开关、循环上限等）。
- **维护 callback_manager**：用于在执行过程中触发回调（观测/调试/扩展点）。
- **绑定 `WorkflowCard`**：用于执行期间识别当前工作流（在 `Workflow.invoke()` 内部设置）。

### get_session_id

```python
get_session_id(self) -> str
```

获取会话 ID。

**返回**：

- **str**：会话 id。\n  - 若由 Agent 会话派生，则通常与 Agent 会话共享同一个 id。\n  - 若未显式传入，则会自动生成（实现细节由创建函数决定）。

### get_envs

```python
get_envs(self)
```

获取环境变量字典。

**返回**：

- **dict[str, Any] | None**：环境变量字典。\n  - 常用于传递运行参数（超时/开关等）与外部配置（如 endpoint、region 等）。\n  - 上层读取通常通过 `Config.get_env(key)` 完成。

### set_workflow_card / get_workflow_card

```python
set_workflow_card(self, card)
get_workflow_card(self)
```

设置/获取当前工作流的 card（执行过程中由 `Workflow.invoke()` 设置）。

**参数**：

- **card**：工作流 card 对象。\n  - 通常包含 workflow 的 id/name/version/描述/输入输出 schema 等元信息。\n  - 执行期用来识别“当前正在运行的是哪个工作流”。

## function openjiuwen.core.session.workflow.create_workflow_session

```python
create_workflow_session(parent: \"AgentSession\" = None, session_id: str = None, envs: dict[str, Any] = None) -> Session
```

创建工作流会话对象。\n+典型用于在调用工作流前准备会话上下文，并在整个执行期间承载状态与运行参数。

**参数**：

- **parent** (\"AgentSession\", 可选)：父会话（通常来自 `openjiuwen.core.session.agent.Session`）。\n  - 传入后：会话 id/envs 会继承自 parent，确保同一执行链路上下文一致。\n  - 适用于“Agent 内部调用 workflow”的场景。
- **session_id** (str, 可选)：显式指定会话 id。\n  - 不传时会自动生成。\n  - 需要“中断后恢复/继续执行”的场景，应复用同一个 session_id（由上层/调用方保证）。
- **envs** (dict[str, Any], 可选)：执行期环境变量。\n  - 用于传递运行参数与外部配置。\n  - 会与内置默认参数合并；相同 key 将覆盖默认值。

**返回**：

- **Session**：工作流会话对象。

**用例**：

```python
from openjiuwen.core.workflow import Workflow
from openjiuwen.core.session.workflow import create_workflow_session

flow = Workflow()
session = create_workflow_session(envs={"WORKFLOW_EXECUTE_TIMEOUT": 60})
await flow.invoke({\"user_inputs\": {...}}, session)
```

