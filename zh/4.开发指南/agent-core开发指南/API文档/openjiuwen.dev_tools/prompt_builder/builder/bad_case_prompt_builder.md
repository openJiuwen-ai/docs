# openjiuwen.dev_tools.prompt_builder.builder.bad_case_prompt_builder

## class openjiuwen.dev_tools.prompt_builder.builder.bad_case_prompt_builder.BadCasePromptBuilder

`BadCasePromptBuilder` 为“错误案例（Bad Case）驱动的提示词优化器”。

它会读取一组评测失败用例（`EvaluatedCase`），先总结“失败原因与改进建议”，再据此对原始提示词进行定向修复。

这类优化特别适合：

- 边缘场景频繁失败（例如格式要求严格、工具调用易错、强约束任务）
- 你已经有评测集与 `EvaluatedCase`（包含 answer/score/reason）
- 希望把“坏例反思”系统化沉淀到提示词里

## 初始化

```python
BadCasePromptBuilder(model_config: ModelRequestConfig, model_client_config: ModelClientConfig)
```

参数语义与 `MetaTemplateBuilder` 一致。

## build

```python
await build(
    self,
    prompt: str | PromptTemplate,
    cases: List[EvaluatedCase],
) -> Optional[str]
```

基于坏例对提示词进行定向优化（非流式）。

**参数**：

- **prompt** (str | PromptTemplate)：原始提示词。
  - 不能为空且不能全空白。
  - 若是 `PromptTemplate`，会先抽取成纯字符串内容参与优化。
- **cases** (List[EvaluatedCase])：评测结果用例列表。
  - 必须非空；否则会抛出异常。
  - 框架默认限制 cases 数量上限（源码默认 10），避免一次性塞入过多 bad cases 导致上下文过长。
  - `EvaluatedCase` 内应包含：用例 inputs/label、模型 answer、评分 score（0~1）、失败原因 reason。
    - 一般推荐只传“失败样本”（例如 score=0 的样本），让优化更聚焦。

**返回**：

- **Optional[str]**：优化后的提示词文本。

**异常（常见）**：

- 当 `prompt` 为空/全空白时抛出 `build_error(StatusCode.TOOLCHAIN_BAD_CASE_TEMPLATE_EXECUTION_ERROR, ...)`。
- 当 `cases` 为空或超过上限时抛出同类错误。
  - 这类校验用于避免“无训练信号”或“上下文爆炸”。

## stream_build

```python
async for chunk in stream_build(...):
    ...
```

流式版本的 build。

参数语义与 `build` 一致，逐段产出优化后的文本片段。

