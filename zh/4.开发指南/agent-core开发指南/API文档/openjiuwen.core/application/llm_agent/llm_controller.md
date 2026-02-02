# openjiuwen.core.application.llm_agent.llm_controller

## class openjiuwen.core.application.llm_agent.llm_controller.LLMController

```python
class LLMController(BaseController):
    ...
```

`LLMController` 是 `LLMAgent` 使用的**ReAct 风格控制器**，基于 `BaseController` 实现。它负责：

1. 接收用户输入，调用大模型进行“思考+规划”（生成 Task 列表）；
2. 执行任务（插件 / 工作流），并把结果写入会话流；
3. 在任务完成后再次调用大模型，决定是否继续（循环）或给出最终回答；
4. 在工作流任务中处理中断、恢复执行、长时记忆等。

源码位置：`openjiuwen.core.application.llm_agent.llm_controller.LLMController`。

---

### 初始化

```python
def __init__(
    self,
    config: ReActAgentConfig,
    context_engine,
    session,
) -> None:
    ...
```

**参数：**

- **config** (`LegacyReActAgentConfig` as `ReActAgentConfig`)：
  - ReAct 单智能体配置（包含工具、工作流、记忆、约束等）。
- **context_engine**：
  - 上下文引擎实例（通常为 `ContextEngine`），用于读写对话上下文。
- **session** (`Session`)：
  - Agent 级别会话对象，用于：
    - 读取/更新状态（包括中断状态）；
    - 写入流式输出（`write_stream/write_custom_stream`）；
    - 管理调用链路 trace。

内部还会初始化：

- `_enable_memory`：是否启用长期记忆（来自 `config.agent_memory_config.enable_long_term_mem`）。
- `_long_term_memory_instance` (`LongTermMemory`)：长期记忆管理实例（懒初始化）。

---

### handle_event

```python
async def handle_event(self, event: Event, session: Session) -> Optional[Dict]:
    ...
```

**用途：** `BaseController` 消费消息队列时的核心回调，仅处理 `USER_INPUT` 类型事件。

**行为（与源码一致）：**

1. 检查 `event.event_type` 是否为 `EventType.USER_INPUT`：
   - 若不是，记录 warning，并抛出 `AGENT_CONTROLLER_USER_INPUT_PROCESS_ERROR` 对应异常。
2. 调用内部 `_handle_user_input(event, session)` 处理用户输入。
3. 捕获异常：
   - 若是 `JiuWenBaseException`，直接向上抛出；
   - 否则包装为 `AGENT_CONTROLLER_RUNTIME_ERROR` 再抛出。

返回值为 `_handle_user_input` 的结果，一般为 `dict` 或 `None`。

---

### _handle_user_input

```python
async def _handle_user_input(self, event: Event, session: Session) -> Optional[Dict]:
    ...
```

**用途：** ReAct 主循环的入口，负责：

1. 把用户输入加入对话历史；
2. 调用 LLM 生成任务计划（Task 列表）；
3. 执行任务（插件/工作流）；
4. 处理工作流中断与恢复；
5. 组织循环迭代直至结束。

**关键流程：**

1. **写入用户消息：**
   - 调用 `MessageUtils.add_user_message(event.get_display_content(), self._context_engine, session)`，将用户可展示内容加入上下文引擎。

2. **InteractiveInput 快速恢复（工作流中断场景）：**
   - 从 `event.content.interactive_input` 中读取交互输入：
     - 若存在且包含 `user_inputs`：
       - 调用 `_find_interrupted_task_by_node_id(interactive_input, session)` 查找对应中断任务；
       - 找到后：
         - 得到 `ai_message, remaining_tasks, saved_iteration`；
         - 将 `ai_message` 写回上下文：`MessageUtils.add_ai_message(...)`；
         - 将首个 `remaining_tasks[0]` 设为 `INTERRUPTED` 并更新 `input.arguments=interactive_input`；
         - 从 `saved_iteration + 1` 开始，调用 `_execute_react_loop(remaining_tasks, session, initial_iteration, ai_message)`；
       - 找不到则记录 warning，退回正常路径。

