# openjiuwen.core.memory.config

`openjiuwen.core.memory.config` 模块定义了记忆引擎使用的三类配置类：

- `MemoryEngineConfig`：全局引擎配置；
- `MemoryScopeConfig`：作用域级配置（用于不同业务场景的模型/向量参数）；
- `AgentMemoryConfig`：Agent 级记忆策略配置（用于定义需要提取的变量记忆和是否开启长期记忆）。

对应源码：`openjiuwen.core.memory.config.config`。

## class MemoryEngineConfig

```python
class openjiuwen.core.memory.config.config.MemoryEngineConfig(BaseModel)
```

全局记忆引擎配置，用于设置引擎级别的通用参数。

### 字段

- `default_model_cfg: ModelRequestConfig | None`：
  - 默认用于生成记忆的大模型请求参数（模型名、温度、最大 token 等）；
  - 若为 `None`，则无法生成记忆（除非通过 `MemoryScopeConfig` 为特定 scope 配置模型）。
- `default_model_client_cfg: ModelClientConfig | None`：
  - 默认的大模型客户端配置（`client_id / client_provider / api_base / api_key / verify_ssl` 等）；
  - 若为 `None`，同样无法生成记忆（除非通过 scope 配置）。
- `input_msg_max_len: int`：
  - 输入消息最大长度（字符数），默认 8192；
  - 在生成记忆时，超过此长度的消息内容会被截断。
- `crypto_key: bytes`：
  - AES 加密密钥，用于加密存储中的敏感字段（如 API key）；
  - 长度必须为 32 字节；若为空字节串 `b''`，则不启用加密；
  - 默认值：`b''`（不加密）。

### 字段校验

`crypto_key` 字段带有 `field_validator`：

- 若长度为 0，则保持为空（不加密）；
- 若长度等于 `AES_KEY_LENGTH`（32），则通过校验；
- 否则抛出 `ValueError`，错误信息为：`"Invalid crypto_key, must be empty or {AES_KEY_LENGTH} bytes length"`。

## class MemoryScopeConfig

```python
class openjiuwen.core.memory.config.config.MemoryScopeConfig(BaseModel)
```

作用域级记忆配置，用于为不同的 `scope_id` 定义独立的模型和向量参数。

### 字段

- `model_cfg: ModelRequestConfig | None`：
  - 该 scope 下使用的大模型请求配置（模型名、温度等）；
  - 若为 `None`，则使用全局 `MemoryEngineConfig.default_model_cfg`。
- `model_client_cfg: ModelClientConfig | None`：
  - 该 scope 下使用的大模型客户端配置（`client_id / api_base / api_key` 等）；
  - 若为 `None`，则使用全局 `MemoryEngineConfig.default_model_client_cfg`。
- `embedding_cfg: EmbeddingConfig | None`：
  - 该 scope 下使用的嵌入模型配置（`model_name / base_url / api_key` 等）；
  - 若为 `None`，则语义检索功能可能不可用（取决于全局是否提供了嵌入模型）。

> **说明**：`MemoryScopeConfig` 中的 `api_key` 字段在保存到 `kv_store` 时会被自动加密（使用 `MemoryEngineConfig.crypto_key`），读取时会自动解密。

## class AgentMemoryConfig

```python
class openjiuwen.core.memory.config.config.AgentMemoryConfig(BaseModel)
```

Agent 级记忆策略配置，描述某个智能体希望提取和管理哪些类型的记忆。

### 字段

- `mem_variables: list[Param]`：
  - 变量记忆配置列表，默认空列表 `[]`；
  - 每个 `Param` 定义一个变量名、描述、类型、是否必填等；
  - 在 `LongTermMemory.add_messages` 时，会根据这些配置从对话中提取变量值并保存。
- `enable_long_term_mem: bool`：
  - 是否开启长期记忆生成，默认 `True`；
  - 为 `True` 时，会从对话中提取用户画像（长期记忆）并保存到语义存储中；
  - 为 `False` 时，仅保存消息和变量记忆，不生成用户画像。

> **说明**：`Param` 类型定义在 `openjiuwen.core.common.schema.param` 中，通常包含 `name / description / type / required` 等字段。

## 使用示例

```python
from openjiuwen.core.memory import (
    MemoryEngineConfig,
    MemoryScopeConfig,
    AgentMemoryConfig,
)
from openjiuwen.core.foundation.llm.schema.config import (
    ModelClientConfig,
    ModelRequestConfig,
)
from openjiuwen.core.retrieval.common.config import EmbeddingConfig
from openjiuwen.core.common.schema.param import Param

# 1. 创建全局引擎配置
engine_config = MemoryEngineConfig(
    default_model_cfg=ModelRequestConfig(
        model="<model_name>",
        temperature=0.0,
    ),
    default_model_client_cfg=ModelClientConfig(
        client_id="default_memory_llm",
        client_provider="OpenAI",
        api_key="sk-xxxx",
        api_base="https://api.openai.com/v1",
    ),
    input_msg_max_len=8192,
    crypto_key=b"your-32-byte-aes-key-here!!",  # 32 字节
)

# 2. 创建作用域配置（可选）
scope_config = MemoryScopeConfig(
    model_cfg=ModelRequestConfig(
        model="<model_name>",
        temperature=0.1,
    ),
    model_client_cfg=ModelClientConfig(
        client_id="scope_llm",
        client_provider="OpenAI",
        api_key="sk-yyyy",
    ),
    embedding_cfg=EmbeddingConfig(
        model_name="text-embedding-3-small",
        base_url="https://api.openai.com/v1",
        api_key="sk-zzzz",
    ),
)

# 3. 创建 Agent 记忆策略配置
agent_config = AgentMemoryConfig(
    mem_variables=[
        Param(
            name="favorite_color",
            description="用户喜欢的颜色",
            type="string",
            required=False,
        ),
        Param(
            name="age",
            description="用户年龄",
            type="number",
            required=False,
        ),
    ],
    enable_long_term_mem=True,
)
```
