# openjiuwen.dev_tools.prompt_builder.builder.feedback_prompt_builder

## class openjiuwen.dev_tools.prompt_builder.builder.feedback_prompt_builder.FeedbackPromptBuilder

`FeedbackPromptBuilder` 为“基于反馈的提示词优化器”。

它适用于你已经有一版提示词，但从用户/评审/线上反馈中得到了明确改进意见，希望把这些反馈转化为“更好的提示词文本”。

## 初始化

```python
FeedbackPromptBuilder(model_config: ModelRequestConfig, model_client_config: ModelClientConfig)
```

参数语义与 `MetaTemplateBuilder` 一致：前者控制优化任务的生成风格，后者指定模型连接配置。

## build

```python
await build(
    self,
    prompt: str | PromptTemplate,
    feedback: str,
    mode: Literal["general", "insert", "select"] = "general",
    start_pos: Optional[int] = None,
    end_pos: Optional[int] = None,
) -> Optional[str]
```

基于反馈生成“优化后的提示词文本”（非流式）。

**参数**：

- **prompt** (str | PromptTemplate)：原始提示词。
  - 不能为空且不能全空白。
  - 若是 `PromptTemplate`，会先抽取成纯字符串内容参与优化。
- **feedback** (str)：反馈信息。
  - 不能为空且不能全空白。
  - 建议写成“问题 + 改进建议”的形式（例如：哪里模糊、哪里遗漏、希望增加什么约束）。
- **mode**：优化模式。
  - `"general"`：对整段提示词做全局改写（默认）。
  - `"insert"`：在指定位置插入优化内容（通过 `start_pos` 指定插入点）。
  - `"select"`：对指定区间（`start_pos:end_pos`）做局部替换优化。
- **start_pos** (int, 可选)：起始位置。
  - `insert` 模式下必须提供，且需满足 `0 <= start_pos <= len(prompt)`。
  - `select` 模式下必须提供，且需满足 `0 <= start_pos < end_pos <= len(prompt)`。
- **end_pos** (int, 可选)：结束位置。
  - 仅 `select` 模式下必须提供。
  - 必须大于 start_pos，且不超过 prompt 长度。

**返回**：

- **Optional[str]**：优化后的提示词。
  - 当底层模型返回为空时，可能返回 `None`。

**异常（常见）**：

- 当 `prompt/feedback` 为 None 或空字符串时会抛出 `build_error(StatusCode.TOOLCHAIN_FEEDBACK_TEMPLATE_EXECUTION_ERROR, ...)`。
- 当 `mode` 为 `insert/select` 但位置参数不合法时会抛出同类错误。
  - 这类校验用于避免“位置越界/区间非法”导致改写结果不可控。

## stream_build

```python
async for chunk in stream_build(...):
    ...
```

流式版本的 build，参数语义与 `build` 一致，逐段产出优化后的文本片段（`chunk.content`）。

