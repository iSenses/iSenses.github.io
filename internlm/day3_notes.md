# Day3 - RAG and huixiangdou
## RAG 技术概要
RAG(Retrieval-Augmented Generation, 检索增强生成)， 是一种基于语言大模型的应用模式。
LLM 在回答问题或生成文本时，先会从指定数据源（数据库， 知识库，权威网站）中检索出相关的信息，然后基于这些信息生成回答或文本，从而提高预测质量。RAG 方法使得开发者不必为每一个特定的任务重新训练整个大模型，只需要外挂上知识库，即可为模型提供额外的信息输入，提高其回答的准确性。RAG模型尤其适合知识密集型的任务。

RAG模型主要由2部分组成，包括检索器与生成器。
检索器从既有结构化数据中生成，生成器负责生成内容，可以是语言大模型，也可以是图像、音频、代码生成器比如VisualGPT, Stable Diffusion, Codex。

检索部分通常分类为稀疏检索器与密集检索器， 稀疏检索器采用一些传统的如TF-IDF/BM25等文档检索算法生成倒排索引；而密集检索通常使用Encoder模型生成的embedding向量，在训练过程中往往采用了对比学习的方法，在推理过程中使用ANN类算法加快搜索。针对代码，知识图谱等也有更多基于其特征的检索器构建方式。
除了这些检索器与生成器直接级联的方式（Query Baed RAG）之外， 还有不是将结果直接输入生成器的如Latent Representation-based RAG，Logit-based RAG和Speculative RAG（可选完全用检索结果代替生成）。



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
- Embedding Optimization
  对原本的稀疏embedding处理生成dense embedding, 并建立ANN索引加速搜索。
- Indexing Optimization
  更好的Chunk method, 并可以加入元数据乃至多模态数据。
- Query Optimization
  Data augmentation, query transformation, multi-query. 
- Context Curation
  重排， 上下文选择， 压缩。
- Iterative Retrieval
  迭代检索。
- Recursive Retrieval
  递归检索。 Chain-of-Thought
- Adaptive Retrieval
  Flare, Self-RAG; 使用LLMs主动决定检索的最佳时机与内容（类似于将Retrieval源作为一个Agent）
- LLM Fine-tuning
  检索微调，生成微调，双重微调。

<image src="img/rag_optim.png"/>
<br/>

### RAG的可能缺陷
- 检索结果的准确性与时效性
- 检索过程带来的额外开销
- 检索的知识与生成器的知识无法对齐带来回答质量下降
- 检索知识占用Context过长导致生成内容不完整


## huixiangdou实现
huixiangdou 是一个开源RAG框架 [技术报告链接：Huixiangdou: Overcoming Group Chat Scenarios With LLM-Based Technical Assistance](https://arxiv.org/abs/2401.08772) [代码](https://github.com/InternLM/HuixiangDou)。
huixiangdou 功能全面，尤其以一个群聊助手为主要应用场景， 是一个包含RAG与web端，Lark前端等功能， 不断进化的RAG应用。
huixiangdou 从架构上说属于最成熟的Query-based RAG, huixiangdou 是通过将不通来源的service加入pipeline过滤后输入LLM进行生成的实现。
huixiangdou支持的外部知识来源包括：1. 向量数据库， 2. 网络搜索结果，3.知识图谱。
经过Reject Pipeline的过滤, 包括对多 ground-truth like source的检索， 最终可以接入本地LLM模型生成也可以接入API, 生成答案， 返回群聊。

<image src="img/huixiangdou_arch.png"/>
<br/>

截至发文， huixiangdou已支持下面大模型直接加载
- InternLM2
- Qwen
- KIMI
- DeepSeek
- ChatGLM (ZHIPU)
- Xi-Api
- OpenAOE
支持下列文件格式作为输入
- pdf
- word
- excel
- ppt
- html
- markdown
- txt
同时有下述app接入支持
- Web
- Lark(飞书)
- WeChat(非官)

huixiangdou部署简单， 特别是建立有效的pipeline, 使得可以直接通过 `resource/good_questions.json`, `resource/bad_questions.json`的修改来控制其应答范围是非常成功的创新。
理论部分先到这里，接下来转战huixiangdou实战篇 [day3 作业 huixiangdou 实战](https://isenses.github.io/internlm/day3_homework)。
