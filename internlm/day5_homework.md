---
layout: post
title: "Day5 - LMDeploy课后作业"
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

# 第五节课作业（请交到第5节课）

## 基础作业（结营必做）
完成以下任务，并将实现过程记录截图：
- 配置lmdeploy运行环境
- 下载internlm-chat-1.8b模型
- 以命令行方式与模型对话

<image src="img/lm_llava_cmd.png" width="960"/> <br/>
<image src="img/lm_llava_cmd2.png" width="960"/> <br/>

## 进阶作业

完成以下任务，并将实现过程记录截图：
- 设置KV Cache最大占用比例为0.4，开启W4A16量化，以命令行方式与模型对话。（优秀学员必做）
- 以API Server方式启动 lmdeploy，开启 W4A16量化，调整KV Cache的占用比例为0.4，分别使用命令行客户端与Gradio网页客户端与模型对话。（优秀学员）
```bash
lmdeploy serve api_server \
	/root/demo/lmdep/internlm2-chat-1_8b-4bit \
    --model-format hf \
    --quant-policy 0 \
    --server-name 0.0.0.0 \
    --server-port 23333 \
	--cache-max-entry-count 0.4 \
    --tp 1
```

- 使用W4A16量化，调整KV Cache的占用比例为0.4，使用Python代码集成的方式运行internlm2-chat-1.8b模型。（优秀学员必做）
- 使用 LMDeploy 运行视觉多模态大模型 llava gradio demo （优秀学员必做）
- 将 LMDeploy Web Demo 部署到 [OpenXLab](../tools/openxlab-deploy/) （OpenXLab cuda 12.2 的镜像还没有 ready，可先跳过，一周之后再来做）

<image src="img/lm_FastAPI.png" width="960"/> <br/>
<image src="img/lm_gradio.png" width="960"/> <br/>
<image src="img/lm_gradio_2.png" width="960"/> <br/>
<image src="img/lm_interlm2_1_8_4bits_cache-max-entry-count_0.1.png" width="960"/> <br/>
<image src="img/lm_interlm2_1_8_4bits_cache-max-entry-count_0.4_cmd.png" width="960"/> <br/>
<image src="img/lm_interlm2_1_8_4bits_cache-max-entry-count_0.8.png" width="960"/> <br/>
<image src="img/lm_llava_gradio.png" width="960"/> <br/>
<image src="img/lm_llava_gradio_1.png" width="960"/> <br/>
<image src="img/lm_llava_gradio_3.png" width="960"/> <br/>
## 基础作业（结营必做）
完成以下任务，并将实现过程记录截图：
- 配置lmdeploy运行环境
- 下载internlm-chat-1.8b模型
- 以命令行方式与模型对话

## 进阶作业

完成以下任务，并将实现过程记录截图：
- 设置KV Cache最大占用比例为0.4，开启W4A16量化，以命令行方式与模型对话。（优秀学员必做）
- 以API Server方式启动 lmdeploy，开启 W4A16量化，调整KV Cache的占用比例为0.4，分别使用命令行客户端与Gradio网页客户端与模型对话。（优秀学员）
- 使用W4A16量化，调整KV Cache的占用比例为0.4，使用Python代码集成的方式运行internlm2-chat-1.8b模型。（优秀学员必做）
- 使用 LMDeploy 运行视觉多模态大模型 llava gradio demo （优秀学员必做）
- 将 LMDeploy Web Demo 部署到 [OpenXLab](../tools/openxlab-deploy/) （OpenXLab cuda 12.2 的镜像还没有 ready，可先跳过，一周之后再来做）




