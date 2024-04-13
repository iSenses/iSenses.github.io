# huixiangdou
huixiangdou 是一个 目标很明确的RAG框架， 它定位于可以在群聊中回答指定领域问题。

## RAG 技术概要
RAG(Retrieval-Augmented Generation, 检索增强生成)， 是一种基于语言大模型的应用模式。
LLM 在回答问题或生成文本时，先会从指定数据源（数据库， 知识库，权威网站）中检索出相关的信息，然后基于这些信息生成回答或文本，从而提高预测质量。RAG 方法使得开发者不必为每一个特定的任务重新训练整个大模型，只需要外挂上知识库，即可为模型提供额外的信息输入，提高其回答的准确性。RAG模型尤其适合知识密集型的任务。

### RAG vs. Fine-Tuning
- RAG
  - No parameters change, 使用独立于深度模型的知识库
  - 资源需求少， 修改容易
- Fine-Tuning
  - 改变深度模型参数，可能带来过拟合
  - 资源需求大。

<image src="img/rag_vs_ft.png"/>
<br/>
### RAG 常见优化方法
### Embedding Optimization
对原本的稀疏embedding处理生成dense embedding, 并建立ANN索引加速搜索。
### Indexing Optimization
更好的Chunk method, 并可以加入元数据乃至多模态数据。
### Query Optimization
Data augmentation, query transformation, multi-query. 
### Context Curation
重排， 上下文选择， 压缩。
### Iterative Retrieval
迭代检索。
### Recursive Retrieval
递归检索。 Chain-of-Thought
### Adaptive Retrieval
Flare, Self-RAG; 使用LLMs主动决定检索的最佳时机与内容（类似于将Retrieval源作为一个Agent）
### LLM Fine-tuning
检索微调，生成微调，双重微调。

<image src="img/rag_optim.png"/>
<br/>
## huixiangdou实现architecture
huixiangdou 是一个群聊助手为主要场景的， 包含RAG与web端，Lark前端等功能， 不断进化的RAG应用。
huixiangdou 从架构上说属于最成熟的Query-based RAG, huixiangdou 是通过将不通来源的service加入pipeline过滤后输入LLM进行生成的实现。
huixiangdou支持的外部知识来源包括：1. 向量数据库， 2. 网络搜索结果，3.知识图谱。
经过Reject Pipeline的过滤, 包括对多 ground-truth like source的检索， 最终可以接入本地LLM模型生成也可以接入API, 生成答案， 返回群聊。

<image src="img/huixiangdou_arch.png"/>
<br/>

<image src="img/cemetery2.png"/>
<br/>

<image src="img/cemetery2.png"/>
<br/>

<image src="img/cemetery2.png"/>
<br/>
