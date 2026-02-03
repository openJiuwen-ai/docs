# openjiuwen.core.sys_operation.code

`openjiuwen.core.sys_operation.code` 模块定义了代码执行操作的抽象基类。

## class BaseCodeOperation

```python
class BaseCodeOperation(BaseOperation, ABC)
```

代码执行操作的基类。

### execute_code

```python
@abstractmethod
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

- `code`: 包含要执行的源代码的非空字符串（必需）。
- `language`: 代码的编程语言。严格限制为 `'python'` 或 `'javascript'`。默认为 `'python'`。
- `time_out`: 最大执行时间（秒）。默认为 300 秒（5 分钟）。
- `environment`: 自定义环境变量的键值字典。
- `options`: 额外的执行配置选项。

**返回**：

- `ExecuteCodeResult`: 执行结果。

### execute_code_stream

```python
@abstractmethod
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

**参数**：

- `code`: 包含要执行的源代码的非空字符串（必需）。
- `language`: 代码的编程语言。严格限制为 `'python'` 或 `'javascript'`。默认为 `'python'`。
- `time_out`: 最大执行时间（秒）。如果超时则终止进程。默认为 300 秒。
- `environment`: 自定义环境变量的键值字典。
- `options`: 额外的执行配置选项。

**返回**：

- `AsyncIterator[ExecuteCodeStreamResult]`: 流式的结构化结果。
