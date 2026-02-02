# openjiuwen.core.sys_operation.sys_operation

## class SysOperationCard

```python
class openjiuwen.core.sys_operation.sys_operation.SysOperationCard(BaseCard)
```

描述一个 `SysOperation` 实例的配置卡片。

### 字段

- `mode: OperationMode`：
  - 运行模式，默认 `OperationMode.LOCAL`；
  - 可选值：`"local"` / `"sandbox"`。
- `work_config: LocalWorkConfig | None`：
  - 本地模式配置；
  - 当 `mode == OperationMode.LOCAL` 时，推荐提供该配置；
  - 若为空，则在 `SysOperation` 内部使用默认的 `LocalWorkConfig()`。
- `gateway_config: SandboxGatewayConfig | None`：
  - 沙箱模式配置；
  - 当 `mode == OperationMode.SANDBOX` 时，推荐提供该配置；
  - 若为空，则在 `SysOperation` 内部使用默认的 `SandboxGatewayConfig()`。

### 字段校验

`mode` 字段带有 `field_validator`：

- 若传入的值不是 `OperationMode` 枚举实例，则会尝试：
  - 将字符串转为小写后构造 `OperationMode(v.lower())`；
  - 如果转换失败，则抛出由 `build_error(StatusCode.SYS_OPERATION_CARD_PARAM_ERROR, ...)` 构造的异常，错误信息中会列出允许的枚举值。

## class SysOperation

```python
class openjiuwen.core.sys_operation.sys_operation.SysOperation
```

系统操作入口类，统一封装文件系统、代码执行、Shell 命令等能力。

### __init__

```python
def __init__(self, card: SysOperationCard)
```

**参数**：

- `card: SysOperationCard`：包含运行模式及工作配置的卡片对象。

**初始化行为**：

- 读取 `card.mode` 确定运行模式：
  - 若为 `OperationMode.LOCAL`：
    - 使用 `card.work_config` 或默认的 `LocalWorkConfig()` 作为运行配置；
  - 否则（沙箱模式）：
    - 使用 `card.gateway_config` 或默认的 `SandboxGatewayConfig()` 作为运行配置。
- 初始化内部实例缓存字典 `_instances: dict[str, Any] = {}`，用于按名称缓存各类 Operation 实例。

### __getattr__

```python
def __getattr__(self, name)
```

对未知属性访问进行拦截，等价于调用 `_get_operation(name)`：

- 例如访问 `sys_op.fs` / `sys_op.code` / `sys_op.shell` 时，会动态返回对应的操作实例。

### fs / code / shell

```python
def fs(self)
def code(self)
def shell(self)
```

三个便捷方法，分别等价于：

- `self._get_operation("fs")`
- `self._get_operation("code")`
- `self._get_operation("shell")`

返回结果通常是继承自 `BaseOperation` 的具体操作类实例。

### _get_operation

```python
def _get_operation(self, name: str)
```

内部方法，用于按名称获取/创建对应的 Operation 实例。

**行为**：

1. 若 `name` 已在 `_instances` 缓存中，则直接返回缓存实例；
2. 否则调用 `OperationRegistry.get_operation_info(name, self.mode)` 获取注册信息：
   - 若返回 `None`，表示没有该名称的操作，直接返回 `None`；
3. 从返回的 `operation_info` 中读取：
   - `operation_cls`：操作类；
   - `operation_desc`：描述字符串。
4. 校验 `operation_cls` 是否为可调用的类型（`isinstance(operation_cls, type) and callable(operation_cls)`）；
5. 使用 `operation_cls(name, self.mode, operation_desc, self._run_config)` 创建实例，并写入 `_instances[name]`；
6. 返回新创建的实例。

> 通过 `OperationRegistry`，可以为不同模式（LOCAL/SANDBOX）注册不同的操作实现，实现灵活的本地/远程切换。

