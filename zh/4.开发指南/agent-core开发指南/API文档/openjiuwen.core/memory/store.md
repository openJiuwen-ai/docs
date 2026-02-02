 # openjiuwen.core.foundation.store
 ## class openjiuwen.core.foundation.store.BaseKVStore
 
 kv_store抽象基类, 包括以下接口。
 
 ### abstractmethod async set
 
 ```python
 set(key: str, value: str)
 ```
 
 set函数，用于设置key,value键值对。
 
 ​**参数**​：
 
 * ​**key**​(str)：设置的键值。默认值：无。
 * ​**value**​(str)：设置的value值。默认值：无。
 
 ### abstractmethod async exclusive_set
 
 ```python
 exclusive_set(key: str, value: str, expiry: int | None = None) -> bool
 ```
 
 原子性地设置key,value键值对，仅当该键不存在时才进行设置。
 
 ​**参数**​：
 
 * ​**key**​(str)：设置的键值。默认值：无。
 * ​**value**​(str)：设置的value值。默认值：无。
 * ​**expiry**​(str)：键值对的过期时间，超过后值可被覆盖，可选。默认值：无。
 
 **返回：**
 ​**bool**，如果键值对设置成功则返回True, 键已存在并且还未过期则返回False。
 
 ### abstractmethod async get
 
 ```python
 get(key: str) -> str | None
 ```
 
 从数据库获取指定的key值对应的value，如果没有则返回None。
 
 ​**参数**​：
 
 * ​**key**​(str)：需要获取的key值。默认值：无。
 
 **返回：**
 ​**str | None**​，返回key对应的value值，如果key不存在则返回None。
 
 ### abstractmethod async exists
 
 ```python
 exists(key: str) -> bool
 ```
 
 判断数据库中某个key值是否存在。
 
 ​**参数**​：
 
 * ​**key**​(str)：键值。默认值：无。
 
 **返回：**
 ​**bool**，如果存在则返回True, 否则返回False。
 
 ### abstractmethod async delete
 
 ```python
 delete(key: str)
 ```
 
 删除数据库中指定key值。
 
 ​**参数**​：
 
 * ​**key**​(str)：键值。默认值：无。
 
 **返回：**
 ​**bool**​，如果值存在且删除成功返回True，否则返回False。
 
 ### abstractmethod async get_by_prefix
 
 ```python
 get_by_prefix(prefix: str) -> dict[str, str]
 ```
 
 按照指定的前缀字符串从数据库获取匹配的若干key,value键值对。
 
 ​**参数**​：
 
 * ​**prefix**​(str)：前缀匹配键值。默认值：无。
 
 **返回：**
 ​**dict[str, str]​**​，返回前缀匹配的若干key,value键值对。
 
 ### abstractmethod async delete_by_prefix
 
 ```python
 delete_by_prefix(prefix: str)
 ```
 
 按照指定的前缀字符串删除数据库中匹配的key,value键值对。
 
 ​**参数**​：
 
 * ​**prefix**​(str)：前缀匹配键值。默认值：无。
 
 ---
 
 ### abstractmethod async mget
 
 ```python
 mget(keys: List[str]) -> List[str | None]
 ```
 
 批量获取键值对，如果某个key不存在，返回的列表中，对应位置的值为None。
 
 ​**参数**​：
 
 * ​**keys**​(List[str])：需要获取的key数组。默认值：无。
 
 **返回：**
 ​**List[str | None]​**​，返回keys对应的value值，如果某个key不存在，则列表对应位置的值为None。
 
 **样例：**
 
 ```python
 import json
 import time
 from typing import Dict, List, Optional
 
 from sqlalchemy import (
     Column,
     String,
     delete,
     select,
 )
 from sqlalchemy.dialects.sqlite import insert
 from sqlalchemy.ext.asyncio import (
     async_sessionmaker, AsyncEngine, AsyncSession,
 )
 from sqlalchemy.orm import DeclarativeBase
 from openjiuwen.core.memory.store.base_kv_store import BaseKVStore
 
 
 class Base(DeclarativeBase):
     pass

 
 class KVStoreTable(Base):
     __tablename__ = "kv_store"
     key = Column(String, primary_key=True)
     value = Column(String, nullable=False)
 
 
 class DefaultKVStore(BaseKVStore):
     def __init__(self, engine: AsyncEngine):
         self.engine = engine
         self.async_session = async_sessionmaker(
             self.engine, expire_on_commit=False,
             class_=AsyncSession
         )
         self.table_created = False
 
     async def set(self, key: str, value: str):
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             async with session.begin():
                 stmt = (
                     insert(KVStoreTable)
                     .values(key=key, value=value)
                     .on_conflict_do_update(
                         index_elements=["key"],
                         set_={"value": value}
                     )
                 )
                 await session.execute(stmt)
 
     async def exclusive_set(
         self, key: str, value: str, expiry: Optional[int] = None
     ) -> bool:
         await self._create_table_if_not_exist()
         now = time.time()
         async with self.async_session() as session:
             async with session.begin():
                 stmt = select(KVStoreTable).where(
                     KVStoreTable.key == key
                 )
                 row = (await session.execute(stmt)).scalar_one_or_none()
                 if row is not None:
                     try:
                         data = json.loads(row.value)
                         old_expire = data.get("expire")
                         if old_expire is None or old_expire > now:
                             return False
                     except json.JSONDecodeError:
                         return False
                 expire_at = now + expiry if expiry else None
                 val = json.dumps({"value": value, "expiry": expire_at})
                 stmt = (
                     insert(KVStoreTable)
                     .values(key=key, value=val)
                     .on_conflict_do_update(
                         index_elements=["key"],
                         set_={"value": val}
                     )
                 )
                 await session.execute(stmt)
                 return True
 
     async def get(self, key: str) -> str | None:
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             stmt = select(KVStoreTable).where(KVStoreTable.key == key)
             rec = (await session.execute(stmt)).scalar_one_or_none()
             if rec is None:
                 return None
             return rec.value
 
     async def exists(self, key: str) -> bool:
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             stmt = select(KVStoreTable).where(KVStoreTable.key == key)
             rec = (await session.execute(stmt)).scalar_one_or_none()
             return rec is not None
 
     async def delete(self, key: str):
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             async with session.begin():
                 await session.execute(
                     delete(KVStoreTable).where(KVStoreTable.key == key)
                 )
 
     async def get_by_prefix(self, prefix: str) -> dict[str, str]:
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             stmt = select(KVStoreTable).where(
                 KVStoreTable.key.startswith(prefix)
             )
             rows = (await session.execute(stmt)).scalars().all()
             result: Dict[str, str] = {}
             for rec in rows:
                 result[rec.key] = rec.value
             return result
 
     async def delete_by_prefix(self, prefix: str):
         await self._create_table_if_not_exist()
         async with self.async_session() as session:
             async with session.begin():
                 await session.execute(
                     delete(KVStoreTable).where(
                         KVStoreTable.key.startswith(prefix)
                     )
                 )

     async def mget(self, keys: List[str]) -> List[str | None]:
         await self._create_table_if_not_exist()
         if not keys:
             return []
         async with self.async_session() as session:
             stmt = select(KVStoreTable).where(KVStoreTable.key.in_(keys))
             rows = (await session.execute(stmt)).scalars().all()
             lookup: Dict[str, str] = {}
             for rec in rows:
                 lookup[rec.key] = rec.value
             return [lookup.get(k) for k in keys]
 
     async def _create_table_if_not_exist(self) -> None:
         if self.table_created:
             return
         async with self.engine.begin() as conn:
             await conn.run_sync(Base.metadata.create_all)
         self.table_created = True
 ```
 
 ## class openjiuwen.core.retrieval.VectorStore
 具体的抽象接口定义请参考retrieval/vector_store/base.md。

 **样例：**

 ```python
 # coding: utf-8
 # Copyright (c) Huawei Technologies Co., Ltd. 2025. All rights reserved.
 import asyncio
 from typing import List, Any, Optional
 from pymilvus import FieldSchema, CollectionSchema, DataType, Collection, connections, utility
 from openjiuwen.core.common.logging import logger
 from openjiuwen.core.retrieval.vector_store.base import VectorStore
 from openjiuwen.core.retrieval.common.retrieval_result import SearchResult
 from openjiuwen.core.common.exception.codes import StatusCode
 from openjiuwen.core.common.exception.errors import build_error
 

 MEMORY_ID_LENGTH = 36
 
 
 def convert_milvus_result(results) -> List[SearchResult]:
     final_results: List[SearchResult] = []
     for hits_per_query in results:

         for hit in hits_per_query:
             memory_id = hit.entity.get("id")
             distance = hit.distance
             final_results.append(
                 SearchResult(
                     id=str(memory_id),
                     score=distance,
                     text="",
                     metadata={}
                 )
             )
     return final_results
 
 
 class MemoryMilvusVectorStore(VectorStore):
 
     def __init__(self, milvus_host: str, milvus_port: str, token: str | None, embedding_dims: int):

         self.embedding_dims = embedding_dims
         self.token = token
         self.milvus_host = milvus_host
         self.milvus_port = milvus_port
         self.timeout = 3
         self.collections = {}

     @staticmethod
     def create_client(database_name: str, path_or_uri: str, token: str = "", **kwargs) -> Any:
         logger.error("create_client not implemented in MemoryMilvusVectorStore")
         pass
 
     async def _ensure_connection(self):
         try:
             await asyncio.to_thread(
                 connections.connect,
                 host=self.milvus_host,
                 port=self.milvus_port,
                 alias="default",
                 token=self.token,
                 timeout=self.timeout
             )
         except Exception as e:
             raise build_error(
                 StatusCode.MEMORY_CONNECT_STORE_EXECUTION_ERROR,
                 store_type="milvus vector store",
                 error_msg=f"milvus connect error: {str(e)}",
                 cause=e
             ) from e

     async def _get_collection(self, collection_name: str) -> Collection:
         await self._ensure_connection()
         if collection_name in self.collections:
             return self.collections[collection_name]
         if not utility.has_collection(collection_name):
             logger.info(f"Collection {collection_name} not found, creating...")
             fields = [
                 FieldSchema(name="id", dtype=DataType.VARCHAR,
                             is_primary=True, max_length=MEMORY_ID_LENGTH),
                 FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR,
                             dim=self.embedding_dims)
             ]
             schema = CollectionSchema(fields, description="embedding collection")
             collection = Collection(name=collection_name, schema=schema, using="default")
             index_params = {"index_type": "IVF_FLAT", "metric_type": "IP", "params": {"nlist": 128}}
             collection.create_index("embedding", index_params)
             logger.info(f"Index created for collection {collection_name}")
         else:
             logger.info(f"milvus collection already exists: {collection_name}")
             collection = Collection(name=collection_name, using="default")
         collection.load()
         self.collections[collection_name] = collection
         return collection
 
     async def add(self, data: dict | List[dict], batch_size: int | None = 128, **kwargs: Any):
         table_name = kwargs.get("table_name")
         if table_name is None:
             raise build_error(
                 StatusCode.MEMORY_STORE_VALIDATION_INVALID,
                 store_type="milvus vector store",
                 error_msg=f"table_name is required for add operation",
             )
         if isinstance(data, dict):
             data = [data]
         collection = await self._get_collection(collection_name=table_name)
         embeddings = [d["embedding"] for d in data]
         memory_ids = [d["id"] for d in data]
         await asyncio.to_thread(
             collection.insert,
             [
                 memory_ids,
                 embeddings,
             ],
             timeout=self.timeout
         )
 
     async def search(self, query_vector: List[float], top_k: int = 5,
                      filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]:
         table_name = kwargs.get("table_name")
         if table_name is None:
             raise build_error(
                 StatusCode.MEMORY_STORE_VALIDATION_INVALID,
                 store_type="milvus vector store",
                 error_msg=f"table_name is required for search operation",
             )
         collection = await self._get_collection(table_name)
         results = await asyncio.to_thread(
             collection.search,
             data=[query_vector],
             anns_field="embedding",
             param={"metric_type": "IP", "params": {"nprobe": 10}},
             limit=top_k,
             timeout=self.timeout,
         )
         parsed_results = convert_milvus_result(results)
         return parsed_results if parsed_results else []
 
     async def sparse_search(self, query_text: str, top_k: int = 5,
                             filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]:
         pass
 
     async def hybrid_search(self, query_text: str, query_vector: Optional[List[float]] = None,
                             top_k: int = 5, alpha: float = 0.5, filters: Optional[dict] = None,
                             **kwargs: Any,
     ) -> List[SearchResult]:
         pass
 
     async def delete(self, ids: Optional[List[str]] = None,
                      filter_expr: Optional[str] = None, **kwargs: Any) -> bool:
         table_name = kwargs.get("table_name")
         if table_name is None:
             raise build_error(
                 StatusCode.MEMORY_STORE_VALIDATION_INVALID,
                 store_type="milvus vector store",
                 error_msg=f"table_name is required for delete operation",
             )
         await self._ensure_connection()
         if not utility.has_collection(table_name, using="default"):
             logger.debug(f"Milvus Collection {table_name} does not exist, skip delete vector")
             return True
         collection = await self._get_collection(table_name)
         ids_str = ", ".join(f'"{i}"' for i in ids)
         expr = f'id in [{ids_str}]'
         await asyncio.to_thread(
             collection.delete,
             expr,
             timeout=self.timeout,
         )
         return True
 
     async def delete_table(self, table_name: str) -> bool:
         await self._ensure_connection()
         if not await self.table_exists(table_name):
             logger.debug(f"Milvus Collection {table_name} does not exist, skip delete collection")
             return True
         await asyncio.to_thread(
             utility.drop_collection,
             table_name,
             timeout=self.timeout,
         )
         self.collections.pop(table_name, None)
         return True
 
     async def table_exists(self, table_name: str) -> bool:
         return utility.has_collection(table_name, using="default")
 
     def check_vector_field(self) -> None:
         """Check if vector field configuration is consistent with actual database"""
         logger.error("check_vector_field not implemented in MemoryMilvusVectorStore")
 ```
 
 ## class openjiuwen.core.foundation.store.BaseDbStore
 
 db_store抽象基类, 包括以下接口。
 
 ### abstractmethod async get_async_engine
 
 ```python
 get_async_engine(self) -> AsyncEngine
 ```
 
 get_async_engine函数，用于获取异步DB Engine对象。
 
 **返回：**
 ​**AsyncEngine**​，异步DB Engine对象。
 
 **样例：**
 
 ```python
 from sqlalchemy.ext.asyncio import AsyncEngine
 
 from openjiuwen.core.memory.store.base_db_store import BaseDbStore
 
 
 class DefaultDbStore(BaseDbStore):
     def __init__(self, async_conn: AsyncEngine):
         self.async_conn = async_conn
 
     def get_async_engine(self) -> AsyncEngine:
         return self.async_conn
 ```
 

