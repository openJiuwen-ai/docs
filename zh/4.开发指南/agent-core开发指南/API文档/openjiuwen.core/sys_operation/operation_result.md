# openjiuwen.core.sys_operation.operation_result

`openjiuwen.core.sys_operation.operation_result` 模块定义了系统操作的所有返回结果数据结构。

## 基础结果类

### class BaseResult

```python
class BaseResult(BaseModel, Generic[T], ABC):
    code: int = Field(..., description="Status code: 0 = success, non-0 = failure")
    message: str = Field(..., description="Message details")
    data: Optional[T] = Field(None, description="Business data (returned only on success)")
```

所有操作结果的通用泛型基类。

- `code: int`: 状态码。`0` 表示成功，非 `0` 表示失败。
- `message: str`: 消息详情。
- `data: Optional[T]`: 业务数据载荷（仅在成功时返回）。

## 代码执行结果

### class ExecuteCodeData

```python
class ExecuteCodeData(BaseModel):
    code_content: str = Field(..., description="Original code executed")
    language: str = Field(..., description="Programming language of the original code")
    exit_code: int = Field(default=0, description="Execution exit code")
    stdout: str = Field(default="", description="The code's standard output (stdout) stream")
    stderr: str = Field(default="", description="The code's standard error (stderr) stream")
```

代码执行结果数据。

### class ExecuteCodeChunkData

```python
class ExecuteCodeChunkData(BaseModel):
    text: str = Field(default="", description="Raw content of the output chunk")
    type: Literal["stdout", "stderr"] = Field(..., description="Type of the output chunk")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    exit_code: int = Field(default=0, description="Execution exit code")
    metadata: Optional[Dict[str, Any]] = Field(default=None, description="Data for execution")
```

流式执行时的分片数据。

### class ExecuteCodeResult

```python
class ExecuteCodeResult(BaseResult[ExecuteCodeData])
```

代码执行的最终结果类型。

### class ExecuteCodeStreamResult

```python
class ExecuteCodeStreamResult(BaseResult[ExecuteCodeChunkData])
```

代码流式执行的结果类型。

## Shell 操作结果

### class ExecuteCmdData

```python
class ExecuteCmdData(BaseModel):
    command: str = Field(..., description="Original shell command executed")
    cwd: str = Field(default=".", description="Current working directory")
    exit_code: int = Field(default=0, description="Command exit code")
    stdout: str = Field(default="", description="The command's standard output (stdout) stream")
    stderr: str = Field(default="", description="The command's standard error (stderr) stream")
```

Shell 命令执行结果数据。

### class ExecuteCmdChunkData

```python
class ExecuteCmdChunkData(BaseModel):
    text: str = Field(default="", description="Raw content of the output chunk")
    type: Literal["stdout", "stderr"] = Field(..., description="Type of the output chunk")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    exit_code: int = Field(default=0, description="Command exit code")
    metadata: Optional[Dict[str, Any]] = Field(default=None, description="Data for command")
```

Shell 流式执行分片数据。

### class ExecuteCmdResult

```python
class ExecuteCmdResult(BaseResult[ExecuteCmdData])
```

Shell 执行的最终结果类型。

### class ExecuteCmdStreamResult

```python
class ExecuteCmdStreamResult(BaseResult[ExecuteCmdChunkData])
```

Shell 流式执行的结果类型。

## 文件系统操作结果

### 通用数据结构

#### class FileSystemItem

```python
class FileSystemItem(BaseModel):
    name: str = Field(..., description="Name of the file/directory")
    path: str = Field(..., description="Full absolute path of the file/directory")
    size: int = Field(..., description="Size in bytes")
    modified_time: str = Field(..., description="Last modification time")
    is_directory: bool = Field(..., description="Whether the item is a directory")
    type: Optional[str] = Field(default=None, description="File extension (only for files)")
```

文件或目录的基本属性。

#### class FileSystemData

