---
layout: post
title: "Some articles are just so short that we have to make the footer stick"
categories: misc
---

# Day3 - 作业：huixiangdou实战

* TOC
{:toc}

## 基础篇
2.在 `InternLM Studio` 上部署茴香豆技术助手

- 根据教程文档搭建 `茴香豆技术助手`，针对问题"茴香豆怎么部署到微信群？"进行提问
- 完成不少于 400 字的笔记 + 截图



进入开发机后，从官方环境复制运行 InternLM 的基础环境，命名为 `InternLM2_Huixiangdou`，在命令行模式下运行：

```bash
studio-conda -o internlm-base -t InternLM2_Huixiangdou

conda activate InternLM2_Huixiangdou

cd /root && mkdir models

# 复制BCE模型
ln -s /root/share/new_models/maidalun1020/bce-embedding-base_v1 /root/demo/huixiangdou/models/bce-embedding-base_v1
ln -s /root/share/new_models/maidalun1020/bce-reranker-base_v1 /root/demo/huixiangdou/models/bce-reranker-base_v1

# 复制大模型参数（下面的模型，根据作业进度和任务进行**选择一个**就行）
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b /root/models/internlm2-chat-7b


pip install protobuf==4.25.3 accelerate==0.28.0 aiohttp==3.9.3 auto-gptq==0.7.1 bcembedding==0.1.3 beautifulsoup4==4.8.2 einops==0.7.0 faiss-gpu==1.7.2 langchain==0.1.14 loguru==0.7.2 lxml_html_clean==0.1.0 openai==1.16.1 openpyxl==3.1.2 pandas==2.2.1 pydantic==2.6.4 pymupdf==1.24.1 python-docx==1.1.0 pytoml==0.1.21 readability-lxml==0.8.1 redis==5.0.3 requests==2.31.0 scikit-learn==1.4.1.post1 sentence_transformers==2.2.2 textract==1.6.5 tiktoken==0.6.0 transformers==4.39.3 transformers_stream_generator==0.0.5 unstructured==0.11.2


git clone https://github.com/internlm/huixiangdou && cd huixiangdou
git checkout 63f2eef0e6e4ac113b05ede0316b193948a42cda

```

