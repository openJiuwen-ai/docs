# openjiuwen.core.sys_operation.sys_operation

`openjiuwen.core.sys_operation.sys_operation` 模块定义了系统操作的主要入口类。

## class SysOperationCard

```python
class SysOperationCard(BaseCard)
```

系统操作配置卡片。

- `mode: OperationMode`
  - 运行模式，可选值：`local` / `sandbox`。
  - 默认为 `OperationMode.LOCAL`。
- `work_config: Optional[LocalWorkConfig]`
  - 本地工作配置（当模式为 `local` 时必需）。
  - 默认为 `None`。
- `gateway_config: Optional[SandboxGatewayConfig]`
  - 沙箱网关配置（当模式为 `sandbox` 时必需）。
  - 默认为 `None`。

## class SysOperation

```python
class SysOperation
```

系统操作入口类，提供对文件、代码和 Shell 操作的访问。

### __init__

```python
def __init__(self, card: SysOperationCard)
```

初始化系统操作实例。

**参数**：

- `card`: 系统操作配置卡片 (`SysOperationCard`)。

### fs

```python
def fs(self)
```

获取文件系统操作实例。

**返回**：

- 对应的 `fs` 操作实例（本地或沙箱）。

### code

```python
def code(self)
```

获取代码执行操作实例。

**返回**：

- 对应的 `code` 操作实例（本地或沙箱）。

### shell

```python
def shell(self)
```

获取 Shell 操作实例。

**返回**：

- 对应的 `shell` 操作实例（本地或沙箱）。
