# openjiuwen.core.memory.long_term_memory

## class LongTermMemory

```python
class openjiuwen.core.memory.long_term_memory.LongTermMemory(metaclass=Singleton)
```

`LongTermMemory` 是 openJiuwen 0.1.4 中统一的**长期记忆管理引擎**，采用单例模式，负责：

- 管理用户对话消息的持久化与检索；
- 管理用户变量记忆（如偏好、个人信息等结构化数据）；
- 管理用户画像（长期记忆，通过 LLM 从对话中提取）；
- 支持基于 `scope_id` 的多租户隔离；
- 支持向量检索、分页查询、按条件删除等操作。

对应源码：`openjiuwen.core.memory.long_term_memory.LongTermMemory`。

> **说明**：与旧版 `MemoryEngine(config: SysMemConfig, ...)` 不同，`LongTermMemory` 采用**无参构造 + 分步初始化**的方式：
> 1. 先调用 `await register_store(...)` 注册底层存储；
> 2. 再调用 `set_config(MemoryEngineConfig(...))` 设置全局配置；
> 3. 可选地通过 `set_scope_config(scope_id, MemoryScopeConfig(...))` 为不同业务场景配置独立的模型/向量参数。

### __init__

```python
def __init__(self) -> None
```

初始化 `LongTermMemory` 实例（单例模式，多次调用返回同一实例）。

**内部状态初始化**：

- 配置相关：`_sys_mem_config: MemoryEngineConfig | None = None`、`_scope_config: dict[str, MemoryScopeConfig] = {}`；
- 存储相关：`kv_store / semantic_store / db_store` 均为 `None`，需通过 `register_store` 注册；
- 管理器相关：`scope_user_mapping_manager / message_manager / user_profile_manager / variable_manager / write_manager / search_manager / generator` 均为 `None`，在 `set_config` 时初始化；
- LLM 相关：`_base_llm: Tuple[str, Model] | None = None`（在 `set_config` 时设置）；
- 嵌入模型缓存：`_scope_embedding: dict[str, Embedding] = {}`。

### async register_store

```python
async def register_store(
    self,
    kv_store: BaseKVStore,
    vector_store: VectorStore | None = None,
    db_store: BaseDbStore | None = None,
    embedding_model: Embedding | None = None,
) -> None
```

注册底层存储实例，必须在调用 `set_config` 之前完成。

**参数**：

- `kv_store: BaseKVStore`：**必填**，键值存储实例，用于快速访问结构化数据（如 scope 配置、用户变量等）。若为 `None`，会抛出 `JiuWenBaseException`（`MEMORY_REGISTER_STORE_EXECUTION_ERROR`）。
- `vector_store: VectorStore | None`：向量存储实例，用于语义相似度检索。若为 `None`，则语义检索功能不可用。
- `db_store: BaseDbStore | None`：关系型数据库存储实例，用于持久化消息、scope-user 映射等。若为 `None`，则消息持久化功能不可用。
- `embedding_model: Embedding | None`：全局嵌入模型实例，用于在注册时初始化 `semantic_store` 的嵌入能力。若为 `None`，后续可通过 `set_scope_config` 为不同 scope 配置独立的嵌入模型。

**行为**：

- 校验 `kv_store` 非空，否则抛出异常；
- 校验 `vector_store` 类型（若提供）是否为 `VectorStore` 实例，否则抛出异常；
- 校验 `db_store` 类型（若提供）是否为 `BaseDbStore` 实例，否则抛出异常；
- 将 `kv_store` 赋值给 `self.kv_store`；
- 创建 `SemanticStore(vector_store=vector_store)` 并赋值给 `self.semantic_store`；
- 若 `semantic_store` 存在且 `embedding_model` 非空，调用 `self.semantic_store.initialize_embedding_model(embedding_model)` 初始化嵌入模型；
- 若 `db_store` 存在，调用 `await create_tables(self.db_store)` 创建必要的数据库表结构。

### set_config

```python
def set_config(self, config: MemoryEngineConfig) -> None
```

设置全局记忆引擎配置，并初始化内部管理器。

**参数**：

