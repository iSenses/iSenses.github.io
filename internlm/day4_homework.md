---
layout: post
title: "Day4 - Xtuner微调LLM实战"
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

## 基础作业
### 训练自己的小助手认知

环境搭建， 利用`studio-conda`构建环境，然后从源码安装`xtuner==0.1.17`。
```shell
studio-conda xtuner0.1.17
conda activate xtuner0.1.17
cd ~
mkdir -p /root/demo/xtuner0117 && cd /root/demo/xtuner0117
git clone -b v0.1.17  https://github.com/InternLM/xtuner
cd /root/demo/xtuner0117/xtuner
pip install -e '.[all]'
```

```shell

mkdir -p /root/demo/ft/model
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b /root/demo/ft/model/internlm2-chat-1_8b
```

搜索配置
```shell
mkdir -p /root/demo/ft/config
xtuner list-cfg -p internlm2_1_8b
xtuner copy-cfg internlm2_1_8b_qlora_alpaca_e3 /root/demo/ft/config
```

修改模型与数据地址
```diff
- pretrained_model_name_or_path = 'internlm/internlm2-1_8b'
+ pretrained_model_name_or_path = '/root/demo/ft/model/internlm2-chat-1_8b'

# 修改数据集地址为本地的json文件地址（在第31行的位置）
- alpaca_en_path = 'tatsu-lab/alpaca'
+ alpaca_en_path = '/root/demo/ft/data/personal_assistant.json'
```

跟据需要修改学习率(lr), max_length, max_epochs, 减少一下训练量, 增加评估问题

```diff
- max_length = 2048
+ max_length = 1024

- max_epochs = 3
+ max_epochs = 2

- save_total_limit = 2
+ save_total_limit = 3


- evaluation_freq = 500
+ evaluation_freq = 300

- evaluation_inputs = ['请给我介绍五个上海的景点', 'Please tell me five scenic spots in Shanghai']
+ evaluation_inputs = ['请你介绍一下你自己', '你是谁', '你是我的小助手吗']
```

最后改变map_fn, 将 dataset_map_fn 改为通用的 OpenAI 数据集格式
``` diff
- from xtuner.dataset.map_fns import alpaca_map_fn, template_map_fn_factory
+ from xtuner.dataset.map_fns import openai_map_fn, template_map_fn_factory

- dataset=dict(type=load_dataset, path=alpaca_en_path),
+ dataset=dict(type=load_dataset, path='json', data_files=dict(train=alpaca_en_path)),

- dataset_map_fn=alpaca_map_fn,
+ dataset_map_fn=openai_map_fn,
```

配置好了， 可以开始训练了，使用 deepspeed 来加速训练
```shell
xtuner train /root/demo/ft/config/internlm2_1_8b_qlora_alpaca_e3_copy.py --work-dir /root/demo/ft/train_deepspeed --deepspeed deepspeed_zero2
```


训练好了，可以准备试用了。  
先要转换模型格式，创建一个保存转换后 Huggingface 格式的文件夹，然后转换，与原模型合并。

``` bash
mkdir -p /root/ft/huggingface

# 模型转换
# xtuner convert pth_to_hf ${配置文件地址} ${权重文件地址} ${转换后模型保存地址}
xtuner convert pth_to_hf /root/ft/train/internlm2_1_8b_qlora_alpaca_e3_copy.py /root/ft/train/iter_500.pth /root/demo/ft/huggingface/500

xtuner convert pth_to_hf /root/ft/train/internlm2_1_8b_qlora_alpaca_e3_copy.py /root/ft/train/iter_500.pth /root/demo/ft/huggingface/500 --fp32 --max-shard-size 2GB

makdir -p /root/demo/final_model/500
export MKL_SERVICE_FORCE_INTEL=1
xtuner convert merge /root/demo/ft/model/internlm2-chat-1_8b  /root/demo/ft/huggingface/500 /root/demo/ft/final_model/500

```

可以试用啦！先直接命令行

```shell
xtuner chat /root/demo/ft/final_model/500 --prompt-template internlm2_chat
```

![image](img/xt_homework1.png)
![image](img/xt_homework2.png)


使用基于`streamlit`的webdemo
```shell
pip install streamlit==1.24.0

```


进入InternLM文件夹， 修改InternLM/chat/web_demo.py
```diff
- model = (AutoModelForCausalLM.from_pretrained('/root/model',
+ model = (AutoModelForCausalLM.from_pretrained('/root/demo/ft/final_model/500',
- tokenizer = AutoTokenizer.from_pretrained('/root/_model',
+ tokenizer = AutoTokenizer.from_pretrained('/root/demo/ft/final_model/500',
```

运行streamlit
```shell
streamlit run /root/demo/InternLM/chat/web_demo.py --server.address 127.0.0.1 --server.port 6006
```

在本机上运行端口映射，使用https://studio.intern-ai.org.cn/console/instance 页面获得的密码：
```shell
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p 44815
```

使在http://127.0.0.1:6006使用模型：

![image](img/xt_homework3.png)
![image](img/xt_homework4.png)
![image](img/xt_homework5.png)


## 进阶作业


### 1. 上传认知模型到OpenXLab并部署

将微调后的模型上传至 [OpenXLab](https://openxlab.org.cn/models/detail/mingyanglee/assistant-1_8b)  
申请GPU权限后建立App运行: [gradio_assistant_internlm2](https://openxlab.org.cn/apps/detail/mingyanglee/gradio_assistant_internlm2)  

![image](img/xt_homework_openxlab.png)
## 2. 复现多模态微调

继续使用前面配置好的环境，并利用提供的Llava数据， 根据需要修改路径, 开始微调：
```

xtuner train /root/demo/tutorial/xtuner/llava/llava_internlm2_chat_1_8b_qlora_clip_vit_large_p14_336_lora_e1_gpu8_finetune_copy.py --deepspeed deepspeed_zero2
```

转换训练成果，与训练前对比：
```console
(xtuner0.1.17) root@intern-studio-40079336:~/demo/xtuner0117/llava# export MKL_SERVICE_FORCE_INTEL=1
(xtuner0.1.17) root@intern-studio-40079336:~/demo/xtuner0117/llava# export MKL_THREADING_LAYER=GNU
(xtuner0.1.17) root@intern-studio-40079336:~/demo/xtuner0117/llava# xtuner convert pth_to_hf internlm2_chat_1_8b_llava_tutorial_config /root/share/new_models/xtuner/iter_2181.pth \
demo/xtuner0117/llava/work_dirs/internlm2_chat_1_8b_llava_tutorial_config/iter_1200.pth
(xtuner0.1.17) root@intern-studio-40079336:~/demo/xtuner0117/llava# xtuner chat /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b \
> --visual-encoder /root/share/new_models/openai/clip-vit-large-patch14-336 \
> --lava /root/demo/xtuner0117/llava/llava_data/iter_2181_hf \
> --prompt-template internlm2_chat \
> --image /root/demo/xtuner0117/llava/llava_data/test_img/oph.jpg 
```

微调前回答很单调：
<image src="img/xt_homework_llava_1.png" width="960"/>
<br/>




微调前回答智商明显增加，仪器也回答正确：
<image src="img/xt_homework_llava_2.png" width="960"/>
<br/>