# openjiuwen.core.memory.store.impl.memory_chroma_vector_store
## class openjiuwen.core.memory.store.impl.memory_chroma_vector_store.MemoryChromaVectorStore

Chroma向量存储实现类，继承自VectorStore抽象类，用于管理和操作Chroma向量数据库中的向量数据。

### method __init__

```python
__init__(persist_directory: str)
```

初始化Chroma向量存储客户端。

**参数**：

* **persist_directory**(str)：Chroma数据库的持久化存储目录路径。默认值：无。

### method create_client

```python
create_client(database_name: str, path_or_uri: str, token: str = "", **kwargs) -> Any
```

创建客户端连接（当前未实现）。

**参数**：

* **database_name**(str)：数据库名称。默认值：无。
* **path_or_uri**(str)：数据库路径或URI。默认值：无。
* **token**(str)：认证令牌。默认值：""。
* **kwargs**(Any)：额外参数。默认值：无。

**返回**：

**Any**，客户端对象（当前返回None）。

### async method get_collection

```python
get_collection(table_name: str)
```

获取或创建指定名称的Chroma集合。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

**返回**：

**Collection**，Chroma集合对象。

### method remove_collection_from_cache

```python
remove_collection_from_cache(table_name: str)
```

从缓存中移除指定的集合。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