本次模型Retrieval阶段采用的是网易有道开源的 `bce-embedding-base_v1` 与 `bce-reranker-base_v1`( [技术报告](https://zhuanlan.zhihu.com/p/681370855) [代码](https://github.com/netease-youdao/BCEmbedding), [模型文件](https://huggingface.co/maidalun1020))
其他一些不错的开源选项包括BAAI的bge系列，阿里的GTE系列，JinaAI系列，指标可以是也可以是colbert,
BCE官方的推荐过滤阈值0.35或0.4, 但在本次实践中太高， 我修改系数0.22过滤出有效的答案。


## 配置huixiangdou RAG助手
通过`sed`修改模型路径
```bash
sed -i '6s#.*#embedding_model_path = "/root/models/bce-embedding-base_v1"#' /root/demo/huixiangdou/config.ini
sed -i '7s#.*#reranker_model_path = "/root/models/bce-reranker-base_v1"#' /root/demo/huixiangdou/config.ini
sed -i '29s#.*#local_llm_path = "/root/models/internlm2-chat-7b"#' /root/demo/huixiangdou/config.ini
```


为RAG添加可供检索的语料，直接使用huixiangdou自身的内容
```bash
cd /root/huixiangdou && mkdir repodir
git clone https://github.com/internlm/huixiangdou --depth=1 repodir/huixiangdou
```


建立huixiangdou有特色的功能， 接受问题、拒绝问题数据列表：

```bash
cd /root/demo/huixiangdou
mv resource/good_questions.json resource/good_questions_bk.json

echo '[
    "mmpose中怎么调用mmyolo接口",
    "mmpose实现姿态估计后怎么实现行为识别",
    "mmpose执行提取关键点命令不是分为两步吗，一步是目标检测，另一步是关键点提取，我现在目标检测这部分的代码是demo/topdown_demo_with_mmdet.py demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth   现在我想把这个mmdet的checkpoints换位yolo的，那么应该怎么操作",
    "在mmdetection中，如何同时加载两个数据集，两个dataloader",
    "如何将mmdetection2.28.2的retinanet配置文件改为单尺度的呢？",
    "1.MMPose_Tutorial.ipynb、inferencer_demo.py、image_demo.py、bottomup_demo.py、body3d_pose_lifter_demo.py这几个文件和topdown_demo_with_mmdet.py的区别是什么，\n2.我如果要使用mmdet是不是就只能使用topdown_demo_with_mmdet.py文件，",
    "mmpose 测试 map 一直是 0 怎么办？",
    "如何使用mmpose检测人体关键点？",
    "我使用的数据集是labelme标注的，我想知道mmpose的数据集都是什么样式的，全都是单目标的数据集标注，还是里边也有多目标然后进行标注",
    "如何生成openmmpose的c++推理脚本",
    "mmpose",
    "mmpose的目标检测阶段调用的模型，一定要是demo文件夹下的文件吗，有没有其他路径下的文件",
    "mmpose可以实现行为识别吗，如果要实现的话应该怎么做",
    "我在mmyolo的v0.6.0 (15/8/2023)更新日志里看到了他新增了支持基于 MMPose 的 YOLOX-Pose，我现在是不是只需要在mmpose/project/yolox-Pose内做出一些设置就可以，换掉demo/mmdetection_cfg/faster_rcnn_r50_fpn_coco.py 改用mmyolo来进行目标检测了",
    "mac m1从源码安装的mmpose是x86_64的",
    "想请教一下mmpose有没有提供可以读取外接摄像头，做3d姿态并达到实时的项目呀？",
    "huixiangdou 是什么？",
    "使用科研仪器需要注意什么？",
    "huixiangdou 是什么？",
    "茴香豆 是什么？",
    "茴香豆 能部署到微信吗？",
    "茴香豆 怎么应用到飞书",
    "茴香豆 能部署到微信群吗？",
    "茴香豆 怎么应用到飞书群",
    "huixiangdou 能部署到微信吗？",
    "huixiangdou 怎么应用到飞书",
    "huixiangdou 能部署到微信群吗？",
    "huixiangdou 怎么应用到飞书群",
    "huixiangdou",
    "茴香豆",
    "茴香豆 有哪些应用场景",
    "huixiangdou 有什么用",
    "huixiangdou 的优势有哪些？",
    "茴香豆 已经应用的场景",
    "huixiangdou 已经应用的场景",
    "huixiangdou 怎么安装",
    "茴香豆 怎么安装",
    "茴香豆 最新版本是什么",
    "茴香豆 支持哪些大模型",
    "茴香豆 支持哪些通讯软件",
    "config.ini 文件怎么配置",
    "remote_llm_model 可以填哪些模型?"
]' > /root/demo/huixiangdou/resource/good_questions.json

```




```bash
# 填入问题
sed -i '74s/.*/    queries = ["huixiangdou 是什么？", "茴香豆怎么部署到微信群", "今天天气怎么样？"]/' /root/huixiangdou/huixiangdou/main.py

# 运行茴香豆
cd /root/huixiangdou/
python3 -m huixiangdou.main --standalone

```

```bash
2024-04-18 10:07:24.588 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('“huixiangdou 是什么？”\n请仔细阅读以上内容，判断句子是否是个有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。\n判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释。', '根据您提供的内容，我无法判断"huixiangdou 是什么？" 这个句子的主题，因为它不包含任何有关于主题的信息。所以，我无法给出 0～10 的分数。请提供更具体的信息，以便我能够更准确地评估。')
2024-04-18 10:07:24.588 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。
判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释 A:根据您提供的内容，我无法判断"huixiangdou 是什么？" 这个句子的主题，因为它不包含任何有关于主题的信息。所以，我无法给出 0～10 的分数。请提供更具体的信息，以便我能够更准确地评估。                  remote local timecost 7.615493535995483 
04/18/2024 10:07:24 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:16 +0800] "POST /inference HTTP/1.1" 200 661 "-" "python-requests/2.31.0"
2024-04-18 10:07:25.143 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('告诉我这句话的主题，直接说主题不要解释：“huixiangdou 是什么？”', '主题："huixiangdou" 的含义或定义。')
2024-04-18 10:07:25.143 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:告诉我这句话的主题，直接说主题不要解释：“huixiangdou 是什么？ A:主题："huixiangdou" 的含义或定义。                  remote local timecost 0.5479512214660645 
04/18/2024 10:07:25 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:24 +0800] "POST /inference HTTP/1.1" 200 246 "-" "python-requests/2.31.0"
You're using a XLMRobertaTokenizerFast tokenizer. Please note that with a fast tokenizer, using the `__call__` method is faster than using a method to encode the text followed by a call to the `pad` method to get a padded encoding.
2024-04-18 10:07:27.199 | INFO     | huixiangdou.service.retriever:query:158 - target README.md file length 15372
2024-04-18 10:07:27.200 | DEBUG    | huixiangdou.service.retriever:query:185 - query:主题："huixiangdou" 的含义或定义。 top1 file:README.md
2024-04-18 10:07:27.773 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('问题：“huixiangdou 是什么？”\n材料：“ <img alt="youtube" src="https://img.shields.io/badge/youtube-black?logo=youtube&logocolor=red" />\n</a>\n<a href="https://www.bilibili.com/video/bv1s2421n7mn" target="_blank">\n<img alt="bilibili" src="https://img.shields.io/badge/bilibili-pink?logo=bilibili&logocolor=white" />\n</a>\n<a href="https://discord.gg/tw4zbpzz" target="_blank">\n<img alt="discord" src="https://img.shields.io/badge/discord-red?logo=discord&logocolor=white" />\n</a>\n</div>  \n</div>  \nhuixiangdou is a **group chat** assistant based on llm (large language model).  \nadvantages:  \n1. design a two-stage pipeline of rejection and response to cope with group chat scenario, answer user questions without message flooding, see arxiv2401.08772”\n请仔细阅读以上内容，判断问题和材料的关联度，用0～10表示。判断标准：非常相关得 10 分；完全没关联得 0 分。直接提供得分不要解释。\n', '8')
2024-04-18 10:07:27.774 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:flooding, see arxiv2401.08772”
请仔细阅读以上内容，判断问题和材料的关联度，用0～10表示。判断标准：非常相关得 10 分；完全没关联得 0 分。直接提供得分不要解释。 A:8               remote local timecost 0.5677931308746338 
04/18/2024 10:07:27 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:27 +0800] "POST /inference HTTP/1.1" 200 171 "-" "python-requests/2.31.0"
2024-04-18 10:07:27.775 | WARNING  | huixiangdou.service.llm_client:generate_response:95 - disable remote LLM while choose remote LLM, auto fixed
2024-04-18 10:07:39.104 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('材料：“ <img alt="youtube" src="https://img.shields.io/badge/youtube-black?logo=youtube&logocolor=red" />\n</a>\n<a href="https://www.bilibili.com/video/bv1s2421n7mn" target="_blank">\n<img alt="bilibili" src="https://img.shields.io/badge/bilibili-pink?logo=bilibili&logocolor=white" />\n</a>\n<a href="https://discord.gg/tw4zbpzz" target="_blank">\n<img alt="discord" src="https://img.shields.io/badge/discord-red?logo=discord&logocolor=white" />\n</a>\n</div>  \n</div>  \nhuixiangdou is a **group chat** assistant based on llm (large language model).  \nadvantages:  \n1. design a two-stage pipeline of rejection and response to cope with group chat scenario, answer user questions without message flooding, see arxiv2401.08772\nEnglish | [简体中文](README_zh.md)\n<div align="center">\n<img src="resource/logo_black.svg" width="555px"/>\n<div align="center">\n <a href="resource/figures/wechat.jpg" target="_blank">\n <img alt="Wechat" src="https://img.shields.io/badge/wechat-robot%20inside-brightgreen?logo=wechat&logoColor=white" />\n </a>\n <a href="https://arxiv.org/abs/2401.08772" target="_blank">\n <img alt="Arxiv" src="https://img.shields.io/badge/arxiv-paper%20-darkred?logo=arxiv&logoColor=white" />\n </a>\n <a href="https://pypi.org/project/huixiangdou" target="_blank">\n <img alt="PyPI" src="https://img.shields.io/badge/PyPI-install-blue?logo=pypi&logoColor=white" />\n </a>\n <a href="https://youtu.be/ylXrT-Tei-Y" target="_blank">\n <img alt="YouTube" src="https://img.shields.io/badge/YouTube-black?logo=youtube&logoColor=red" />\n </a>\n <a href="https://www.bilibili.com/video/BV1S2421N7mn" target="_blank">\n <img alt="BiliBili" src="https://img.shields.io/badge/BiliBili-pink?logo=bilibili&logoColor=white" />\n </a>\n <a href="https://discord.gg/TW4ZBpZZ" target="_blank">\n <img alt="discord" src="https://img.shields.io/badge/discord-red?logo=discord&logoColor=white" />\n </a>\n</div>\n</div>\nHuixiangDou is a **group chat** assistant based on LLM (Large Language Model).\nAdvantages:\n1. Design a two-stage pipeline of rejection and response to cope with group chat scenario, answer user questions without message flooding, see [arxiv2401.08772](https://arxiv.org/abs/2401.08772)\n2. Low cost, requiring only 1.5GB memory and no need for training\n3. Offers a complete suite of Web, Android, and pipeline source code, which is industrial-grade and commercially viable\nCheck out the [scenes in which HuixiangDou are running](./huixiangdou-inside.md) and join [WeChat Group](resource/figures/wechat.jpg) to try AI assistant inside.\nIf this helps you, please give it a star ⭐\n# 🔆 News\nThe web portal is available on [OpenXLab](https://openxlab.org.cn/apps/detail/tpoisonooo/huixiangdou-web), where you can build your own knowledge assistant without any coding, using WeChat and Feishu groups.\nVisit web portal usage video on [YouTube](https://www.youtube.com/watch?v=ylXrT-Tei-Y) and [BiliBili](https://www.bilibili.com/video/BV1S2421N7mn).\n- \\[2024/04\\] ”\n 问题：“huixiangdou 是什么？” \n 请仔细阅读参考材料回答问题。', 'HuixiangDou 是一个基于 LLM（大型语言模型）的群组聊天助手。它设计了一个两阶段管道，以处理群组聊天场景，并能够回答用户问题，而不会造成信息过载。该模型具有低成本的特点，仅需 1.5GB 内存，且不需要进行训练。HuixiangDou 还提供了 Web、Android 和管道源代码，这些代码是工业级和商业可行的。您可以在 [WeChat 群](resource/figures/wechat.jpg) 中尝试 AI 助手内部，并使用 [OpenXLab](https://openxlab.org.cn/apps/detail/tpoisonooo/huixiangdou-web) 的 Web 门户，无需编写任何代码即可构建自己的知识助手，使用 WeChat 和 Feishu 群组。')
2024-04-18 10:07:39.105 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:ttps://www.bilibili.com/video/BV1S2421N7mn).
- \[2024/04\] ”
 问题：“huixiangdou 是什么？” 
 请仔细阅读参考材料回答问题 A:HuixiangDou 是一个基于 LLM（大型语言模型）的群组聊天助手。它设计了一个两阶段管道，以处理群组聊天场景，并能够回答用户问题，而不会造成信息过载。该模型具有低成本的特点，仅需 1.5GB 内存，且不需要进行训练。HuixiangDou 还提供了 Web、Android 和管道源代码，这些代码是工业级和商业可行的。您可以在 [WeChat 群](resource/figures/wechat.jpg) 中尝试 AI 助手内部，并使用 [OpenXLab](https://openxlab.org.cn/apps/detail/tpoisonooo/huixiangdou-web) 的 Web 门户，无需编写任何代码即可构建自己的知识助手，使用 WeChat 和 Feishu 群组。          remote local timecost 11.327838659286499 
04/18/2024 10:07:39 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:27 +0800] "POST /inference HTTP/1.1" 200 1332 "-" "python-requests/2.31.0"
2024-04-18 10:07:39.108 | INFO     | __main__:lark_send_only:80 - ErrorCode.SUCCESS, huixiangdou 是什么？, HuixiangDou 是一个基于 LLM（大型语言模型）的群组聊天助手。它设计了一个两阶段管道，以处理群组聊天场景，并能够回答用户问题，而不会造成信息过载。该模型具有低成本的特点，仅需 1.5GB 内存，且不需要进行训练。HuixiangDou 还提供了 Web、Android 和管道源代码，这些代码是工业级和商业可行的。您可以在 [WeChat 群](resource/figures/wechat.jpg) 中尝试 AI 助手内部，并使用 [OpenXLab](https://openxlab.org.cn/apps/detail/tpoisonooo/huixiangdou-web) 的 Web 门户，无需编写任何代码即可构建自己的知识助手，使用 WeChat 和 Feishu 群组。, ['README.md']
2024-04-18 10:07:40.715 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('“你最喜欢什么颜色？”\n请仔细阅读以上内容，判断句子是否是个有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。\n判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释。', '10分。\n- 有主语（你）、谓语（喜欢）、宾语（什么颜色），符合疑问句的构成。\n- 是疑问句，所以得10分。')
2024-04-18 10:07:40.715 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。
判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释 A:10分。
- 有主语（你）、谓语（喜欢）、宾语（什么颜色），符合疑问句的构成。
- 是疑问句，所以得10分。                 remote local timecost 1.603487253189087 
04/18/2024 10:07:40 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:39 +0800] "POST /inference HTTP/1.1" 200 447 "-" "python-requests/2.31.0"
2024-04-18 10:07:40.954 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('告诉我这句话的主题，直接说主题不要解释：“你最喜欢什么颜色？”', '主题：颜色偏好。')
2024-04-18 10:07:40.954 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:告诉我这句话的主题，直接说主题不要解释：“你最喜欢什么颜色？ A:主题：颜色偏好。     remote local timecost 0.23428058624267578 
04/18/2024 10:07:40 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:40 +0800] "POST /inference HTTP/1.1" 200 218 "-" "python-requests/2.31.0"
2024-04-18 10:07:40.971 | INFO     | __main__:lark_send_only:80 - ErrorCode.UNRELATED, 你最喜欢什么颜色？, , ['HuixiangDou.pdf']
2024-04-18 10:07:46.588 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('“茴香豆怎么部署到微信群？”\n请仔细阅读以上内容，判断句子是否是个有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。\n判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释。', '8.0\n\n解释：\n- 该句子有主语 "茴香豆" 和谓语 "怎么部署到微信群"，因此符合主谓宾的句法结构。\n- 该句子以 "怎么" 开头，是疑问句的形式，因此得分为 8 分。\n- 该句子询问的是 "茴香豆" 如何部署到微信群，是关于技术或方法的问题，因此与主题相关。\n- 该句子不是陈述句，因此不扣分。\n- 该句子是疑问句，因此得分为 8 分。')
2024-04-18 10:07:46.588 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。
判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释 A:8.0

解释：
- 该句子有主语 "茴香豆" 和谓语 "怎么部署到微信群"，因此符合主谓宾的句法结构。
- 该句子以 "怎么" 开头，是疑问句的形式，因此得分为 8 分。
- 该句子询问的是 "茴香豆" 如何部署到微信群，是关于技术或方法的问题，因此与主题相关。
- 该句子不是陈述句，因此不扣分。
- 该句子是疑问句，因此得分为 8 分。              remote local timecost 5.614309787750244 
04/18/2024 10:07:46 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:40 +0800] "POST /inference HTTP/1.1" 200 1007 "-" "python-requests/2.31.0"
2024-04-18 10:07:47.293 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('告诉我这句话的主题，直接说主题不要解释：“茴香豆怎么部署到微信群？”', '这句话的主题是：“茴香豆的微信部署”。')
2024-04-18 10:07:47.293 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:告诉我这句话的主题，直接说主题不要解释：“茴香豆怎么部署到微信群？ A:这句话的主题是：“茴香豆的微信部署”。           remote local timecost 0.6987035274505615 
04/18/2024 10:07:47 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:46 +0800] "POST /inference HTTP/1.1" 200 285 "-" "python-requests/2.31.0"
2024-04-18 10:07:47.309 | INFO     | __main__:lark_send_only:80 - ErrorCode.UNRELATED, 茴香豆怎么部署到微信群？, , ['README_zh.md']
2024-04-18 10:07:49.683 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('“今天天气怎么样？”\n请仔细阅读以上内容，判断句子是否是个有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。\n判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释。', '根据给定的标准，"今天天气怎么样？" 是一个有主语、谓语和宾语，并且是疑问句的句子。因此，它的得分是 10 分。')
2024-04-18 10:07:49.684 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:有主题的疑问句，结果用 0～10 表示。直接提供得分不要解释。
判断标准：有主语谓语宾语并且是疑问句得 10 分；缺少主谓宾扣分；陈述句直接得 0 分；不是疑问句直接得 0 分。直接提供得分不要解释 A:根据给定的标准，"今天天气怎么样？" 是一个有主语、谓语和宾语，并且是疑问句的句子。因此，它的得分是 10 分。                 remote local timecost 2.3721837997436523 
04/18/2024 10:07:49 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:47 +0800] "POST /inference HTTP/1.1" 200 474 "-" "python-requests/2.31.0"
2024-04-18 10:07:50.074 | INFO     | huixiangdou.service.llm_server_hybrid:generate_response:519 - ('告诉我这句话的主题，直接说主题不要解释：“今天天气怎么样？”', '主题：天气。')
2024-04-18 10:07:50.074 | DEBUG    | huixiangdou.service.llm_server_hybrid:generate_response:522 - Q:告诉我这句话的主题，直接说主题不要解释：“今天天气怎么样？ A:主题：天气。           remote local timecost 0.3870413303375244 
04/18/2024 10:07:50 - [INFO] -aiohttp.access->>>    127.0.0.1 [18/Apr/2024:10:07:49 +0800] "POST /inference HTTP/1.1" 200 206 "-" "python-requests/2.31.0"
2024-04-18 10:07:50.086 | INFO     | __main__:lark_send_only:80 - ErrorCode.UNRELATED, 今天天气怎么样？, , ['HuixiangDou.pdf']
======== Running on http://0.0.0.0:8888 ========
(Press CTRL+C to quit)
```




![image](img/hxd_hw_success_1.png)
![image](img/hxd_hw_unrelated_1.png)

![image](img/hxd_hw_serper_1.png)
![image](img/hxd_hw_serper_2.png)


### 3.1 加入网络搜索

茴香豆除了可以从本地向量数据库中检索内容进行回答，也可以加入网络的搜索结果，生成回答。 

开启网络搜索功能需要用到 **Serper** 提供的 API：

1. 登录 [Serper](https://serper.dev/) ，注册：

![成功注册serper.dev](img/hxd_hw_serper_1.png)

2. 进入 [Serper API](https://serper.dev/api-key) 界面，复制自己的 API-key：

![image]()
得到搜索列表

![从serper.dev中得到搜索结果](img/hxd_hw_serper_2.png)

3. 替换 `/huixiangdou/config.ini` 中的 ***${YOUR-API-KEY}*** 为自己的API-key：

```
[web_search]
# check https://serper.dev/api-key to get a free API key
x_api_key = "${YOUR-API-KEY}"
domain_partial_order = ["openai.com", "pytorch.org", "readthedocs.io", "nvidia.com", "stackoverflow.com", "juejin.cn", "zhuanlan.zhihu.com", "www.cnblogs.com"]
save_dir = "logs/web_search_result"
```
其中 `domain_partial_order` 可以设置网络搜索的范围。



![image](img/hxd_hw_lark_1.png)
![image](img/hxd_hw_lark_2.png)
![image](img/hxd_hw_lark_3.png)
![image](img/hxd_hw_lark_4.png)

## 进阶篇
### 拓展实践：为huixiangdou增加百度星河社区api调用
对huixiangdou进行探索实践中，发现其架构清晰，易于扩展，特别是基于远程大模型api，能够极大减少对客户机的要求，甚至在适配faiss-cpu后可以在无gpu情况下运行。
为huixiangdou添加api后端百度星河社区aistudio（刚好新注册的API用不完）。

在 huixiangdou/service/llm_server_hybrid.py 的HybridLLMServer类中添加函数
```python
def call_aistudio(self, prompt, history):
	"""Generate a response from aistudio erniebot-sdk
	Args:
		prompt (str): The prompt to send.
		history (list): List of previous interactions.

	Returns:
		str: Generated response.
	"""

	try:
		import erniebot
		erniebot.api_type = "aistudio"
		erniebot.access_token = self.server_config['remote_api_key']
	except Exception as e:
		logger.error(str(e))
		logger.error('please `pip install erniebot` and check API_KEY')
		return ''

	model_temp =  (self.server_config["remote_llm_model"])
	if model_temp.startswith("ernie"):
		model = model_temp
	else:
		model = "ernie-speed-128k";

	logger.info('choose aistudio model: {}'.format(model))
	temperature = 0.1
	# self.rpm.wait()
	time.sleep(1)
	messages = build_messages(prompt=prompt, history=history)

	response = erniebot.ChatCompletion.create(
		model=model,
		messages=messages)
	text = response.get_result()
	return text
```

修改
```
def generate_response(self, prompt, history=[], backend='remote'):
	...
	
	elif backend == 'aistudio':
		output_text = self.call_aistudio(prompt=prompt, history=history)
```


service/retriever
huixiangdou建立将会根据提供的`good_question`与`bad_question`列表 利用 `sklearn.metrics.precision_recall_curve` 函数计算更新 `reject_throttle` 。 但是我在调试过程中发现其值可能过高造成好问题被拒绝，我采用临时手动修改`config.ini`的方式修改feature_store下的`reject_throttle`阈值，将其改小为约0.618倍增加了其对相关问题的敏感度。这种情况可以临时使用，但是正道还提供更具有代表性的问题或者更合适地修改`reject_throttle`的算法。




### A.【应用方向】 结合自己擅长的领域知识（游戏、法律、电子等）、专业背景，搭建个人工作助手或者垂直领域问答助手，参考茴香豆官方文档，部署到下列任一平台。
  - 飞书、微信
  - 可以使用 茴香豆 Web 版 或 InternLM Studio 云端服务器部署
  - 涵盖部署全过程的作业报告和个人助手问答截图

### B.【算法方向】尝试修改 `good_questions.json`、调试 prompt 或应用其他 NLP 技术，如其他 chunk 方法，提高个人工作助手的表现。
  - 完成不少于 400 字的笔记 ，记录自己的尝试和调试思路，涵盖全过程和改进效果截图

## 大作业项目选题

### A.【工程方向】 参与贡献茴香豆前端，将茴香豆助手部署到下列平台
  - Github issue、Discord、钉钉、X
### B.【应用方向】 茴香豆RAG-Agent
  - 应用茴香豆建立一个 ROS2 的机器人Agent
### C.【算法方向】 茴香豆多模态
  - 参与茴香豆多模态的工作
