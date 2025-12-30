# retrieval

`openjiuwen.core.retrieval`是openJiuwen框架的检索模块，提供知识库管理、文档索引构建、向量检索、图检索等能力。

**Classes**：

| CLASS                                                         | DESCRIPTION     |
|---------------------------------------------------------------|-----------------|
| [KnowledgeBase](./retrieval/knowledge_base.md)                | 知识库抽象基类。         |
| [SimpleKnowledgeBase](./retrieval/simple_knowledge_base.md)   | 简单知识库实现。         |
| [GraphKnowledgeBase](./retrieval/graph_knowledge_base.md)     | 图知识库实现。          |
| [Retriever](./retrieval/retriever/base.md)                    | 检索器抽象基类。         |
| [VectorRetriever](./retrieval/retriever/vector_retriever.md)  | 向量检索器。            |
| [SparseRetriever](./retrieval/retriever/sparse_retriever.md)  | 稀疏检索器（BM25）。     |
| [HybridRetriever](./retrieval/retriever/hybrid_retriever.md)  | 混合检索器。            |
| [GraphRetriever](./retrieval/retriever/graph_retriever.md)    | 图检索器。              |
| [AgenticRetriever](./retrieval/retriever/agentic_retriever.md)| 智能检索器。            |
| [VectorStore](./retrieval/vector_store/base.md)               | 向量存储抽象基类。       |
| [ChromaVectorStore](./retrieval/vector_store/chroma_store.md) | ChromaDB向量存储实现。   |
| [MilvusVectorStore](./retrieval/vector_store/milvus_store.md)| Milvus向量存储实现。     |
| [Embedding](./retrieval/embedding/base.md)                     | 嵌入模型抽象基类。       |
| [APIEmbedding](./retrieval/embedding/api_embedding.md)        | API嵌入模型实现。        |
| [OllamaEmbedding](./retrieval/embedding/ollama_embedding.md)  | Ollama嵌入模型实现。     |
| [Indexer](./retrieval/indexing/indexer/base.md)               | 索引管理器抽象基类。     |
| [ChromaIndexer](./retrieval/indexing/indexer/chroma_indexer.md)| ChromaDB索引管理器实现。 |
| [MilvusIndexer](./retrieval/indexing/indexer/milvus_indexer.md)| Milvus索引管理器实现。   |

