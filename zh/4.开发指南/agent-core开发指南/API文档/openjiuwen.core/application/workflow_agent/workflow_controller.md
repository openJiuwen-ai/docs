# openjiuwen.core.application.workflow_agent.workflow_controller

## class openjiuwen.core.application.workflow_agent.workflow_controller.WorkflowController

```python
class WorkflowController(IntentDetectionController):
    ...
```

`WorkflowController` 是面向 `WorkflowAgent` 的**工作流控制器**，负责把「用户输入 → 意图判断 → 选择/恢复工作流 → 调用 Runner 执行」这一整条链路串起来，并在中断场景下管理任务状态。

源码位置：`openjiuwen.core.application.workflow_agent.workflow_controller.WorkflowController`。

### 核心职责

1. **意图识别（intent_detection）**：
   - 根据 `Event` 内容和当前 Agent 中配置的 `workflows`，选择要执行的工作流。
   - 支持多工作流场景下通过 LLM 做工作流选择；无匹配时返回默认回复。
   - 识别是否存在需要恢复的中断任务（包括基于 `InteractiveInput` 的快速恢复）。
2. **任务执行（exec_task）**：
   - 新任务：调用 `Runner.run_workflow_streaming` 执行工作流。
   - 中断任务：基于 `InteractiveInput` 与保存的中断状态恢复执行。
   - 通过 `Session` 写入流式输出（`OutputSchema/CustomSchema`），并构造 `WorkflowOutput`。
3. **中断管理**：
   - 使用内部 `task_queue` 管理当前会话的运行任务。
   - 在中断时将中断信息写入 `Session` 状态（默认 key 为 `"workflow_controller"`），支持后续恢复。
   - 支持“仅返回中断信息、暂不继续执行”的模式（`metadata["return_interruption"]`）。

---

### 初始化

```python
def __init__(
    self,
    config: AgentConfig | None = None,
    context_engine=None,
    session=None,
) -> None:
    ...
```

**参数：**

- **config** (`AgentConfig | None`)：
  - 单智能体配置（`WorkflowAgent` 的 AgentConfig）。
  - 未传入时，会在 `ControllerAgent.setup_from_agent` 阶段注入。
- **context_engine**：
  - 上下文引擎实例（通常为 `ContextEngine`）。
  - 未传入时，同样由 `ControllerAgent` 注入。
- **session**：
  - Agent 级别的 `Session` 实例。
  - 主要用于：
    - 追踪对话会话（`session.session_id()`）；
    - 管理流式输出（`write_stream/write_custom_stream`）；
    - 存取中断相关状态（`get_state/update_state` 等）。

> 说明：如果构造时参数为空，`WorkflowController` 仍然可以通过 `ControllerAgent.setup_from_agent(agent)` 在运行前完成依赖注入。

---

### intent_detection

```python
async def intent_detection(
    self,
    event: Event,
    session: Session,
) -> Intent:
    ...
```

**用途：** 根据当前输入事件选择工作流，并判断是**新建任务**还是**恢复任务**。

**关键行为（与源码一致）：**

1. **获取工作流列表：**
   - 从 `self.agent_config.workflows` 读取 `WorkflowSchema` 列表；
   - 若列表为空，抛出 `ValueError("No workflows configured for single_agent")`。
2. **基于 `InteractiveInput` 的快速恢复：**
   - 若 `event.content.interactive_input` 不为空且包含 `user_inputs`：
     - 调用 `_find_interrupted_task_by_node_id(interactive_input, session)`。
     - 找到匹配的中断任务后，直接返回：
       ```python
       Intent(
           intent_type=IntentType.ResumeTask,
           task=task,
           workflow=workflow,
       )
       ```
3. **工作流选择：**
   - **单工作流模式**：若只配置一个工作流，直接选用该工作流。
   - **多工作流模式**：调用 `_detect_workflow_via_llm(event, session)` 使用 LLM 进行 workflow 选择。
     - 若返回 `None`，则使用 `self.agent_config.default_response.text` 生成 `IntentType.DefaultResponse`：
       ```python
       Intent(
           intent_type=IntentType.DefaultResponse,
           metadata={"default_response_text": default_text},
       )
       ```
4. **中断任务检测：**
   - 对选中的工作流调用 `_find_interrupted_task(detected_workflow, session)`。
   - 若存在中断任务：
     - 先调用 `_should_resume_interrupted_task(...)` 决定是恢复执行，还是仅返回中断信息：
       - **需要恢复**：返回 `IntentType.ResumeTask`，`task` 为中断任务；
       - **只返回中断**：同样返回 `IntentType.ResumeTask`，但在 `metadata` 中加入 `{"return_interruption": True}`。
   - 若不存在中断任务：
     - 通过 `_create_new_task(event, detected_workflow)` 创建新的 `Task`；
     - 返回 `IntentType.ExecNewTask`。

**返回值：**

- **`Intent`**：包含以下信息：
  - `intent_type`：`ExecNewTask` / `ResumeTask` / `DefaultResponse` 等。
  - `task`：要执行或要恢复的任务（对于 `DefaultResponse` 可为空）。
  - `workflow`：选中的 `WorkflowSchema`。
  - `metadata`：附加控制信息，例如 `{"return_interruption": True}`。

---

