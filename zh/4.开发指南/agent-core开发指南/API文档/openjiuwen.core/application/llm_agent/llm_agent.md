# openjiuwen.core.application.llm_agent.llm_agent

## class openjiuwen.core.application.llm_agent.llm_agent.LLMAgent

```python
class LLMAgent(ControllerAgent):
    ...
```

`LLMAgent` 是基于 **ReAct** 的单智能体实现，继承 `ControllerAgent`，内部持有 `LLMController` 完成“思考 → 计划 → 执行 → 汇报”闭环。特点：

- 使用消息队列模型处理会话输入/输出；
- 通过 LLM 生成任务计划（工具 / 工作流），支持多轮流式输出；
- 可选接入长期记忆，将对话问答写入 `LongTermMemory`；
- 与 `Runner`/`ContextEngine`/`Session` 深度协同。

源码位置：`openjiuwen.core.application.llm_agent.llm_agent.LLMAgent`。

---

### 初始化

```python
def __init__(self, agent_config: ReActAgentConfig):
    ...
```

**参数与约束：**
- `agent_config` (`LegacyReActAgentConfig` 别名 `ReActAgentConfig`)：ReAct Agent 配置，要求 `controller_type == ControllerType.ReActController`，否则抛出 `NotImplementedError`。

**内部注入：**
- `controller = LLMController(config, context_engine, session)`：核心控制器；
- `_long_term_memory_instance = LongTermMemory()`；
- `_memory_scope_id` 与 `_enable_memory`：当配置存在 `memory_scope_id` 且开启 `agent_memory_config.enable_long_term_mem` 或配置了 `mem_variables` 时启用写记忆。

---

### invoke

```python
async def invoke(self, inputs: Dict, session: Session = None) -> Dict:
    ...
```

**用途：** 非流式调用，完全委托给基类 `ControllerAgent.invoke`（底层走 `LLMController.invoke`），常用于一次性获取完整结果。

**输入要求：**
- `inputs["query"]`：用户问题（字符串）；
- `inputs["conversation_id"]`：会话标识（可选，默认 `"default_session"`）；
- `inputs["user_id"]`：写入长期记忆时必须提供。

**记忆写入：**
- 若 `_enable_memory=True`，在返回后异步调用 `_write_messages_to_memory(inputs, result)`，把用户 query 与回答写入 `LongTermMemory`；失败会记录日志但不影响主流程。

返回值由控制器决定（通常是包含回答和 trace 的字典/列表）。

---

### stream

```python
async def stream(self, inputs: Dict, session: Session = None) -> AsyncIterator[Any]:
    ...
```

**用途：** 流式执行，边执行边产出 `OutputSchema`/字典块。

**行为要点：**
1. 若未传入 `session`：通过 `self._session.pre_run(conversation_id)` 创建内部会话，`own_stream=True`，负责读写 `session.stream_iterator()` 并在结束时 `post_run()`。
2. 传入外部 `session` 时：`own_stream=False`，不负责关闭会话；会把 Agent 的 tools/workflows 同步注册到 `Runner.resource_mgr`（工具已实现，同步工作流需按注释补充）。
3. 执行主体 `stream_process()`：调用 `controller.invoke(inputs, agent_session)` 以便将最终结果写入 `final_result_holder`，用于非自持流场景的补发。
4. `own_stream=True` 时实时 `yield` `agent_session.stream_iterator()` 中的块；同时收集回答文本用于记忆。
5. 结束后若 `_enable_memory`：异步写入长期记忆（拼接收集到的回答文本）。

**典型输入：**
```python
inputs = {
    "conversation_id": "chat-001",
    "query": "请帮我规划一次上海周末行程",
    "user_id": "u123"      # 需要写记忆时必填
}
async for chunk in agent.stream(inputs):
    print(chunk)
```

---

### set_prompt_template

```python
def set_prompt_template(self, prompt_template: List[Dict]):
    ...
```

更新 Agent 与 LLMController 的提示模板（会同步刷新配置包装器 `_config_wrapper`）。可在运行期热更新提示词。

---

### 工厂方法（向后兼容）

```python
create_llm_agent_config(agent_id, agent_version, description,
                        workflows, plugins, model, prompt_template, tools=None)
```
- 返回 `ReActAgentConfig`，便于旧代码快速构建配置。

```python
create_llm_agent(agent_config, workflows=None, tools=None)
```
- 基于配置实例化 `LLMAgent`，并批量 `add_workflows/add_tools`（空列表安全）；适合迁移旧项目时的一行式创建。

---

### 记忆写入（内部）

```python
async def _write_messages_to_memory(self, inputs, result=None):
    ...
```

- 仅当存在 `user_id` 且 `_enable_memory=True` 执行；
- 收集用户 query 与 AI 回答（支持 `OutputSchema`/dict/str 三种格式转 `AssistantMessage`）；
- 调用 `LongTermMemory.add_messages(user_id, scope_id, messages, timestamp, agent_config)` 持久化；
- 异常仅记录日志，不中断主流程。
