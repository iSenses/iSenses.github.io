# Day2 InternLM2 Demo && Homework
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
## Demo 4 实践部署 浦语·灵笔2 模型
