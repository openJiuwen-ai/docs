## openjiuwen.core.sys_operation

`openjiuwen.core.sys_operation` 模块提供对底层系统资源的**受控访问能力**，主要包括：

- 文件系统操作（读写文件、列目录等）；
- 代码执行（如执行 Python 代码块）；
- Shell 命令执行；
- 本地模式与沙箱模式的隔离。

这些能力最终通过 `SysOperation` 统一对外暴露，并被 `SkillToolKit` 等高级能力复用。

### 对外导出的核心类型

对应源码入口：`openjiuwen.core.sys_operation.__init__`：

- `OperationMode`（来自 `base.py`）：
  - 表示系统操作的运行模式枚举，如本地执行、本地沙箱等。
- `LocalWorkConfig`（来自 `config.py`）：
  - 本地模式下的工作配置（如根目录、权限范围等）。
- `SandboxGatewayConfig`（来自 `config.py`）：
  - 沙箱模式的网关配置（如沙箱服务地址、认证信息等）。
- `SysOperationCard`（来自 `sys_operation.py`）：
  - 系统操作实例的“卡片”元数据，描述一个 SysOperation 的 id、模式、配置等。
- `SysOperation`（来自 `sys_operation.py`）：
  - 系统操作入口类，封装了对文件系统、代码执行、Shell 的实际访问。

### SysOperation 能力概览

完整实现分布在多个子模块中，这里从使用视角概述其能力：

- **文件系统相关（`fs` 子接口）**：
  - 功能：读取文件、写入文件、列出目录等。
  - 用法示例（伪代码）：
    ```python
    sys_op = SysOperation(...)
    fs = sys_op.fs()
    res = await fs.read_file("/path/to/file.txt", mode="text")
    content = res.data.content
    ```

- **代码执行相关（`code` 子接口）**：
  - 功能：执行一段代码（例如 Python），并返回 stdout/stderr 等结果。
  - 用法示例（伪代码）：
    ```python
    code = sys_op.code()
    res = await code.execute_code("print('hello')", language="python")
    stdout = res.data.stdout
    stderr = res.data.stderr
    ```

- **Shell 命令相关（`shell` 子接口）**：
  - 功能：执行 Shell 命令（如 bash），返回命令输出。
  - 用法示例（伪代码）：
    ```python
    shell = sys_op.shell()
    res = await shell.execute_cmd("ls -l")
    stdout = res.data.stdout
    stderr = res.data.stderr
    ```

内部还有两套实现：

- `local` 子包：针对本地环境的 `code_operation/fs_operation/shell_operation`；
- `sandbox` 子包：针对沙箱环境的 `code_operation/fs_operation/shell_operation` 与 `sandbox_gateway`；

二者通过 `OperationMode` 和 `*Config` 进行切换和配置。

### 与其他模块的关系

- `SkillManager` / `SkillToolKit` 会使用 `SysOperation` 来：
  - 读取技能目录中的 `Skill.md`；
  - 执行 Python 代码、Shell 命令；
  - 通过这些操作构建技能相关的 Tool。
- Runner 的资源管理器会注册 `SysOperation` 实例，并通过 `sys_operation_id` 将其暴露给上层功能。

在文档使用上，你可以把 `openjiuwen.core.sys_operation` 理解为“**受控系统操作层**”，为上层 Agent 能力（技能、工具、调试等）提供统一的安全抽象。

