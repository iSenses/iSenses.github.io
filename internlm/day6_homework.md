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

## 基础作业

### 1. 完成 Lagent Web Demo 使用


一个终端运行api_server
```bash
conda activate agent
lmdeploy serve api_server /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b \
                            --server-name 127.0.0.1 \
                            --model-name internlm2-chat-7b \
                            --cache-max-entry-count 0.1
```

另一个运行`streamlit`App
```bash

conda activate agent
cd /root/agent/lagent/examples
streamlit run internlm2_agent_web_demo.py --server.address 127.0.0.1 --server.port 7860
```


接下来在本地的浏览器页面中打开 http://localhost:7860 以使用 Lagent Web Demo。首先输入模型 IP 为 127.0.0.1:23333，**在输入完成后按下回车键以确认**。并选择插件为 ArxivSearch，以让模型获得在 arxiv 上搜索论文的能力。

我们输入“请帮我搜索 Llama 3” 以让模型搜索书生·浦语2的技术报告。效果如下图所示，可以看到模型正确输出了 Llama 3 技术报告的相关信息。尽管还输出了其他论文，但这是由 arxiv 搜索 API 的相关行为导致的。

<image src="img/ag_hw_arxiv_1.png" width="960"/> <br/>

<image src="img/ag_hw_arxiv_2.png" width="960"/> <br/>

### 2. 完成 AgentLego 直接使用部分

<image src="img/ag_hw_road_1.jpg" width="478"/>
<image src="img/ag_hw_road_2.jpg" width="478"/> <br/>

   
## 进阶作业

### 1. 完成 AgentLego WebUI 使用

<image src="img/ag_hw_ObjectDetection_0.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_1.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_2.png" width="960"/> <br/>
<image src="img/ag_hw_ObjectDetection_3.png" width="960"/> <br/>


### 2. 使用 Lagent 实现自定义工具查询天气

首先通过 `touch agent/lagent/lagent/actions/weather.py`新建工具文件，该文件内容如下：

```python
import json
import os
import requests
from typing import Optional, Type

from lagent.actions.base_action import BaseAction, tool_api
from lagent.actions.parser import BaseParser, JsonParser
from lagent.schema import ActionReturn, ActionStatusCode

class WeatherQuery(BaseAction):
    """Weather plugin for querying weather information."""
    
    def __init__(self,
                 key: Optional[str] = None,
                 description: Optional[dict] = None,
                 parser: Type[BaseParser] = JsonParser,
                 enable: bool = True) -> None:
        super().__init__(description, parser, enable)
        key = os.environ.get('WEATHER_API_KEY', key)
        if key is None:
            raise ValueError(
                'Please set Weather API key either in the environment '
                'as WEATHER_API_KEY or pass it as `key`')
        self.key = key
        self.location_query_url = 'https://geoapi.qweather.com/v2/city/lookup'
        self.weather_query_url = 'https://devapi.qweather.com/v7/weather/now'

    @tool_api
    def run(self, query: str) -> ActionReturn:
        """一个天气查询API。可以根据城市名查询天气信息。
        
        Args:
            query (:class:`str`): The city name to query.
        """
        tool_return = ActionReturn(type=self.name)
        status_code, response = self._search(query)
        if status_code == -1:
            tool_return.errmsg = response
            tool_return.state = ActionStatusCode.HTTP_ERROR
        elif status_code == 200:
            parsed_res = self._parse_results(response)
            tool_return.result = [dict(type='text', content=str(parsed_res))]
            tool_return.state = ActionStatusCode.SUCCESS
        else:
            tool_return.errmsg = str(status_code)
            tool_return.state = ActionStatusCode.API_ERROR
        return tool_return
    
    def _parse_results(self, results: dict) -> str:
        """Parse the weather results from QWeather API.
        
        Args:
            results (dict): The weather content from QWeather API
                in json format.
        
        Returns:
            str: The parsed weather results.
        """
        now = results['now']
        data = [
            f'数据观测时间: {now["obsTime"]}',
            f'温度: {now["temp"]}°C',
            f'体感温度: {now["feelsLike"]}°C',
            f'天气: {now["text"]}',
            f'风向: {now["windDir"]}，角度为 {now["wind360"]}°',
            f'风力等级: {now["windScale"]}，风速为 {now["windSpeed"]} km/h',
            f'相对湿度: {now["humidity"]}',
            f'当前小时累计降水量: {now["precip"]} mm',
            f'大气压强: {now["pressure"]} 百帕',
            f'能见度: {now["vis"]} km',
        ]
        return '\n'.join(data)

    def _search(self, query: str):
        # get city_code
        try:
            city_code_response = requests.get(
                self.location_query_url,
                params={'key': self.key, 'location': query}
            )
        except Exception as e:
            return -1, str(e)
        if city_code_response.status_code != 200:
            return city_code_response.status_code, city_code_response.json()
        city_code_response = city_code_response.json()
        if len(city_code_response['location']) == 0:
            return -1, '未查询到城市'
        city_code = city_code_response['location'][0]['id']
        # get weather
        try:
            weather_response = requests.get(
                self.weather_query_url,
                params={'key': self.key, 'location': city_code}
            )
        except Exception as e:
            return -1, str(e)
        return weather_response.status_code, weather_response.json()
```
在 https://dev.qweather.com/docs/api/ 获得API KEY,存入环境变量：

使用lmdeploy部署lagent后端：
```bash
conda activate agent
lmdeploy serve api_server /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b \
                            --server-name 127.0.0.1 \
                            --model-name internlm2-chat-7b \
                            --cache-max-entry-count 0.1
```

运行`streamlit`App：

```bash
export WEATHER_API_KEY=在2.2节获取的API KEY
# 比如 export WEATHER_API_KEY=1234567890abcdef
conda activate agent
cd /root/agent/Tutorial/agent
streamlit run internlm2_weather_web_demo.py --server.address 127.0.0.1 --server.port 7860
```


<image src="img/ag_hw_weather_1.png" width="960"/> <br/>
<image src="img/ag_hw_weather_2.png" width="960"/> <br/>

### 3. 使用 AgentLego 实现适配MagicMaker生成创意图片
<image src="img/ag_hw_magic_1.png" width="960"/> <br/>
<image src="img/ag_hw_magic_2.png" width="960"/> <br/>
<image src="img/ag_hw_magic_3.png" width="960"/> <br/>

