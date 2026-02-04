# openjiuwen.core.sys_operation.fs

## class openjiuwen.core.sys_operation.fs.BaseFsOperation

```python
class openjiuwen.core.sys_operation.fs.BaseFsOperation()
```

提供文件读写、上传下载、列表搜索等功能。

### async read_file(path: str, mode: Literal['text', 'bytes'] = "text", head: Optional[int] = None, tail: Optional[int] = None, line_range: Optional[Tuple[int, int]] = None, encoding: str = "utf-8", chunk_size: int = 8192, options: Optional[Dict[str, Any]] = None) -> ReadFileResult

使用指定模式和参数异步读取文件。

**参数**：

* **path**(str)：要读取的文件的完整路径或相对路径。
* **mode**(Literal['text', 'bytes'], 可选)：读取模式。'text'（基于行，默认）或 'bytes'（原始字节）。默认值："text"。
* **head**(int, 可选)：从开头读取的行数（仅限文本模式）。
* **tail**(int, 可选)：从末尾读取的行数（仅限文本模式）。
* **line_range**(Tuple[int, int], 可选)：要读取的具体行范围 (start, end) - 从 1 开始，包含首尾（仅限文本模式）。
* **encoding**(str, 可选)：文本模式的字符编码。默认值："utf-8"。
* **chunk_size**(int, 可选)：字节模式读取的缓冲区大小。默认值：8192。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**ReadFileResult**，读取文件结果。

### async read_file_stream(path: str, mode: Literal['text', 'bytes'] = "text", head: Optional[int] = None, tail: Optional[int] = None, line_range: Optional[Tuple[int, int]] = None, encoding: str = "utf-8", chunk_size: int = 8192, options: Optional[Dict[str, Any]] = None) -> AsyncIterator[ReadFileStreamResult]

使用指定模式和参数异步流式读取文件。

**参数**：

* **path**(str)：要读取的文件的完整路径或相对路径。
* **mode**(Literal['text', 'bytes'], 可选)：读取模式。'text'（基于行，默认）或 'bytes'（原始字节）。默认值："text"。
* **head**(int, 可选)：从开头读取的行数（仅限文本模式）。
* **tail**(int, 可选)：从末尾读取的行数（仅限文本模式）。
* **line_range**(Tuple[int, int], 可选)：要读取的具体行范围 (start, end) - 从 1 开始，包含首尾（仅限文本模式）。
* **encoding**(str, 可选)：文本模式的字符编码。默认值："utf-8"。
* **chunk_size**(int, 可选)：字节模式读取的缓冲区大小。默认值：8192。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**AsyncIterator[ReadFileStreamResult]**，流式读取文件结果。

### async write_file(path: str, content: str | bytes, mode: Literal['text', 'bytes'] = "text", prepend_newline: bool = True, append_newline: bool = False, create_if_not_exist: bool = True, permissions: str = "644", encoding: str = "utf-8", options: Optional[Dict[str, Any]] = None) -> WriteFileResult

异步将内容写入文件。

**参数**：

* **path**(str)：要写入的文件的完整路径或相对路径。
* **content**(str | bytes)：要写入的数据（文本模式为字符串，二进制模式为字节）。
* **mode**(Literal['text', 'bytes'], 可选)：写入模式。'text' (默认) 或 'bytes'。默认值："text"。
* **prepend_newline**(bool, 可选)：内容前添加换行符（仅文本模式）。默认值：True。
* **append_newline**(bool, 可选)：内容后添加换行符（仅文本模式）。默认值：False。
* **create_if_not_exist**(bool, 可选)：若文件不存在则自动创建。默认值：True。
* **permissions**(str, 可选)：八进制文件权限（仅 Unix/Linux）。默认值："644"。
* **encoding**(str, 可选)：字符编码。默认值："utf-8"。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**WriteFileResult**，写入文件结果。

### async upload_file(local_path: str, target_path: str, overwrite: bool = False, create_parent_dirs: bool = True, preserve_permissions: bool = True, chunk_size: int = 1024 * 1024, options: Optional[Dict[str, Any]] = None) -> UploadFileResult

异步上传文件（本地文件 -> 目标路径）。

**参数**：

* **local_path**(str)：本地源文件路径。
* **target_path**(str)：上传目标路径。
* **overwrite**(bool, 可选)：是否覆盖现有目标文件。默认值：False。
* **create_parent_dirs**(bool, 可选)：是否自动创建目标父目录。默认值：True。
* **preserve_permissions**(bool, 可选)：是否保留文件权限。默认值：True。
* **chunk_size**(int, 可选)：传输分块大小。默认值：1MB。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**UploadFileResult**，上传文件结果。

