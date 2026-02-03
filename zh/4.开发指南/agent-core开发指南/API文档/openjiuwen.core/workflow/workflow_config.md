# openjiuwen.core.workflow.workflow_config

## class openjiuwen.core.workflow.workflow_config.WorkflowConfig

工作流配置类，对应 `openjiuwen.core.workflow.workflow_config.WorkflowConfig`。

### 字段

- **card** ([WorkflowCard](./base.md))：工作流身份卡片，必填。包含 id、name、version、description、input_params 等。
- **spec** (Optional[WorkflowSpec])：工作流结构规约（边、组件配置等），默认 `WorkflowSpec()`。`WorkflowSpec` 定义在 `openjiuwen.core.common.schema.workflow_spec`。
- **workflow_max_nesting_depth** (int)：工作流最大嵌套深度，默认 5，取值范围 [0, 10]。

### 样例

```python
>>> from openjiuwen.core.workflow import WorkflowCard, WorkflowConfig
>>>
>>> config = WorkflowConfig(
...     card=WorkflowCard(
...         id="workflow_001",
...         name="示例工作流",
...         version="1.0",
...     )
... )
```
