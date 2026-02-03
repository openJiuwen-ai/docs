# openjiuwen.core.sys_operation.result.fs_operation_result

`openjiuwen.core.sys_operation.result.fs_operation_result` 模块定义了文件系统操作相关的数据模型。

## 读文件相关

### class ReadFileData

```python
class ReadFileData(BaseModel):
    path: str = Field(..., description="File path of the read file")
    content: Union[str, bytes] = Field(..., description="File content (text string or binary bytes)")
    mode: Literal['text', 'bytes'] = Field(..., description="File read mode: 'text' (string) or 'bytes' (binary)")
```

读文件结果数据。

- `path: str`: 读取的文件路径。
- `content: Union[str, bytes]`: 文件内容（文本字符串或二进制字节）。
- `mode: Literal['text', 'bytes']`: 文件读取模式：`'text'` (字符串) 或 `'bytes'` (二进制)。

### class ReadFileChunkData

```python
class ReadFileChunkData(BaseModel):
    path: str = Field(..., description="File path of the read file")
    chunk_content: Union[str, bytes] = Field(..., description="Current chunk content (text string or binary bytes)")
    mode: Literal['text', 'bytes'] = Field(..., description="File read mode: 'text' (string) or 'bytes' (binary)")
    chunk_size: int = Field(..., description="Size of each chunk (in bytes)")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

流式读文件的分片数据。

- `path: str`: 读取的文件路径。
- `chunk_content: Union[str, bytes]`: 当前分片的内容（文本字符串或二进制字节）。
- `mode: Literal['text', 'bytes']`: 文件读取模式：`'text'` (字符串) 或 `'bytes'` (二进制)。
- `chunk_size: int`: 当前分片的大小（字节数）。
- `chunk_index: int`: 当前分片的序号（从 0 开始）。
- `is_last_chunk: bool`: 是否为最后一个分片。

### class ReadFileResult

```python
class ReadFileResult(BaseResult[ReadFileData])
```

读文件操作的最终结果类型。

### class ReadFileStreamResult

```python
class ReadFileStreamResult(BaseResult[ReadFileChunkData])
```

流式读文件操作的结果类型。

## 写文件相关

### class WriteFileData

```python
class WriteFileData(BaseModel):
    path: str = Field(..., description="File path of the write file")
    size: int = Field(..., description="File content size in bytes")
    mode: Literal['text', 'bytes'] = Field(..., description="File write mode: 'text' (string) or 'bytes' (binary)")
```

写文件结果数据。

- `path: str`: 写入的文件路径。
- `size: int`: 写入的文件内容大小（字节数）。
- `mode: Literal['text', 'bytes']`: 文件写入模式：`'text'` (字符串) 或 `'bytes'` (二进制)。

### class WriteFileResult

```python
class WriteFileResult(BaseResult[WriteFileData])
```

写文件操作的最终结果类型。

## 上传文件相关

### class UploadFileData

```python
class UploadFileData(BaseModel):
    local_path: str = Field(..., description="File path of the local file")
    target_path: str = Field(..., description="File path of the target file")
    size: int = Field(..., description="File content size in bytes")
```

上传文件结果数据。

- `local_path: str`: 本地源文件路径。
- `target_path: str`: 目标文件路径。
- `size: int`: 文件内容大小（字节数）。

### class UploadFileChunkData

```python
class UploadFileChunkData(BaseModel):
    local_path: str = Field(..., description="File path of the local file")
    target_path: str = Field(..., description="File path of the target file")
    chunk_size: int = Field(..., description="Size of each chunk (in bytes)")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

流式上传文件的分片数据。

- `local_path: str`: 本地源文件路径。
- `target_path: str`: 目标文件路径。
- `chunk_size: int`: 当前分片的大小（字节数）。
- `chunk_index: int`: 当前分片的序号（从 0 开始）。
- `is_last_chunk: bool`: 是否为最后一个分片。

### class UploadFileResult

```python
class UploadFileResult(BaseResult[UploadFileData])
```

上传文件操作的最终结果类型。

### class UploadFileStreamResult

```python
class UploadFileStreamResult(BaseResult[UploadFileChunkData])
```

流式上传文件操作的结果类型。

## 下载文件相关

### class DownloadFileData

```python
class DownloadFileData(BaseModel):
    source_path: str = Field(..., description="File path of the source file")
    local_path: str = Field(..., description="File path of the local file")
    size: int = Field(..., description="File content size in bytes")
```