### async upload_file_stream(local_path: str, target_path: str, overwrite: bool = False, create_parent_dirs: bool = True, preserve_permissions: bool = True, chunk_size: int = 1024 * 1024, options: Optional[Dict[str, Any]] = None) -> AsyncIterator[UploadFileStreamResult]

异步流式上传文件（本地文件 -> 目标路径）。

**参数**：

* **local_path**(str)：本地源文件路径。
* **target_path**(str)：上传目标路径。
* **overwrite**(bool, 可选)：是否覆盖现有目标文件。默认值：False。
* **create_parent_dirs**(bool, 可选)：是否自动创建目标父目录。默认值：True。
* **preserve_permissions**(bool, 可选)：是否保留文件权限。默认值：True。
* **chunk_size**(int, 可选)：传输分块大小。默认值：1MB。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**AsyncIterator[UploadFileStreamResult]**，流式上传文件结果。

### async download_file(source_path: str, local_path: str, overwrite: bool = False, create_parent_dirs: bool = True, preserve_permissions: bool = True, chunk_size: int = 1024 * 1024, options: Optional[Dict[str, Any]] = None) -> DownloadFileResult

异步下载文件（源文件 -> 本地路径）。

**参数**：

* **source_path**(str)：源文件路径。
* **local_path**(str)：本地路径。
* **overwrite**(bool, 可选)：是否覆盖现有本地文件。默认值：False。
* **create_parent_dirs**(bool, 可选)：是否自动创建本地父目录。默认值：True。
* **preserve_permissions**(bool, 可选)：是否保留文件权限。默认值：True。
* **chunk_size**(int, 可选)：传输分块大小。默认值：1MB。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**DownloadFileResult**，下载文件结果。

### async download_file_stream(source_path: str, local_path: str, overwrite: bool = False, create_parent_dirs: bool = True, preserve_permissions: bool = True, chunk_size: int = 1024 * 1024, options: Optional[Dict[str, Any]] = None) -> AsyncIterator[DownloadFileStreamResult]

异步流式下载文件（源文件 -> 本地路径）。

**参数**：

* **source_path**(str)：源文件路径。
* **local_path**(str)：本地路径。
* **overwrite**(bool, 可选)：是否覆盖现有本地文件。默认值：False。
* **create_parent_dirs**(bool, 可选)：是否自动创建本地父目录。默认值：True。
* **preserve_permissions**(bool, 可选)：是否保留文件权限。默认值：True。
* **chunk_size**(int, 可选)：传输分块大小。默认值：1MB。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**AsyncIterator[DownloadFileStreamResult]**，流式下载文件结果。

### async list_files(path: str, recursive: bool = False, max_depth: Optional[int] = None, sort_by: Literal['name', 'modified_time', 'size'] = "name", sort_descending: bool = False, file_types: Optional[List[str]] = None, options: Optional[Dict[str, Any]] = None) -> ListFilesResult

异步列出指定路径下的文件。

**参数**：

* **path**(str)：目标父目录路径。
* **recursive**(bool, 可选)：是否递归列出子目录中的文件。默认值：False。
* **max_depth**(int, 可选)：最大递归深度（仅在 recursive=True 时有效）。
* **sort_by**(Literal['name', 'modified_time', 'size'], 可选)：排序字段。默认值："name"。
* **sort_descending**(bool, 可选)：是否降序排列。默认值：False。
* **file_types**(List[str], 可选)：按扩展名过滤文件（如 ['.txt', '.pdf']）。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**ListFilesResult**，文件列表结果。

### async list_directories(path: str, recursive: bool = False, max_depth: Optional[int] = None, sort_by: Literal['name', 'modified_time', 'size'] = "name", sort_descending: bool = False, options: Optional[Dict[str, Any]] = None) -> ListDirsResult

异步列出指定路径下的目录。

**参数**：

* **path**(str)：目标父目录路径。
* **recursive**(bool, 可选)：是否递归列出子目录。默认值：False。
* **max_depth**(int, 可选)：最大递归深度（仅在 recursive=True 时有效）。
* **sort_by**(Literal['name', 'modified_time', 'size'], 可选)：排序字段。默认值："name"。
* **sort_descending**(bool, 可选)：是否降序排列。默认值：False。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**ListDirsResult**，目录列表结果。

### async search_files(path: str, pattern: str, exclude_patterns: Optional[List[str]] = None) -> SearchFilesResult

异步搜索指定路径下的文件。

**参数**：

* **path**(str)：搜索起始目录路径。
* **pattern**(str)：文件名匹配模式。
* **exclude_patterns**(List[str], 可选)：排除模式列表。

**返回**：

**SearchFilesResult**，搜索文件结果。