### method check_table_name

```python
check_table_name(table_name: Optional[str] = None, operation: Optional[str] = None)
```

检查集合名称是否有效。

**参数**：

* **table_name**(Optional[str])：集合名称。默认值：None。
* **operation**(Optional[str])：当前操作名称，用于错误信息。默认值：None。

**异常**：

* 如果集合名称为空或None，抛出VALIDATION_INVALID异常。

### async method table_exists

```python
table_exists(table_name: str) -> bool
```

检查指定名称的集合是否存在。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

**返回**：

**bool**，如果集合存在返回True，否则返回False。

### async method add

```python
add(data: dict | List[dict], batch_size: int | None = 128, **kwargs: Any) -> None
```

向向量存储中添加向量数据。

**参数**：

* **data**(dict | List[dict])：要添加的向量数据，每个字典包含"id"和"embedding"字段。默认值：无。
* **batch_size**(int | None)：批量处理大小。默认值：128。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

### async method search

```python
search(query_vector: List[float], top_k: int = 5, filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]
```

根据查询向量搜索相似向量。

**参数**：

* **query_vector**(List[float])：查询向量。默认值：无。
* **top_k**(int)：返回结果的数量。默认值：5。
* **filters**(Optional[dict])：过滤条件（当前未使用）。默认值：None。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

