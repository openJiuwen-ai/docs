# openjiuwen.core.sys_operation.local.code_operation

`openjiuwen.core.sys_operation.local.code_operation` 模块提供了本地代码执行的具体实现。

## class CodeOperation

```python
class openjiuwen.core.sys_operation.local.code_operation.CodeOperation(BaseCodeOperation)
```

继承自 `BaseCodeOperation`，实现了本地代码执行功能。

注册信息：
- `name`: `"code"`
- `mode`: `OperationMode.LOCAL`
- `description`: `"local code operation"`

### execute_code

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

**参数说明**：
- `code` (str): 包含要执行的源代码的非空字符串（必需的位置参数）。
- `language` (Literal['python', 'javascript']): 代码的编程语言。严格类型约束为 'python' 或 'javascript'。
- `time_out` (int): 最大执行时间（秒）。默认为 300 秒（5 分钟）。
- `environment` (Optional[Dict[str, str]]): 自定义环境变量的键值对字典。
- `options` (Optional[Dict[str, Any]]): 额外的执行配置选项。

**实现细节**：
- 使用 `asyncio.create_subprocess_exec` 创建子进程执行代码。
- 支持 Python 和 JavaScript (Node.js) 环境。
- 包含超时控制机制，超时后会自动终止进程。
- 捕获 stdout 和 stderr 输出。

### execute_code_stream

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

**参数说明**：
- `code` (str): 包含要执行的源代码的非空字符串（必需的位置参数）。
- `language` (Literal['python', 'javascript']): 代码的编程语言。严格类型约束为 'python' 或 'javascript'。默认为 "python"。
- `time_out` (int): 最大执行时间（秒）。如果超时则终止进程。必须为正整数。默认为 300 秒（5 分钟）。
- `environment` (Optional[Dict[str, str]]): 自定义环境变量的键值对字典。
- `options` (Optional[Dict[str, Any]]): 额外的执行配置选项。
