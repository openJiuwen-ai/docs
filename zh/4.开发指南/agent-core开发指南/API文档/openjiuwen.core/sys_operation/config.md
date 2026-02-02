# openjiuwen.core.sys_operation.config

## class LocalWorkConfig

```python
class openjiuwen.core.sys_operation.config.LocalWorkConfig(BaseModel)
```

本地工作模式下的配置，用于约束和描述在本地环境中执行系统操作的行为。

### 字段

- `shell_allowlist: list[str] | None`：
  - 允许执行的命令前缀白名单；
  - 默认值包含了常见的安全命令：`["echo", "ls", "dir", "cd", "pwd", "python", "python3", "pip", "pip3", "npm", "node", "git", "cat", "type", "mkdir", "md", "rm", "rd", "cp", "copy", "mv", "move", "grep", "find", "curl", "wget", "ps", "df", "ping"]`；
  - 若设置为 `None`，则表示不限制（不推荐，存在安全风险）。
- `work_dir: str | None`：
  - 本地工作目录路径；
  - 若为 `None`，则使用默认工作目录。

## class SandboxGatewayConfig

```python
class openjiuwen.core.sys_operation.config.SandboxGatewayConfig(BaseModel)
```

沙箱模式下的远程网关配置，用于描述如何连接到远程沙箱服务。

### 字段

- `gateway_url: str`：
  - 远程沙箱网关服务的 URL，默认为空字符串；
  - 调用方应根据实际部署填入正确的网关地址。
- `params: dict[str, Any]`：
  - 全局请求参数字典，可用于配置通用的请求级参数（如超时、重试等）。
- `auth_headers: dict[str, str]`：
  - 鉴权使用的 HTTP Header 字典，例如 `{"Authorization": "Bearer xxx"}`。
- `auth_query_params: dict[str, str]`：
  - 鉴权使用的 QueryString 参数字典，例如 `{"token": "xxx"}`。

> 这两个配置类都继承自 `pydantic.BaseModel`，支持基于字段名的构造和自动校验，可直接用于序列化、反序列化等场景。

