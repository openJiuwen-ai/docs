# openjiuwen.core.workflow.components.flow.end_comp

`openjiuwen.core.workflow.components.flow.end_comp` 模块提供工作流结束组件，用于定义工作流的最终输出。支持无配置（直接透传上游输入）或配置响应模板（使用 `{{variable}}` 占位符格式化输出）。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import End, EndConfig` 导入。更多组件说明见 [components](../../components.README.md)。

## class EndConfig

```python
class openjiuwen.core.workflow.components.flow.end_comp.EndConfig(BaseModel)
```

`End` 组件的配置数据类，用于指定响应模板。字段通过 Pydantic 的 `alias` 支持 JSON 风格命名（如 `responseTemplate`）。

### 字段

- **response_template**（str，别名 `responseTemplate`）：用于格式化最终输出的响应模板，输出格式为 `{"response": "..."}`。模板中可使用 `{{name}}`、`{{error_msg}}` 等占位符，最小长度为 1。示例：`"Hello {{name}}"`、`"Error: {{error_msg}}"`。

---

## class End

```python
class openjiuwen.core.workflow.components.flow.end_comp.End(WorkflowComponent)
```

工作流结束组件，继承自 [WorkflowComponent](../components.md#class-workflowcomponent)。无配置时直接将上游输入整理为 `output` 输出；有配置时使用 [EndConfig](end_comp.md#class-endconfig) 中的 `response_template` 渲染结果，并支持流式输出。

### \_\_init\_\_

```python
def __init__(self, conf: Union[EndConfig, dict] = None) -> None
```

**参数**：

- **conf**（EndConfig | dict | None）：可选配置。若传入则校验并解析为 `EndConfig`，用于模板渲染；若为 `None` 则不做模板渲染。

**异常**：

- **JiuWenBaseException**：当 `conf` 解析失败（如 `response_template` 类型错误）时，错误码参见 [StatusCode](../../../common/exception/status_code.md) 中的 `COMPONENT_END_INIT_FAILED`。

### set_mix

```python
def set_mix(self) -> None
```

开启混合模式。在配置了响应模板且存在多路并发流式渲染时，用于控制等待与合并行为。

### invoke

```python
async def invoke(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

执行结束节点：有模板时按模板渲染 `inputs` 并返回 `{"response": ...}`；无模板时过滤掉 `None` 值并返回 `{"output": ...}` 或等价结构。

**参数**：

- **inputs**（Input）：上游传入的输入。
- **session**（Session）：工作流会话，用于获取模板渲染相关环境（如超时）。
- **context**（ModelContext）：上下文。

**返回**：

- **Output**：`{"response": str}` 或 `{"output": Any}`。

### stream

```python
async def stream(self, inputs: Input, session: Session, context: ModelContext) -> AsyncIterator[Output]
```

流式输出：有模板时按片段流式渲染并 yield `OutputSchema`；无模板时按输入字段逐个 yield。

### transform

```python
async def transform(self, inputs: Input, session: Session, context: ModelContext) -> AsyncIterator[Output]
```

流式转换：有模板时与 `stream` 类似；无模板时遍历 `inputs` 的叶子节点，若值为 `AsyncGenerator` 则逐帧 yield，否则按路径 yield 单条输出。

### collect

```python
async def collect(self, inputs: Input, session: Session, context: ModelContext) -> Output
```

收集流式输入为单次输出：有模板时等价于一次 `_render`；无模板时将所有叶子值（含 AsyncGenerator 的帧）收集为列表，返回 `{"output": chunks}`。
