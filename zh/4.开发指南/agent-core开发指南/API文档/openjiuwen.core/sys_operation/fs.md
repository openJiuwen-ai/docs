# openjiuwen.core.sys_operation.fs

`openjiuwen.core.sys_operation.fs` 模块定义了文件系统操作的抽象基类。

## class BaseFsOperation

```python
class BaseFsOperation(BaseOperation, ABC)
```

文件系统操作的基类。

### read_file

```python
@abstractmethod
async def read_file(
    self,
    path: str,
    *,
    mode: Literal['text', 'bytes'] = "text",
    head: Optional[int] = None,
    tail: Optional[int] = None,
    line_range: Optional[Tuple[int, int]] = None,
    encoding: str = "utf-8",
    chunk_size: int = 8192,
    options: Optional[Dict[str, Any]] = None
) -> ReadFileResult
```

使用指定模式和参数异步读取文件。

**参数**：

- `path`: 要读取的文件的完整路径或相对路径（必需）。
- `mode`: 读取模式 - `'text'`（基于行，默认）或 `'bytes'`（原始字节）。
- `head`: 从开头读取的行数（仅限文本模式）。
- `tail`: 从末尾读取的行数（仅限文本模式）。
- `line_range`: 要读取的具体行范围 (start, end) - 从 1 开始，包含首尾（仅限文本模式）。
- `encoding`: 文本模式的字符编码（默认：`utf-8`）。
- `chunk_size`: 字节模式读取的缓冲区大小（默认：8192 字节）。
- `options`: 扩展配置选项（可选）。

**返回**：

- `ReadFileResult`: 结构化结果。

### read_file_stream

```python
@abstractmethod
async def read_file_stream(
    self,
    path: str,
    *,
    mode: Literal['text', 'bytes'] = "text",
    head: Optional[int] = None,
    tail: Optional[int] = None,
    line_range: Optional[Tuple[int, int]] = None,
    encoding: str = "utf-8",
    chunk_size: int = 8192,
    options: Optional[Dict[str, Any]] = None
) -> AsyncIterator[ReadFileStreamResult]
```

使用指定模式和参数异步流式读取文件。

**参数**：

- 同 `read_file`。

**返回**：

- `AsyncIterator[ReadFileStreamResult]`: 流式的结构化结果（逐行或逐块）。

### write_file

```python
@abstractmethod
async def write_file(
    self,
    path: str,
    content: str | bytes,
    *,
    mode: Literal['text', 'bytes'] = "text",
    prepend_newline: bool = True,
    append_newline: bool = False,
    create_if_not_exist: bool = True,
    permissions: str = "644",
    encoding: str = "utf-8",
    options: Optional[Dict[str, Any]] = None
) -> WriteFileResult
```

异步将内容写入文件，配置灵活。

**参数**：

- `path`: 要写入的文件的完整路径或相对路径（必需）。
- `content`: 要写入的数据（文本模式为字符串，二进制模式为字节）。
- `mode`: 写入模式：`'text'` 或 `'bytes'`（默认：`'text'`）。
- `prepend_newline`: 在内容前添加换行符 `\n`（仅限文本模式；默认：`True`）。
- `append_newline`: 在内容后添加换行符 `\n`（仅限文本模式；默认：`False`）。
- `create_if_not_exist`: 如果文件不存在则自动创建（默认：`True`）。
- `permissions`: 八进制文件权限（仅限 Unix/Linux；Windows 上忽略）（默认：`"644"`）。
- `encoding`: 文本模式的字符编码（默认：`utf-8`）。
- `options`: 扩展配置选项。

**返回**：

- `WriteFileResult`: 结构化结果。

### upload_file

```python
@abstractmethod
async def upload_file(
    self,
    local_path: str,
    target_path: str,
    *,
    overwrite: bool = False,
    create_parent_dirs: bool = True,
    preserve_permissions: bool = True,
    chunk_size: int = 1024 * 1024,
    options: Optional[Dict[str, Any]] = None
) -> UploadFileResult
```

异步文件上传（语义：本地文件 -> 目标路径）。

**参数**：

- `local_path`: 本地源文件路径（必需）。
- `target_path`: 上传目标路径（必需）。
- `overwrite`: 是否覆盖现有的目标文件（默认：`False`）。
- `create_parent_dirs`: 是否自动创建目标父目录（默认：`True`）。
- `preserve_permissions`: 是否保留文件权限（默认：`True`，仅限 Unix/Linux）。
- `chunk_size`: 跨文件系统传输的分块大小（默认：1MB）。
- `options`: 扩展配置选项。

