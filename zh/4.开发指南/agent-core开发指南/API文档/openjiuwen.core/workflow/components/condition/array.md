# openjiuwen.core.workflow.components.condition.array

`openjiuwen.core.workflow.components.condition.array` 模块提供基于数组的循环条件实现，用于在 [LoopComponent](../flow/loop/loop_comp.md#class-loopcomponent) 中按数组（或输入中的数组引用）逐元素迭代，并在每轮将当前索引对应的值写入会话状态。与 [loop_comp](../flow/loop/loop_comp.md) 配合使用。条件类通过 `openjiuwen.core.workflow` 导出，建议使用 `from openjiuwen.core.workflow import ArrayCondition` 导入。

## class ArrayCondition

```python
class openjiuwen.core.workflow.components.condition.array.ArrayCondition(Condition)
```

基于数组的循环条件：根据 `arrays` 配置从输入或会话中按当前轮次索引取各数组的对应元素，更新 `session.state()`，并判断当前索引是否小于最短数组长度以决定是否继续循环。继承自 [Condition](condition.md)。

### \_\_init\_\_

```python
def __init__(self, arrays: dict[str, Union[str, list[Any]]]) -> None
```

**参数**：

- **arrays**（dict[str, Union[str, list[Any]]]）：键为变量名（写入会话状态的 key），值为输入中的数组 key（str）或数组本身（list）。循环时按 `session.state()` 中的 `INDEX` 取当前下标，各数组长度取最小值为有效长度；当 `current_idx >= min_length` 时条件为假。

### invoke

```python
def invoke(self, inputs: Input, session: BaseSession) -> Output
```

求值当前轮次是否继续循环：从 `session.state()` 读取 `INDEX`，对 `arrays` 中每项从 `inputs` 取数组（或使用传入的 list），取最短长度，若当前索引超出则返回 `False`；否则将当前索引对应的各数组元素写入 `session.state()` 并返回 `True` 及本次更新的键值（供上层写回输出）。

**参数**：

- **inputs**（Input）：当前输入，用于解析 `arrays` 中值为 str 的数组引用。
- **session**（BaseSession）：会话，用于读写 `INDEX` 与状态。

**返回**：

- **Output**：`True` 表示继续循环，可带元组 `(True, io_updates)`；`False` 表示结束循环。

**异常**：

- **JiuWenBaseException**：当 `arrays` 中某项类型不符合或访问越界时，错误码参见 [StatusCode](../../../common/exception/status_code.md) 中的 `ARRAY_CONDITION_ERROR`。

---

## ArrayConditionInSession（会话内数组条件）

内部实现类 `ArrayConditionInSession` 与 `ArrayCondition` 类似，但构造参数为 `dict[str, Union[list[Any], tuple[Any]]]`，即直接传入数组/元组而非输入 key；在构造时会校验各值为 list 或 tuple，并计算最小长度。适用于循环输入在会话或上层已解析好的场景。若 `arrays` 中某 key 的值为 `None` 或非 list/tuple，会抛出 `ARRAY_CONDITION_ERROR`。
