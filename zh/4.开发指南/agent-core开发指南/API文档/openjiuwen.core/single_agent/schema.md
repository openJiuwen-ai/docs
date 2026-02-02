# openjiuwen.core.single_agent.schema

本模块提供单 Agent 相关数据结构：`AgentCard` 定义 Agent 身份与输入输出参数，`AgentResult` 与 `Artifact` 用于描述执行结果与产出物。`AgentCard` 定义在 `openjiuwen.core.single_agent.schema.agent_card`，`AgentResult`、`Artifact` 定义在 `openjiuwen.core.single_agent.schema.agent_result`。

> **注意**：`AgentCard` 通过 `openjiuwen.core.single_agent` 导出；`AgentResult`、`Artifact` 需从 `openjiuwen.core.single_agent.schema.agent_result` 导入。

---

## class openjiuwen.core.single_agent.schema.agent_card.AgentCard

Agent 卡片数据类，继承自 [BaseCard](../common.README.md)，用于描述 Agent 的身份、说明与输入输出参数。作为 `BaseAgent` 构造必填参数，也用于将 Agent 作为能力（Ability）注册到其他 Agent 的 `AbilityKit` 中。

### 继承字段（BaseCard）

- **id** (str)：可选唯一标识。
- **name** (str)：Agent 名称，用于展示与能力查找。
- **description** (str)：Agent 说明，供 LLM 或上层选择能力时使用。

### 字段

- **input_params** (List[Param])：输入参数列表，默认空列表。每个 [Param](../common.README.md) 描述参数名、类型、描述、是否必填等；用于生成 `ToolInfo.parameters` 及子 Agent 调用时的 schema。
- **output_params** (List[Param])：输出参数列表，默认空列表。描述 Agent 返回结果的结构，供上层或工作流编排使用。

### 使用说明

- 创建 `BaseAgent` 或 `ReActAgent` 时必须传入 `AgentCard`。
- 将本 Agent 作为子 Agent 注册到其他 Agent 时，会使用 `AgentCard` 的 name、description、input_params 生成 `ToolInfo`。

**样例**：

```python
from openjiuwen.core.single_agent import AgentCard
from openjiuwen.core.common.schema import Param

card = AgentCard(
    name="assistant",
    description="通用助手 Agent",
    input_params=[
        Param(name="user_input", type="string", description="用户输入", required=True)
    ],
    output_params=[
        Param(name="reply", type="string", description="回复内容")
    ]
)
```

---

## class openjiuwen.core.single_agent.schema.agent_result.Artifact

表示 Agent 产出物（如摘要、图表、文件等）的语义单元。用于在 `AgentResult` 中承载结构化产出列表。

### 字段

- **artifactId** (str)：产出物唯一标识，默认 `None`。
- **name** (Optional[str])：语义名称，如 `"summary"`、`"chart"`。
- **description** (Optional[str])：产出物描述。
- **parts** (List[Part])：组成部件列表，默认空列表。`Part` 定义见 [common.schema.part](../common.README.md)。
- **metadata** (Dict[str, Any])：扩展元数据，默认空字典。

---

## class openjiuwen.core.single_agent.schema.agent_result.AgentResult

Agent 执行结果的数据模型（Pydantic BaseModel）。用于统一描述任务 id、会话 id、状态、产出物列表与元数据。

### 字段

- **task_id** (str)：任务 id，默认 `None`。
- **sessionId** (Optional[str])：会话 id。
- **status** (TaskStatus)：任务状态，见 [common.schema.task](../common.README.md) 中的 `TaskStatus`。
- **artifacts** (List[Artifact])：产出物列表，默认空列表。
- **metadata** (Dict[str, Any])：扩展元数据，默认空字典。

### 使用说明

- 子类实现 `BaseAgent.invoke` 或 `stream` 时，可返回 `AgentResult` 实例，便于上层统一解析任务状态与产出物。
- 与 `Session`、流式输出的结合方式由具体 Runner/执行链路约定。

**样例**：

```python
from openjiuwen.core.single_agent.schema.agent_result import AgentResult, Artifact
from openjiuwen.core.common.schema.task import TaskStatus

result = AgentResult(
    task_id="task_001",
    sessionId="session_123",
    status=TaskStatus.SUCCESS,
    artifacts=[
        Artifact(
            artifactId="a1",
            name="summary",
            description="对话摘要",
            parts=[],
            metadata={}
        )
    ],
    metadata={"source": "react_agent"}
)
```
