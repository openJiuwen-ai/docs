# openjiuwen.core.sys_operation.shell

## class openjiuwen.core.sys_operation.shell.BaseShellOperation

```python
class openjiuwen.core.sys_operation.shell.BaseShellOperation()
```

提供 Shell 命令执行功能。

### async execute_cmd(command: str, cwd: Optional[str] = None, timeout: Optional[int] = 300, environment: Optional[Dict[str, str]] = None, options: Optional[Dict[str, Any]] = None) -> ExecuteCmdResult

异步执行 Shell 命令（仅限 Shell 模式）。

**参数**：

* **command**(str)：要执行的命令。
* **cwd**(str, 可选)：命令执行的工作目录。默认值：当前目录。
* **timeout**(int, 可选)：命令执行超时时间（秒）。默认值：300。
* **environment**(Dict[str, str], 可选)：自定义环境变量。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**ExecuteCmdResult**，命令执行结果。

### async execute_cmd_stream(command: str, cwd: Optional[str] = None, timeout: Optional[int] = 300, environment: Optional[Dict[str, str]] = None, options: Optional[Dict[str, Any]] = None) -> AsyncIterator[ExecuteCmdStreamResult]

异步流式执行 Shell 命令（仅限 Shell 模式）。

**参数**：

* **command**(str)：要执行的命令。
* **cwd**(str, 可选)：命令执行的工作目录。默认值：当前目录。
* **timeout**(int, 可选)：命令执行超时时间（秒）。默认值：300。
* **environment**(Dict[str, str], 可选)：自定义环境变量。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**AsyncIterator[ExecuteCmdStreamResult]**，流式命令执行结果。