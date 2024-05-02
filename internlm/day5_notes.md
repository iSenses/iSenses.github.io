# Day5 LMDeploy 大模型部署工具
## 大模型量化部署必要性
- 模型巨大参数量
- k/v cache缓存
- 请求数不固定
- decoder-only 结构

## 大模型部署方案
- 模型并行
- 低比特量化
- Page Attention
- Transformer 计算和访存优化
- Continuous Batch

## 竞品方案
- huggingface transformers
- lmdeploy
- vllm
- tensorrt-llm
- deepspeed
- llama.cpp
- mlc-llm


## 关于量化算法

有效利用GPU提高模型推理效率的方法：
1.基于warp分组并行化,2.压缩参数表示以减少访存.

## 关于LMDeploy 命令行

lmdeploy 实现了一个

```shell


lmdeploy serve api_server 
 [-h]
 [--server-name SERVER_NAME]
 [--server-port SERVER_PORT]
 [--allow-origins ALLOW_ORIGINS [ALLOW_ORIGINS ...]]
 [--allow-credentials]
 [--allow-methods ALLOW_METHODS [ALLOW_METHODS ...]]
 [--allow-headers ALLOW_HEADERS [ALLOW_HEADERS ...]]
 [--qos-config-path QOS_CONFIG_PATH]
 [--backend {pytorch,turbomind}]
 [--log-level {CRITICAL,FATAL,ERROR,WARN,WARNING,INFO,DEBUG,NOTSET}]
 [--api-keys [API_KEYS ...]]
 [--ssl]
 [--meta-instruction META_INSTRUCTION]  # System prompt for ChatTemplateConfig. Deprecated. Please use --chat-template instead. Default: None. Type: str

 [--chat-template CHAT_TEMPLATE]  # https://lmdeploy.readthedocs.io/en/latest/advance/chat_template.html
 [--cap {completion,infilling,chat,python}] # Deprecated. Please use --chat-template instead. Default: chat. Type: str

 [--adapters [ADAPTERS ...]] # Used to set path(s) of lora adapter(s). One can input key-value pairs in xxx=yyy format for multiple lora adapters. If only have one adapter, one can only input the path of the adapter.. Default: None. Type: str
 [--tp TP]
 [--model-name MODEL_NAME] # GPU number used in tensor parallelism. Should be 2^n. Default: 1. Type: int
 [--session-len SESSION_LEN]
 [--max-batch-size MAX_BATCH_SIZE]
 [--cache-max-entry-count CACHE_MAX_ENTRY_COUNT]
 [--cache-block-seq-len CACHE_BLOCK_SEQ_LEN]
 [--model-format {hf,llama,awq}]
 [--quant-policy QUANT_POLICY]
 [--rope-scaling-factor ROPE_SCALING_FACTOR]

 model_path

```


```
lmdeploy
    chat                Chat with pytorch or turbomind engine.
    lite                Compressing and accelerating LLMs with lmdeploy.lite module
    serve               Serve LLMs with gradio, openai API or triton server.
    convert             Convert LLMs to turbomind format.
    list                List the supported model names.
    check_env           Check the environmental information.
    chat                Chat with pytorch or turbomind engine.
```

```json
{
	"model_name": "Chat Template Name",
	"system": "<im_start>system\n",
	"meta_instruction": "You are a robot developed by LMDeploy.",
	"eosys": "<|im_end|>\n",
	"user": "<|im_start|>user\n",
	"eoh": "<|im_end|>\n",
	"assistant": "<|im_start|>assistant\n",
	"eoa": "<|im_end|>",
	"separator": "\n",
	"capability": "chat",
	"stop_words": ["<|im_end|>"]
}
```

```python
from lmdeploy import ChatTemplateConfig, serve
serve('internlm/internlm2-chat-7b', chat_template_config=ChatTemplateConfig.from_json('${JSON_FILE}'))
```

