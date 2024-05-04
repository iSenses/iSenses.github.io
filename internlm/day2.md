---
layout: post
title: "Day2 InternLM2 Demo && Homework"
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

## Demo1 部署 InternLM2-Chat-1.8B 模型进行智能对话
### 1. 创建开发机 CUDA-11.7 conda




```bash
studio-conda -o internlm-base -t demo
# activate
conda activate demo
pip install huggingface-hub==0.17.3
pip install transformers==4.34 
pip install psutil==5.9.8
pip install accelerate==0.24.1
pip install streamlit==1.32.2 
pip install matplotlib==3.8.3 
pip install modelscope==1.9.5
pip install sentencepiece==0.1.99
```

### 2. 下载 InternLM2-Chat-1.8B 模型
```bash
mkdir -p /root/demo
cd /root/demo
touch cli_demo.py
touch download_mini.py
```

download_mini.py 用于下载模型文件
```python

import os
from modelscope.hub.snapshot_download import snapshot_download

os.system("mkdir /root/models")

save_dir="/root/models"

snapshot_download("Shanghai_AI_Laboratory/internlm2-chat-1_8b", cache_dir=save_dir, revision='v1.1.0')
```

### 3. 运行
cli_demo.py 用于CLI启动
```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM


model_name_or_path = "/root/models/Shanghai_AI_Laboratory/internlm2-chat-1_8b"

tokenizer = AutoTokenizer.from_pretrained(model_name_or_path, trust_remote_code=True, device_map='cuda:0')
model = AutoModelForCausalLM.from_pretrained(model_name_or_path, trust_remote_code=True, torch_dtype=torch.bfloat16, device_map='cuda:0')
model = model.eval()

system_prompt = """You are an AI assistant whose name is InternLM (书生·浦语).
- InternLM (书生·浦语) is a conversational language model that is developed by Shanghai AI Laboratory (上海人工智能实验室). It is designed to be helpful, honest, and harmless.
- InternLM (书生·浦语) can understand and communicate fluently in the language chosen by the user such as English and 中文.
"""

messages = [(system_prompt, '')]

print("=============Welcome to InternLM chatbot, type 'exit' to exit.=============")

while True:
    input_text = input("\nUser  >>> ")
    input_text = input_text.replace(' ', '')
    if input_text == "exit":
        break

    length = 0
    for response, _ in model.stream_chat(tokenizer, input_text, messages):
        if response is not None:
            print(response[length:], flush=True, end="")
            length = len(response)
```

完成环境安装（时间不短）：

<image src="img/env1.8B.png"/>
<br/>

运行cli_demo.py 就可以对话了：

<image src="img/1.8B.png"/>
<br/>
## Demo2 角色扮演 八戒-Chat-1.8B

### 1. 环境配置
```bash
conda activate demo
cd /root/
git clone https://gitee.com/InternLM/Tutorial -b camp2
cd /root/Tutorial
python helloworld/bajie_download.py
streamlit run /root/Tutorial/helloworld/bajie_chat.py --server.address 127.0.0.1 --server.port 6006
```

利用端口反射映射到本地端口（还是第一次这么做）。
```bash
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p 41488
```

<image src="img/port.jpg"/>
<br/>
### 2. 聊起来
<image src="img/pig.png"/>
<br/>

<image src="img/pig2.png"/>
<br/>

## Demo 3 使用 Lagent 运行 InternLM2-Chat-7B 模型
### 1. 环境配置
因为继承了前面横型的部署环境， 部署步骤简单了许多：
```bash
cd /root/demo
git clone https://gitee.com/internlm/lagent.git
cd lagent && git checkout 581d9fb8987a5d9b72bb9ebd37a95efd47d479ac
pip install -e .
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b /root/models/internlm2-chat-7b
```

然后修改模型位置, streamlit启动
```bash
streamlit run /root/demo/lagent/examples/internlm2_agent_web_demo_hf.py --server.address 127.0.0.1 --server.port 6006
```
在本地机上端口映射：
```bash
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p 41501
```
### 2. 运行效果
在尝试利用Lagent功能时， 需要选中“数据分析”， 实际可能还需要利用prompt提示：
没有强调使用工具， 答案错误(虽然也选中“数据分析”)：
<image src="img/lagent1.png"/>
<br/>
有要求使用sympy, 有代码且结果正确
<image src="img/lagent2.png"/>
<br/>
## Demo 4 实践部署 浦语·灵笔2 模型
浦语·灵笔2 是基于 书生·浦语2 大语言模型研发的突破性的图文多模态大模型，具有非凡的图文写作和图像理解能力，在多种应用场景表现出色，总结起来其具有：

