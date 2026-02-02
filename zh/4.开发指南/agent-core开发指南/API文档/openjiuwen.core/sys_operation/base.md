# openjiuwen.core.sys_operation.base

## class OperationMode

```python
class openjiuwen.core.sys_operation.base.OperationMode(str, Enum)
```

系统操作运行模式的枚举类型。

**可选值**：

- `OperationMode.LOCAL`：本地模式，在当前机器的文件系统 / 解释器 / Shell 环境下执行；
- `OperationMode.SANDBOX`：沙箱模式，通过远程沙箱网关进行隔离执行。

> 该枚举主要用于配置 `SysOperationCard.mode`，并在运行时选择对应的本地/沙箱实现。

## class BaseOperation

```python
class openjiuwen.core.sys_operation.base.BaseOperation
```

系统操作（如 `fs/code/shell`）的抽象基类。

### __init__

```python
def __init__(
    self,
    name: str,
    mode: OperationMode,
    description: str,
    run_config: Union[LocalWorkConfig, SandboxGatewayConfig],
)
```

**参数**：

- `name: str`：操作名称（如 `"fs"`、`"code"`、`"shell"`）；
- `mode: OperationMode`：运行模式（LOCAL 或 SANDBOX）；
- `description: str`：该操作的描述信息；
- `run_config: LocalWorkConfig | SandboxGatewayConfig`：
  - 本地模式下为 `LocalWorkConfig`；
  - 沙箱模式下为 `SandboxGatewayConfig`。

### list_tools

```python
def list_tools(self) -> list[ToolCard]
```

返回当前操作下可用的工具列表（`ToolCard` 列表）。

> 该方法在基类中仅作占位，具体工具枚举由各子类（如文件操作、代码执行操作等）实现。

