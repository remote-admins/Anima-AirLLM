![airllm_logo](https://github.com/lyogavin/Anima/blob/main/assets/airllm_logo_sm.png?v=3&raw=true)

**AirLLM** optimizes inference memory usage, allowing 70B large language models to run inference on a single 4GB GPU card. No quantization, distillation, pruning or other model compression techniques that would result in degraded model performance are needed.

AirLLM优化inference内存，4GB单卡GPU可以运行70B大语言模型推理。不需要任何损失模型性能的量化和蒸馏，剪枝等模型压缩。

## Updates


[2023/12/03] added support of **ChatGLM**, **QWen**, **Baichuan**, **Mistral**, **InternLM**!

支持ChatGLM, QWEN, Baichuan, Mistral, InternLM!

[2023/12/02] added support for safetensors. Now support all top 10 models in open llm leaderboard.

支持safetensor系列模型，现在open llm leaderboard前10的模型都已经支持。

[2023/12/01] airllm 2.0. Support compressions: **3x run time speed up!**

airllm2.0。支持模型压缩，速度提升3倍。

[2023/11/20] airllm Initial verion!

airllm发布。



## Quickstart

### 1. install package

First, install airllm pip package.

首先安装airllm包。

```bash
pip install airllm
```

如果找不到package，可能是因为默认的镜像问题。可以尝试制定原始镜像：
```bash
pip install -i https://pypi.org/simple/ airllm
```

### 2. Inference

Then, initialize AirLLMLlama2, pass in the huggingface repo ID of the model being used, or the local path, and inference can be performed similar to a regular transformer model.

然后，初始化AirLLMLlama2，传入所使用模型的huggingface repo ID，或者本地路径即可类似于普通的transformer模型进行推理。

(*You can can also specify the path to save the splitted layered model through **layer_shards_saving_path** when init AirLLMLlama2.*

*如果需要指定另外的路径来存储分层的模型可以在初始化AirLLMLlama2是传入参数：**layer_shards_saving_path**。*)

```python
from airllm import AirLLMLlama2

MAX_LENGTH = 128
# could use hugging face model repo id:
model = AirLLMLlama2("garage-bAInd/Platypus2-70B-instruct")

# or use model's local path...
#model = AirLLMLlama2("/home/ubuntu/.cache/huggingface/hub/models--garage-bAInd--Platypus2-70B-instruct/snapshots/b585e74bcaae02e52665d9ac6d23f4d0dbc81a0f")

input_text = [
        'What is the capital of United States?',
        #'I like',
    ]

input_tokens = model.tokenizer(input_text,
    return_tensors="pt", 
    return_attention_mask=False, 
    truncation=True, 
    max_length=MAX_LENGTH, 
    padding=True)
           
generation_output = model.generate(
    input_tokens['input_ids'].cuda(), 
    max_new_tokens=20,
    use_cache=True,
    return_dict_in_generate=True)

output = model.tokenizer.decode(generation_output.sequences[0])

print(output)

```
 
 
Note: During inference, the original model will first be decomposed and saved layer-wise. Please ensure there is sufficient disk space in the huggingface cache directory.
 
注意：推理过程会首先将原始模型按层分拆，转存。请保证huggingface cache目录有足够的磁盘空间。


### 3. Model Compression - 3x Inference Speed Up!

We just added model compression based on block-wise quantization based model compression. Which can further **speed up the inference speed** for up to **3x** , with **almost ignorable accuracy loss!** (see more performance evaluation and why we use block-wise quantization in [this paper](https://arxiv.org/abs/2212.09720))

我们增加了基于block-wise quantization的模型压缩，推理速度提升3倍几乎没有精度损失。精度评测可以参考此paper：[this paper](https://arxiv.org/abs/2212.09720)

![speed_improvement](https://github.com/lyogavin/Anima/blob/main/assets/airllm2_time_improvement.png?v=2&raw=true)

#### how to enalbe model compression speed up:

* Step 1. make sure you have [bitsandbytes](https://github.com/TimDettmers/bitsandbytes) installed by `pip install -U bitsandbytes `
* Step 2. make sure airllm verion later than 2.0.0: `pip install -U airllm` 
* Step 3. when initialize the model, passing the argument compression ('4bit' or '8bit'):

```python
model = AirLLMLlama2("garage-bAInd/Platypus2-70B-instruct",
                     compression='4bit' # specify '8bit' for 8-bit block-wise quantization 
                    )
```

#### how model compression here is different from quantization?

Quantization normally needs to quantize both weights and activations to really speed things up. Which makes it harder to maintain accuracy and avoid the impact of outliers in all kinds of inputs.

While in our case the bottleneck is mainly at the disk loading, we only need to make the model loading size smaller. So we get to only quantize the weights part, which is easier to ensure the accuracy.

### 4. All supported configurations
 
When initialize the model, we support the following configurations:

初始化model的时候，可以指定以下的配置参数：

* **compression**: supported options: 4bit,  8bit for 4-bit or 8-bit block-wise quantization, or by default None for no compression
* **profiling_mode**: supported options: True to output time consumptions or by default False
* **layer_shards_saving_path**: optionally another path to save the splitted model

### 5. Supported Models

#### [HF open llm leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard) top models 


**Including but not limited to the following:** (Most of the open models are based on llama2, so should be supported by default)

@12/01/23

| Rank  | Model | Supported | Model Class |
| ------------- | ------------- | ------------- | ------------- |
| 1 | TigerResearch/tigerbot-70b-chat-v2  | ✅ | AirLLMLlama2 |
| 2 | upstage/SOLAR-0-70b-16bit | ✅ | AirLLMLlama2 |
| 3 | ICBU-NPU/FashionGPT-70B-V1.1 | ✅ | AirLLMLlama2 |
| 4 | sequelbox/StellarBright | ✅ | AirLLMLlama2 |
| 5 | bhenrym14/platypus-yi-34b  | ✅ | AirLLMLlama2 |
| 6 | MayaPH/GodziLLa2-70B  | ✅ | AirLLMLlama2 |
| 7 | 01-ai/Yi-34B | ✅ | AirLLMLlama2 |
| 8 | garage-bAInd/Platypus2-70B-instruct  | ✅ | AirLLMLlama2 |
| 9 | jondurbin/airoboros-l2-70b-2.2.1  | ✅ | AirLLMLlama2 |
| 10 | chargoddard/Yi-34B-Llama  | ✅ | AirLLMLlama2 |
| ？ | mistralai/Mistral-7B-Instruct-v0.1  | ✅ | AirLLMMistral |


#### [opencompass leaderboard](https://opencompass.org.cn/leaderboard-llm) top models

**Including but not limited to the following:** (Most of the open models are based on llama2, so should be supported by default)

@12/01/23

| Rank  | Model | Supported | Model Class |
| ------------- | ------------- | ------------- | ------------- |
| 1 | GPT-4  | closed.ai😓 | N/A |
| 2 | TigerResearch/tigerbot-70b-chat-v2 | ✅ | AirLLMLlama2 |
| 3 | THUDM/chatglm3-6b-base | ✅ | AirLLMChatGLM |
| 4 | Qwen/Qwen-14B | ✅| AirLLMQWen |
| 5 | 01-ai/Yi-34B  | ✅ | AirLLMLlama2 |
| 6 | ChatGPT  | closed.ai😓  | N/A |
| 7 | OrionStarAI/OrionStar-Yi-34B-Chat | ✅ | AirLLMLlama2 |
| 8 | Qwen/Qwen-14B-Chat  | ✅ | AirLLMQWen |
| 9 | Duxiaoman-DI/XuanYuan-70B  | ✅ | AirLLMLlama2 |
| 10 | internlm/internlm-20b  | ✅ | AirLLMInternLM |
| 26 | baichuan-inc/Baichuan2-13B-Chat | ✅ | AirLLMBaichuan |

#### example of other models (ChatGLM, QWen, Baichuan, Mistral, etc):

<details>


* ChatGLM:

```python
from airllm import AirLLMChatGLM
MAX_LENGTH = 128
model = AirLLMChatGLM("THUDM/chatglm3-6b-base")
input_text = ['What is the capital of China?',]
input_tokens = model.tokenizer(input_text,
    return_tensors="pt", 
    return_attention_mask=False, 
    truncation=True, 
    max_length=MAX_LENGTH, 
    padding=True)
generation_output = model.generate(
    input_tokens['input_ids'].cuda(), 
    max_new_tokens=5,
    use_cache= True,
    return_dict_in_generate=True)
model.tokenizer.decode(generation_output.sequences[0])
```

* QWen:

```python
from airllm import AirLLMQWen
MAX_LENGTH = 128
model = AirLLMQWen("Qwen/Qwen-7B")
input_text = ['What is the capital of China?',]
input_tokens = model.tokenizer(input_text,
    return_tensors="pt", 
    return_attention_mask=False, 
    truncation=True, 
    max_length=MAX_LENGTH)
generation_output = model.generate(
    input_tokens['input_ids'].cuda(), 
    max_new_tokens=5,
    use_cache=True,
    return_dict_in_generate=True)
model.tokenizer.decode(generation_output.sequences[0])
```


* Baichuan, InternLM, Mistral, etc:

```python
from airllm import AirLLMBaichuan # AirLLMInternLM, AirLLMMistral
MAX_LENGTH = 128
model = AirLLMBaichuan("baichuan-inc/Baichuan2-7B-Base")
#model = AirLLMInternLM("internlm/internlm-20b")
#model = AirLLMMistral("mistralai/Mistral-7B-Instruct-v0.1")
input_text = ['What is the capital of China?',]
input_tokens = model.tokenizer(input_text,
    return_tensors="pt", 
    return_attention_mask=False, 
    truncation=True, 
    max_length=MAX_LENGTH)
generation_output = model.generate(
    input_tokens['input_ids'].cuda(), 
    max_new_tokens=5,
    use_cache=True,
    return_dict_in_generate=True)
model.tokenizer.decode(generation_output.sequences[0])
```


</details>


#### To request other model support: [here](https://docs.google.com/forms/d/e/1FAIpQLSe0Io9ANMT964Zi-OQOq1TJmnvP-G3_ZgQDhP7SatN0IEdbOg/viewform?usp=sf_link)



## Acknowledgement

A lot of the code are based on SimJeg's great work in the Kaggle exam competition. Big shoutout to SimJeg:

[GitHub account @SimJeg](https://github.com/SimJeg), 
[the code on Kaggle](https://www.kaggle.com/code/simjeg/platypus2-70b-with-wikipedia-rag), 
[the associated discussion](https://www.kaggle.com/competitions/kaggle-llm-science-exam/discussion/446414).


## FAQ

### 1. MetadataIncompleteBuffer

safetensors_rust.SafetensorError: Error while deserializing header: MetadataIncompleteBuffer

If you run into this error, most possible cause is you run out of disk space. The process of splitting model is very disk-consuming. See [this](https://huggingface.co/TheBloke/guanaco-65B-GPTQ/discussions/12). You may need to extend your disk space, clear huggingface [.cache](https://huggingface.co/docs/datasets/cache) and rerun. 

如果你碰到这个error，很有可能是空间不足。可以参考一下[这个](https://huggingface.co/TheBloke/guanaco-65B-GPTQ/discussions/12) 可能需要扩大硬盘空间，删除huggingface的[.cache](https://huggingface.co/docs/datasets/cache)，然后重新run。

## Contribution 

Welcome contribution, ideas and discussions!

If you find it useful, please ⭐ or buy me a coffee! 🙏

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://bmc.link/lyogavinQ)
