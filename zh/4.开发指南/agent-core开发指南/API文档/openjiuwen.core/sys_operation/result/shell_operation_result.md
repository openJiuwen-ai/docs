# openjiuwen.core.sys_operation.result.shell_operation_result

`openjiuwen.core.sys_operation.result.shell_operation_result` 模块定义了 Shell 命令执行相关的数据模型。

## class ExecuteCmdData

```python
class ExecuteCmdData(BaseModel):
    command: str = Field(..., description="Original shell command executed")
    cwd: str = Field(default=".", description="Current working directory")
    exit_code: int = Field(default=0, description="Command exit code")
    stdout: str = Field(default="", description="The command's standard output (stdout) stream")
    stderr: str = Field(default="", description="The command's standard error (stderr) stream")
```

Shell 命令执行结果数据。

- `command: str`: 执行的命令。
- `cwd: str`: 执行目录。
- `exit_code: int`: 退出码。
- `stdout: str`: 标准输出。
- `stderr: str`: 标准错误。

## class ExecuteCmdChunkData

```python
class ExecuteCmdChunkData(BaseModel):
    text: str = Field(default="", description="Raw content of the output chunk")
    type: Literal["stdout", "stderr"] = Field(..., description="Type of the output chunk")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    exit_code: int = Field(default=0, description="Command exit code")
    metadata: Optional[Dict[str, Any]] = Field(default=None, description="Data for command")
```

Shell 流式执行分片数据。

- `text: str`: 分片内容。
- `type: Literal["stdout", "stderr"]`: 类型。
- `chunk_index: int`: 序号。
- `exit_code: int`: 退出码。
- `metadata: Optional[Dict[str, Any]]`: 元数据。

## class ExecuteCmdResult

```python
class ExecuteCmdResult(BaseResult[ExecuteCmdData])
```

Shell 执行的最终结果类型。

## class ExecuteCmdStreamResult

```python
class ExecuteCmdStreamResult(BaseResult[ExecuteCmdChunkData])
```

Shell 流式执行的结果类型。
