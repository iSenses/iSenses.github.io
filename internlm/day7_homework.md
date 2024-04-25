## 1. 作业(第7节课)

### 基础作业

- 使用 OpenCompass 评测 internlm2-chat-1_8b 模型在 C-Eval 数据集上的性能


### 进阶作业

- 将自定义数据集提交至OpenCompass官网

提交地址：https://hub.opencompass.org.cn/dataset-submit?lang=[object%20Object]  
提交指南：https://mp.weixin.qq.com/s/_s0a9nYRye0bmqVdwXRVCg  
Tips：不强制要求配置数据集对应榜单（ leaderboard.xlsx ），可仅上传 EADME_OPENCOMPASS.md 文档  


欢迎扫码关注 OpenCompass 公众号了解更多评测相关知识～

![image](https://github.com/InternLM/Tutorial/assets/25839884/1e8f3fd4-cf73-4d13-b5f0-03b1f925d825)

1. 启动OpenCompass的评测
2. opencompass代码的运行逻辑
3. 实现一个自定义的数据集, 并评测
4. 其他功能

```
studio-conda -o internlm-base -t opencompass
source activate opencompass
conda activate xtuner0117
```

```bash
cd ~/demo/opencompass

export MLK_SERVICE_FORCE_INTEL=1

python tools/list_configs.py internlm ceval
python run.py --datasets ceval_gen --hf-path /share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b --tokenizer-path /share/new_model/Shanghai_AI_Laboratory/internlm2-chat-1_8b --tokenizer-kwargs padding_side='left' truncation='left' trust_remote_code=True --max-seq-len 2048 --max-out-len 16 --batch-size 4 --num-gpus 1 --reuse latest --workdir /root/demo/opencompass/results #--debug
```

Partitioners Runners openicl Summarizer

```python
ceval_datasets.append(
	dict(
		type=CEvalEvent,
		path="./data/ceval/formal_ceval",
		name=_name,
		abbr="eval-" + _name if _split == "val" else "ceval-test-" + _name,
		reader_cfg=dict(
			input_columns = ["question", "A", "B", "C", "D"],
			output_column = "answer",
			train_split = "dev",
			test_split = _split),
		infer_cfg = ceval_infer_cfg,
		eval_cfg = ceval_eval_cfg,
	)
)

```
