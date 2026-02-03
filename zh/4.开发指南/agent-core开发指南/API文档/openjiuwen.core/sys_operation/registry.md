# openjiuwen.core.sys_operation.registry

`openjiuwen.core.sys_operation.registry` 模块提供了操作注册表，用于管理和检索系统操作类。

## class OperationRegistry

```python
class OperationRegistry
```

操作注册表，管理操作类的注册和获取。

### register

```python
@classmethod
def register(
    cls,
    operation_cls: Type[BaseOperation],
    name: str,
    mode: OperationMode,
    description: str
)
```

注册操作类到仓库。

**参数**：

- `operation_cls`: 操作类（必须继承自 `BaseOperation`）。
- `name`: 操作的唯一标识符（例如 "fs", "code", "shell"）。
- `mode`: 运行模式（例如 `OperationMode.LOCAL` 或 `OperationMode.SANDBOX`）。
- `description`: 操作的人类可读描述。

### get_operation_info

```python
@classmethod
def get_operation_info(
    cls,
    name: str,
    mode: OperationMode,
    auto_load: bool = True
) -> Dict[str, Any]
```

根据名称和模式获取操作类信息。

**参数**：

- `name`: 操作的唯一标识符。
- `mode`: 运行模式。
- `auto_load`: 是否自动加载未注册的模块（默认为 `True`）。

**返回**：

- 包含操作类 (`cls`) 和描述 (`description`) 的字典。

## operation

```python
def operation(name: str, mode: OperationMode, description: str = "")
```

装饰器，用于将类注册为 `OperationRegistry` 中的操作。

**参数**：

- `name`: 操作的唯一标识符。
- `mode`: 运行模式。
- `description`: 操作描述，默认为 `""`。

**返回**：

- 原始类。