### exec_task

```python
async def exec_task(
    self,
    message_content: EventContent,
    task: Task,
    session: Session,
) -> Dict:
    ...
```

**用途：** 执行或恢复单个工作流任务，并处理中断/完成等状态。

**执行流程（与源码一致）：**

1. **基础信息：**
   - 从 `task.input.target_id` 读取 `workflow_id`；
   - 从 `session.session_id()` 读取 `conversation_id`。
2. **检查并取消会话内旧任务：**
   - 若 `self.task_queue.has_running_task(conversation_id)` 为真：
     - 调用 `cancel_running_task(conversation_id)` 取消旧任务；
     - 通过 `self.task_queue.find_task(conversation_id)` 查到旧任务信息，并：
       - 构造一个 `TaskStatus.CANCELLED` 的临时 `Task`；
       - 调用 `_clear_interrupted_state(temp_task, session)` 清理旧的中断状态。
3. **标记任务为运行中：**
   - `task.status = TaskStatus.RUNNING`。
4. **查找工作流对象：**
   - 调用 `_find_workflow_from_agent(workflow_id, session)` 获取真正的 `Workflow` 实例；
   - 若找不到，抛出 `ValueError(f"Workflow not found: {workflow_id}")`。
5. **创建工作流 Session：**
   - 通过 Agent 级 Session 创建工作流级别会话：
     ```python
     workflow_session = session.create_workflow_session()
     ```
6. **准备输入参数：**
   - `inputs = task.input.arguments`；
   - 若是恢复任务（`TaskStatus.INTERRUPTED`），此处通常为 `InteractiveInput`。
7. **调用 Runner 以流式方式执行工作流：**
   - 内部定义 `run_workflow_streaming()` 协程：
     ```python
     workflow_stream = Runner.run_workflow_streaming(
         workflow,
         inputs=inputs,
         session=workflow_session,
         context=await self._context_engine.create_context(
             context_id=workflow_id,
             session=session,
         ),
     )
     ```
   - 对流中的每个 `chunk`：
     - 若是 `OutputSchema`：
       - `type == "__interaction__"`：标记有中断，但不立刻透传，由 Controller 统一处理；
       - `type == "workflow_final"`：视为工作流最终结果，保存在 `final_result`，并 `await session.write_stream(chunk)` 透传；
       - 其他类型：直接 `await session.write_stream(chunk)`（例如 trace 等）。
     - 若是 `CustomSchema`：调用 `await session.write_custom_stream(chunk)`。
   - 最后根据是否存在中断构造 `WorkflowOutput`：
     - 有中断：`WorkflowOutput(result=chunks, state=WorkflowExecutionState.INPUT_REQUIRED)`；
     - 无中断：`WorkflowOutput(result=final_result, state=WorkflowExecutionState.COMPLETED)`。
8. **注册并等待任务完成：**
   - 使用 `asyncio.create_task(run_workflow_streaming())` 创建任务；
   - 通过 `self.task_queue.register_task(conversation_id, task, workflow_task, target_id=workflow_id)` 注册；
   - 等待 `workflow_task` 完成，或捕获 `CancelledError`。
9. **结果处理：**
   - 无论成功与否，都会在 `finally` 中调用 `self.task_queue.unregister_task(conversation_id)` 解除注册。
   - 对于正常完成结果：
     - 通过 `_is_workflow_interrupted(result)` 判断是否为中断：
       - 若中断：
         - 设置 `task.status = TaskStatus.INTERRUPTED`；
         - 从 `result.result` 中抽取中断列表，调用 `interrupt_task(task, session, interaction_data)` 保存所有中断；
         - 通过 `_get_first_interrupt` 取第一个中断作为本次返回值（流式场景下仅返回首个中断）。
       - 若完成：
         - 设置 `task.status = TaskStatus.SUCCESS`；
         - 调用 `_clear_interrupted_state(task, session)` 清理中断状态；
         - 返回 `{"output": result, "result_type": "answer"}`。
   - 对于执行异常：
     - 记录日志，设置 `task.status = TaskStatus.FAILED`，并重新抛出异常。
   - 对于取消场景：
     - 设置 `task.status = TaskStatus.CANCELLED`，返回包含 `status/task_id/workflow_id` 的字典。

**返回值：**

- **`dict`**：
  - 正常完成时：`{"output": WorkflowOutput(...), "result_type": "answer"}`；
  - 中断时：返回第一个中断对应的结构（用于前端展示交互）；
  - 取消时：`{"status": "cancelled", "task_id": ..., "workflow_id": ...}`。

---

### 与文档的对齐说明

- `WorkflowController` 依赖的是 **`Session` + `Runner.run_workflow_streaming` + `ContextEngine.create_context`** 这一套新接口，不再使用任何 `Runtime/WorkflowRuntime` 相关类型。
- 控制器对外主要暴露两个协程方法：
  - `intent_detection(event, session)`：选择/恢复工作流，返回 `Intent`；
  - `exec_task(message_content, task, session)`：执行或恢复任务，返回执行结果或中断信息。
- 本文档中的方法签名、参数命名、状态流转描述，均以 0.1.4 源码为准，确保与 `openjiuwen.core.application.workflow_agent.workflow_controller.WorkflowController` 的实现保持一致。**