下载文件结果数据。

- `source_path: str`: 源文件路径。
- `local_path: str`: 本地目标路径。
- `size: int`: 文件内容大小（字节数）。

### class DownloadFileChunkData

```python
class DownloadFileChunkData(BaseModel):
    source_path: str = Field(..., description="File path of the source file")
    local_path: str = Field(..., description="File path of the local file")
    chunk_size: int = Field(..., description="Size of each chunk (in bytes)")
    chunk_index: int = Field(..., description="Index of current chunk (starting from 0)")
    is_last_chunk: bool = Field(..., description="Whether current chunk is the last one")
```

流式下载文件的分片数据。

- `source_path: str`: 源文件路径。
- `local_path: str`: 本地目标路径。
- `chunk_size: int`: 当前分片的大小（字节数）。
- `chunk_index: int`: 当前分片的序号（从 0 开始）。
- `is_last_chunk: bool`: 是否为最后一个分片。

### class DownloadFileResult

```python
class DownloadFileResult(BaseResult[DownloadFileData])
```

下载文件操作的最终结果类型。

### class DownloadFileStreamResult

```python
class DownloadFileStreamResult(BaseResult[DownloadFileChunkData])
```

流式下载文件操作的结果类型。

## 文件列表/搜索相关

### class FileSystemItem

```python
class FileSystemItem(BaseModel):
    name: str = Field(..., description="Name of the file/directory")
    path: str = Field(..., description="Full absolute path of the file/directory")
    size: int = Field(..., description="Size in bytes (file size for files; total contents size for directories)")
    modified_time: str = Field(..., description="Last modification time")
    is_directory: bool = Field(..., description="Whether the item is a directory (True) or file (False)")
    type: Optional[str] = Field(default=None, description="File extension (only for files)")
```

文件/目录的基本属性模型。

- `name: str`: 文件或目录的名称。
- `path: str`: 文件或目录的完整绝对路径。
- `size: int`: 大小（字节数）。如果是文件，则为文件大小；如果是目录，则为包含内容的总大小。
- `modified_time: str`: 最后修改时间。
- `is_directory: bool`: 是否为目录（True）或文件（False）。
- `type: Optional[str]`: 文件扩展名（仅针对文件，默认为 None）。

### class FileSystemData

```python
class FileSystemData(BaseModel):
    total_count: int = Field(..., description="Total number of items (files/directories)")
    list_items: List[FileSystemItem] = Field(..., description="List of file/directory details")
    root_path: str = Field(..., description="Original input directory path")
    recursive: bool = Field(..., description="Actual recursive status used")
    max_depth: Optional[int] = Field(default=None, description="Actual maximum recursion depth used")
```

文件列表或目录列表的结果数据。

- `total_count: int`: 项目（文件/目录）总数。
- `list_items: List[FileSystemItem]`: 文件/目录详情列表。
- `root_path: str`: 原始输入的目录路径。
- `recursive: bool`: 实际使用的递归状态。
- `max_depth: Optional[int]`: 实际使用的最大递归深度。

### class SearchFilesData

```python
class SearchFilesData(BaseModel):
    total_matches: int = Field(..., description="Total number of files matching the search pattern")
    matching_files: List[FileSystemItem] = Field(..., description="List of matching files")
    search_path: str = Field(..., description="Original base path used for the search")
    search_pattern: str = Field(..., description="Original search pattern used")
    exclude_patterns: Optional[List[str]] = Field(default=None, description="Original exclude patterns used")
```

搜索文件的结果数据。

- `total_matches: int`: 匹配搜索模式的文件总数。
- `matching_files: List[FileSystemItem]`: 匹配的文件列表。
- `search_path: str`: 搜索使用的原始基础路径。
- `search_pattern: str`: 搜索使用的原始模式。
- `exclude_patterns: Optional[List[str]]`: 搜索使用的原始排除模式列表（默认为 None）。

### class ListFilesResult

```python
class ListFilesResult(BaseResult[FileSystemData])
```

列出文件操作的最终结果类型。

### class ListDirsResult

```python
class ListDirsResult(BaseResult[FileSystemData])
```

列出目录操作的最终结果类型。

### class SearchFilesResult

```python
class SearchFilesResult(BaseResult[SearchFilesData])
```

搜索文件操作的最终结果类型。
