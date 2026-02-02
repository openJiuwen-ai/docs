# openjiuwen.core

`openjiuwen.core` 是 openJiuwen 的核心模块集合，涵盖工作流编排与执行、会话与状态管理、执行器、检索、记忆、上下文引擎等能力。

**Modules**：

| MODULE | DESCRIPTION |
|---|---|
| [foundation](./openjiuwen.core/foundation.README.md) | 基础能力：LLM/Tool/Prompt/Store。 |
| [workflow](./openjiuwen.core/workflow.README.md) | 工作流编排与执行。 |
| [session](./openjiuwen.core/session.README.md) | 会话：状态、交互、流式输出、trace 等。 |
| [runner](./openjiuwen.core/runner.README.md) | 执行器与资源管理。 |
| [context_engine](./openjiuwen.core/context_engine.README.md) | 上下文引擎。 |
| [memory](./openjiuwen.core/memory.README.md) | 记忆引擎。 |
| [retrieval](./openjiuwen.core/retrieval.README.md) | 知识检索/RAG。 |
| [graph](./openjiuwen.core/graph.README.md) | 执行图与路由。 |
| [controller](./openjiuwen.core/controller.README.md) | 控制器抽象与事件/意图/任务建模（旧版控制器 API 入口）。 |
| [operator](./openjiuwen.core/operator.README.md) | 操作算子（如 LLMCall）。 |
| [skills](./openjiuwen.core/skills.README.md) | 技能管理与内置技能工具。 |
| [sys_operation](./openjiuwen.core/sys_operation.README.md) | 系统操作抽象（文件/代码/命令执行），支持本地与沙箱模式。 |
| [security](./openjiuwen.core/security.README.md) | 安全相关能力（预留扩展）。 |
| [application](./openjiuwen.core/application.README.md) | 应用层：预置 Agent（WorkflowAgent/LLMAgent）。 |
| [single_agent](./openjiuwen.core/single_agent.README.md) | 单 Agent（新 API 为主）。 |
| [multi_agent](./openjiuwen.core/multi_agent.README.md) | 多 Agent（新 API 为主）。 |