- `config: MemoryEngineConfig`：全局引擎配置，包含：
  - `default_model_cfg: ModelRequestConfig`：默认用于生成记忆的大模型请求参数；
  - `default_model_client_cfg: ModelClientConfig`：默认大模型客户端配置；
  - `input_msg_max_len: int`：输入消息最大长度（默认 8192）；
  - `crypto_key: bytes`：AES 加密密钥（长度必须为 32 字节；为空则不加密）。

**前置条件**：

- 必须已调用 `register_store` 注册 `kv_store`、`semantic_store`、`db_store`，否则会抛出 `JiuWenBaseException`（`MEMORY_SET_CONFIG_EXECUTION_ERROR`）。

**行为**：

1. 将 `config` 保存到 `self._sys_mem_config`；
2. 创建 `DataIdManager()` 用于生成唯一 id；
3. 创建 `UserMemStore(self.kv_store)` 用于用户记忆的 KV 存储封装；
4. 若 `db_store` 存在：
   - 创建 `SqlDbStore(self.db_store)`；
   - 创建 `ScopeUserMappingManager(sql_db_store)` 并赋值给 `self.scope_user_mapping_manager`；
   - 创建 `MessageManager(sql_db_store, data_id_generator, config.crypto_key)` 并赋值给 `self.message_manager`；
5. 创建 `UserProfileManager(semantic_recall_instance=self.semantic_store, user_mem_store=user_mem_store, data_id_generator=data_id_generator, crypto_key=config.crypto_key)` 并赋值给 `self.user_profile_manager`；
6. 创建 `VariableManager(self.kv_store, config.crypto_key)` 并赋值给 `self.variable_manager`；
7. 创建 `WriteManager(managers={MemoryType.USER_PROFILE: user_profile_manager, MemoryType.VARIABLE: variable_manager}, user_mem_store=user_mem_store)` 并赋值给 `self.write_manager`；
8. 创建 `SearchManager(managers, user_mem_store, config.crypto_key)` 并赋值给 `self.search_manager`；
9. 创建 `Generator()` 并赋值给 `self.generator`；
10. 根据 `config.default_model_cfg` 和 `config.default_model_client_cfg` 创建默认 LLM 实例，保存为 `self._base_llm = (model_name, Model(...))`。

### async set_scope_config

```python
async def set_scope_config(
    self,
    scope_id: str,
    memory_scope_config: MemoryScopeConfig,
) -> bool
```

为指定 `scope_id` 设置作用域级记忆配置，并持久化到 `kv_store`。

**参数**：

- `scope_id: str`：作用域标识符，不能包含 `/`，长度不能超过 128 字符；若格式无效，返回 `False` 并记录错误日志。
- `memory_scope_config: MemoryScopeConfig`：作用域配置，包含：
  - `model_cfg: ModelRequestConfig | None`：该 scope 下使用的大模型请求配置；
  - `model_client_cfg: ModelClientConfig | None`：该 scope 下使用的大模型客户端配置；
  - `embedding_cfg: EmbeddingConfig | None`：该 scope 下使用的嵌入模型配置。

**行为**：

1. 校验 `scope_id` 格式（调用 `_validate_id`），无效则返回 `False`；
2. 深拷贝 `memory_scope_config` 为 `encrypted_config`；
3. 若 `encrypted_config.model_client_cfg.api_key` 存在，使用 `BaseMemoryManager.encrypt_memory_if_needed(key=self._sys_mem_config.crypto_key, plaintext=api_key)` 加密后覆盖；
4. 若 `encrypted_config.embedding_cfg.api_key` 存在，同样加密后覆盖；
5. 将 `encrypted_config` 存入内存缓存 `self._scope_config[scope_id]`；
6. 将配置序列化为 JSON（`encrypted_config.model_dump_json(by_alias=True)`），写入 `kv_store`，key 为 `f"{self.SCOPE_CONFIG_KEY}/{scope_id}"`（即 `"memory_scope_config/{scope_id}"`）；
7. 若 `scope_id` 在 `self._scope_embedding` 缓存中，删除该缓存项（因为配置变更可能导致嵌入模型需要重新创建）；
8. 返回 `True`。

### async get_scope_config

```python
async def get_scope_config(self, scope_id: str) -> MemoryScopeConfig | None
```

