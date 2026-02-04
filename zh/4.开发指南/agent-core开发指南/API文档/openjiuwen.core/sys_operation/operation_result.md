# openjiuwen.core.sys_operation.operation_result

`openjiuwen.core.sys_operation.operation_result` 模块定义了系统操作的所有返回结果数据结构。

## class openjiuwen.core.sys_operation.result.base_result.BaseResult

```python
class openjiuwen.core.sys_operation.result.base_result.(BaseModel, Generic[T], ABC)
```

所有操作结果的通用泛型基类。

**参数**：

* **code**(int)：状态码。0 表示成功，非 0 表示失败。
* **message**(str)：消息详情。
* **data**(Optional[T], 可选)：业务数据载荷（仅在成功时返回）。默认值：None。

## class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeData

```python
class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeData(BaseModel)
```

代码执行结果数据。

**参数**：

* **code_content**(str)：执行的原始代码。
* **language**(str)：原始代码的编程语言。
* **exit_code**(int, 可选)：执行退出码。默认值：0。
* **stdout**(str, 可选)：标准输出流。默认值：""。
* **stderr**(str, 可选)：标准错误流。默认值：""。

## class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeChunkData

```python
class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeChunkData(BaseModel)
```

流式执行时的分片数据。

**参数**：

* **text**(str, 可选)：输出分片的原始内容。默认值：""。
* **type**(Literal["stdout", "stderr"])：输出分片的类型。
* **chunk_index**(int)：当前分片的索引（从 0 开始）。
* **exit_code**(int, 可选)：执行退出码。默认值：0。
* **metadata**(Dict[str, Any], 可选)：执行数据。默认值：None。

## class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeResult

```python
class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeResult(BaseResult[ExecuteCodeData])
```

代码执行的最终结果类型。

## class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeStreamResult

```python
class openjiuwen.core.sys_operation.result.code_operation_result.ExecuteCodeStreamResult(BaseResult[ExecuteCodeChunkData])
```

代码流式执行的结果类型。

## class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdData

```python
class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdData(BaseModel)
```

Shell 命令执行结果数据。

**参数**：

* **command**(str)：执行的原始 Shell 命令。
* **cwd**(str, 可选)：当前工作目录。默认值："."。
* **exit_code**(int, 可选)：命令退出码。默认值：0。
* **stdout**(str, 可选)：标准输出流。默认值：""。
* **stderr**(str, 可选)：标准错误流。默认值：""。

## class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdChunkData

```python
class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdChunkData(BaseModel)
```

Shell 流式执行分片数据。

**参数**：

* **text**(str, 可选)：输出分片的原始内容。默认值：""。
* **type**(Literal["stdout", "stderr"])：输出分片的类型。
* **chunk_index**(int)：当前分片的索引（从 0 开始）。
* **exit_code**(int, 可选)：命令退出码。默认值：0。
* **metadata**(Dict[str, Any], 可选)：命令数据。默认值：None。

## class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdResult

```python
class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdResult(BaseResult[ExecuteCmdData])
```

Shell 执行的最终结果类型。

## class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdStreamResult

```python
class openjiuwen.core.sys_operation.result.shell_operation_result.ExecuteCmdStreamResult(BaseResult[ExecuteCmdChunkData])
```

Shell 流式执行的结果类型。

## class openjiuwen.core.sys_operation.result.fs_operation_result.FileSystemItem

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.FileSystemItem(BaseModel)
```

文件或目录的基本属性。

**参数**：

* **name**(str)：文件/目录名称。
* **path**(str)：文件/目录的完整绝对路径。
* **size**(int)：大小（字节）。
* **modified_time**(str)：最后修改时间。
* **is_directory**(bool)：是否为目录。
* **type**(str, 可选)：文件扩展名（仅文件有）。默认值：None。

## class openjiuwen.core.sys_operation.result.fs_operation_result.FileSystemData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.FileSystemData(BaseModel)
```

列出文件或目录的结果数据。

