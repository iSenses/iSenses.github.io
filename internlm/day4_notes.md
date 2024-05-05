---
layout: post
title: "Day4 - Xtuner微调LLM课程笔记"
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

## FineTuning 微调

增量预训练和指令跟随是经常会用到两种的微调模式  
<image src="img/xt_01.jpg" width="478"/>
<image src="img/xt_02.jpg" width="478" /><br/>


### 增量预训练微调
使用场景：让基座模型学习到一些新知识，如某个垂类领域的常识  
训练数据：文章、书籍、代码等  

### 指令跟随微调
使用场景：让模型学会对话模板，根据人类指令进行对话  
训练数据：高质量的对话、问答数据  

### 一条数据的一生

- 生活中的原始数据，统一格式化为标准格式数据
- 标准格式数据添加对话模板， 将用于输入LLM微调  
  对话模板的构建：  
  System(<|System|>), User(<|User|>, <eoh>), Assistant(<|Bot|>, <eoa>)  
  用户输入的内容自动放入User部分  
- 数据进入LLM首先用其特定tokenizer转化
- 添加Label用于损失计算
- 开始训练

<image src="img/xt_03.jpg" width="478" />
<image src="img/xt_04.jpg" width="478" /><br/>

## LoRA
LoRA的发明， 源于对少量样本微调的有效性的探索性研究[Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning]()，尽管参数矩阵是高维向量， 但本征维数相较而言却低得多， 因此为参数层旁路添加一个可分解为两低秩矩阵的乘积的adapter参数矩阵, 训练中仅更新adapter能够在大大减少训练参数的情况下实现适配, 即LoRA方法[LoRA_Low-Rank Adaptation Of Large Language Models](2021), 而QLoRA则是更进一步基于量化后的模型进行低秩适应， 大大减少微调时所需要的显存。


<image src="img/xt_lora.jpg"/>
<br/>


<image src="img/xt_qlora.jpg"/>
<br/>


## XTuner实现

<image src="img/xt_07.jpg"/><br/>
<image src="img/xt_08.jpg"/><br/>
<image src="img/xt_09.jpg"/><br/>
<image src="img/xt_10.jpg"/><br/>
<image src="img/xt_11.jpg"/><br/>
<image src="img/xt_12.jpg"/><br/>
<image src="img/xt_13.jpg"/><br/>
<image src="img/xt_14.jpg"/><br/>
<image src="img/xt_15.jpg"/><br/>
<image src="img/xt_16.jpg"/><br/>
<image src="img/xt_17.jpg"/><br/>
<image src="img/xt_18.jpg"/><br/>
<image src="img/xt_19.jpg"/><br/>
<image src="img/xt_20.jpg"/><br/>
<image src="img/xt_21.jpg"/><br/>
<image src="img/xt_22.jpg"/><br/>