```
(agent) root@intern-studio-40079336:~/demo/lmdep# lmdeploy chat /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b
2024-04-27 15:02:37,739 - lmdeploy - WARNING - model_source: hf_model
2024-04-27 15:02:37,739 - lmdeploy - WARNING - kwargs max_batch_size is deprecated to initialize model, use TurbomindEngineConfig instead.
2024-04-27 15:02:37,739 - lmdeploy - WARNING - kwargs cache_max_entry_count is deprecated to initialize model, use TurbomindEngineConfig instead.
2024-04-27 15:02:40,487 - lmdeploy - WARNING - model_config:

[llama]
model_name = internlm2
tensor_para_size = 1
head_num = 16
kv_head_num = 8
vocab_size = 92544
num_layer = 24
inter_size = 8192
norm_eps = 1e-05
attn_bias = 0
start_id = 1
end_id = 2
session_len = 32776
weight_type = bf16
rotary_embedding = 128
rope_theta = 1000000.0
size_per_head = 128
group_size = 0
max_batch_size = 128
max_context_token_num = 1
step_length = 1
cache_max_entry_count = 0.8
cache_block_seq_len = 64
cache_chunk_size = -1
num_tokens_per_iter = 0
max_prefill_iters = 1
extra_tokens_per_iter = 0
use_context_fmha = 1
quant_policy = 0
max_position_embeddings = 32768
rope_scaling_factor = 0.0
use_dynamic_ntk = 0
use_logn_attn = 0


2024-04-27 15:02:42,225 - lmdeploy - WARNING - get 195 model params
2024-04-27 15:02:49,175 - lmdeploy - WARNING - Input chat template with model_name is None. Forcing to use internlm2                                                                   
[WARNING] gemm_config.in is not found; using default GEMM algo
session 1

double enter to end input >>> please provide three suggestions about time management

<|im_start|>system
You are an AI assistant whose name is InternLM (书生·浦语).
- InternLM (书生·浦语) is a conversational language model that is developed by Shanghai AI Laboratory (上海人工智能实验室). It is designed to be helpful, honest, and harmless.
- InternLM (书生·浦语) can understand and communicate fluently in the language chosen by the user such as English and 中文.
<|im_end|>
<|im_start|>user
please provide three suggestions about time management<|im_end|>
<|im_start|>assistant
 2024-04-27 15:04:04,022 - lmdeploy - WARNING - kwargs ignore_eos is deprecated for inference, use GenerationConfig instead.
2024-04-27 15:04:04,023 - lmdeploy - WARNING - kwargs random_seed is deprecated for inference, use GenerationConfig instead.
当然，以下是三个关于时间管理的建议：

1. **设定优先级**：
   - 将任务按照重要性和紧急性进行分类。
   - 先完成那些紧急且重要的任务，然后处理那些重要但不紧急的事。
   - 学会说“不”，只接受自己可以完成的任务。

2. **使用时间管理工具**：
   - 使用日历应用程序来规划日常工作和会议。
   - 利用日曆和提醒功能来确保不会错过重要的邮件和通知。
   - 利用待办事项列表或任务管理应用程序来跟踪任务进度。

3. **休息和放松**：
   - 确保保持适当的休息和睡眠时间。
   - 定期进行身体锻炼以提高效率和精神状态。
   - 给自己设置一些放松和娱乐的时间，有助于减轻压力并增加生产力。

```

```shell
https://b-aide-20240422-633ec18-40079336.intern-ai.org.cn/files/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b/configuration_internlm2.py?_xsrf=2%7Cc7513c7d%7C032cf67f9c577899b672130f7fc796f2%7C1713747595
```

