# openjiuwen.core.session.checkpointer

## class openjiuwen.core.session.checkpointer.base.Checkpointer

`Checkpointer` 是 checkpoint 管理接口，用于保存/恢复会话与工作流执行状态，从而支持：

- **交互中断恢复**：进入 `INPUT_REQUIRED`/中断态后可在补齐输入后继续执行
- **异常场景留痕**：异常发生时保存可用于排查的执行上下文
- **资源释放**：会话结束时清理状态与存储

### get_thread_id

```python
get_thread_id(session: BaseSession) -> str
```

生成 thread_id（用于标识同一“会话 + 工作流”执行线程）。

**参数**：

- **session** (BaseSession)：底层会话对象。\n  - 必须能提供 `session_id()` 与 `workflow_id()`。\n  - thread_id 的生成规则为 `"{session_id}:{workflow_id}"`。

**返回**：

- **str**：thread_id。

### pre_workflow_execute

```python
async pre_workflow_execute(self, session: BaseSession, inputs: InteractiveInput)
```

在工作流执行前触发，用于“创建或恢复”工作流 checkpoint。

**参数**：

- **session** (BaseSession)：工作流底层会话。\n  - 用于定位要恢复/创建的状态命名空间。\n  - 一般会包含当前 workflow_id、session_id、state、config 等。
- **inputs** (InteractiveInput)：交互输入。\n  - 当 inputs 为交互输入时，通常表示“从交互恢复”，需要基于 inputs 恢复此前中断点。\n  - 当 inputs 非 InteractiveInput（例如普通 dict）时，通常表示“首次执行/非恢复执行”，具体行为由实现决定（可能拒绝覆盖已有状态）。

### post_workflow_execute

```python
async post_workflow_execute(self, session: BaseSession, result, exception)
```

在工作流执行后触发。\n+常见行为：

- **异常不为空**：保存 checkpoint，并将异常继续抛出
- **执行结束且无中断**：清理该 workflow 的 checkpoint
- **需要交互（中断）**：保存 checkpoint，等待后续恢复

**参数**：

- **session** (BaseSession)：工作流底层会话。\n  - 用于定位要保存/清理的状态。
- **result**：执行结果对象。\n  - 可能包含“是否中断”等标记（具体由执行引擎约定）。\n  - 实现通常会据此判断是清理还是保留 checkpoint。
- **exception**：异常对象。\n  - 为 `None` 表示正常执行。\n  - 不为 `None` 时一般会保存 checkpoint 并 re-raise。

### pre_agent_execute

```python
async pre_agent_execute(self, session: BaseSession, inputs)
```

在 Agent 执行前触发，用于恢复/初始化 Agent 侧 checkpoint。\n+当 inputs 不为空时，通常表示“带交互输入继续执行”。

**参数**：

- **session** (BaseSession)：Agent 底层会话。\n  - 用于定位要恢复/创建的 agent 状态。
- **inputs**：交互输入。\n  - 具体类型由 agent 执行链路决定。\n  - 常见场景：用户补齐输入后继续运行。

### interrupt_agent_execute

```python
async interrupt_agent_execute(self, session: BaseSession)
```

当 Agent 执行过程中触发交互中断时调用，用于保存“可恢复的中断点”。

**参数**：

- **session** (BaseSession)：Agent 底层会话。\n  - 用于保存当前 agent 的状态快照。

### post_agent_execute

```python
async post_agent_execute(self, session: BaseSession)
```

在 Agent 执行结束后调用，用于保存最终状态或进行必要清理（具体取决于实现策略）。

**参数**：

- **session** (BaseSession)：Agent 底层会话。

### release

```python
async release(self, session_id: str)
```

释放某个 session_id 下的 checkpoint 资源。

**参数**：

- **session_id** (str)：会话 id。\n  - 用于定位并清理该会话下的 workflow/agent checkpoint。\n  - 通常在会话结束或显式清理时调用。

### graph_store

```python
graph_store(self) -> Store
```

返回用于保存/加载图执行状态的 store 实例（供执行引擎读写）。

## class openjiuwen.core.session.checkpointer.checkpointer.InMemoryCheckpointer

`InMemoryCheckpointer` 是内存实现，适合本地/单进程运行与调试场景。\n+它的主要特点：

- **进程内存存储**：进程退出后状态丢失（不适合生产长期恢复）
- **支持 workflow 与 agent 两侧 checkpoint**：分别存储并按 session_id 组织
- **与 graph_store 集成**：用于保存图执行状态（例如用于中断恢复）

### 行为说明（关键点）

- 当工作流**发生异常**时：会保存 checkpoint 并继续抛出异常，便于上层处理。\n+- 当工作流**正常结束且无中断**时：会清理 workflow checkpoint，并在非 agent 父会话场景下可能清理整个 session。\n+- 当工作流**需要交互**（进入中断态）时：会保存 checkpoint，等待用户输入后恢复继续执行。

## function openjiuwen.core.session.base.get_default_inmemory_checkpointer

```python
get_default_inmemory_checkpointer()
```

获取默认的内存 checkpointer。

