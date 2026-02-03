# openjiuwen.core.workflow.components.llm.intent_detection_comp

`openjiuwen.core.workflow.components.llm.intent_detection_comp` 模块提供工作流中的意图识别组件，用于根据用户输入与可选对话历史调用大模型进行意图分类，并配合 [BranchRouter](../flow/branch_router.md#class-branchrouter) 将执行路由到不同下游节点。分类结果通过 [IntentDetectionCompConfig](intent_detection_comp.md#class-intentdetectioncompconfig) 中的 `category_name_list` 等配置，输出为分类 id、分类名称及原因。组件通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import IntentDetectionComponent, IntentDetectionCompConfig` 导入。更多组件说明见 [components](../../components.README.md)。

## class IntentDetectionCompConfig

```python
@dataclass
class openjiuwen.core.workflow.components.llm.intent_detection_comp.IntentDetectionCompConfig(ComponentConfig)
```

意图识别组件的配置数据类。

### 字段

- **model_id**（str | None）：已注册模型 id，与 `model_client_config` + `model_config` 二选一。
- **model_client_config**（ModelClientConfig | None）：模型客户端配置。
- **model_config**（ModelRequestConfig | None）：模型请求配置。
- **category_name_list**（list[str]）：分类名称列表，与内部生成的 `分类1`、`分类2` 等一一对应，用于解析 LLM 输出与展示。
- **user_prompt**（str）：用户侧提示词内容，会填入默认模板中的 `user_prompt` 占位符。
- **example_content**（list[str]）：示例内容列表，用于 few-shot 提示。
- **enable_history**（bool）：是否将上下文中的对话历史加入意图识别输入，默认 `False`。
- **chat_history_max_turn**（int）：使用的对话历史最大轮数，默认 3。

---

## class IntentDetectionComponent

```python
class openjiuwen.core.workflow.components.llm.intent_detection_comp.IntentDetectionComponent(ComponentComposable)
```

意图识别组件，实现 [ComponentComposable](../components.md)。内部持有一个 [BranchRouter](../flow/branch_router.md#class-branchrouter)，通过 `add_component` 将本节点加入图并添加条件边，通过 `add_branch` 配置“分类 → 下游节点”的映射；`to_executable` 返回 IntentDetectionExecutable，执行时调用大模型得到分类结果并写入会话，供路由器求值。

### \_\_init\_\_

```python
def __init__(self, component_config: Optional[IntentDetectionCompConfig] = None) -> None
```

**参数**：

- **component_config**（IntentDetectionCompConfig | None）：意图识别配置；为 `None` 时由可执行对象侧按需处理。

### add_component

```python
def add_component(self, graph: Graph, node_id: str, wait_for_all: bool = False) -> None
```

将本组件作为节点加入 [Graph](../../../graph/graph.md#class-graph)，并为该节点添加条件边，由内部 [BranchRouter](../flow/branch_router.md#class-branchrouter) 根据意图识别结果路由到下游。

**参数**：

- **graph**（Graph）：工作流图实例。
- **node_id**（str）：本节点在图中的唯一 id。
- **wait_for_all**（bool）：是否等待所有前驱完成，默认 `False`。

### add_branch

```python
def add_branch(self, condition: Union[str, Callable[[], bool], Condition], target: Union[str, list[str]], branch_id: str = None) -> None
```

为内部 [BranchRouter](../flow/branch_router.md#class-branchrouter) 添加一条分支：当 `condition` 成立时路由到 `target`。`condition` 通常为基于意图识别输出的表达式（如 `"${intent.result} == '出行'"`），参见 [ExpressionCondition](../condition/expression.md)。

**参数**：

- **condition**：条件，支持 str、无参 Callable 或 [Condition](../condition/condition.md) 子类。
- **target**：下游节点 id 或 id 列表。
- **branch_id**：可选分支标识。

### router

```python
def router(self) -> BranchRouter
```

返回内部使用的 [BranchRouter](../flow/branch_router.md#class-branchrouter) 实例。

**返回**：

- **BranchRouter**：本组件的分支路由器。

### to_executable

```python
def to_executable(self) -> IntentDetectionExecutable
```

返回 IntentDetectionExecutable 实例，并为其设置内部 [BranchRouter](../flow/branch_router.md#class-branchrouter)（用于在执行时 set_session）。

**返回**：

- **IntentDetectionExecutable**：本组件的可执行实现。

---

## IntentDetectionExecutable（可执行实现）

意图识别组件的可执行实现，由 `IntentDetectionComponent.to_executable` 返回。主要行为：

- **invoke**：从输入与上下文中准备意图识别输入（含 `query`、分类列表、对话历史等），调用大模型，解析 JSON 输出得到 `class` 与 `reason`，映射为分类 id 与名称后返回 IntentDetectionOutput 风格的字典（如 `classification_id`、`category_name`、`reason`），并调用内部路由器的 [set_session](../flow/branch_router.md#set_session) 以便条件边求值。
- **set_router**：由组件在构造可执行对象后调用，注入 [BranchRouter](../flow/branch_router.md#class-branchrouter)。
- 输入校验：必选字段 `query` 通过 `IntentDetectionInput` 校验，失败时错误码为 `COMPONENT_INTENT_DETECTION_INPUT_PARAM_ERROR`；模型创建或调用失败参见 [StatusCode](../../../common/exception/status_code.md) 中的意图识别相关项。

输出结构（IntentDetectionOutput）包含：**classification_id**（int）、**reason**（str）、**category_name**（str）。
