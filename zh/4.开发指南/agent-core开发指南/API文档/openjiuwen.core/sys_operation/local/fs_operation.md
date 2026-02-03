# openjiuwen.core.sys_operation.local.fs_operation

`openjiuwen.core.sys_operation.local.fs_operation` 模块提供了本地文件系统操作的具体实现。

## class FsOperation

```python
class openjiuwen.core.sys_operation.local.fs_operation.FsOperation(BaseFsOperation)
```

继承自 `BaseFsOperation`，实现了本地文件系统的读写操作。

注册信息：
- `name`: `"fs"`
- `mode`: `OperationMode.LOCAL`
- `description`: `"local fs operation"`

### read_file

```python
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

异步读取本地文件。

**参数说明**：
- `path` (str): 要读取的文件的完整路径或相对路径（必需）。
- `mode` (Literal['text', 'bytes']): 读取模式 - "text" (基于行，默认) 或 "bytes" (原始字节)。
- `head` (Optional[int]): 从开头读取的行数（仅限文本模式）。
- `tail` (Optional[int]): 从末尾读取的行数（仅限文本模式）。
- `line_range` (Optional[Tuple[int, int]]): 要读取的具体行范围 (start, end) - 1-based，包含首尾（仅限文本模式）。
- `encoding` (str): 文本模式的字符编码（默认：utf-8）。
- `chunk_size` (int): 字节模式读取的缓冲区大小（默认：8192 字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

**实现细节**：
- 使用 `aiofiles` 进行异步文件操作。
- 支持全量读取优化。
- 支持 `head`, `tail`, `line_range` 等分片读取参数。
- 支持文本模式和二进制模式。

### read_file_stream

```python
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

异步流式读取本地文件。

**参数说明**：
- `path` (str): 要读取的文件的完整路径或相对路径（必需）。
- `mode` (Literal['text', 'bytes']): 读取模式 - "text" (基于行，默认) 或 "bytes" (原始字节)。
- `head` (Optional[int]): 从开头读取的行数（仅限文本模式）。
- `tail` (Optional[int]): 从末尾读取的行数（仅限文本模式）。
- `line_range` (Optional[Tuple[int, int]]): 要读取的具体行范围 (start, end) - 1-based，包含首尾（仅限文本模式）。
- `encoding` (str): 文本模式的字符编码（默认：utf-8）。
- `chunk_size` (int): 字节模式读取的缓冲区大小（默认：8192 字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### write_file

```python
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

异步写入本地文件。

**参数说明**：
- `path` (str): 要写入的文件的完整路径或相对路径（必需）。
- `content` (str | bytes): 要写入文件的数据（文本模式为字符串，二进制模式为字节）。
- `mode` (Literal['text', 'bytes']): 写入模式 - "text" (用于字符串内容，默认) 或 "bytes" (用于二进制数据)。
- `prepend_newline` (bool): 在内容前添加换行符 `\n`（仅限文本模式；默认：True）。
- `append_newline` (bool): 在内容后添加换行符 `\n`（仅限文本模式；默认：False）。
- `create_if_not_exist` (bool): 如果文件不存在，是否自动创建（默认：True）。
- `permissions` (str): 八进制文件权限（仅限 Unix/Linux；Windows 上忽略）（默认："644"）。
- `encoding` (str): 文本模式的字符编码（默认：utf-8）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

**实现细节**：
- 自动处理父目录创建。
- 支持写入前检查文件是否存在。
- 支持文本追加换行符。
- 支持设置文件权限（Unix/Linux）。

### upload_file

```python
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

本地文件上传（复制）。在 Local 模式下，语义等同于本地文件复制。

**参数说明**：
- `local_path` (str): 本地源文件路径（必需，例如 /tmp/local_file.txt）。
- `target_path` (str): 上传目标路径（必需，例如 /mnt/storage/file.txt 或 sandbox:/opt/bucket/file.txt）。
- `overwrite` (bool): 是否覆盖现有的目标文件（默认：False）。
- `create_parent_dirs` (bool): 是否自动创建目标父目录（默认：True）。
- `preserve_permissions` (bool): 是否保留文件权限（默认：True，仅限 Unix/Linux）。
- `chunk_size` (int): 跨文件系统传输的块大小（默认：1MB，字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### upload_file_stream