自由指令输入的图文写作能力： 浦语·灵笔2 可以理解自由形式的图文指令输入，包括大纲、文章细节要求、参考图片等，为用户打造图文并貌的专属文章。
具有海量图文知识，可以准确的回复各种图文问答难题，在识别、感知、细节描述、视觉推理等能力上表现惊人。
浦语·灵笔2-7B 基于 书生·浦语2-7B 模型，在13项多模态评测中大幅领先同量级多模态模型，在其中6项评测中超过 GPT-4V 和 Gemini Pro。

### 1. 环境配置
```bash
conda activate demo
pip install timm==0.4.12 sentencepiece==0.1.99 markdown2==2.4.10 xlsxwriter==3.1.2 gradio==4.13.0 modelscope==1.9.5
cd /root/demo
git clone https://gitee.com/internlm/InternLM-XComposer.git
# git clone https://github.com/internlm/InternLM-XComposer.git
cd /root/demo/InternLM-XComposer
git checkout f31220eddca2cf6246ee2ddf8e375a40457ff626
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm-xcomposer2-7b /root/models/internlm-xcomposer2-7b
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm-xcomposer2-vl-7b /root/models/internlm-xcomposer2-vl-7b
```

### 2. 运行效果

#### 灵笔之图文写作实战
启用图文写作模型:internlm-xcomposer2-7b
```bash
cd /root/demo/InternLM-XComposer
python /root/demo/InternLM-XComposer/examples/gradio_demo_composition.py  \
--code_path /root/models/internlm-xcomposer2-7b \
--private \
--num_gpus 1 \
--port 6006
```

映射到本地端口
```bash
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p <port>
```

我让他写一篇“中国足球”相关的文章， 他的内容流畅并符合大众观感（具体历史有错误）。最让我惊喜的是配图，配图的选择与叙事感情匹配非常好。
甚至包括历史上夺冠的东亚运动会， 这确实是快速生成热点文章的神器。

<image src="img/soccer1.png"/>
<br/>
<image src="img/soccer2.png"/>
<br/>
<image src="img/soccer3.png"/>
<br/>
<image src="img/soccer4.png"/>
<br/>

相比之下要求它写“多模态发动机”相关的文章可读性就差一些，配图选择多样性很好，具有启发性。

<image src="img/engine1.png"/>
<br/>
<image src="img/engine2.png"/>
<br/>
#### 灵笔之图片理解实战
启用图片理解模型：internlm-xcomposer2-vl-7b
```bash
cd /root/demo/InternLM-XComposer
python /root/demo/InternLM-XComposer/examples/gradio_demo_composition.py  \
--code_path /root/models/internlm-xcomposer2-7b \
--private \
--num_gpus 1 \
--port 6006
```

映射到本地端口
```bash
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p <port>
```
我上传了一张F-35的图片，问它具体的型号（调皮）， 它没有正确回答，但在我不断引导下还是给出了答案。

<image src="img/plane1.png"/>
<br/>
<image src="img/plane2.png"/>
<br/>
<image src="img/plane3.png"/>
<br/>

"不知为不知"是一个好习惯

<image src="img/giraffe.png"/>
<br/>

可贵的是这个对话模型确实有很强的沟通能力：

<image src="img/cemetery2.png"/>
<br/>

<image src="img/cemetery1.png"/>
<br/>

## 总结
有幸实践了四个InternLM 模型， 从小到大。对话模型（角色扮演）的效果很好， 就个人微调来说， 我很想知道具体的训练量要多大。 Lagent潜力不可限量，不同的domain knownledge将在它手发扬光大。 但最惊艳的还是灵笔模型， 生成文章的配图检索的很准确的， 文章本身也流畅, 与图片对话实用价值与娱乐价值都很大。希望能有更多的机会研究它们。


