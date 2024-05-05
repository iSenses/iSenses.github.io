---
layout: post
title: "Day6 - Lagent  & AgentLego <br/>
  智能体应用搭建作业"
output: html_document
---

<nav class="toc-fixed" markdown="1">
* TOC
{:toc}
</nav>

## 基础作业7

### 1. 完成 Lagent Web Demo 使用

<image src="img/ag_hw_arxiv_1.png" width="960"/> <br/>

<image src="img/ag_hw_arxiv_2.png" width="960"/> <br/>


```bash
conda activate agent
lmdeploy serve api_server /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b \
                            --server-name 127.0.0.1 \
                            --model-name internlm2-chat-7b \
                            --cache-max-entry-count 0.1
```

### 2. 完成 AgentLego 直接使用部分

<image src="img/ag_hw_road_1.jpg" width="960"/> <br/>
<image src="img/ag_hw_road_2.jpg" width="960"/> <br/>

   
## 进阶作业

### 1. 完成 AgentLego WebUI 使用，并在作业中上传截图

<image src="img/ag_hw_ObjectDetection_0.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_1.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_2.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_3.png" width="960"/> <br/>


### 2. 使用 Lagent 实现自定义工具查询天气

<image src="img/ag_hw_weather_1.png" width="960"/> <br/>
<image src="img/ag_hw_weather_2.png" width="960"/> <br/>

### 3. 使用 AgentLego 实现适配MagicMaker生成创意图片
<image src="img/ag_hw_magic_1.png" width="960"/> <br/>
<image src="img/ag_hw_magic_2.png" width="960"/> <br/>