从 `kv_store` 中读取指定 `scope_id` 的作用域配置，并解密 API key。

**参数**：

- `scope_id: str`：作用域标识符。

**返回**：

- `MemoryScopeConfig | None`：若配置存在，返回解密后的配置对象；若不存在或 `scope_id` 格式无效，返回 `None`。

**行为**：

1. 校验 `scope_id` 格式，无效则返回 `None`；
2. 从 `kv_store` 读取 key `f"memory_scope_config/{scope_id}"` 的 JSON 字符串；
3. 若不存在，返回 `None`；
4. 使用 `MemoryScopeConfig.model_validate_json(config_json)` 解析为配置对象；
5. 若 `model_client_cfg.api_key` 存在，使用 `BaseMemoryManager.decrypt_memory_if_needed(...)` 解密后覆盖；
6. 若 `embedding_cfg.api_key` 存在，同样解密后覆盖；
7. 返回解密后的配置对象。

### async delete_scope_config

```python
async def delete_scope_config(self, scope_id: str) -> bool
```

删除指定 `scope_id` 的作用域配置（从 `kv_store` 和内存缓存中移除）。

**参数**：

- `scope_id: str`：作用域标识符。

**返回**：

- `bool`：删除成功返回 `True`，`scope_id` 格式无效或删除失败返回 `False`。

**行为**：

1. 校验 `scope_id` 格式，无效则返回 `False`；
2. 从 `kv_store` 删除 key `f"memory_scope_config/{scope_id}"`；
3. 从内存缓存 `self._scope_config` 中删除 `scope_id`（若存在）；
4. 从嵌入模型缓存 `self._scope_embedding` 中删除 `scope_id`（若存在）；
5. 记录调试日志，返回 `True`；若发生异常，记录错误日志并返回 `False`。

### async delete_mem_by_scope

```python
async def delete_mem_by_scope(self, scope_id: str) -> bool
```

删除指定 `scope_id` 下的所有记忆数据（包括消息、用户画像、变量等）。

**参数**：

- `scope_id: str`：作用域标识符。

**返回**：

- `bool`：删除成功返回 `True`，`scope_id` 格式无效返回 `False`。

**行为**：

1. 校验 `scope_id` 格式，无效则返回 `False`；
2. 通过 `scope_user_mapping_manager.get_by_scope_id(scope_id)` 获取该 scope 下所有 `user_id` 列表；
3. 对每个 `user_id`：
   - 获取分布式锁 `DistributedLock(self.kv_store, f"user/{user_id}")`；
   - 在锁内调用 `write_manager.delete_mem_by_user_id(scope_id=scope_id, user_id=user_id)` 删除该用户在该 scope 下的所有记忆；
4. 调用 `scope_user_mapping_manager.delete_by_scope_id(scope_id=scope_id)` 删除 scope-user 映射关系；
5. 记录调试日志，返回 `True`。

### async add_messages

```python
async def add_messages(
    self,
    messages: list[BaseMessage],
    agent_config: AgentMemoryConfig,
    *,
    user_id: str = "__default__",
    scope_id: str = "__default__",
    session_id: str = "__default__",
    timestamp: datetime | None = None,
    gen_mem: bool = True,
    gen_mem_with_history_msg_num: int = 5,
) -> None
```

添加对话消息到记忆引擎，并根据 `agent_config` 生成记忆（用户画像、变量等）。

**参数**：

- `messages: list[BaseMessage]`：要添加的消息列表（通常包含用户消息和 AI 回复）。
- `agent_config: AgentMemoryConfig`：Agent 记忆策略配置，包含：
  - `mem_variables: list[Param]`：需要提取的变量记忆配置（变量名、描述、类型等）；
  - `enable_long_term_mem: bool`：是否开启长期记忆生成（默认 `True`）。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时直接返回，不抛异常。
- `session_id: str`：会话标识符，默认 `"__default__"`。
- `timestamp: datetime | None`：消息时间戳，若为 `None` 则使用当前 UTC 时间。
- `gen_mem: bool`：是否生成记忆，默认 `True`；为 `False` 时仅保存消息，不触发记忆提取。
- `gen_mem_with_history_msg_num: int`：生成记忆时参考的历史消息数量，默认 5。

