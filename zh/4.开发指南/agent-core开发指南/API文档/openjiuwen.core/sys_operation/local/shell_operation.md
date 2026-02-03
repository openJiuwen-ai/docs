# openjiuwen.core.sys_operation.local.shell_operation

`openjiuwen.core.sys_operation.local.shell_operation` 模块提供了本地 Shell 命令执行的具体实现。

## class ShellOperation

```python
class openjiuwen.core.sys_operation.local.shell_operation.ShellOperation(BaseShellOperation)
```

继承自 `BaseShellOperation`，实现了本地 Shell 命令执行功能。

注册信息：
- `name`: `"shell"`
- `mode`: `OperationMode.LOCAL`
- `description`: `"local shell operation"`

### execute_cmd

```python
async def execute_cmd(
    self,
    command: str,
    *,
    cwd: Optional[str] = None,
    timeout: Optional[int] = 300,
    environment: Optional[Dict[str, str]] = None,
    options: Optional[Dict[str, Any]] = None
) -> ExecuteCmdResult
```

异步执行 Shell 命令（仅限 Shell 模式）。

**参数说明**：
- `command` (str): 要执行的命令。
- `cwd` (Optional[str]): 命令执行的工作目录（默认：当前目录）。
- `timeout` (Optional[int]): 命令执行超时时间（秒）（默认：300 秒）。
- `environment` (Optional[Dict[str, str]]): 自定义环境变量的键值对字典。
- `options` (Optional[Dict[str, Any]]): 额外的执行配置选项。

**实现细节**：
- **安全检查**：执行前会检查命令是否在白名单 (`shell_allowlist`) 中。
- **环境准备**：支持注入自定义环境变量。
- **路径解析**：支持基于 `work_dir` 配置解析相对路径。
- **超时控制**：支持执行超时自动终止。
- **输出捕获**：异步捕获标准输出和标准错误。

### execute_cmd_stream

```python
async def execute_cmd_stream(
    self,
    command: str,
    *,
    cwd: Optional[str] = None,
    timeout: Optional[int] = 300,
    environment: Optional[Dict[str, str]] = None,
    options: Optional[Dict[str, Any]] = None
) -> AsyncIterator[ExecuteCmdStreamResult]
```

以流式方式异步执行 Shell 命令（仅限 Shell 模式）。

**参数说明**：
- `command` (str): 要执行的命令。
- `cwd` (Optional[str]): 命令执行的工作目录（默认：当前目录）。
- `timeout` (Optional[int]): 命令执行超时时间（秒）（默认：300 秒）。
- `environment` (Optional[Dict[str, str]]): 自定义环境变量的键值对字典。
- `options` (Optional[Dict[str, Any]]): 额外的执行配置选项。
