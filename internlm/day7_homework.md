---
layout: post
title: "Day7 - opencompass 作业实战"
output: html_document
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

## 基础作业

### 使用 OpenCompass 评测 internlm2-chat-1_8b <br/> 模型在 C-Eval 数据集上的性能
首先安装并激活环境，安装 opencompass-0.2.4：  


```shell
studio-conda -o internlm-base -t opencompass
source activate opencompass
git clone -b 0.2.4 https://github.com/open-compass/opencompass
cd opencompass
pip install -r requirements.txt
pip install -e .
```

环境安装好之后准备数据集，贴心的老师们已经准备好啦！

```shell
cp /share/temp/datasets/OpenCompassData-core-20231110.zip /root/demo/opencompass
unzip -q OpenCompassData-core-20231110.zip
```


列出所有跟 InternLM 及 C-Eval 相关的配置：
```shell
python tools/list_configs.py internlm ceval

```

<image src="img/op_hw_list.png" width="960"/>
<br/>


可以直接调用`run.py`运行测试, 记得之前先修改环境变量`MLK_SERVICE_FORCE_INTEL`, `--debug`选项将过程回显到屏幕。

```bash
cd ~/demo/opencompass

export MLK_SERVICE_FORCE_INTEL=1

python tools/list_configs.py internlm ceval
python run.py --datasets ceval_gen --hf-path /share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b \
--tokenizer-path /share/new_model/Shanghai_AI_Laboratory/internlm2-chat-1_8b \
--tokenizer-kwargs padding_side='left' truncation='left' trust_remote_code=True \
--max-seq-len 2048 \ # 模型可以接受的最大序列长度
--max-out-len 16 \ # 生成的最大 token 数, 客观题16就够了
--batch-size 4 \
--num-gpus 1 \
--reuse latest \
--workdir /root/demo/opencompass/results #--debug
```

结果如下： 
<image src="img/op_hw_ceval.png" width="960"/>
<br/>

随手对`internlm-chat-7b` 用C-Eval数据集做一个测评， 方便与1.8b模型对比，发现这个7b模型accuracy比1.8b略差（见下图），其 naive_average 为44.16， 小于 `internlm2-chat-1_8b`的46.04， 怎么模型大了分值还小了呢？是我做错了吗？  
我核查了predictions文件夹里的结果， 发现结果确实如此。再仔细对比， 才意识到我用的7b是internlm第一版本， 而1.8b是internlm2了，原来1.8b已经在ceval上略超了上一版的7b了， 大佬们真是太给力了！！  

<image src="img/op_hw_comp.png" width="960"/>
<br/>



## 进阶作业

### 将自定义数据集提交至OpenCompass官网

首先我找回之前收集的一些GQL查询例子的一个parquet，只有几十个仅供演示。进行数据分割，写入`train.jsonl`, `dev.jsonl`, `test.jsonl`三个数据集
```python
import pandas as pd
import numpy as np

df = pd.read_parquet("G:/InternLM_Course/train-00000-of-00001.parquet")
train , validate, test = np.split(df.sample(frac=1), [int(.6*len(df)), int(.8*len(df))])
with open("G:/InternLM_Course/op_gql_tiny_data/train.jsonl", "a") as f:
     test.to_json(f, orient='records', lines=True)
with open("G:/InternLM_Course/op_gql_tiny_data/dev.jsonl", "a") as f:
     validate.to_json(f, orient='records', lines=True)
with open("G:/InternLM_Course/op_gql_tiny_data/test.jsonl", "a") as f:
     test.to_json(f, orient='records', lines=True)

```

数据集需要有下载地址， 放到自己的github上了[op_gql_tiny_data](https://github.com/iSenses/op_gql_tiny_data)  
接下来就是根据[OpenCompass平台指引 | 贡献数据集](https://mp.weixin.qq.com/s/_s0a9nYRye0bmqVdwXRVCg)填写`README_OPENCOMPASS.md`   

```
---
name: GQL_Tiny
desc: A Tiny GQL dataset means as seed for further GQL data collection and exploration.
language:
- en
dimension:
- code
sub_dimension:
- gql
website: https://github.com/iSenses/op_gql_tiny_data
github: https://github.com/iSenses/op_gql_tiny_data
paper: https://github.com/iSenses/op_gql_tiny_data
release_date: 2024-04-30
tag:
- cypher
- query
- text
download_url: https://github.com/iSenses/op_gql_tiny_data

---
## Introduction
A Tiny GQL dataset means as example for further GQL data collection and exploration.
## Meta Data
- code: cypher
- datasize: 36
- language: English
## Example
```json
{
"input_text":"Schema:
	(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_SKILL {}]->(:Skill {label:'Skill', id: string,name: string,level: string})
	(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_EDUCATION {}]->(:Education {label:'Education', id: string, degree: string, university: string, graduationDate: string, score: string, url: string})
	Question: How many people have a Ph.D. in Physics and are experts in C++?",

"output_text":"```MATCH (p:Person)-[:HAS_SKILL]->(s:Skill), (p)-[:HAS_EDUCATION]->(e:Education) WHERE toLower(s.name) CONTAINS 'c++' AND toLower(s.level) CONTAINS 'expert' AND toLower(e.degree) CONTAINS 'ph.d.' AND toLower(e.university) CONTAINS 'physics' RETURN COUNT(p)```"}
```

## Citation
```

最后上传： https://hub.opencompass.org.cn/dataset-submit?lang=[object%20Object]


[提交后数据集的链接](https://hub.opencompass.org.cn/dataset-detail/GQL_tiny)


<image src="img/op_hw_GQL.png" width="960"/>
<br/>

完结撒花~  