**行为**：

1. 校验 `scope_id` 格式，无效则直接返回；
2. 调用 `_get_scope_llm(scope_id)` 获取该 scope 的 LLM 实例（优先使用 scope 配置，否则使用全局默认配置）；
3. 调用 `_set_semantic_store_embedding_model(scope_id)` 设置该 scope 的嵌入模型到 `semantic_store`；
4. 获取用户级分布式锁 `DistributedLock(self.kv_store, f"user/{user_id}")`；
5. 在锁内：
   - 若 LLM 未初始化，记录错误日志并返回；
   - 调用 `_get_history_messages(user_id, scope_id, session_id, history_window_size=gen_mem_with_history_msg_num)` 获取历史消息；
   - 调用 `scope_user_mapping_manager.add(user_id=user_id, scope_id=scope_id)` 记录 scope-user 映射；
   - 若 `timestamp` 为 `None`，使用 `datetime.now(timezone.utc)`；
   - 遍历 `messages`，为每条消息调用 `message_manager.add(MessageAddRequest(...))` 保存到数据库，并记录最后一条消息的 `msg_id`；
6. 若 `gen_mem=False`，直接返回；
7. 调用 `_check_messages(messages)` 校验消息格式（必须包含至少一条用户消息，并截断超长消息）；
8. 若校验失败，记录调试日志并返回；
9. 调用 `generator.gen_all_memory(scope_id, user_id, messages, history_messages, session_id, config=agent_config, base_chat_model=llm, message_mem_id=msg_id)` 生成所有记忆单元（用户画像、变量等）；
10. 调用 `write_manager.add_mem(mem_units=all_memory, llm=llm)` 写入记忆；若失败，抛出 `JiuWenBaseException`（`MEMORY_ADD_MEMORY_EXECUTION_ERROR`）。

### async get_recent_messages

```python
async def get_recent_messages(
    self,
    user_id: str = "__default__",
    scope_id: str = "__default__",
    session_id: str = "__default__",
    num: int = 10,
) -> list[BaseMessage]
```

获取指定用户/scope/会话的最近 N 条消息，按写入顺序返回。

**参数**：

- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回空列表。
- `session_id: str`：会话标识符，默认 `"__default__"`。
- `num: int`：要获取的消息数量，默认 10。

**返回**：

- `list[BaseMessage]`：消息列表，按写入时间顺序排列；若 `scope_id` 格式无效或 `message_manager` 未初始化，返回空列表。

**行为**：

1. 校验 `scope_id` 格式，无效则返回空列表；
2. 调用 `message_manager.get(user_id, scope_id, session_id, message_len=num)` 获取最近 `num` 条消息（返回 `list[Tuple[BaseMessage, datetime]]`）；
3. 提取消息部分（忽略时间戳），返回 `[msg for msg, _ in recent_messages_tuple]`。

### async get_message_by_id

```python
async def get_message_by_id(self, msg_id: str) -> Tuple[BaseMessage, datetime] | None
```

根据消息 id 获取单条消息及其创建时间戳。

**参数**：

- `msg_id: str`：消息唯一标识符。

**返回**：

- `Tuple[BaseMessage, datetime] | None`：若消息存在，返回 `(消息对象, 创建时间)`；若 `message_manager` 未初始化或消息不存在，返回 `None`。

### async delete_mem_by_id

```python
async def delete_mem_by_id(
    self,
    mem_id: str,
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> None
```

删除指定 id 的记忆条目（用户画像或变量）。

**参数**：

- `mem_id: str`：记忆唯一标识符。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时直接返回。

**行为**：

1. 校验 `scope_id` 格式，无效则返回；
2. 调用 `_set_semantic_store_embedding_model(scope_id)` 设置嵌入模型；
3. 获取用户级分布式锁；
4. 在锁内：若 `write_manager` 未初始化，抛出 `JiuWenBaseException`（`MEMORY_DELETE_MEMORY_EXECUTION_ERROR`）；否则调用 `write_manager.delete_mem_by_id(user_id, scope_id, mem_id)` 删除。

### async delete_mem_by_user_id

