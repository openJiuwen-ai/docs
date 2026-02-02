# openjiuwen.core.controller

`openjiuwen.core.controller` 模块提供了一套 **老版 Agent 控制器及其任务/事件/意图建模** 的公共类型，这些类型目前主要通过 `legacy` 子模块实现，并在 `__init__.py` 中统一导出，用于兼容历史代码以及为高层控制器（如 `LLMController`）提供任务/事件模型的基础。

## class BaseController

```python
class openjiuwen.core.controller.legacy.BaseController
```

控制器抽象基类，负责：

- 接收外部事件（`Event`）；
- 维护内部任务队列（`TaskQueue`）；
- 调度任务执行（`Task`）。

> **说明**：在 0.1.4 中，更推荐直接使用 `openjiuwen.core.application.llm_agent.LLMController`、`openjiuwen.core.application.workflow_agent.WorkflowController` 等高层控制器；`BaseController` 主要用于理解和兼容旧有控制器体系。

## class Intent / IntentType

```python
class openjiuwen.core.controller.legacy.Intent
class openjiuwen.core.controller.legacy.IntentType
```

- **IntentType**：枚举类型，表示不同的意图类别（如任务执行、信息查询等）。
- **Intent**：单个意图对象，描述“用户想做什么”，通常由 `IntentDetector` 从输入中识别产生。

常见字段（以源码为准）：

- `type: IntentType`：意图类型；
- `content: str`：意图文本或摘要；
- 其他与上下文、置信度相关的字段。

## class Task / TaskInput / TaskStatus / TaskResult / TaskQueue

```python
class openjiuwen.core.controller.legacy.Task
class openjiuwen.core.controller.legacy.TaskInput
class openjiuwen.core.controller.legacy.TaskStatus
class openjiuwen.core.controller.legacy.TaskResult
class openjiuwen.core.controller.legacy.TaskQueue
```

这些类型共同完成“任务”的生命周期建模：

- **TaskInput**：任务输入结构，封装原始请求、上下文等；
- **Task**：单个任务对象，包含 `TaskInput`、任务 id、当前状态等；
- **TaskStatus**：任务状态枚举，如 PENDING/RUNNING/COMPLETED/FAILED；
- **TaskResult**：任务执行结果对象，封装最终输出及可能的错误信息；
- **TaskQueue**：任务队列，内部维护一组待执行/执行中的任务。

在高层控制器（例如 `LLMController`）中，通常会：

- 从对话/输入中抽取或构造 `TaskInput`；
- 生成一个或多个 `Task` 并入队；
- 根据执行结果构造 `TaskResult` 返回给上层。

## class Event 及相关类型

```python
class openjiuwen.core.controller.legacy.Event
class openjiuwen.core.controller.legacy.EventType
class openjiuwen.core.controller.legacy.EventPriority
class openjiuwen.core.controller.legacy.EventSource
class openjiuwen.core.controller.legacy.EventContent
class openjiuwen.core.controller.legacy.EventContext
class openjiuwen.core.controller.legacy.SourceType
```

事件模型用于描述“控制器在什么时候、因为什么原因被触发”：

- **EventType**：事件类型（如用户输入事件、系统回调事件等）；
- **EventPriority**：事件优先级；
- **EventSource/SourceType**：事件来源；
- **EventContent**：事件内容载体（原始文本、结构化 payload 等）；
- **EventContext**：事件上下文（如关联的会话、任务 id 等）；
- **Event**：综合上述信息的事件对象，作为控制器的主要输入。

## IntentDetectionController / IntentDetector / Planner 及配置

```python
class openjiuwen.core.controller.legacy.IntentDetectionController
class openjiuwen.core.controller.legacy.IntentDetector
class openjiuwen.core.controller.legacy.Planner
class openjiuwen.core.controller.legacy.IntentDetectionConfig
class openjiuwen.core.controller.legacy.PlannerConfig
class openjiuwen.core.controller.legacy.ProactiveIdentifierConfig
class openjiuwen.core.controller.legacy.ReflectorConfig
class openjiuwen.core.controller.legacy.ReasonerConfig
```

- **IntentDetectionController**：基于意图识别的控制器实现，内部组合 `IntentDetector`、`Planner` 等组件：
  - `IntentDetector` 负责“从输入中识别 Intent”；
  - `Planner` 负责“根据 Intent 规划 Task 序列”。
- 各类 `*Config` 则用于控制上述组件的行为（如是否启用主动识别、反思、推理策略等）。

> 由于这部分属于旧控制器体系的实现细节，新开发如果希望使用 ReAct 风格控制器，建议优先参考 `openjiuwen.core.application.llm_agent.LLMController` 的文档和示例；当需要深入理解事件/任务/意图的底层数据结构时，可回到本模块查看这些类型定义。