```python
class FileSystemData(BaseModel):
    total_count: int = Field(..., description="Total number of items")
    list_items: List[FileSystemItem] = Field(..., description="List of file/directory details")
    root_path: str = Field(..., description="Original input directory path")
    recursive: bool = Field(..., description="Actual recursive status used")
    max_depth: Optional[int] = Field(default=None, description="Actual maximum recursion depth used")
```

列出文件或目录的结果数据。

#### class SearchFilesData

```python
class SearchFilesData(BaseModel):
    total_matches: int = Field(..., description="Total number of files matching the search pattern")
    matching_files: List[FileSystemItem] = Field(..., description="List of matching files")
    search_path: str = Field(..., description="Original base path used for the search")
    search_pattern: str = Field(..., description="Original search pattern used")
    exclude_patterns: Optional[List[str]] = Field(default=None, description="Original exclude patterns used")
```

搜索文件的结果数据。

### 读文件结果

#### class ReadFileData

```python
class ReadFileData(BaseModel):
    path: str = Field(..., description="File path of the read file")
    content: Union[str, bytes] = Field(..., description="File content")
    mode: Literal['text', 'bytes'] = Field(..., description="File read mode")
```

#### class ReadFileChunkData

```python
class ReadFileChunkData(BaseModel):
    path: str = Field(..., description="File path of the read file")
    chunk_content: Union[str, bytes] = Field(..., description="Current chunk content")
    mode: Literal['text', 'bytes'] = Field(..., description="File read mode")
    chunk_size: int = Field(..., description="Size of each chunk")
    chunk_index: int = Field(..., description="Index of current chunk")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

#### class ReadFileResult

```python
class ReadFileResult(BaseResult[ReadFileData])
```

#### class ReadFileStreamResult

```python
class ReadFileStreamResult(BaseResult[ReadFileChunkData])
```

### 写文件结果

#### class WriteFileData

```python
class WriteFileData(BaseModel):
    path: str = Field(..., description="File path of the write file")
    size: int = Field(..., description="File content size in bytes")
    mode: Literal['text', 'bytes'] = Field(..., description="File write mode")
```

#### class WriteFileResult

```python
class WriteFileResult(BaseResult[WriteFileData])
```

### 上传/下载文件结果

#### class UploadFileData

```python
class UploadFileData(BaseModel):
    local_path: str = Field(..., description="File path of the local file")
    target_path: str = Field(..., description="File path of the target file")
    size: int = Field(..., description="File content size in bytes")
```

#### class UploadFileChunkData

```python
class UploadFileChunkData(BaseModel):
    local_path: str = Field(..., description="File path of the local file")
    target_path: str = Field(..., description="File path of the target file")
    chunk_size: int = Field(..., description="Size of each chunk")
    chunk_index: int = Field(..., description="Index of current chunk")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

#### class UploadFileResult

```python
class UploadFileResult(BaseResult[UploadFileData])
```

#### class UploadFileStreamResult

```python
class UploadFileStreamResult(BaseResult[UploadFileChunkData])
```

#### class DownloadFileData

```python
class DownloadFileData(BaseModel):
    source_path: str = Field(..., description="File path of the source file")
    local_path: str = Field(..., description="File path of the local file")
    size: int = Field(..., description="File content size in bytes")
```

#### class DownloadFileChunkData

```python
class DownloadFileChunkData(BaseModel):
    source_path: str = Field(..., description="File path of the source file")
    local_path: str = Field(..., description="File path of the local file")
    chunk_size: int = Field(..., description="Size of each chunk")
    chunk_index: int = Field(..., description="Index of current chunk")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

#### class DownloadFileResult

```python
class DownloadFileResult(BaseResult[DownloadFileData])
```

#### class DownloadFileStreamResult

```python
class DownloadFileStreamResult(BaseResult[DownloadFileChunkData])
```

### 列表/搜索结果

#### class ListFilesResult

```python
class ListFilesResult(BaseResult[FileSystemData])
```

#### class ListDirsResult

```python
class ListDirsResult(BaseResult[FileSystemData])
```

#### class SearchFilesResult

```python
class SearchFilesResult(BaseResult[SearchFilesData])
```
