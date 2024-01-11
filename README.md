# OpenMoE
| [**Blog**](https://xuefuzhao.notion.site/Aug-2023-OpenMoE-v0-2-Release-43808efc0f5845caa788f2db52021879) | [**Twitter**](https://twitter.com/XueFz) | [**Discord**](https://discord.gg/bjGnGfjegU) |

OpenMoE is a project aimed at igniting the open-source MoE community! We are releasing a family of open-sourced Mixture-of-Experts (MoE) Large Language Models.

Since we are a small team working on a huge project, we cannot handle everything. Instead, we release some intermediate checkpoints in this repo to invite more contributors to work on open-sourced MoE project together!

## News

[2024/01] 🔥 OpenMoE-8B-Chat is now available. We've provided a Colab inference [demo](https://colab.research.google.com/drive/1xIfIVafnlCP2XVICmRwkUFK3cwTJYjCY) for everyone to try, as well as a [tutorial](https://colab.research.google.com/drive/1eIT1rtG7pORRQAYtQoMOAekUg7aZLDdn) on converting JAX checkpoints to PyTorch checkpoints(Note: both require Colab Pro).

[2023/11] 🔥 Thanks to Colossal AI! They released one [PyTorch OpenMoE implementation](https://github.com/hpcaitech/ColossalAI/tree/main/examples/language/openmoe) including both training and inference with expert parallelism.

[2023/08] 🔥 We released an intermediate OpenMoE-8B checkpoint (OpenMoE-v0.2) along with two other models. Check out the blog [post](https://xuefuzhao.notion.site/Aug-2023-OpenMoE-v0-2-Release-43808efc0f5845caa788f2db52021879).

## TODO List

- [x] PyTorch Implementation with Colossal AI
- [ ] More Evaluation
- [x] Continue Training to 1T tokens
- [ ] Paper

## Contents
- [Model Weights](#model-weights)
- [Get Started](#get-started)
- [Approach](#approach)
- [License](#license)
- [Authors](#authors)
- [Citation](#citation)


## Model Weights
Currently, three models are released in total.


| Model Name     | Description                      | #Param   |Huggingface | Gin File   |
|----------------|-------------------------------------------------|----------|-------------|----------  |
| OpenMoE-base/16E   | A small MoE model for debugging                 |637M      |[Link](https://huggingface.co/OrionZheng/openmoe-base) |[Link](https://github.com/XueFuzhao/t5x/blob/main/t5x/examples/t5/t5_1_1/examples/openmoe_base.gin)  |   
| OpenLLaMA-base | A dense counter-part of OpenMoE-base            |310M      |[Link](https://huggingface.co/fuzhao/OpenLLaMA_Base) |[Link](https://github.com/XueFuzhao/t5x/blob/main/t5x/examples/t5/t5_1_1/examples/openllama_base.gin)  |     
| OpenMoE-8B/32E (200B)    | 8B MoE with comparable FLOPs of a 1.6B LLaMA      |8B        |[Link](https://huggingface.co/fuzhao/OpenMoE_8B) |[Link](https://github.com/XueFuzhao/t5x/blob/main/t5x/examples/t5/t5_1_1/examples/openmoe_large.gin) |
| OpenMoE-8B/32E (1.1T)   | 8B MoE with comparable FLOPs of a 1.6B LLaMA(No SFT)      |8B        |[Link](https://huggingface.co/fuzhao/OpenMoE-8B-32E-Jax) |[Link](https://github.com/XueFuzhao/t5x/blob/main/t5x/examples/t5/t5_1_1/examples/openmoe_large_full_lm_stage2.gin) |
| OpenMoE-8B/32E (SFT)   | 8B Chat Model supervised finetuned on the [WildChat-nontoxic](https://huggingface.co/datasets/allenai/WildChat-nontoxic)   |8B        |[Link](https://huggingface.co/OrionZheng/openmoe-8b-chat) |[Link](https://github.com/XueFuzhao/t5x/blob/main/t5x/examples/t5/t5_1_1/examples/openmoe_large_full_lm_stage2.gin) |

We release all these checkpoints on Huggingface and Google Cloud Storage. For instance, you can download openmoe-8B with 
```
gsutil cp -r gs://openmoe/openmoe-8b/checkpoint_100000 $YOUR_DIR
```

The base models are trained with 128B tokens. The openmoe-8B checkpoint with 4 MoE layers and 32 experts has been trained by 200B tokens. We are still training OpenMoE-8B. So if you are interested in the latest checkpoint, please feel free to drop Fuzhao an email (f.xue@u.nus.edu).

Note: downloading data from Google Cloud Storage is not free, but you can sign up to Google Cloud and get some credits. You can also use Huggingface directly.


## Get Started

### Inference

We provide a Colab [tutorial](https://colab.research.google.com/drive/1eIT1rtG7pORRQAYtQoMOAekUg7aZLDdn) explaining the setup and execution of PyTorch model inference. You can now experiment with OpenMoE-8B-Chat on Colab. Running OpenMoE-8B requires ~49GB of memory in float32 or ~23GB in bfloat16. It can be executed on a Colab CPU High-RAM runtime or an A100-40GB runtime, both of which require Colab Pro.


### Training

Get a TPU-vm and run the following code on all TPUs. Researcher can apply [TPU Research Cloud](https://sites.research.google/trc/about/) to get the TPU resource.

[Colossal AI](https://github.com/hpcaitech/ColossalAI) has the PyTorch + GPU implementation for OpenMoE.
```
# On TPUs
git clone https://github.com/XueFuzhao/OpenMoE.git
bash OpenMoE/script/run_pretrain.sh
```


### Eval

Get a TPU-vm and run the following code on all TPUs.
```
git clone https://github.com/XueFuzhao/OpenMoE.git
bash OpenMoE/script/run_eval.sh
```

## Approach
### Data
#### Before 780B tokens:
50% The RedPajama + 50% The Stack Dedup.
We use a high ratio of coding data to improve reasoning ability.

#### After 780B tokens:

| dataset                        | Ratio (%) |
| -------------------------------| ----------- |
| redpajama_c4                   | 15.0        |
| redpajama_wikipedia            | 6.5         |
| wikipedia                      | 6.5         |
| redpajama_stackexchange        | 2.5         |
| redpajama_arxiv                | 4.5         |
| redpajama_book                 | 6.5         |
| redpajama_github               | 5.0         |
| redpajama_common_crawl         | 43.5        |
| the_stack_dedup                | 10.0        |

We found model tends to learn code faster than language. So we decide to reduce the coding data at the later stage of training.

Below are scripts to generate TFDS for pre-training datasets:   
The RedPajama: https://github.com/Orion-Zheng/redpajama_tfds  
The-Stack-Dedup: https://github.com/Orion-Zheng/the_stack_tfds  

### Tokenizer
We use the [umt5 Tokenizer](https://arxiv.org/abs/2304.09151) to support multi-lingual continue learning in the future, which can be downloaded on [Huggingface](https://huggingface.co/google/umt5-small/tree/main) or [Google Cloud](https://github.com/google-research/t5x/blob/main/docs/models.md#umt5-checkpoints).

### Model Architecture
OpenMoE is based on [ST-MoE](https://arxiv.org/abs/2202.08906) but uses Decoder-only architecture. The detailed implementation can be found in Fuzhao's [T5x](https://github.com/XueFuzhao/t5x) and [Flaxformer](https://github.com/XueFuzhao/flaxformer) repo.

### Training Objective

#### Before 780B tokens:
We use a modified UL2 training objective but Casual Attention Mask (We use more prefix LM and high mask ratio because it saves computation.):
- 50% prefix LM
- 10% span len=3 mask ratio=0.15
- 10% span len=8 mask ratio=0.15
- 10% span len=3 mask ratio=0.5
- 10% span len=8 mask ratio=0.5
- 10% span len=64 mask ratio=0.5

#### After 780B tokens:
Vanilla next token prediction, because we observed that UL2 objective tends to saturate at the later stage of training, although it enables model to learn things faster at start.

### Other Designs
RoPE, SwiGLU activation, 2K context length. We will release a more detailed report soon.

## Evaluation

We evaluate our model on BigBench-Lite as our first step. We plot the cost-effectiveness curve in the figure below. 

Relative Cost is approximated by multiplying activated parameters and training tokens. The size of dots denotes the number of activated parameters for each token. The lightgray dot denotes the total parameters of MoE models.
![Plot](figure/bblite-3-shot.png)

For more detailed results, please see our [Blog](https://www.notion.so/Aug-2023-OpenMoE-v0-2-Release-43808efc0f5845caa788f2db52021879)



## License

Our code is under Apache 2.0 License.

Since the models are trained on The Redpajama and The Stack dataset, please check the license of these two datasets for your model usage.


## Authors

This project is currently contributed by the following authors:

- [Fuzhao Xue](https://xuefuzhao.github.io/)
- [Zian Zheng](https://zheng-zian-andy.com)
- [Yao Fu](https://franxyao.github.io/)
- [Jinjie Ni](http://jinjie.one/)
- [Zangwei Zheng](https://zhengzangw.github.io/)
- [Wangchunshu Zhou](https://michaelzhouwang.github.io/)
- [Yang You](https://www.comp.nus.edu.sg/~youy/)


## Citation

Please cite the repo if you use the model and code in this repo.

```bibtex
@misc{openmoe2023,
  author = {Fuzhao Xue, Zian Zheng, Yao Fu, Jinjie Ni, Zangwei Zheng, Wangchunshu Zhou and Yang You},
  title = {OpenMoE: Open Mixture-of-Experts Language Models},
  year = {2023},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/XueFuzhao/OpenMoE}},
}
```


