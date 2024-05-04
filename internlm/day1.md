---
layout: post
title: "InternLM 第一节课后笔记"
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>


## InternLM开放全体系

<image src="img/journey.jpg"/>
<br/>
开源发展之路一步一个脚印。

<image src="img/features.jpg"/>
<br/>
开源体系非常之全面

<image src="img/comparison.jpg"/>
<br/>
在与相近体量模型的对比中表现优异。

### 训练数据集
包含多模态， 大量科研相关语料
<image src="img/proc.jpg"/>
<br/>

### 微调XTuner

<image src="img/xtuner.jpg"/>
<br/>
Xtuner 是我最兴趣的部分， 因为目前没有资源做大规模的训练。而针对项目进行微调量化是最为现实的路径。

### 部署LMDeploy
<image src="img/lmdeploy1.jpg"/>
<br/>

<image src="img/lmdeploy2.jpg"/>
<br/>

### 评测 OpenCompass
<image src="img/opencompass.jpg"/>
<br/>
一个完整客观的评测体系至关重要，短期内可能有发现， 但要长期进步， 必须要一个有效的评测体系， OpenCompass我愿称这为最佳。

### 应用 Lagent AgentLego
最后是智能体的框架, Lagent能够去支持多种的智能体的能力, 从输入然后接下来怎么去选择和执行工具以及是否有计划材拆分等等。
这样不同的一些智能体的流程其实都是在里面可以去一键支持以及说可以去开发新的流程并且也可以去凝获的支持各种大语言模型。
<image src="img/lmagent1.jpg"/>
<br/>

<image src="img/lmagent2.jpg"/>
<br/>

<image src="img/lmagent3.jpg"/>
<br/>


# InternLM2 技术报告解读 
 [技术报告链接：InternLM2 Technical Report(2024年3月版)](https://arxiv.org/abs/2403.17297)

## 基础架构
- 三个模型大小
  1.8B, 7B, 20B
- 200K Context Window
- 数据整理指南
- Conditional Online RLHF (COOL RLHF)技巧
#### InternEvo 预训练模型
高速可扩展
一个概念MFU(Model FLOPs Utilization), 代表实际训练计算速度与训练用硬件理论的比值, InternEvo 可以达到88%MFU.
对计算系统重要的优化体现在减少GPU的通信， InternEvo采用了一系列adaptive sharding 技术， 包括但不限于Full-Replica, Full-sharding, Partial sharding, 建立了一套优化框架， 扩展阅读 [Reducing communication overhead of zero for efficient llm training, 2024b](https://arxiv.org/abs/2311.00257)
 [InternEvo: Efficient longsequence large language model training via hybrid parallelism and redundant sharding.](https://arxiv.org/abs/2401.09149)
InternLM2 采用Grouped-Query Attention (GQA) ,高速率，低存储，长文本。

## 预训练
### 数据
文本数据以中英文为主，包括常用的Common Crawl, Wiki 等， 并进行了细致的Pipeline处理,采用了“域名拦截”、“文字拦截”、“色情分类器”、“毒性分类器”相结合的综合安全策略来过滤数据。并使用了“Toxic Comment Classification Challenge”数据集对 BERT 模型进行了微调，得到了一个分类器；进一步用Perspective API 对其进行注释，以创建色情分类数据集。然后使用该数据集对 BERT 模型进行微调，生成色情分类器。最后用这两个分类器对数据进行二次过滤，过滤掉分数低于阈值的数据，得到安全数据。

代码数据来自github,同样经过pipeline处理,增加了dependency sorting等环节。

<image src="img/data_pipeline.png"/>
<br/>
长文本能力是 LLMs 研究中越来越受欢迎的主题，它扩大和促进了LLM的应用
- 数据过滤流程：数据处理流程旨在过滤掉低质量的长文本数据。它包括三个阶段： 
  - 长度选择，一个基于规则的过滤器，选择超过 32K 字节的数据样本；
  - 统计过滤器，利用统计特征来识别和删除异常数据； 
  - 困惑度过滤器，利用困惑度的差异来评估文本片段之间的连贯性，过滤掉具有分散注意力的上下文的样本。请注意，为长上下文训练选择的所有数据都是标准预训练语料库的子集，这意味着在预训练期间将至少学习两次长上下文数据。
- 统计过滤器：采用各种词汇和语言特征来构建统计过滤器。不符合既定规则的数据样本将被排除在预训练语料库之外。
- 困惑度过滤：计算相邻文本段落的困惑度差异，过滤掉混杂的上下文。
- 阈值选择：针对每个领域定制阈值，而不是寻求通用解决方案。例如，连词的统计过滤器不适用于长代码数据，因为长代码数据通常不具有任何连词。同样，教科书、研究论文、小说和专利都表现出独特的特征。通用阈值可能会对大量数据进行错误分类。相同的逻辑适用于设置不同语言的阈值；因此，单独调整每个域的阈值。
经过以上过滤之后，长文本的文献来源发生了变化。

<image src="img/data_filtered.png"/>
<br/>

### 分词 Tokenization
基于Tiktoken的c100k加入32397中文token， 仍小于100k。
c100k top-60004 + Chinese 32397 + 147 spare tokens,  共92548.

### 预训练过程
AdamW优化器 
β1 = 0.9, 
β2 = 0.95, 
ϵ = 1e − 8 
weight decay = 0.1  采用  cosine 学习率下降最低至1e-9.
训练采用先4k以下文本， 然后32k以下文本，最后进行功能提升(Capability Specific Enhancement Training)， 每个阶段都是代码中英文混合。

## 对齐
### SFT &&& Cool RLHF
ChatML格式的tool calling, reasoning interleaved with coding (RICO) 策略进行， 这个值得细读(Internlm-math: Open math large language models
toward verifiable reasoning)
## 验证分析
测试分为了6个维度, 每个下面又有细分，更多的信息见 [opencompass](https://github.com/open-compass/opencompass) 。 我个人最感兴真趣的是工具调用这部分。
### 综合测试
### 语言与知识
### 推理与数学
### 代码
### 长文本
### 工具调用
评测采用 GSM8K, MATH, MathBench, T-Eval 还有CIBench。 测试结果优异。

<image src="img/tool_eval.png"/>
<br/>
## 一致性测试

### 对齐表现
  在多个主观对齐相关数据集上评估模型，包括AlpacaEval、MTBench、CompassArena、AlignBench、IFEval等。结果显示，InternLM2系列模型在对齐任务上取得了显著成绩。

### 数据污染评估
  评估模型在GSM8K数据集上的语言建模损失，以判断是否存在数据污染。结果显示，InternLM2系列模型在数据污染评估上表现良好。

## 个人总结
报告花了很大报告花了很大的篇幅来讲数据的处理与训练的过程,指标等其他方面， 就对于我这样没法真正去训练模型的人是一个非常好的补充。 很多方面我缺乏实践的经验，所以也不好真正的做一个判断，但是这对于我开阔视野是有非常大的好处。
整个书生模型最牛的一点就是他的体系完整可靠，从数据处理，预训练，对齐，微调，到部署测评啊，包括几个对其他模型的横向评比，和非深度和广度都非常的大。 全部开源做到非常难，因为其实能做到这么大工作量几乎都是大企业，有非常大的资金支持。
接下来我的目标是基于InternLM2进行微调。所以,我要努力学习吧关于这个整个系统的体系。
20b的模型从我的角度来讲进行全量训练也是不太可能，但是做微调是可以做到的。接下来我的工作重点会放在对微调技术的实战, 细读XTuner, 以及LAgent部分。