**参数**：

* **total_count**(int)：项目总数。
* **list_items**(List[FileSystemItem])：文件/目录详情列表。
* **root_path**(str)：原始输入目录路径。
* **recursive**(bool)：是否递归。
* **max_depth**(int, 可选)：最大递归深度。默认值：None。

## class openjiuwen.core.sys_operation.result.fs_operation_result.SearchFilesData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.SearchFilesData(BaseModel)
```

搜索文件的结果数据。

**参数**：

* **total_matches**(int)：匹配文件总数。
* **matching_files**(List[FileSystemItem])：匹配文件列表。
* **search_path**(str)：搜索起始路径。
* **search_pattern**(str)：搜索模式。
* **exclude_patterns**(List[str], 可选)：排除模式列表。默认值：None。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileData(BaseModel)
```

读取文件数据。

**参数**：

* **path**(str)：文件路径。
* **content**(Union[str, bytes])：文件内容。
* **mode**(Literal['text', 'bytes'])：读取模式。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileChunkData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileChunkData(BaseModel)
```

读取文件分片数据。

**参数**：

* **path**(str)：文件路径。
* **chunk_content**(Union[str, bytes])：当前分片内容。
* **mode**(Literal['text', 'bytes'])：读取模式。
* **chunk_size**(int)：分片大小。
* **chunk_index**(int)：分片索引。
* **is_last_chunk**(bool)：是否为最后一个分片。

## class openjiuwen.core.sys_operation.result.fs_operation_result.WriteFileData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.WriteFileData(BaseModel)
```

写入文件数据。

**参数**：

* **path**(str)：文件路径。
* **size**(int)：文件内容大小（字节）。
* **mode**(Literal['text', 'bytes'])：写入模式。

## class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileData(BaseModel)
```

上传文件数据。

**参数**：

* **local_path**(str)：本地文件路径。
* **target_path**(str)：目标文件路径。
* **size**(int)：文件内容大小（字节）。

## class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileChunkData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileChunkData(BaseModel)
```

上传文件分片数据。

**参数**：

* **local_path**(str)：本地文件路径。
* **target_path**(str)：目标文件路径。
* **chunk_size**(int)：分片大小。
* **chunk_index**(int)：分片索引。
* **is_last_chunk**(bool)：是否为最后一个分片。

## class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileData(BaseModel)
```

下载文件数据。

**参数**：

* **source_path**(str)：源文件路径。
* **local_path**(str)：本地文件路径。
* **size**(int)：文件内容大小（字节）。

## class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileChunkData

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileChunkData(BaseModel)
```

下载文件分片数据。

**参数**：

* **source_path**(str)：源文件路径。
* **local_path**(str)：本地文件路径。
* **chunk_size**(int)：分片大小。
* **chunk_index**(int)：分片索引。
* **is_last_chunk**(bool)：是否为最后一个分片。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileResult(BaseResult[ReadFileData])
```

读取文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileStreamResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ReadFileStreamResult(BaseResult[ReadFileChunkData])
```

流式读取文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.WriteFileResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.WriteFileResult(BaseResult[WriteFileData])
```

写入文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileResult(BaseResult[UploadFileData])
```

上传文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileStreamResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.UploadFileStreamResult(BaseResult[UploadFileChunkData])
```

流式上传文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileResult(BaseResult[DownloadFileData])
```

下载文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileStreamResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.DownloadFileStreamResult(BaseResult[DownloadFileChunkData])
```

流式下载文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ListFilesResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ListFilesResult(BaseResult[FileSystemData])
```

列出文件结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.ListDirsResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.ListDirsResult(BaseResult[FileSystemData])
```

列出目录结果。

## class openjiuwen.core.sys_operation.result.fs_operation_result.SearchFilesResult

```python
class openjiuwen.core.sys_operation.result.fs_operation_result.SearchFilesResult(BaseResult[SearchFilesData])
```

搜索文件结果。
