# openjiuwen.dev_tools.prompt_builder.builder.meta_template_builder

## class openjiuwen.dev_tools.prompt_builder.builder.meta_template_builder.MetaTemplateBuilder

`MetaTemplateBuilder` 为“提示词生成/重写”优化器。

它通过一组内置的 meta-template（以及可选的自定义 meta-template），把输入的“任务说明/指令草稿 + 工具列表”转换为更规范、更可执行的提示词文本。

## 适用场景

- 你只有一段自然语言的“任务描述/需求”，想快速生成一版结构化提示词。
- 你希望在“通用/规划（plan）”等不同模板策略之间切换。
- 你希望注入工具列表，让生成的提示词能正确引导模型使用工具。

## 初始化

```python
MetaTemplateBuilder(model_config: ModelRequestConfig, model_client_config: ModelClientConfig)
```

**参数**：

- **model_config** (`ModelRequestConfig`)：模型请求配置。
  - 建议明确 temperature/top_p/max_tokens 等参数，以控制生成风格与长度。
  - 这是“优化任务”的模型配置，不一定等同于你业务 Agent 的配置。
- **model_client_config** (`ModelClientConfig`)：模型客户端配置。
  - 用于指定 provider/base_url/api_key 等连接信息。
  - 建议在受控环境中注入，避免在文档/日志中泄露敏感字段。

## register_meta_template

```python
register_meta_template(self, name: str, meta_template: str | PromptTemplate)
```

注册自定义 meta-template。

当你在 `build(..., template_type="other", custom_template_name=...)` 时，会从这里取模板。

**参数**：

- **name** (str)：模板名（不含前缀）。
  - 内部会自动加上固定前缀 `META_TEMPLATE_`。
  - 建议用稳定且可读的命名（例如 `my_team_general_v1`）。
- **meta_template** (str | PromptTemplate)：模板内容。
  - 传 `str` 时会自动包装为 `PromptTemplate(content=...)`。
  - 传 `PromptTemplate` 时会 deepcopy 一份，避免外部引用被意外修改。
  - 只支持 `str` 或 `PromptTemplate`；其他类型会抛出异常。

**异常**：

- 当 `meta_template` 类型不合法时会抛出 `build_error(StatusCode.TOOLCHAIN_META_TEMPLATE_EXECUTION_ERROR, ...)`。

## build

```python
await build(
    self,
    prompt: str | PromptTemplate,
    tools: Optional[List[ToolInfo]] = None,
    template_type: Literal["general", "plan", "other"] = "general",
    custom_template_name: Optional[str] = None
) -> Optional[str]
```

生成/优化提示词文本（非流式）。

**参数**：

- **prompt** (str | PromptTemplate)：原始提示词/任务说明。
  - 允许传 `PromptTemplate`，会先转成纯字符串内容再处理。
  - 不能为空且不能是空白字符串，否则会抛出异常。
- **tools** (List[ToolInfo] | None)：可用工具列表。
  - 传入后会被写入 meta-template 的 `tools` 字段，帮助模型生成“可正确调用工具”的提示词。
  - 要求每个元素必须是 `ToolInfo`，否则会抛出异常。
- **template_type**：模板策略。
  - `"general"`：通用生成策略（默认）。
  - `"plan"`：偏规划/分步的生成策略。
  - `"other"`：使用自定义模板（必须同时提供 `custom_template_name`）。
- **custom_template_name**：自定义模板名。
  - 仅当 `template_type="other"` 时使用。
  - 必须先通过 `register_meta_template` 注册同名模板，否则会抛出异常。

**返回**：

- **Optional[str]**：生成后的提示词文本。
  - 当底层模型返回为空时，可能返回 `None`。

## stream_build

```python
async for chunk in stream_build(...):
    ...
```

流式生成提示词文本。

参数语义与 `build` 一致，区别在于返回值为异步生成器，逐段产出 `chunk.content`。

