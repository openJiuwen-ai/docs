# openjiuwen.core.sys_operation.sys_operation

`openjiuwen.core.sys_operation.sys_operation` 模块定义了系统操作的主要入口类及相关接口。

## 配置类

### class OperationMode

```python
class OperationMode(str, Enum)
```

系统运行模式枚举。

- `LOCAL = "local"`: 本地运行模式。
- `SANDBOX = "sandbox"`: 沙箱运行模式。

### class LocalWorkConfig

```python
class LocalWorkConfig(BaseModel)
```

本地工作环境配置（当模式为 `local` 时使用）。

- `shell_allowlist: Optional[List[str]]`
  - 允许执行的 Shell 命令前缀白名单。
  - 默认为一组常见命令（如 "echo", "ls", "python", "git" 等）。
  - 如果为 `None`，则允许执行所有命令（注意：不安全）。
- `work_dir: Optional[str]`
  - 本地工作目录路径。
  - 默认为 `None`。

### class SandboxGatewayConfig

```python
class SandboxGatewayConfig(BaseModel)
```

远程沙箱网关连接配置（当模式为 `sandbox` 时使用）。

- `gateway_url: str`
  - 远程沙箱网关服务地址。
  - 默认为 `""`。
- `params: Dict[str, Any]`
  - 全局请求参数。
  - 默认为空字典。
- `auth_headers: Dict[str, str]`
  - 认证 HTTP 头。
  - 默认为空字典。
- `auth_query_params: Dict[str, str]`
  - 认证查询参数。
  - 默认为空字典。

### class SysOperationCard

```python
class SysOperationCard(BaseCard)
```

系统操作配置卡片。

- `mode: OperationMode`
  - 运行模式，可选值：`local` / `sandbox`。
  - 默认为 `OperationMode.LOCAL`。
- `work_config: Optional[LocalWorkConfig]`
  - 本地工作配置（当模式为 `local` 时必需）。
  - 默认为 `None`。
- `gateway_config: Optional[SandboxGatewayConfig]`
  - 沙箱网关配置（当模式为 `sandbox` 时必需）。
  - 默认为 `None`。

## 入口类

### class SysOperation

```python
class SysOperation
```

系统操作入口类，提供对文件、代码和 Shell 操作的访问。

#### \_\_init\_\_

```python
def __init__(self, card: SysOperationCard)
```

初始化系统操作实例。

**参数**：

- `card`: 系统操作配置卡片 (`SysOperationCard`)。

#### fs

```python
def fs(self) -> BaseFsOperation
```

获取文件系统操作实例。

**返回**：

- 对应的 `fs` 操作实例（本地或沙箱）。

#### code

```python
def code(self) -> BaseCodeOperation
```

获取代码执行操作实例。

**返回**：

- 对应的 `code` 操作实例（本地或沙箱）。

#### shell

```python
def shell(self) -> BaseShellOperation
```

获取 Shell 操作实例。

**返回**：

- 对应的 `shell` 操作实例（本地或沙箱）。

## 操作接口

### 文件系统操作 (FsOperation)

提供文件读写、上传下载、列表搜索等功能。

#### read_file

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

#### read_file_stream

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

使用指定模式和参数异步流式读取文件。

#### write_file

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

异步将内容写入文件。

**参数**：
- `path`: 要写入的文件的完整路径或相对路径（必需）。
- `content`: 要写入的数据（文本模式为字符串，二进制模式为字节）。
- `mode`: 写入模式：`'text'` (默认) 或 `'bytes'`。
- `prepend_newline`: 内容前添加换行符（仅文本模式；默认：True）。
- `append_newline`: 内容后添加换行符（仅文本模式；默认：False）。
- `create_if_not_exist`: 若文件不存在则自动创建（默认：True）。
- `permissions`: 八进制文件权限（仅 Unix/Linux；默认："644"）。
- `encoding`: 字符编码（默认：utf-8）。

#### upload_file

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

异步上传文件（本地文件 -> 目标路径）。

**参数**：
- `local_path`: 本地源文件路径。
- `target_path`: 上传目标路径。
- `overwrite`: 是否覆盖现有目标文件（默认：False）。
- `create_parent_dirs`: 是否自动创建目标父目录（默认：True）。
- `preserve_permissions`: 是否保留文件权限（默认：True）。
- `chunk_size`: 传输分块大小（默认：1MB）。

#### download_file

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

异步下载文件（源文件 -> 本地路径）。

#### list_files

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

#### list_directories

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

#### search_files

```python
async def search_files(
    self,
    path: str,
    pattern: str,
    exclude_patterns: Optional[List[str]] = None
) -> SearchFilesResult
```

异步搜索指定路径下的文件。

### 代码执行操作 (CodeOperation)

提供代码执行功能。

#### execute_code

```python
async def execute_code(
    self,
    code: str,
    *,
    language: Literal['python', 'javascript'] = "python",
    time_out: int = 300,
    environment: Optional[Dict[str, str]] = None,
    options: Optional[Dict[str, Any]] = None
) -> ExecuteCodeResult
```

异步执行任意代码。

**参数**：
- `code`: 源代码字符串（必需）。
- `language`: 编程语言，`'python'` 或 `'javascript'`（默认：`'python'`）。
- `time_out`: 最大执行时间（秒）（默认：300）。
- `environment`: 自定义环境变量。
- `options`: 额外配置。

#### execute_code_stream

```python
async def execute_code_stream(
    self,
    code: str,
    *,
    language: Literal['python', 'javascript'] = "python",
    time_out: int = 300,
    environment: Optional[Dict[str, str]] = None,
    options: Optional[Dict[str, Any]] = None
) -> AsyncIterator[ExecuteCodeStreamResult]
```

以流式方式异步执行任意代码。

### Shell 操作 (ShellOperation)

提供 Shell 命令执行功能。

#### execute_cmd

```python
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

异步执行 Shell 命令。

**参数**：
- `command`: 要执行的命令。
- `cwd`: 工作目录（默认：当前目录）。
- `timeout`: 超时时间（秒）（默认：300）。
- `environment`: 自定义环境变量。
- `options`: 额外配置。

#### execute_cmd_stream

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

异步流式执行 Shell 命令。
