# openjiuwen.core.sys_operation.shell

`openjiuwen.core.sys_operation.shell` 模块定义了 Shell 操作的抽象基类。

## class BaseShellOperation

```python
class BaseShellOperation(BaseOperation, ABC)
```

Shell 操作的基类。

### execute_cmd

```python
@abstractmethod
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

异步执行命令（仅限 Shell 模式）。

**参数**：

- `command`: 要执行的命令。
- `cwd`: 命令执行的工作目录（默认：当前目录）。
- `timeout`: 命令执行超时时间（秒）（默认：300 秒）。
- `environment`: 自定义环境变量的键值字典。
- `options`: 额外的执行配置选项。

**返回**：

- `ExecuteCmdResult`: 执行结果。

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

异步流式执行命令（仅限 Shell 模式）。

**参数**：

- 同 `execute_cmd`。

**返回**：

- `AsyncIterator[ExecuteCmdStreamResult]`: 流式的结构化结果。
