# Day4 - Xtuner微调LLM

## FineTuning 微调
### 指令跟随微调
对话模板的构建
System(<|System|>), User(<|User|>, <eoh>), Assistant(<|Bot|>, <eoa>)
用户输入的内容自动放入User部分
### 增量预训练微调
BOS: <s>; EOS: </s>.

## LoRA
LoRA的发明， 源于对少量样本微调的有效性的探索性研究[Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning]()，尽管参数矩阵是高维向量， 但本征维数相较而言却低得多， 因此为参数层旁路添加一个可分解为两低秩矩阵的乘积的adapter参数矩阵, 训练中仅更新adapter能够在大大减少训练参数的情况下实现适配, 即LoRA方法[LoRA_Low-Rank Adaptation Of Large Language Models](2021), 而QLoRA则是更进一步基于量化后的模型进行低秩适应， 大大减少微调时所需要的显存。


<image src="img/xtuner_lora.png"/>
<br/>


<image src="img/rag_optim.png"/>
<br/>


## XTuner 实现