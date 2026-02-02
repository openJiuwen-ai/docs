# openjiuwen.core.application.workflow_agent.workflow_agent

## class openjiuwen.core.application.workflow_agent.workflow_agent.WorkflowAgent

```python
class WorkflowAgent(ControllerAgent):
    ...
```

`WorkflowAgent` 面向“预定义工作流”场景，继承 `ControllerAgent`，内部持有 `WorkflowController`，将意图识别、工作流选择/恢复与执行全权交给控制器。适合需要稳定、可重复编排的流程式任务。

- 基于消息队列模型处理输入/输出；
- 支持多工作流选择、工作流中断恢复（由 `WorkflowController` 负责）；
- 代码极简，仅做参数校验与调用委托。

源码位置：`openjiuwen.core.application.workflow_agent.workflow_agent.WorkflowAgent`。

---

### 初始化

```python
def __init__(self, agent_config: WorkflowAgentConfig):
    ...
```

**参数与约束：**
- `agent_config`：`WorkflowAgentConfig`，要求 `controller_type == ControllerType.WorkflowController`，否则抛出 `NotImplementedError`。

**内部注入：**
- 创建 `WorkflowController()`，交给父类 `ControllerAgent` 进行后续注入（上下文、会话、配置等在 `setup_from_agent` 时完成）。

---

### invoke

```python
async def invoke(self, inputs: Dict, session: Session | None = None) -> Dict:
    ...
```

**用途：** 非流式调用。完全委托 `ControllerAgent.invoke`（底层走 `WorkflowController.invoke`），常用于一次性拿到完整执行结果或中断信息。

**典型输入字段：**
- `query`：用户请求（可选，若工作流需要纯结构化入参可自定义）。
- `conversation_id`：会话标识；未提供时由控制器默认生成。

返回值由控制器决定，通常为包含最终输出或中断描述的字典。

---

### stream

```python
async def stream(self, inputs: Dict, session: Session | None = None) -> AsyncIterator[Any]:
    ...
```

**用途：** 流式执行，逐块产出执行过程（`OutputSchema` 或自定义字典块）。内部完全沿用 `ControllerAgent.stream` 的逻辑：创建/复用 Session，委托 `WorkflowController` 写入会话流，再逐块向调用方 `yield`。

适合长耗时或需中途交互的工作流（如表单补全、人工确认等）。

**示例：**
```python
inputs = {"conversation_id": "wf-001", "query": "启动文档审阅流程"}
async for chunk in agent.stream(inputs):
    print(chunk)
```

---

### 适用场景与注意事项
- 预先在 `WorkflowAgentConfig.workflows` 中注册 `WorkflowSchema`；未配置工作流时控制器会抛出错误。
- 多工作流场景的选择、中断恢复、默认回复等行为由 `WorkflowController` 处理（详见对应文档）。
- 若需工具/插件执行，请在工作流节点中配置；`WorkflowAgent` 本身不直接管理工具列表。 