```python
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

流式上传（复制）文件。

**参数说明**：
- `local_path` (str): 本地源文件路径（必需，例如 /tmp/local_file.txt）。
- `target_path` (str): 上传目标路径（必需，例如 /mnt/storage/file.txt 或 sandbox:/opt/bucket/file.txt）。
- `overwrite` (bool): 是否覆盖现有的目标文件（默认：False）。
- `create_parent_dirs` (bool): 是否自动创建目标父目录（默认：True）。
- `preserve_permissions` (bool): 是否保留文件权限（默认：True，仅限 Unix/Linux）。
- `chunk_size` (int): 跨文件系统传输的块大小（默认：1MB，字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### download_file

```python
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

异步下载文件（语义：源文件 → 本地目标路径）。

**参数说明**：
- `source_path` (str): 源文件路径（必需，例如 /mnt/storage/file.txt 或 sandbox:/opt/bucket/file.txt）。
- `local_path` (str): 本地目标文件路径（必需，例如 /home/user/downloads/file.txt）。
- `overwrite` (bool): 是否覆盖现有的目标文件（默认：False）。
- `create_parent_dirs` (bool): 是否自动创建目标父目录（默认：True）。
- `preserve_permissions` (bool): 是否保留文件权限（默认：True，仅限 Unix/Linux）。
- `chunk_size` (int): 跨文件系统传输的块大小（默认：1MB，字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### download_file_stream

```python
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

异步流式下载文件。

**参数说明**：
- `source_path` (str): 源文件路径（必需，例如 /mnt/storage/file.txt 或 sandbox:/opt/bucket/file.txt）。
- `local_path` (str): 本地目标文件路径（必需，例如 /home/user/downloads/file.txt）。
- `overwrite` (bool): 是否覆盖现有的目标文件（默认：False）。
- `create_parent_dirs` (bool): 是否自动创建目标父目录（默认：True）。
- `preserve_permissions` (bool): 是否保留文件权限（默认：True，仅限 Unix/Linux）。
- `chunk_size` (int): 跨文件系统传输的块大小（默认：1MB，字节）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### list_files

```python
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

**参数说明**：
- `path` (str): 目标父目录路径（必需）。
- `recursive` (bool): 是否递归列出子目录中的文件（默认：False）。
- `max_depth` (Optional[int]): 最大递归深度限制，仅在 recursive=True 时有效。
- `sort_by` (Literal['name', 'modified_time', 'size']): 排序字段，支持：'name' (文件名, 默认), 'modified_time' (最后修改时间), 'size' (文件大小)。
- `sort_descending` (bool): 是否降序排序（默认：False，升序）。
- `file_types` (Optional[List[str]]): 按扩展名筛选文件（扩展名列表），例如 ['.txt', '.pdf']。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### list_directories

```python
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

**参数说明**：
- `path` (str): 目标父目录路径（必需）。
- `recursive` (bool): 是否递归列出子目录（默认：False）。
- `max_depth` (Optional[int]): 最大递归深度限制，仅在 recursive=True 时有效。
- `sort_by` (Literal['name', 'modified_time', 'size']): 排序字段，支持：'name' (文件名, 默认), 'modified_time' (最后修改时间), 'size' (文件大小)。
- `sort_descending` (bool): 是否降序排序（默认：False，升序）。
- `options` (Optional[Dict[str, Any]]): 扩展配置选项。

### search_files

```python
async def search_files(
    self,
    path: str,
    pattern: str,
    exclude_patterns: Optional[List[str]] = None
) -> SearchFilesResult
```

异步搜索指定路径下的文件。

**参数说明**：
- `path` (str): 开始搜索的基础目录路径（必需）。
- `pattern` (str): 匹配文件名的搜索模式 (glob pattern)。
- `exclude_patterns` (Optional[List[str]]): 从结果中排除的模式列表。