3. **正常路径：调用 LLM 生成任务计划：**
   - 调用 `tasks, llm_output = await self._generate_plan_from_llm(event, session)`：
     - 由 LLM 输出 ReAct 风格的 `llm_output`（含 tool_calls 等），并解析成结构化的任务列表 `tasks`。
   - 若 `tasks` 为空：
     - 认为是“仅回答、不需要任务执行”的场景；
     - 调用 `_send_final_stream(llm_output.content, session)` 写入最终回答的流式输出；
     - 通过 `_unwrap_result` 返回给上层。

4. **工作流任务中断恢复：**
   - 调用 `_resolve_workflow_from_tasks(tasks)` 判断是否包含工作流任务；
   - 若存在 `workflow_task`：
     - 调用 `_find_interrupted_task(workflow_task, session)` 查找已有中断：
       - 若存在，则解包为 `saved_ai_message, remaining_tasks, saved_iteration, component_ids`；
       - 可能会用新的 `llm_output.tool_calls[0].id` 更新首个任务的 `task_id`，保证工具调用 id 一致；
       - 构造或复用 `InteractiveInput`（可能基于 `component_ids` 与当前用户 query）；
       - 设置 `remaining_tasks[0].input.arguments = interactive_input, status = INTERRUPTED`；
       - 从 `saved_iteration + 1` 开始，调用 `_execute_react_loop(remaining_tasks, session, initial_iteration, ai_message=saved_ai_message)`。

5. **执行新任务：**
   - 若不存在需要恢复的工作流任务：
     - 以 `initial_iteration = 1` 调用 `_execute_react_loop(tasks, session, initial_iteration, ai_message=llm_output)`。

**返回值：**

- 取决于 `_execute_react_loop` 或 `_send_final_stream` 的结果，一般为最终回答或中断结构的字典形式。

> 注意：`_handle_user_input` 为内部方法，主要由 `handle_event` 调用，遵循 `BaseController` 的消息队列模型。

---

### 与 BaseController 的关系

- `LLMController` 继承自 `BaseController`，因此：
  - 通过 `BaseController.invoke(inputs, session)` 暴露统一的调用入口：
    - 从 `inputs["conversation_id"]` 推导会话队列 topic；
    - 封装为 `Event`；
    - 发送到内存消息队列；
    - 由 `_handle_message_wrapper` 调用 `handle_event` 并返回结果。
  - `LLMController.handle_event` 专注处理 `EventType.USER_INPUT`，其他事件类型会被拒绝并抛错。

---

### 依赖与协作组件

`LLMController` 在实现中依赖以下核心组件（文档中只简要列出，详细请参考对应模块的 API 文档）：

- **消息与任务：**
  - `BaseController`：控制器基础抽象，提供消息队列、订阅和 `invoke` 调用入口。
  - `Event / EventType / Task / TaskStatus / TaskResult`（`openjiuwen.core.controller`）：控制器任务与事件模型。
  - `MessageHandlerUtils`、`MessageUtils`：封装 LLM 消息的序列化 / 反序列化 / 存储逻辑。
- **模型与记忆：**
  - `ModelClientConfig / ModelRequestConfig / Model / AssistantMessage / ToolMessage / ToolCall`：大模型调用与消息类型。
  - `LongTermMemory`：长期记忆组件，用于在 ReAct 过程中读写 agent 级记忆。
- **执行与会话：**
  - `Runner`：工作流 / 工具的统一执行器。
  - `Session / InteractiveInput / OutputSchema`：执行期会话、中断交互对象、流式输出结构。
  - `WorkflowExecutionState / WorkflowOutput`：工作流执行状态与输出封装。

---

### 文档与源码的一致性说明

- 本文档仅描述源码中实际存在的方法与职责：
  - `__init__(config, context_engine, session)`、`handle_event(event, session)`、`_handle_user_input(event, session)` 等核心接口的签名和含义均与源码匹配。
  - 控制器使用的是 **`Session` + `Runner` + `ContextEngine` + `LongTermMemory`** 这一套 0.1.4 的设计，不依赖已删除的 `Runtime/WorkflowRuntime/ModelFactory` 等旧接口。
- 对于内部私有方法（如 `_execute_react_loop`、`_generate_plan_from_llm` 等），本文仅在行为描述中引用，不虚构额外参数或返回值，保持与实现一致。**

