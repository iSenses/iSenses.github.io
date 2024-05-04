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

- 使用 OpenCompass 评测 internlm2-chat-1_8b 模型在 C-Eval 数据集上的性能
首先安装并激活环境，安装opencompass-0.2.4：  


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
我核查了predictions文件夹里的结果， 发现结果确实如此。再仔细对比， 才意识到7b是interlm第一版本， 而1.8b是internlm2了，原来1.8b已经在ceval上略超了上一版的7b了， 大佬们真是太给力了！！  

<image src="img/op_hw_comp.png" width="960"/>
<br/>



## 进阶作业

- 将自定义数据集提交至OpenCompass官网

提交地址：https://hub.opencompass.org.cn/dataset-submit?lang=[object%20Object]  
提交指南：https://mp.weixin.qq.com/s/_s0a9nYRye0bmqVdwXRVCg  
Tips：不强制要求配置数据集对应榜单（ leaderboard.xlsx ），可仅上传 EADME_OPENCOMPASS.md 文档  
TODO:  
1. 启动OpenCompass的评测
2. opencompass代码的运行逻辑
3. 实现一个自定义的数据集, 并评测
4. 其他功能
