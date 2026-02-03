# openjiuwen.core.sys_operation.base

`openjiuwen.core.sys_operation.base` 模块定义了系统操作的基础类和运行模式枚举。

## class OperationMode

```python
class OperationMode(str, Enum):
    LOCAL = "local"
    SANDBOX = "sandbox"
```

系统操作的运行模式枚举。

- `LOCAL`：本地模式 (`"local"`)，在当前运行环境中直接执行操作。
- `SANDBOX`：沙箱模式 (`"sandbox"`)，通过网关连接远程沙箱环境执行操作。

## class BaseOperation

```python
class BaseOperation
```

所有具体操作（如文件、代码、Shell）的基类。

### __init__

```python
def __init__(
    self,
    name: str,
    mode: OperationMode,
    description: str,
    run_config: Union[LocalWorkConfig, SandboxGatewayConfig]
)
```

**参数**：

- `name`: 操作的唯一标识符（例如 "fs", "code", "shell"）。
- `mode`: 运行模式 (`OperationMode`)。
- `description`: 操作的人类可读描述。
- `run_config`: 运行配置，根据模式不同可能是 `LocalWorkConfig` 或 `SandboxGatewayConfig`。

### list_tools

```python
def list_tools(self) -> list[ToolCard]
```

列出该操作提供的所有工具定义（`ToolCard` 列表）。具体的工具列表由子类实现。
