# openjiuwen.core.sys_operation.sys_operation

`openjiuwen.core.sys_operation.sys_operation` 模块定义了系统操作的主要入口类及相关接口。

## class openjiuwen.core.sys_operation.base.OperationMode

```python
class openjiuwen.core.sys_operation.base.OperationMode(str, Enum)
```

系统运行模式枚举。

**参数**：

* **LOCAL**：本地运行模式。值："local"。
* **SANDBOX**：沙箱运行模式。值："sandbox"。

## class openjiuwen.core.sys_operation.config.LocalWorkConfig

```python
class openjiuwen.core.sys_operation.config.LocalWorkConfig(shell_allowlist: Optional[List[str]] = ["echo", ...], work_dir: Optional[str] = None)
```

本地工作环境配置（当模式为 `local` 时使用）。

**参数**：

* **shell_allowlist**(List[str], 可选)：允许执行的 Shell 命令前缀白名单。默认为一组常见命令（如 "echo", "ls", "python", "git" 等）。如果为 `None`，则允许执行所有命令（注意：不安全）。
* **work_dir**(str, 可选)：本地工作目录路径。默认值：`None`。


## class openjiuwen.core.sys_operation.sys_operation.SysOperationCard

```python
class openjiuwen.core.sys_operation.sys_operation.SysOperationCard(mode: OperationMode = OperationMode.LOCAL, work_config: Optional[LocalWorkConfig] = None, gateway_config: Optional[SandboxGatewayConfig] = None)
```

系统操作配置卡片。

**参数**：

* **mode**(OperationMode, 可选)：运行模式。默认值：`OperationMode.LOCAL`。
* **work_config**(LocalWorkConfig, 可选)：本地工作配置（当模式为 `local` 时必需）。默认值：`None`。
* **gateway_config**(SandboxGatewayConfig, 可选)：沙箱网关配置（当模式为 `sandbox` 时必需）。默认值：`None`。

## class openjiuwen.core.sys_operation.sys_operation.SysOperation

```python
class openjiuwen.core.sys_operation.sys_operation.SysOperation(card: SysOperationCard)
```

系统操作入口类，提供对文件、代码和 Shell 操作的访问。

**参数**：

* **card**(SysOperationCard)：系统操作配置卡片。

### fs() -> BaseFsOperation

获取文件系统操作实例。

**返回**：

**BaseFsOperation**，对应的 `fs` 操作实例（本地或沙箱）。

### code() -> BaseCodeOperation

获取代码执行操作实例。

**返回**：

**BaseCodeOperation**，对应的 `code` 操作实例（本地或沙箱）。

### shell() -> BaseShellOperation

获取 Shell 操作实例。

**返回**：

**BaseShellOperation**，对应的 `shell` 操作实例（本地或沙箱）。

## class openjiuwen.core.sys_operation.fs.BaseFsOperation

```python
class openjiuwen.core.sys_operation.fs.BaseFsOperation
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

## class openjiuwen.core.sys_operation.code.BaseCodeOperation

```python
class openjiuwen.core.sys_operation.code.BaseCodeOperation
```

提供代码执行功能。

### async execute_code(code: str, language: Literal['python', 'javascript'] = "python", time_out: int = 300, environment: Optional[Dict[str, str]] = None, options: Optional[Dict[str, Any]] = None) -> ExecuteCodeResult

异步执行代码。

**参数**：

* **code**(str)：要执行的源代码字符串。
* **language**(Literal['python', 'javascript'], 可选)：编程语言。默认值："python"。
* **time_out**(int, 可选)：最大执行时间（秒）。默认值：300。
* **environment**(Dict[str, str], 可选)：自定义环境变量。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**ExecuteCodeResult**，代码执行结果。

### async execute_code_stream(code: str, language: Literal['python', 'javascript'] = "python", time_out: int = 300, environment: Optional[Dict[str, str]] = None, options: Optional[Dict[str, Any]] = None) -> AsyncIterator[ExecuteCodeStreamResult]

异步流式执行代码。

**参数**：

* **code**(str)：要执行的源代码字符串。
* **language**(Literal['python', 'javascript'], 可选)：编程语言。默认值："python"。
* **time_out**(int, 可选)：最大执行时间（秒）。默认值：300。
* **environment**(Dict[str, str], 可选)：自定义环境变量。
* **options**(Dict[str, Any], 可选)：扩展配置选项。

**返回**：

**AsyncIterator[ExecuteCodeStreamResult]**，流式代码执行结果。

## class openjiuwen.core.sys_operation.shell.BaseShellOperation

```python
class openjiuwen.core.sys_operation.shell.BaseShellOperation
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