internlm2-chat-1_8b-4bit
```shell
(agent) root@intern-studio-40079336:~/demo/lmdep# lmdeploy chat /root/demo/lmdep/internlm2-chat-1_8b-4bit --model-format awq --cache-max-entry-count 0.01
2024-04-27 16:34:36,064 - lmdeploy - WARNING - model_source: hf_model
2024-04-27 16:34:36,065 - lmdeploy - WARNING - kwargs model_format is deprecated to initialize model, use TurbomindEngineConfig instead.
2024-04-27 16:34:36,065 - lmdeploy - WARNING - kwargs max_batch_size is deprecated to initialize model, use TurbomindEngineConfig instead.
2024-04-27 16:34:36,065 - lmdeploy - WARNING - kwargs cache_max_entry_count is deprecated to initialize model, use TurbomindEngineConfig instead.
2024-04-27 16:34:38,615 - lmdeploy - WARNING - model_config:

[llama]
model_name = internlm2
tensor_para_size = 1
head_num = 16
kv_head_num = 8
vocab_size = 92544
num_layer = 24
inter_size = 8192
norm_eps = 1e-05
attn_bias = 0
start_id = 1
end_id = 2
session_len = 32776
weight_type = int4
rotary_embedding = 128
rope_theta = 1000000.0
size_per_head = 128
group_size = 128
max_batch_size = 128
max_context_token_num = 1
step_length = 1
cache_max_entry_count = 0.01
cache_block_seq_len = 64
cache_chunk_size = -1
num_tokens_per_iter = 0
max_prefill_iters = 1
extra_tokens_per_iter = 0
use_context_fmha = 1
quant_policy = 0
max_position_embeddings = 32768
rope_scaling_factor = 0.0
use_dynamic_ntk = 0
use_logn_attn = 0


2024-04-27 16:34:39,337 - lmdeploy - WARNING - get 267 model params
2024-04-27 16:34:45,908 - lmdeploy - WARNING - Input chat template with model_name is None. Forcing to use internlm2                                                                   
[WARNING] gemm_config.in is not found; using default GEMM algo
session 1

double enter to end input >>> 你是谁？

<|im_start|>system
You are an AI assistant whose name is InternLM (书生·浦语).
- InternLM (书生·浦语) is a conversational language model that is developed by Shanghai AI Laboratory (上海人工智能实验室). It is designed to be helpful, honest, and harmless.
- InternLM (书生·浦语) can understand and communicate fluently in the language chosen by the user such as English and 中文.
<|im_end|>
<|im_start|>user
你是谁？<|im_end|>
<|im_start|>assistant
 2024-04-27 16:35:17,344 - lmdeploy - WARNING - kwargs ignore_eos is deprecated for inference, use GenerationConfig instead.
2024-04-27 16:35:17,344 - lmdeploy - WARNING - kwargs random_seed is deprecated for inference, use GenerationConfig instead.
你好，我是书生·浦语。我是一位语言模型，致力于帮助人类使用英语进行交流。如果你有学习英语的需求，我愿尽我所能，助你一路前行。

double enter to end input >>> Write cypher code based on following schema and question:\nSchema:\n(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_SKILL {}]->(:Skill {label:'Skill', id: string,name: string,level: string})\n(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_EDUCATION {}]->(:Education {label:'Education', id: string, degree: string, university: string, graduationDate: string, score: string, url: string})\nQuestion: How many java developers attend more than one universities? Return code only!


<|im_start|>user
Write cypher code based on following schema and question:\nSchema:\n(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_SKILL {}]->(:Skill {label:'Skill', id: string,name: string,level: string})\n(:Person {label: 'Person',id: string, role: string, description: string})-[:HAS_EDUCATION {}]->(:Education {label:'Education', id: string, degree: string, university: string, graduationDate: string, score: string, url: string})\nQuestion: How many java developers attend more than one universities? Return code only!<|im_end|>
<|im_start|>assistant
 根据您给出的问题和提供的数据，可以编写如下代码：

```python
import requests
from bs4 import BeautifulSoup

# 获取所有教育背景相关的资源
def get_all_educations():
    url = "https://www.example.com/resources/best-jobs-in-java"
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "lxml")
    resources = []
    for resource in soup.find_all("div", class_="resources-card")["class_"]:
        title = resource.find_all("h3", class_="title")["class"][0].text
        description = resource.find_all("p", class_="description")["class"][0].text
        if not description or not title or "no resources found" not in description:
            resources.append({"title": title, "description": description})
    return resources

# 获得所有教育背景信息
def get_all_educations():
    resources = get_all_educations()
    total_count = 0
    for resource in resources:
        for education in resource["education"]:
            if education["degree"] == "Java Developer":
                total_count += 1
    return total_count

# 获取超过一个大学的Java开发人员数量
def get_java_developers_count():
    total_count = get_all_educations()
    return 1 + total_count

if __name__ == '__main__':
    result = get_java_developers_count()
    print(result)
```

该代码首先获取所有教育背景信息，并根据“Java Developer”是否在教育背景中列出，统计获得者数量。最后返回所有获得者的数量，确保满足条件。请注意，此代码仅返回满足特定需求的结果。
```
LMDeploy_internlm2-chat-1_8b-4bit