```python
async def delete_mem_by_user_id(
    self,
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> None
```

删除指定用户在某 scope 下的所有类型记忆（用户画像、变量等）。

**参数**：

- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时直接返回。

**行为**：

1. 校验 `scope_id` 格式，无效则返回；
2. 设置嵌入模型；
3. 获取用户级分布式锁；
4. 在锁内：若 `write_manager` 未初始化，抛出异常；否则调用 `write_manager.delete_mem_by_user_id(user_id, scope_id)` 删除该用户在该 scope 下的所有记忆。

### async update_mem_by_id

```python
async def update_mem_by_id(
    self,
    mem_id: str,
    memory: str,
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> None
```

更新指定 id 的记忆内容。

**参数**：

- `mem_id: str`：记忆唯一标识符。
- `memory: str`：新的记忆内容。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时直接返回。

**行为**：

1. 校验 `scope_id` 格式，无效则返回；
2. 设置嵌入模型；
3. 获取用户级分布式锁；
4. 在锁内：若 `write_manager` 未初始化，抛出 `JiuWenBaseException`（`MEMORY_UPDATE_MEMORY_EXECUTION_ERROR`）；否则调用 `write_manager.update_mem_by_id(user_id, scope_id, mem_id, memory)` 更新。

### async get_variables

```python
async def get_variables(
    self,
    names: list[str] | str | None = None,
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> dict[str, str]
```

获取用户变量（一个或多个）。

**参数**：

- `names: list[str] | str | None`：
  - 若为 `None`：返回该用户在该 scope 下的所有变量；
  - 若为 `str`：返回单个变量（`{name: value}`）；
  - 若为 `list[str]`：返回多个变量（`{name1: value1, name2: value2, ...}`）。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回空字典。

**返回**：

- `dict[str, str]`：变量名到变量值的映射；若 `scope_id` 格式无效或 `search_manager` 未初始化，返回空字典或抛出异常。

**行为**：

1. 校验 `scope_id` 格式，无效则返回空字典；
2. 若 `search_manager` 未初始化，抛出 `JiuWenBaseException`（`MEMORY_GET_MEMORY_EXECUTION_ERROR`）；
3. 若 `names` 为 `None`：调用 `search_manager.get_all_user_variable(user_id, scope_id)` 返回所有变量；
4. 若 `names` 为 `str`：调用 `search_manager.get_user_variable(user_id, scope_id, names)` 返回单个变量；
5. 若 `names` 为 `list[str]`：遍历列表，逐个调用 `get_user_variable` 并合并结果；
6. 若 `names` 类型不符合预期，抛出异常。

### async search_user_mem

```python
async def search_user_mem(
    self,
    query: str,
    num: int,
    user_id: str = "__default__",
    scope_id: str = "__default__",
    threshold: float = 0.3,
) -> list[MemResult]
```

基于语义相似度搜索用户记忆（用户画像、变量等），返回与查询最相关的 N 条记忆。

**参数**：

- `query: str`：查询文本。
- `num: int`：要返回的记忆数量（top-k）。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回空列表。
- `threshold: float`：相似度阈值，默认 0.3；低于该阈值的记忆会被过滤。

**返回**：

- `list[MemResult]`：记忆结果列表，每个 `MemResult` 包含：
  - `mem_info: MemInfo`（`mem_id / content / type`）；
  - `score: float`（相似度分数）。

**行为**：

1. 校验 `scope_id` 格式，无效则返回空列表；
2. 设置嵌入模型；
3. 若 `search_manager` 未初始化，抛出异常；
4. 构造 `SearchParams(query, scope_id, top_k=num, user_id, threshold)`；
5. 调用 `search_manager.search(params)` 执行搜索；
6. 将搜索结果转换为 `list[MemResult]` 格式并返回；若发生异常，记录日志并返回空列表。

### async user_mem_total_num

```python
async def user_mem_total_num(
    self,
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> int
```

返回指定用户在某 scope 下的记忆总数。

**参数**：

- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回 0。

**返回**：

- `int`：记忆总数；若 `scope_id` 格式无效，返回 0。

**行为**：

1. 校验 `scope_id` 格式，无效则返回 0；
2. 调用 `search_manager.list_user_profile(user_id, scope_id)` 获取所有用户画像列表；
3. 返回列表长度。