**返回**：

**List[SearchResult]**，搜索结果列表，每个结果包含id、text和score字段。

### async method delete

```python
delete(ids: Optional[List[str]] = None, filter_expr: Optional[str] = None, **kwargs: Any) -> bool
```

删除向量数据。

**参数**：

* **ids**(Optional[List[str]])：要删除的向量ID列表。默认值：None。
* **filter_expr**(Optional[str])：过滤表达式（当前未使用）。默认值：None。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

**返回**：

**bool**，删除操作是否成功。

### async method delete_table

```python
delete_table(table_name: str) -> bool
```

删除指定名称的集合。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

**返回**：

**bool**，删除操作是否成功。

### method check_vector_field

```python
check_vector_field() -> None
```

检查向量字段配置是否与实际数据库一致（当前未实现）。

**返回**：

None


# openjiuwen.core.memory.store.impl.memory_milvus_vector_store
## function openjiuwen.core.memory.store.impl.memory_milvus_vector_store.convert_milvus_result

将Milvus搜索结果转换为SearchResult对象列表。

```python
convert_milvus_result(results) -> List[SearchResult]
```

**参数**：

* **results**：Milvus搜索返回的原始结果。默认值：无。

**返回**：

**List[SearchResult]**，转换后的搜索结果列表，每个结果包含id、text、score和metadata字段。

