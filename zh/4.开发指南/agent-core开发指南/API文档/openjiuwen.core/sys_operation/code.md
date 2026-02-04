# openjiuwen.core.sys_operation.code

## class openjiuwen.core.sys_operation.code.BaseCodeOperation

```python
class openjiuwen.core.sys_operation.code.BaseCodeOperation()
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