# openjiuwen.core.skills.skill_manager

## class Skill

```python
class openjiuwen.core.skills.skill_manager.Skill(BaseModel)
```

表示单个技能（Skill）的元数据信息。

**字段**：

- `name: str`：技能名称，一般对应技能目录名。
- `description: str | None`：技能描述，从 `Skill.md` 的 YAML front matter 中解析得到。
- `directory: pathlib.Path`：技能所在目录路径。

## class SkillManager

```python
class openjiuwen.core.skills.skill_manager.SkillManager
```

负责技能元数据的注册、查询与管理。

### __init__

```python
def __init__(self, sys_operation_id: str)
``>

**参数**：

- `sys_operation_id: str`：用于从 Runner 的资源管理器中获取 `SysOperation` 实例的 id（`Runner.resource_mgr.get_sys_operation(sys_operation_id)`）。

### set_sys_operation_id

```python
def set_sys_operation_id(self, sys_operation_id: str) -> None
```

更新内部使用的 `sys_operation_id`，后续文件系统操作都会基于新的 SysOperation 实例执行。

### register

```python
async def register(
    self,
    skill_path: Union[Path, List[Path]],
    session_id: str = None,
    overwrite: bool = False
) -> None
```

注册一个或多个技能目录。

**参数**：

- `skill_path`：
  - 若为单个 `Path`：表示技能根目录，内部会遍历其子目录，并在每个包含 `Skill.md` 的目录上创建一个 `Skill` 实例并注册；
  - 若为 `List[Path]`：对列表中的每个路径执行同样的逻辑。
- `session_id: str | None`：执行文件操作时使用的会话 id，用于 SysOperation。
- `overwrite: bool`：
  - 为 `False` 且技能已存在时，会抛出 `ValueError`；
  - 为 `True` 时，允许覆盖已存在的技能记录。

**行为说明**：

- 内部通过 `SysOperation.fs().list_directories / list_files / read_file` 访问文件系统；
- 在每个候选目录中查找名为 `Skill.md`（大小写不敏感）的文件，从其 YAML front matter 中读取 `description` 字段，构造 `Skill`；
- 最终写入 `_registry[name] = Skill(...)`。

### unregister

```python
def unregister(self, name: str) -> None
```

按照技能名称注销一个技能（若名称不存在则忽略）。

### get

```python
def get(self, name: str) -> Optional[Skill]
```

按名称获取技能元数据。

**返回**：

- 对应的 `Skill` 实例，若不存在返回 `None`。

### get_all

```python
def get_all(self) -> List[Skill]
```

返回当前注册的所有技能列表。

### get_names

```python
def get_names(self) -> List[str]
```

返回当前注册的所有技能名称列表。

### has

```python
def has(self, name: str) -> bool
```

判断指定名称的技能是否存在于注册表中。

### clear

```python
def clear(self) -> None
```

清空所有已注册技能。

### count

```python
def count(self) -> int
```

返回当前注册技能的数量。

