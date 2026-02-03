# openjiuwen.core.sys_operation.result.code_operation_result

`openjiuwen.core.sys_operation.result.code_operation_result` 模块定义了代码执行相关的数据模型。

## class ExecuteCodeData

```python
class ExecuteCodeData(BaseModel):
    code_content: str = Field(..., description="Original code executed")
    language: str = Field(..., description="Programming language of the original code")
    exit_code: int = Field(default=0, description="Execution exit code")
    stdout: str = Field(default="", description="The code's standard output (stdout) stream")
    stderr: str = Field(default="", description="The code's standard error (stderr) stream")

```

代码执行结果数据。

- `code_content: str`: 原始执行的代码内容。
- `language: str`: 编程语言。
- `exit_code: int`: 进程退出码，`0` 表示成功。
- `stdout: str`: 标准输出内容。
- `stderr: str`: 标准错误内容。

## class ExecuteCodeChunkData

```python
class ExecuteCodeChunkData(BaseModel):
    text: str = Field(default="", description="Raw content of the output chunk")
    type: Literal["stdout", "stderr"] = Field(..., description="Type of the output chunk")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    exit_code: int = Field(default=0, description="Execution exit code")
    metadata: Optional[Dict[str, Any]] = Field(default=None, description="Data for execution")
```

流式执行时的分片数据。

- `text: str`: 当前分片文本内容。
- `type: Literal["stdout", "stderr"]`: 输出类型。
- `chunk_index: int`: 分片序号。
- `exit_code: int`: 退出码。
- `metadata: Optional[Dict[str, Any]]`: 额外元数据。

## class ExecuteCodeResult

```python
class ExecuteCodeResult(BaseResult[ExecuteCodeData])
```

代码执行的最终结果类型。

## class ExecuteCodeStreamResult

```python
class ExecuteCodeStreamResult(BaseResult[ExecuteCodeChunkData])
```

代码流式执行的结果类型。