**返回**：

- `UploadFileResult`: 结构化结果。

### upload_file_stream

```python
@abstractmethod
async def upload_file_stream(
    self,
    local_path: str,
    target_path: str,
    *,
    overwrite: bool = False,
    create_parent_dirs: bool = True,
    preserve_permissions: bool = True,
    chunk_size: int = 1024 * 1024,
    options: Optional[Dict[str, Any]] = None
) -> AsyncIterator[UploadFileStreamResult]
```

异步流式文件上传。

**参数**：

- 同 `upload_file`。

**返回**：

- `AsyncIterator[UploadFileStreamResult]`: 流式的结构化结果。

### download_file

```python
@abstractmethod
async def download_file(
    self,
    source_path: str,
    local_path: str,
    *,
    overwrite: bool = False,
    create_parent_dirs: bool = True,
    preserve_permissions: bool = True,
    chunk_size: int = 1024 * 1024,
    options: Optional[Dict[str, Any]] = None
) -> DownloadFileResult
```

异步文件下载（语义：源文件 -> 本地目标路径）。

**参数**：

- `source_path`: 源文件路径（必需）。
- `local_path`: 本地目标文件路径（必需）。
- `overwrite`: 是否覆盖现有的目标文件（默认：`False`）。
- `create_parent_dirs`: 是否自动创建目标父目录（默认：`True`）。
- `preserve_permissions`: 是否保留文件权限（默认：`True`，仅限 Unix/Linux）。
- `chunk_size`: 跨文件系统传输的分块大小（默认：1MB）。
- `options`: 扩展配置选项。

**返回**：

- `DownloadFileResult`: 结构化结果。

### download_file_stream

```python
@abstractmethod
async def download_file_stream(
    self,
    source_path: str,
    local_path: str,
    *,
    overwrite: bool = False,
    create_parent_dirs: bool = True,
    preserve_permissions: bool = True,
    chunk_size: int = 1024 * 1024,
    options: Optional[Dict[str, Any]] = None
) -> AsyncIterator[DownloadFileStreamResult]
```

异步流式文件下载。

**参数**：

- 同 `download_file`。

**返回**：

- `AsyncIterator[DownloadFileStreamResult]`: 流式的结构化结果。

### list_files

```python
@abstractmethod
async def list_files(
    self,
    path: str,
    *,
    recursive: bool = False,
    max_depth: Optional[int] = None,
    sort_by: Literal['name', 'modified_time', 'size'] = "name",
    sort_descending: bool = False,
    file_types: Optional[List[str]] = None,
    options: Optional[Dict[str, Any]] = None
) -> ListFilesResult
```

异步列出指定路径下的文件。

**参数**：

- `path`: 目标父目录路径（必需）。
- `recursive`: 是否递归列出子目录中的文件。默认为 `False`。
- `max_depth`: 最大递归深度限制，仅在 `recursive=True` 时有效。
- `sort_by`: 排序字段，支持：`'name'`（默认）, `'modified_time'`, `'size'`。
- `sort_descending`: 是否降序排序。默认为 `False`（升序）。
- `file_types`: 按扩展名过滤文件（扩展名列表），例如 `['.txt', '.pdf']`。
- `options`: 扩展配置选项。

**返回**：

- `ListFilesResult`: 结构化结果。

### list_directories

```python
@abstractmethod
async def list_directories(
    self,
    path: str,
    *,
    recursive: bool = False,
    max_depth: Optional[int] = None,
    sort_by: Literal['name', 'modified_time', 'size'] = "name",
    sort_descending: bool = False,
    options: Optional[Dict[str, Any]] = None
) -> ListDirsResult
```

异步列出指定路径下的目录。

**参数**：

- `path`: 目标父目录路径（必需）。
- `recursive`: 是否递归列出子目录。默认为 `False`。
- `max_depth`: 最大递归深度限制，仅在 `recursive=True` 时有效。
- `sort_by`: 排序字段。
- `sort_descending`: 是否降序排序。
- `options`: 扩展配置选项。

**返回**：

- `ListDirsResult`: 结构化结果。

### search_files

```python
@abstractmethod
async def search_files(
    self,
    path: str,
    pattern: str,
    exclude_patterns: Optional[List[str]] = None
) -> SearchFilesResult
```

异步在指定路径下搜索文件。

**参数**：

- `path`: 开始搜索的基础目录路径（必需）。
- `pattern`: 匹配文件名的搜索模式。
- `exclude_patterns`: 从结果中排除的模式列表（可选）。

**返回**：

- `SearchFilesResult`: 结构化结果。