### async get_user_mem_by_page

```python
async def get_user_mem_by_page(
    self,
    user_id: str = "__default__",
    scope_id: str = "__default__",
    page_size: int = 10,
    page_idx: int = 0,
    memory_type: MemoryType = MemoryType.UNKNOWN,
) -> list[MemInfo]
```

分页获取用户记忆列表，支持按记忆类型过滤。

**参数**：

- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回空列表。
- `page_size: int`：每页数量，默认 10。
- `page_idx: int`：页码（从 0 开始），默认 0。
- `memory_type: MemoryType`：记忆类型过滤，默认 `MemoryType.UNKNOWN`（不过滤）；可选值包括 `MemoryType.USER_PROFILE`、`MemoryType.VARIABLE` 等。

**返回**：

- `list[MemInfo]`：记忆信息列表，每个 `MemInfo` 包含 `mem_id / content / type`；若 `scope_id` 格式无效或 `search_manager` 未初始化，返回空列表或抛出异常。

**行为**：

1. 校验 `scope_id` 格式，无效则返回空列表；
2. 若 `search_manager` 未初始化，抛出异常；
3. 调用 `search_manager.list_user_mem(user_id, scope_id, nums=page_size, pages=page_idx)` 获取分页数据；
4. 若结果为空，返回空列表；
5. 遍历结果，根据 `memory_type` 过滤（若 `memory_type != MemoryType.UNKNOWN`，只保留匹配类型的记忆）；
6. 构造 `list[MemInfo]` 并返回。

### async update_variables

```python
async def update_variables(
    self,
    variables: dict[str, str],
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> None
```

批量更新用户变量。

**参数**：

- `variables: dict[str, str]`：变量名到变量值的映射字典。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时直接返回。

**行为**：

1. 校验 `scope_id` 格式，无效则返回；
2. 获取用户级分布式锁；
3. 在锁内：若 `variable_manager` 未初始化，抛出 `JiuWenBaseException`（`MEMORY_UPDATE_MEMORY_EXECUTION_ERROR`）；否则遍历 `variables`，对每个 `(name, value)` 调用 `variable_manager.update_user_variable(user_id, scope_id, var_name=name, var_mem=value)`。

### async delete_variables

```python
async def delete_variables(
    self,
    names: list[str],
    user_id: str = "__default__",
    scope_id: str = "__default__",
) -> bool
```

批量删除用户变量。

**参数**：

- `names: list[str]`：要删除的变量名列表。
- `user_id: str`：用户标识符，默认 `"__default__"`。
- `scope_id: str`：作用域标识符，默认 `"__default__"`；格式无效时返回 `False`。

**返回**：

- `bool`：删除成功返回 `True`，`scope_id` 格式无效返回 `False`。

**行为**：

1. 校验 `scope_id` 格式，无效则返回 `False`；
2. 获取用户级分布式锁；
3. 在锁内：若 `variable_manager` 未初始化，抛出 `JiuWenBaseException`（`MEMORY_DELETE_MEMORY_EXECUTION_ERROR`）；否则遍历 `names`，对每个 `name` 调用 `variable_manager.delete_user_variable(user_id, scope_id, var_name=name)`；
4. 返回 `True`。

## 辅助类型

### class MemInfo

```python
class openjiuwen.core.memory.long_term_memory.MemInfo(BaseModel)
```

表示单条记忆的元数据信息。

**字段**：

- `mem_id: str`：记忆唯一标识符。
- `content: str`：记忆内容文本。
- `type: MemoryType`：记忆类型（如 `USER_PROFILE`、`VARIABLE` 等）。

### class MemResult

```python
class openjiuwen.core.memory.long_term_memory.MemResult(BaseModel)
```

表示语义搜索返回的记忆结果，包含记忆信息和相似度分数。

**字段**：

- `mem_info: MemInfo`：记忆信息对象。
- `score: float`：相似度分数（0.0-1.0 之间，越高越相关）。

## 典型使用流程示例

