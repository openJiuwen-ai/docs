# openjiuwen.core.sys_operation

## class openjiuwen.core.sys_operation.OperationMode

定义了系统操作执行过程中不同模式的标识。

**参数**：

* **LOCAL**：表示系统操作在本地环境中执行。
* **SANDBOX**：表示系统操作在隔离的沙箱环境中执行。

## class openjiuwen.core.sys_operation.LocalWorkConfig

表示系统操作在本地环境中执行所使用的本地工作环境配置。

**参数**：

* **shell_allowlist**(List[str], 可选)：允许执行的Shell命令前缀白名单。默认值：`["echo", "ls", "dir", "cd", "pwd", "python", "python3", "pip", "pip3", "npm", "node", "git", "cat", "type", "mkdir", "md", "rm", "rd", "cp", "copy", "mv", "move", "grep", "find", "curl", "wget", "ps", "df", "ping"]`。如果值为`None`，表示允许执行所有Shell命令，需要注意，这是不满足安全要求的，所以不建议设置为`None`。
* **work_dir**(str, 可选)：本地工作目录路径。默认值：`None`。

## class openjiuwen.core.sys_operation.SysOperationCard

```python
class openjiuwen.core.sys_operation.SysOperationCard(BaseCard)
```

系统操作配置卡片。

**参数**：

* **mode**(OperationMode, 可选)：运行模式。默认值：`OperationMode.LOCAL`。
* **work_config**(LocalWorkConfig, 可选)：本地工作配置（当模式为 `local` 时必需）。默认值：`None`。
* **gateway_config**(SandboxGatewayConfig, 可选)：沙箱网关配置（当模式为 `sandbox` 时必需）。默认值：`None`。

## class openjiuwen.core.sys_operation.SysOperation

```python
class openjiuwen.core.sys_operation.SysOperation(card: SysOperationCard)
```

系统操作入口类，提供对文件、代码和 Shell 操作的访问。

**参数**：

* **card**(SysOperationCard)：系统操作配置卡片。

### fs() -> BaseFsOperation

获取文件系统操作实例。

**返回**：

**BaseFsOperation**，对应的 `fs` 操作实例（本地或沙箱）。

### code() -> BaseCodeOperation

获取代码执行操作实例。

**返回**：

**BaseCodeOperation**，对应的 `code` 操作实例（本地或沙箱）。

### shell() -> BaseShellOperation

获取 Shell 操作实例。

**返回**：

**BaseShellOperation**，对应的 `shell` 操作实例（本地或沙箱）。
