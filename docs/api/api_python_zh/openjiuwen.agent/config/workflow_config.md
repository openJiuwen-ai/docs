# openjiuwen.agent.config.workflow_config

## class openjiuwen.agent.config.workflow_config.WorkflowAgentConfig

```python
openjiuwen.agent.config.workflow_config.WorkflowAgentConfig(id: str, version: str, description: str, workflows: List[WorkflowSchema], controller_type: ControllerType = ControllerType.WorkflowController, timeout: int = 60, start_workflow: WorkflowSchema = WorkflowSchema(), end_workflow: WorkflowSchema = WorkflowSchema(), global_variables: List[dict] = [], global_params: Dict[str, Any] = {}, constrain: ConstrainConfig = ConstrainConfig())
```

WorkflowAgentConfig类用于配置WorkflowAgent的所有参数。

**参数**：

- **id**(str)：agent的唯一标识符。
- **version**(str)：agent的版本号。
- **description**(str)：agent的描述信息。
- **workflows**(List[[WorkflowSchema](../common/schema.md#class-openjiuwenagentcommonschemaworkflowschema)])：agent支持的工作流描述信息列表。
- **controller_type**([ControllerType](../common/enum.md#class-openjiuwenagentcommonenumcontrollertype), 可选)：控制器类型。此处必须是`ControllerType.WorkflowController`。默认值：`ControllerType.WorkflowController`。
- **timeout**(int, 可选)：全局超时时间。单位：秒。默认值：60秒。
- **start_workflow**([WorkflowSchema](../common/schema.md#class-openjiuwenagentcommonschemaworkflowschema), 可选)：预留参数，用于多级Agent，当前还未使用。
- **end_workflow**([WorkflowSchema](../common/schema.md#class-openjiuwenagentcommonschemaworkflowschema), 可选)：预留参数，用于多级Agent，当前还未使用。
- **global_variables**(List[dict], 可选)：预留参数，用于多级Agent，当前还未使用。
- **global_params**(Dict[str, Any], 可选)：预留参数，用于多级Agent，当前还未使用。
- **constrain**([ConstrainConfig](react_config.md#class-openjiuwenagentconfigreact_configconstrainconfig), 可选)：约束配置。默认值：`ConstrainConfig()`。

**样例**：

```python
>>> import os
>>> from openjiuwen.agent.common.schema import WorkflowSchema
>>> from openjiuwen.agent.config.workflow_config import WorkflowAgentConfig
>>> from openjiuwen.core.workflow.base import Workflow
>>> from openjiuwen.core.workflow.workflow_config import WorkflowConfig, WorkflowMetadata
>>> 
>>> API_BASE = os.getenv("API_BASE", "")
>>> API_KEY = os.getenv("API_KEY", "")
>>> MODEL_NAME = os.getenv("MODEL_NAME", "")
>>> MODEL_PROVIDER = os.getenv("MODEL_PROVIDER", "")
>>> 
>>> def _create_workflow_schema(id, name: str, version: str) -> WorkflowSchema:
...     return WorkflowSchema(
...         id=id,
...         name=name,
...         description="天气查询工作流",
...         version=version,
...         inputs={"query": {"type": "string"}}
...     )
>>> 
>>> workflow_id = "test_weather_agent"
>>> workflow_name = "weather"
>>> workflow_version = "1.0"
>>> schema = _create_workflow_schema(workflow_id, workflow_name, workflow_version)
>>> 
>>> # 创建WorkflowAgentConfig
>>> config = WorkflowAgentConfig(
...     id="test_weather_agent",
...     version="0.1.0",
...     description="测试用天气 agent",
...     workflows=[schema],
...     timeout=120
... )
```

### is_single_workflow

```python
is_single_workflow(self) -> bool
```

判断当前配置是否只包含单个工作流。

**返回**：

**bool**，如果只有一个工作流返回`True`，否则返回`False`。

**样例**：

```python
>>> from openjiuwen.agent.config.workflow_config import WorkflowAgentConfig
>>> 
>>> def _create_workflow_schema(id, name: str, version: str) -> WorkflowSchema:
...     return WorkflowSchema(
...         id=id,
...         name=name,
...         description="天气查询工作流",
...         version=version,
...         inputs={"query": {"type": "string"}}
...     )
>>> 
>>> # 创建单个工作流的配置
>>> schema = _create_workflow_schema("workflow_1", "weather", "1.0")
>>> config = WorkflowAgentConfig(
...     id="test_agent",
...     version="0.1.0",
...     description="测试agent",
...     workflows=[schema]
... )
>>> 
>>> # 检查是否为单工作流
>>> is_single = config.is_single_workflow
>>> print(f"是否为单工作流: {is_single}")  # 输出: 是否为单工作流: True
是否为单工作流: True
>>> 
>>> # 创建多个工作流的配置
>>> schema2 = _create_workflow_schema("workflow_2", "translate", "1.0")
>>> config_multi = WorkflowAgentConfig(
...     id="test_agent_multi",
...     version="0.1.0",
...     description="测试多工作流agent",
...     workflows=[schema, schema2]
... )
>>> 
>>> is_single_multi = config_multi.is_single_workflow
>>> print(f"是否为单工作流: {is_single_multi}")  # 输出: 是否为单工作流: False
是否为单工作流: False
```
