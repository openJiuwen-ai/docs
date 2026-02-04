# openjiuwen.core.application.workflow_agent.workflow_agent

## class openjiuwen.core.application.workflow_agent.workflow_agent.WorkflowAgent

```
class openjiuwen.core.application.workflow_agent.workflow_agent.WorkflowAgent(agent_config: WorkflowAgentConfig)
```

基于多工作流控制器的 Agent，实现预定义工作流的执行，继承 `ControllerAgent`。

### async invoke(inputs: Dict, session: Session = None) -> Dict

同步（非流式）执行入口：直接调用 `ControllerAgent.invoke`，由内部的 `WorkflowController` 执行工作流并返回结果。

**参数：**

* **inputs**(Dict)：输入数据，通常包含 `query`、`conversation_id` 等。
* **session**(Session, 可选)：会话上下文；不传则自动创建。

**返回：**

* **Dict**：工作流执行结果。

### async stream(inputs: Dict, session: Session = None) -> AsyncIterator[Any]

流式执行入口：委托 `ControllerAgent.stream`，由 `WorkflowController` 执行并逐块产出流式输出。

**参数：**

* **inputs**(Dict)：输入数据。
* **session**(Session, 可选)：会话上下文；不传则自动创建。

**返回：**

* **AsyncIterator[Any]**：流式输出迭代器。
