# openjiuwen.core.application.llm_agent.llm_agent

## class openjiuwen.core.application.llm_agent.llm_agent.LLMAgent

```
class openjiuwen.core.application.llm_agent.llm_agent.LLMAgent(agent_config: ReActAgentConfig)
```

基于新架构的 ReAct 风格智能体，继承`ControllerAgent`，内部使用`LLMController`完成“思考-规划-执行-汇报”流程，可选接入长期记忆。


### async stream(inputs: Dict, session: Session = None) -> AsyncIterator[Any]

流式调用：为给定`conversation_id`创建/复用会话，注册工具与工作流，委托控制器执行并实时从会话流产出`OutputSchema`/字典块；外部会话时最终再产出控制器返回值。若开启记忆，在流结束后异步写入长记忆。

**参数：**

* **inputs**(Dict)：同`invoke`，需`query`，可选`conversation_id`/`user_id`。
* **session**(Session, 可选)：外部会话；未提供则内部创建并负责生命周期。

**返回：**

* **AsyncIterator[Any]**：流式输出块。

**异常：**

* **RuntimeError**：未初始化控制器。

### set_prompt_template(prompt_template: List[Dict]) -> None

更新Agent与LLMController的提示模板，并刷新内部配置包装器。

**参数：**

* **prompt_template**(List[Dict])：新的提示模板列表。

### async _write_messages_to_memory(inputs, result=None) -> None

内部记忆写入：提取用户`query`与模型回答（支持`OutputSchema`/dict/str 三种），调用`LongTermMemory.add_messages`追加到指定`user_id`/`memory_scope_id`，异常仅记录日志不影响主流程。

**参数：**

* **inputs**：原始输入字典，需包含`user_id`。
* **result**(可选)：控制器返回结果或流式拼接文本。