## class openjiuwen.core.memory.store.impl.memory_milvus_vector_store.MemoryMilvusVectorStore

Milvus向量存储实现类，继承自VectorStore抽象类，用于管理和操作Milvus向量数据库中的向量数据。

### method __init__

```python
__init__(milvus_host: str, milvus_port: str, token: str | None, embedding_dims: int)
```

初始化Milvus向量存储客户端连接参数。

**参数**：

* **milvus_host**(str)：Milvus服务器主机地址。默认值：无。
* **milvus_port**(str)：Milvus服务器端口号。默认值：无。
* **token**(str | None)：Milvus认证令牌。默认值：None。
* **embedding_dims**(int)：向量嵌入的维度。默认值：无。

### method create_client

```python
create_client(database_name: str, path_or_uri: str, token: str = "", **kwargs) -> Any
```

创建客户端连接（当前未实现）。

**参数**：

* **database_name**(str)：数据库名称。默认值：无。
* **path_or_uri**(str)：数据库路径或URI。默认值：无。
* **token**(str)：认证令牌。默认值：""。
* **kwargs**(Any)：额外参数。默认值：无。

**返回**：

**Any**，客户端对象（当前返回None）。

### async method _ensure_connection

```python
_ensure_connection()
```

确保Milvus连接已建立，如未建立则创建连接。

