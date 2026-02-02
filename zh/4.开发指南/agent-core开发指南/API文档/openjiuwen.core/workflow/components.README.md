# workflow.components

`openjiuwen.core.workflow.components` 提供工作流预置组件，与 [workflow](../workflow.README.md) 基类配合使用。组件类通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import ...` 导入。

| 文档 | 说明 |
|------|------|
| [base](./components/base.md) | WorkflowComponent、ComponentExecutable、ComponentConfig |
| [branch_comp](./components/branch_comp.md) | BranchComponent |
| [branch_router](./components/branch_router.md) | BranchRouter、Branch |
| [condition/condition](./components/condition/condition.md) | Condition、FuncCondition、AlwaysTrue |
| [condition/expression](./components/condition/expression.md) | ExpressionCondition |
| [condition/array](./components/condition/array.md) | ArrayCondition |
| [condition/number](./components/condition/number.md) | NumberCondition |
| [end_comp](./components/end_comp.md) | End、EndConfig |
| [intent_detection_comp](./components/intent_detection_comp.md) | IntentDetectionComponent、IntentDetectionCompConfig |
| [llm_comp](./components/llm_comp.md) | LLMComponent、LLMCompConfig |
| [loop_comp](./components/loop_comp.md) | LoopComponent、LoopGroup、LoopBreakComponent、LoopSetVariableComponent |
| [questioner_comp](./components/questioner_comp.md) | QuestionerComponent、QuestionerConfig、FieldInfo |
| [start_comp](./components/start_comp.md) | Start |
| [tool_comp](./components/tool_comp.md) | ToolComponent、ToolComponentConfig |
| [workflow_comp](./components/workflow_comp.md) | SubWorkflowComponent |
