# openjiuwen.core.sys_operation.config

`openjiuwen.core.sys_operation.config` 模块定义了本地工作环境和沙箱网关的配置类。

## class LocalWorkConfig

```python
class LocalWorkConfig(BaseModel)
```

本地工作环境配置。

- `shell_allowlist: Optional[List[str]]`
  - 允许执行的 Shell 命令前缀白名单。
  - 默认为一组常见命令（如 "echo", "ls", "python", "git" 等）。
  - 如果为 `None`，则允许执行所有命令（注意：不安全）。
- `work_dir: Optional[str]`
  - 本地工作目录路径。
  - 默认为 `None`。

## class SandboxGatewayConfig

```python
class SandboxGatewayConfig(BaseModel)
```

远程沙箱网关连接配置。

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
