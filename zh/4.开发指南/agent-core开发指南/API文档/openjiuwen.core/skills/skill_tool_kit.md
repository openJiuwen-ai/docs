# openjiuwen.core.skills.skill_tool_kit

## class SkillToolKit

```python
class openjiuwen.core.skills.skill_tool_kit.SkillToolKit
```

`SkillToolKit` 负责基于 `SysOperation` 创建一组与技能相关的内置工具，并将这些工具注册到 Runner 的资源管理器中，同时挂载到 Agent 的 `ability_kit` 上，方便在推理过程中调用。

### __init__

```python
def __init__(self, sys_operation_id: str)
```

**参数**：

- `sys_operation_id: str`：用于从 `Runner.resource_mgr` 中获取 `SysOperation` 实例的 id。

### 属性 sys_operation_id

```python
@property
def sys_operation_id(self) -> str

@sys_operation_id.setter
def sys_operation_id(self, sys_operation_id: str)
```

用于读取与更新内部使用的 `sys_operation_id`，当上层切换到新的 `SysOperation` 实例时，需要同步更新该字段。

### set_runner

```python
def set_runner(self, runner) -> None
```

预留的 Runner 注入接口，当前实现仅保存引用，方便未来在创建工具时访问 Runner 的上下文。

### 内部方法 _get_sys_operation

```python
def _get_sys_operation(self) -> Any
```

根据当前的 `sys_operation_id`，从 `Runner.resource_mgr` 中获取对应的 `SysOperation` 实例：

- 若 `sys_operation_id` 为空，则返回 `None`；
- 若未能获取到 `SysOperation`，则后续工具函数会返回错误字符串（如 `"sys_operation is not available"`）。

### create_view_file_tool

```python
def create_view_file_tool(self) -> LocalFunction
```

创建一个用于**查看文本文件**的本地工具，返回 `LocalFunction` 实例。

工具卡片定义：

- `id="_internal_view_file"`
- `name="view_file"`
- `description`：说明只读取文本文件，不读取二进制文件（如 `.pdf/.xlsx/.ppt` 等）。
- `input_params`：
  - `file_path: str`：要查看的文件路径。

工具函数行为：

- 调用 `sys_operation.fs().read_file(file_path, mode="text")` 读取文件内容；
- 若内容为二进制（bytes/bytearray），返回提示信息而不是原始二进制；

### create_execute_python_code_tool

```python
def create_execute_python_code_tool(self) -> LocalFunction
```

创建一个用于**执行 Python 代码块**的本地工具。

工具卡片定义：

- `id="_internal_execute_python_code"`
- `name="execute_python_code"`
- `input_params`：
  - `code_block: str`：要执行的 Python 代码。

工具函数行为：

- 调用 `sys_operation.code().execute_code(code_block, language="python")` 执行代码；
- 从返回结果中提取 `stdout`/`stderr` 并拼接为最终输出字符串；
- 若二者均为空，则返回原始结果的字符串形式。

### create_execute_command_tool

```python
def create_execute_command_tool(self) -> LocalFunction
```

创建一个用于**执行 Shell 命令**的本地工具。

工具卡片定义：

- `id="_internal_run_command"`
- `name="run_command"`
- `input_params`：
  - `bash_command: str`：要执行的一条或多条命令字符串。

工具函数行为：

- 调用 `sys_operation.shell().execute_cmd(bash_command)` 执行命令；
- 从返回结果中提取 `stdout`/`stderr` 并返回；
- 若无标准输出/错误，则返回原始结果的字符串形式。

### add_skill_tools

```python
def add_skill_tools(self, agent)
```

为指定的 Agent 创建并注册所有技能相关的内置工具：

1. 依次调用：
   - `create_execute_python_code_tool()`
   - `create_execute_command_tool()`
   - `create_view_file_tool()`
2. 将上述三个工具列表传给 `Runner.resource_mgr.add_tool([...])`，统一注册到资源管理器；
3. 将每个工具的 `card` 对象添加到 `agent.ability_kit` 中，使 Agent 在推理过程中可以通过工具调用这些能力。

> **典型用法**：通常不直接单独调用这些方法，而是通过 `SkillUtil.register_skills(...)` 一次性完成“注册技能 + 挂载工具”的流程。