```python
import os
import asyncio
from sqlalchemy.ext.asyncio import create_async_engine

from openjiuwen.core.memory import (
    LongTermMemory,
    MemoryEngineConfig,
    MemoryScopeConfig,
    AgentMemoryConfig,
    MemoryMilvusVectorStore,
)
from openjiuwen.core.foundation.store.in_memory_kv_store import InMemoryKVStore
from openjiuwen.core.foundation.store.default_db_store import DefaultDbStore
from openjiuwen.core.foundation.llm.schema.config import ModelClientConfig, ModelRequestConfig
from openjiuwen.core.foundation.llm.schema.message import UserMessage, AIMessage
from openjiuwen.core.common.schema.param import Param


async def demo_long_term_memory():
    # 1. 创建 LongTermMemory 实例（单例）
    engine = LongTermMemory()

    # 2. 注册底层存储
    kv_store = InMemoryKVStore()
    vector_store = MemoryMilvusVectorStore(
        milvus_host=os.getenv("MILVUS_HOST", "localhost"),
        milvus_port=os.getenv("MILVUS_PORT", "19530"),
        token=os.getenv("MILVUS_TOKEN"),
        embedding_dims=1024,
    )
    async_engine = create_async_engine(
        f"mysql+aiomysql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('AGENT_DB_NAME')}?charset=utf8mb4",
        pool_size=20,
    )
    db_store = DefaultDbStore(async_engine)

    await engine.register_store(
        kv_store=kv_store,
        vector_store=vector_store,
        db_store=db_store,
    )

    # 3. 设置全局配置
    engine_config = MemoryEngineConfig(
        default_model_cfg=ModelRequestConfig(model="<model_name>", temperature=0.0),
        default_model_client_cfg=ModelClientConfig(
            client_id="default_memory_llm",
            client_provider="OpenAI",
            api_key=os.getenv("MEMORY_MODEL_API_KEY"),
            api_base=os.getenv("MEMORY_MODEL_API_BASE"),
        ),
        crypto_key=b"",  # 不启用加密
    )
    engine.set_config(engine_config)

    # 4. 可选：为特定 scope 配置独立的模型/向量参数
    scope_config = MemoryScopeConfig(
        model_cfg=ModelRequestConfig(model="<model_name>", temperature=0.1),
        model_client_cfg=ModelClientConfig(
            client_id="scope_llm",
            client_provider="OpenAI",
            api_key=os.getenv("SCOPE_API_KEY"),
        ),
    )
    await engine.set_scope_config("app_demo_scope", scope_config)

    # 5. 添加消息并生成记忆
    agent_config = AgentMemoryConfig(
        mem_variables=[
            Param(name="favorite_color", description="用户喜欢的颜色", type="string"),
            Param(name="age", description="用户年龄", type="number"),
        ],
        enable_long_term_mem=True,
    )

    messages = [
        UserMessage(content="我喜欢蓝色，今年25岁"),
        AIMessage(content="好的，我记住了您喜欢蓝色，年龄是25岁。"),
    ]

    await engine.add_messages(
        messages=messages,
        agent_config=agent_config,
        user_id="user_001",
        scope_id="app_demo_scope",
        session_id="session_001",
    )

    # 6. 查询变量
    variables = await engine.get_variables(
        names=["favorite_color", "age"],
        user_id="user_001",
        scope_id="app_demo_scope",
    )
    print(f"用户变量: {variables}")  # {'favorite_color': '蓝色', 'age': '25'}

    # 7. 语义搜索记忆
    results = await engine.search_user_mem(
        query="用户喜欢的颜色",
        num=3,
        user_id="user_001",
        scope_id="app_demo_scope",
    )
    for result in results:
        print(f"记忆: {result.mem_info.content}, 相似度: {result.score}")

    # 8. 获取最近消息
    recent = await engine.get_recent_messages(
        user_id="user_001",
        scope_id="app_demo_scope",
        session_id="session_001",
        num=10,
    )
    print(f"最近消息数: {len(recent)}")


asyncio.run(demo_long_term_memory())
```

> **说明**：所有方法中涉及的 `user_id`、`scope_id`、`session_id` 若使用默认值 `"__default__"`，表示使用系统默认标识符；在实际业务中，建议传入有意义的业务标识符以支持多租户隔离和精确查询。
