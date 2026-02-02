# openjiuwen.core.controller

`openjiuwen.core.controller` 提供了一套 **新版 Agent 控制器及其任务/事件/意图建模** 的公共类型。

## class TextDataFrame / FileDataFrame / JsonDataFrame / DataFrame
```python
class openjiuwen.core.controller.schema.TextDataFrame
class openjiuwen.core.controller.schema.FileDataFrame
class openjiuwen.core.controller.schema.JsonDataFrame
class openjiuwen.core.controller.schema.DataFrame
```
- 统一的输入/输出载荷模型：文本、文件、JSON 及基类。
- 事件与控制器输出均以 DataFrame 形式封装，便于序列化、校验与多模态扩展。

## class Event / EventType / InputEvent / TaskInteractionEvent / TaskCompletionEvent / TaskFailedEvent
```python
class openjiuwen.core.controller.schema.Event
class openjiuwen.core.controller.schema.EventType
class openjiuwen.core.controller.schema.InputEvent
class openjiuwen.core.controller.schema.TaskInteractionEvent
class openjiuwen.core.controller.schema.TaskCompletionEvent
class openjiuwen.core.controller.schema.TaskFailedEvent
```

- **EventType**：input / task_interaction / task_completion / task_failed。
- **Event**：事件基类，含 `event_type`、`event_id`、`metadata`。
- **InputEvent**：用户输入事件；`from_user_input` 支持 str/dict/InputEvent 快速构造。
- **TaskInteractionEvent / TaskCompletionEvent / TaskFailedEvent**：执行过程中的交互、成功、失败事件，携带任务上下文与结果/错误信息。

## class ControllerOutputPayload / ControllerOutputChunk / ControllerOutput
```python
class openjiuwen.core.controller.schema.ControllerOutputPayload
class openjiuwen.core.controller.schema.ControllerOutputChunk
class openjiuwen.core.controller.schema.ControllerOutput
```

- 流式输出与批量输出模型。
- `ControllerOutputChunk`：type 固定 `controller_output`，包含 `payload`、`index`、`last_chunk`。
- `ControllerOutputPayload.type` 支持 task_completion / task_interaction / task_failed / processing / all_tasks_processed；`ControllerOutput` 用于一次性返回。

## class Intent / IntentType
```python
class openjiuwen.core.controller.schema.Intent
class openjiuwen.core.controller.schema.IntentType
```

- **IntentType**：枚举类型，表示不同的意图类别（如任务执行、信息查询等）。
- **Intent**：单个意图对象，描述“用户想做什么”，通常由 `IntentDetector` 从输入中识别产生。

常见字段（以源码为准）：

- `type: IntentType`：意图类型；
- `content: str`：意图文本或摘要；
- 其他与上下文、置信度相关的字段。

## class Task / TaskStatus
```python
class openjiuwen.core.controller.schema.Task
class openjiuwen.core.controller.schema.TaskStatus
```

- **Task**：任务模型，记录 `session_id`、`task_id`、`task_type`、`priority`、`inputs`、`outputs (List[ControllerOutputChunk])`、父子关系、错误信息等。
- **TaskStatus**：submitted / working / input-required / completed / failed / paused / canceled / waiting / unknown。

## class EventHandlerInput / EventHandler
```python
class openjiuwen.core.controller.modules.EventHandlerInput
class openjiuwen.core.controller.modules.EventHandler
```

- `EventHandlerInput`：事件回调入参包装（event + session）。
- `EventHandler`：事件处理抽象，需要实现 input / task_interaction / task_completion / task_failed 四类处理方法。

## class EventQueue
```python
class openjiuwen.core.controller.modules.EventQueue
```

- 基于内存消息队列的发布/订阅器；topic `{agent_id}_{session_id}_{event_type}`；负责调用对应 `EventHandler` 方法。

## class TaskManagerState / TaskManager / TaskFilter
```python
class openjiuwen.core.controller.modules.TaskManagerState
class openjiuwen.core.controller.modules.TaskManager
class openjiuwen.core.controller.modules.TaskFilter
```

- `TaskManager`：任务存储与索引，支持 CRUD、优先级、父子关系管理。
- `TaskManagerState`：用于任务状态持久化/恢复。
- `TaskFilter`：组合过滤条件（需至少指定 task_id / session_id / priority / status / is_root 等之一）。

## class TaskExecutor / TaskExecutorRegistry / TaskScheduler
```python
class openjiuwen.core.controller.modules.TaskExecutor
class openjiuwen.core.controller.modules.TaskExecutorRegistry
class openjiuwen.core.controller.modules.TaskScheduler
```

- `TaskExecutor`：任务执行器抽象。
- `TaskExecutorRegistry`：按 task_type 注册/移除/获取执行器。
- `TaskScheduler`：从队列取任务并分发到执行器，维护会话映射与调度循环。

## class IntentRecognizer
```python
class openjiuwen.core.controller.modules.IntentRecognizer
```

- 可插拔的意图识别组件，可在事件处理链路中提升路由准确性。

## class EventHandlerWithIntentRecognition
```python
class openjiuwen.core.controller.modules.EventHandlerWithIntentRecognition
```

- 带意图识别的事件处理器封装：先识别 `Intent`，再调用业务处理逻辑，实现“识别+处理”一体化。

## class ControllerConfig
```python
class openjiuwen.core.controller.config.ControllerConfig
```

- 配置项涵盖：调度并发/间隔、任务超时与持久化、事件队列大小/超时、意图识别阈值与类型列表等。

## class Controller
```python
class openjiuwen.core.controller.base.Controller
```
- 新版推荐控制器主体：负责事件订阅、任务调度、流式输出与状态持久化；提供 `init`、`set_event_handler`、`add/remove_task_executor`、`invoke`（批量）与 `stream`（流式）等接口。

