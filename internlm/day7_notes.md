# Day7: OpenCompass 大模型评测实战

## 
- 全面性
- 评测成本
- 数据污染
- 鲁棒性

<image src="img/op_challenge"/>
<br/>
<image src="img/op_comps.png"/>
<br/>

唯一获得Meta官方推荐的国产大模型评测体系, 包含100+评测集， 50万+题目(竞品LM Evaluation, Harness(HF Leaderboard), HELM, BIG-bench)
## 评测类型
- 客观题
- 主观题

## 评测内容

- Prompt Engineering
- Few-shot learning
- Chain-of-thought
- needle-in-a-stack
- Instruction-following

### 三位一体： CompassHub, CompassRank, CompassKit



### CompassHub
https://hub.opencompass.org.cn/home
包括各种测试标准， 方法， 数据集

<image src="img/op_comps.png"/>
<br/>
### CompassRank
https://rank.opencompass.org.cn/home
包括对多达模型的评测数据排名

<image src="img/op_rank.png"/>
<br/>

### CompassKit
https://github.com/open-compass
- 支持对数据污染的检测， 让评测更准确
- 支持近20个商业模型API, LMDeploy, vLLM, LightLLM等推理后端
- 长文本1M长度多种评测基准
- 主观， 打分， 模型对战等多种能力， 可切换上百种评价模型

<image src="img/op_pipeline.png"/>
<br/>
<image src="img/op_tools.png"/>
<br/>
各种能力维度全面测试

<image src="img/op_caps.png"/>
<br/>
<image src="img/op_datasets.png"/>
<br/>
### 代码结构分析

<image src="img/op_code1.png"/>
<br/>

<image src="img/op_code2.png"/>
<br/>