# openjiuwen.core.skills.skill_util

## class SkillUtil

```python
class openjiuwen.core.skills.skill_util.SkillUtil
```

`SkillUtil` 是对 `SkillManager` 与 `SkillToolKit` 的高级封装，为业务侧提供一站式的**技能注册、工具挂载与技能提示词生成**能力。

### __init__

```python
def __init__(self, sys_operation_id: str)
```

**参数**：

- `sys_operation_id: str`：用于构造内部的 `SkillManager` 与 `SkillToolKit`，并通过 `SysOperation` 完成文件/代码/命令相关操作。

内部成员：

- `_skill_manager: SkillManager`：技能元数据管理器；
- `_skill_tool_kit: SkillToolKit`：技能相关工具创建与挂载组件。

### set_sys_operation_id

```python
def set_sys_operation_id(self, sys_operation_id: str) -> None
```

同步更新内部所有与 `SysOperation` 相关的组件：

- 调用 `self.skill_manager.set_sys_operation_id(sys_operation_id)`；
- 设置 `self.skill_tool_kit.sys_operation_id = sys_operation_id`。

### 属性 skill_manager / skill_tool_kit

```python
@property
def skill_manager(self) -> SkillManager

@property
def skill_tool_kit(self) -> SkillToolKit
```

返回内部持有的 `SkillManager` 与 `SkillToolKit` 实例，便于在需要时直接访问底层能力。

### register_skills

```python
async def register_skills(
    self,
    skill_path: str,
    agent: "BaseAgent",
    session_id: str | None = None,
) -> bool
```

注册指定目录下的技能，并为 Agent 挂载所有技能相关内置工具。

**参数**：

- `skill_path: str`：技能根目录路径，内部会转换为 `Path(skill_path)` 传给 `SkillManager.register`；
- `agent: BaseAgent`：要挂载工具的 Agent 实例；
- `session_id: str | None`：执行文件系统操作时使用的会话 id。

**行为**：

1. 调用 `self._skill_tool_kit.add_skill_tools(agent)`，为 Agent 添加：
   - `view_file` 工具；
   - `execute_python_code` 工具；
   - `run_command` 工具。
2. 调用 `self._skill_manager.register(Path(skill_path), session_id)` 注册技能。  

注册成功后返回 `True`。

### has_skill

```python
def has_skill(self) -> bool
```

判断当前是否已经注册了至少一个技能。

**实现**：

- 实际调用 `self._skill_manager.count()`，若大于 0 则返回 `True`，否则返回 `False`。

### get_skill_prompt

```python
def get_skill_prompt(self) -> str
```

生成包含**所有已注册技能信息**的提示词片段，用于拼接到 Agent 的系统提示词中。

**行为**：

1. 调用 `self._skill_manager.get_all()` 获取所有技能列表；
2. 将每个技能格式化为一行字符串：
   - `"{index}.Skill name: {skill.name}; Skill description: {skill.description}; Skill file path: {skill.directory}"`；
3. 使用预置的 `PromptTemplate`：

   ```python
   SKILL_PROMPT_CONTENT = '''
   To help you better complete tasks, the following skill knowledge is provided:
   {{skills}}
   You can use the view_file tool to read the corresponding Skill.md file to obtain the relevant skill knowledge.
   '''
   skill_prompt = PromptTemplate(content=SKILL_PROMPT_CONTENT)
   ```

4. 调用 `skill_prompt.format({"skills": "\n".join(skills_info)})`，返回格式化后的提示词字符串。

### 典型用法示例

```python
from openjiuwen.core.skills import SkillUtil
from openjiuwen.core.single_agent import BaseAgent


async def setup_agent_with_skills(agent: BaseAgent):
    # 1. 创建 SkillUtil，指定 SysOperation id
    skill_util = SkillUtil(sys_operation_id="default_sys_op")

    # 2. 注册技能目录并为 Agent 挂载技能相关工具
    await skill_util.register_skills(
        skill_path="/path/to/skills_root",
        agent=agent,
        session_id="session_001",
    )

    # 3. 构造技能提示词片段，拼接到 Agent 系统提示词中
    skill_prompt = skill_util.get_skill_prompt()
    print(skill_prompt)
```

通过 `SkillUtil`，上层只需要关心“技能目录路径 + Agent 实例”这两个要素，就可以完成：

- 技能元数据的注册；
- 相关内置工具的挂载；
- 技能提示词片段的自动生成。