**异常**：

* 如果连接失败，抛出MEMORY_CONNECT_STORE_EXECUTION_ERROR异常。

### async method _get_collection

```python
_get_collection(collection_name: str) -> Collection
```

获取或创建指定名称的Milvus集合。

**参数**：

* **collection_name**(str)：集合名称。默认值：无。

**返回**：

**Collection**，Milvus集合对象。

### async method add

```python
add(data: dict | List[dict], batch_size: int | None = 128, **kwargs: Any) -> None
```

向向量存储中添加向量数据。

**参数**：

* **data**(dict | List[dict])：要添加的向量数据，每个字典包含"id"和"embedding"字段。默认值：无。
* **batch_size**(int | None)：批量处理大小。默认值：128。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

**异常**：

* 如果未提供table_name，抛出MEMORY_STORE_VALIDATION_INVALID异常。

### async method search

```python
search(query_vector: List[float], top_k: int = 5, filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]
```

根据查询向量搜索相似向量。

**参数**：

* **query_vector**(List[float])：查询向量。默认值：无。
* **top_k**(int)：返回结果的数量。默认值：5。
* **filters**(Optional[dict])：过滤条件（当前未使用）。默认值：None。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

**返回**：

**List[SearchResult]**，搜索结果列表，每个结果包含id、text、score和metadata字段。

**异常**：

* 如果未提供table_name，抛出MEMORY_STORE_VALIDATION_INVALID异常。

### async method sparse_search

```python
sparse_search(query_text: str, top_k: int = 5, filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]
```

稀疏搜索方法（当前未实现）。

**参数**：

* **query_text**(str)：查询文本。默认值：无。
* **top_k**(int)：返回结果的数量。默认值：5。
* **filters**(Optional[dict])：过滤条件。默认值：None。
* **kwargs**(Any)：额外参数。

**返回**：

**List[SearchResult]**，搜索结果列表。

### async method hybrid_search

```python
hybrid_search(query_text: str, query_vector: Optional[List[float]] = None, top_k: int = 5, alpha: float = 0.5, filters: Optional[dict] = None, **kwargs: Any) -> List[SearchResult]
```

混合搜索方法（当前未实现）。

**参数**：

* **query_text**(str)：查询文本。默认值：无。
* **query_vector**(Optional[List[float]])：查询向量。默认值：None。
* **top_k**(int)：返回结果的数量。默认值：5。
* **alpha**(float)：文本搜索和向量搜索的权重比例。默认值：0.5。
* **filters**(Optional[dict])：过滤条件。默认值：None。
* **kwargs**(Any)：额外参数。

**返回**：

**List[SearchResult]**，搜索结果列表。

### async method delete

```python
delete(ids: Optional[List[str]] = None, filter_expr: Optional[str] = None, **kwargs: Any) -> bool
```

删除向量数据。

**参数**：

* **ids**(Optional[List[str]])：要删除的向量ID列表。默认值：None。
* **filter_expr**(Optional[str])：过滤表达式（当前未使用）。默认值：None。
* **kwargs**(Any)：额外参数，必须包含"table_name"指定集合名称。

**返回**：

**bool**，删除操作是否成功。

**异常**：

* 如果未提供table_name，抛出MEMORY_STORE_VALIDATION_INVALID异常。

### async method delete_table

```python
delete_table(table_name: str) -> bool
```

删除指定名称的Milvus集合。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

**返回**：

**bool**，删除操作是否成功。

### async method table_exists

```python
table_exists(table_name: str) -> bool
```

检查指定名称的Milvus集合是否存在。

**参数**：

* **table_name**(str)：集合名称。默认值：无。

**返回**：

**bool**，如果集合存在返回True，否则返回False。

### method check_vector_field

```python
check_vector_field() -> None
```

检查向量字段配置是否与实际数据库一致（当前未实现）。

