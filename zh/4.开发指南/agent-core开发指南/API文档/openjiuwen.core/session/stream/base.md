# openjiuwen.core.session.stream.base

流式输出相关类已迁移至 `openjiuwen.core.session.stream`。源码中无顶级 `openjiuwen.core.stream`，请使用 `from openjiuwen.core.session.stream import BaseStreamMode, OutputSchema, TraceSchema, CustomSchema` 导入。

## class openjiuwen.core.session.stream.base.BaseStreamMode

```python
class openjiuwen.core.session.stream.base.BaseStreamMode(StreamMode)
```

在工作流的 `stream` 方法中设置不同的 `BaseStreamMode` 类型，可以控制流式输出的数据格式和内容。

- **OUTPUT**：标准流式输出数据格式，输出为 `OutputSchema` 类型。
- **TRACE**：调试信息流式输出数据格式，输出为 `TraceSchema` 类型。
- **CUSTOM**：自定义流式输出数据格式，输出为 `CustomSchema` 类型。

## class openjiuwen.core.session.stream.base.OutputSchema

```python
class openjiuwen.core.session.stream.base.OutputSchema(BaseModel)
```

openJiuwen 标准流式输出数据格式的数据类。

- **type** (str)：流式数据的类型。当前支持两种内置的流式消息类型：
  - `'INTERACTION'`：表示流式消息为中断交互的中断流式消息
  - `'end node stream'`：表示流式消息为 [End](../../workflow/components/flow/end_comp.md) 组件的流式消息。
- **index** (int)：流式数据的下标。
- **payload** (Any)：流式数据的数据内容。

**样例**：

```python
from openjiuwen.core.session.stream import OutputSchema

OutputSchema(type='end node stream', index=0, payload={'answer': '结果:'}),
OutputSchema(type='end node stream', index=1, payload={'answer': 'Check the weather in Shanghai on 2025-08-22'})
```

## class openjiuwen.core.session.stream.base.CustomSchema

```python
class openjiuwen.core.session.stream.base.CustomSchema(BaseModel)
```

用户自定义流式输出数据格式的数据类。

- **kwargs** (dict，可选)：用户自定义的流式数据。

**样例**：

```python
from openjiuwen.core.session.stream import CustomSchema

CustomSchema(custom_output='Check the weather in Shanghai on 2025-08-22')
```

## class openjiuwen.core.session.stream.base.TraceSchema

```python
class openjiuwen.core.session.stream.base.TraceSchema(BaseModel)
```

调试信息流式输出数据格式的数据类。

- **type** (str)：流式数据的类型。当前支持两种内置的调试信息类型：
  - `'tracer_workflow'`：表示 workflow 的组件调测信息。
  - `'tracer_agent'`：表示 agent 的调测信息，包含大模型和插件等调用。
- **payload** (Any)：调测信息数据，有内置的格式定义，包含唯一标识、起止时间、输入输出、错误详情及组件状态等关键字段。

  - 基础调测字段：traceId、startTime、endTime、inputs、outputs、error、invokeId、parentInvokeId、executionId、onInvokeData、componentId、componentName、componentType、status
  - 循环组件字段：loopNodeId、loopIndex
  - 嵌套子 workflow 字段：parentNodeId

**样例**：TraceSchema 使用方式与原先一致，仅需将导入改为 `from openjiuwen.core.session.stream import TraceSchema`。
