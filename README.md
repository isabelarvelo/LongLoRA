# DS 5690: LongLoRA Paper Presentation

## Motivation 

**Context Length:** The maximum amount of information an LLM can take as input for a query. LLMs are stateless (each incoming query is processed independently of other interactions), so their 'memory' is essentially their context window

* Longer context allows LLMs to handle:
	* Entire documents, including research papers or even books
	* Large databases or codebases
	* Multiple data sources simultaneously
 	* Extended conversations or multi-turn dialogues
 	* Complex tasks requiring integration of information from various parts of a long input

* Current Limitations
	* Most LLMs are pre-trained with fixed context sizes which restrict LLMs in many applications such as:
		* Summarizing long documents
		* Answering questions about extensive content
		* Maintaining coherence in long conversations
		* Analyzing large datasets or codebases in their entirety
		* Performing tasks that require understanding of broader context or long-term dependencies
 

## Problem Statement : Extending context length is not trivial 

* Computational Complexity:
	* The primary challenge is the quadratic O(n^2) complexity of self-attention with respect to sequence length
	* For instance, extending from 2,048 to 8,192 context length requires **16×** more computation in self-attention layers



* Memory Requirements:
	* Longer sequences require more GPU memory to store attention matrices and intermediate activations
	* This often exceeds the capacity of available hardware, especially for larger models
* Overfitting and Generalization:
	* Models may overfit to specific long-context patterns seen during training
	* Ensuring generalization to various types of long-context tasks is crucial

## Current Approaches 

* Long-context Transformers and their limitations
	* Sparse Attention Mechanisms:
		* Approach: Using various sparse attention patterns to reduce computational complexity
  		* Examples: [Longformer](https://huggingface.co/docs/transformers/en/model_doc/longformer), [BigBird](https://huggingface.co/docs/transformers/en/model_doc/big_bird)
		* Limitation: May sacrifice some performance or struggle with certain types of long-range dependencies

	* Recurrent Memory Mechanisms:
		* Approach: Incorporating explicit memory structures to extend effective context
		* Examples: Transformer-XL, Compressive Transformers
		* Limitation: Compressions have a large gap to full attention, making it infeasible to fine-tune pre-trained LLMs

	* Retrieval-Augmented Models:

		* Approach: Combining LLMs with external retrieval systems to access relevant information
		* Examples: REALM, RAG (Retrieval-Augmented Generation)
		* Limitation: Relies on effective retrieval systems and may struggle with seamless integration of retrieved information

	* Context Compression Techniques:

		* Approach: Dynamically compressing or summarizing past context
		* Examples: LongNet, Memorizing Transformers
		* Limitation: May lose some fine-grained details in the compression process


* Long-context LLMS and their limitations:
 	* Full Fine-tuning:
 		* Approach: Retraining the entire model on longer sequences
 		* Limitation: Extremely resource-intensive
   		* Example: [Position Interpolation](https://arxiv.org/abs/2306.15595) method used 32 A100 GPUs for 2k to 8k context extension, 128 A100 GPUs for longer context fine-tuning
     	* Other methods like [Landmark Attention](https://arxiv.org/abs/2305.16300) are efficient, but prone to losing information 

**Core Tradeoff**: Efficiency vs Full Information Retention 


| Model             | Context Length |
|:------------------|----------------|
| Ministral 8B      | 128k           |
| GPT 3.5           | 4,096          |
| GPT 4             | 8,192          |
| Llama 1           | 2,048          |
| Llama 2           | 4,096          |
| Llama 3           | 8,192          |
| Claude Sonnet 3.5 | 200k           |
| gpt-4o            | 128k           |
| Gemma 7B          |  8,192         |

*For reference, the novel The Great Gatsby (208 pages) is 72k tokens*

## Solution 

* Having general purpose long context models available for our everyday workflow is incredible, but what if _we_ (collective of AI students and researchers without acces to swaths of VC funding or infinite time) want to
	* fine-tune a model to understand and summarize entire research papers, including methods, results, and discussions?
	* fine-tune a model to analyze entire contracts, identifying key clauses and potential issues across hundreds of pages?
	* fine-tune a model to analyze a patient's complete medical record, including years of notes, test results, and treatments, to assist in diagnosis and treatment planning?
	* fine-tune a model to understand entire codebases, providing more context-aware suggestions and bug detection across multiple files and functions?
	* fine-tune a model to analyze textbooks and generate tailored lesson plans, quizzes, and study guides that cover entire subjects or courses?
	* fine-tune a model to process entire quarterly reports, market analyses, and historical data to provide more comprehensive investment insights?

LongLoRA enables individuals and small teams to fine-tune models for specialized tasks that require processing and understanding large amounts of context-rich information, which was previously impractical or impossible due to computational constraints! It offers a balance between computational efficiency and maintaining the full capabilities of the original model architecture.



## LoRA Refresher  

* LoRA(Low-Rank Adaptation) is a method for efficiently fine-tuning large AI models by reducing the number of parameters that need to be updated
* **Key Idea**: Instead of updating the entire weight matrix during fine-tuning, LoRA constrains the weight updates to a low-rank space, significantly reducing computational cost.
* **How it works**
	* Weight updates are represented as a low-rank decomposition: ΔW = BA (B and A are low-rank matrices).
	* This reduces the number of parameters to update from nk to r(n + k), where r << min(n, k).
 * Enables the adaptation of large models to specific tasks without the heavy computational burden of traditional fine-tuning, which is particularly useful in low-data regimes or when computation resources are limited
 * Good for AI teams' budgets and the environment

<p align="center" width="100%">
<img src="original_lora_viz.jpeg" alt="LoRA:" style="width: 70%; min-width: 300px; display: block; margin: auto;">
</p>


* Empirical evidence has found that using LoRA for context extension is neither _effective_ or _efficient_
	* has high perplexity (less certainty in predictions)
  	* standard self-attention mechanism still yields dramatic increases in computational complexity as context length extends 



**How can we can efficently and effectively extend the context of pre-trained LLMs?**

Introducing ... LongLoRA ! Using this method, researchers were able to extend Llama2 7B to 100k context length and 70B model to 32k context length, on a **single** 8× A100 machine.



## Architecture Overview

### Major Contributions of LongLoRA:

a. Shifted Sparse Attention (S2-Attn)
b. Improved LoRA (LoRA+)

## Key Questions 

### Q1: How does S2-Attn work and why is it effective?

* Explanation of S2-Attn mechanism
* Benefits over full attention during training
* Consistency with full attention during inference

### Q2: What improvements does LongLoRA make to standard LoRA?

* Role of trainable embedding and normalization layers
* Why these changes are crucial for long-context adaptation

### Key components of LongLoRA:

a. Shifted Sparse Attention (S2-Attn)

<p align="center" width="100%">
<img src="imgs/Shift-short-attention2.png" alt="LoRA:" style="width: 70%; min-width: 300px; display: block; margin: auto;">
</p>


* **Explanation:**

	* S2-Attn is an efficient approximation of full attention used during training.
	* It allows for efficient computation while maintaining performance close to full attention.

* **Key points**:
  
	* How it works:
		* Splits the input sequence into several groups.
		* Applies attention separately within each group.
		* In half of the attention heads, shifts the group partition by half the group size.

	* Implementation:
		* Can be implemented with just two lines of code during training.
		* Does not require changes to the model architecture for inference.
  		* See [s2_attention_example.ipynb](https://github.com/isabelarvelo/LongLoRA/blob/main/s2_attention_example.ipynb) for a more in depth walk through 
 
<p align="center" width="100%">
<img src="shifted-sparse-attention-pseudocode.jpeg" alt="LoRA:" style="width: 70%; min-width: 300px; display: block; margin: auto;">
</p>

	* Benefits:
		* Reduces computational cost significantly during training.
		* Enables information flow between different groups through shifting.
		* Achieves performance close to full attention fine-tuning.

	* Comparison to other methods:
  		* Unlike other efficient attention designs (e.g., dilated or sparse attention), S2-Attn has a smaller gap to standard attention.
		* Models trained with S2-Attn can use full attention during inference.
  			* The shifting mechanism in S2-Attn prevents the model from overfitting to specific attention patterns, which allows it to generalize better when using full attention during inference.
		* Models fine-tuned with S2-Attn retain the original attention architecture during inference which allows the use of existing optimizations and infrastructure for inference.


      [Insert table here]

* When testing with the same attention pattern used in training (first row), S2-Attn performs well (8.64 perplexity).
* When testing with full attention (second row), S2-Attn still performs well (8.12 perplexity).
Other attention patterns like dilated, block sparse, and stride sparse attention show larger discrepancies between training and full-attention testing, or perform poorly overall.


b. Improved LoRA (LoRA+)

<p align="center" width="100%">
<img src="long_lora_plus.jpeg" alt="LoRA:" style="width: 70%; min-width: 300px; display: block; margin: auto;">
</p>

**Explanation:**

	* Makes embedding and normalization layers trainable.
	* These layers occupy a small proportion of total parameters but are crucial for long-context learning.

**Importance of trainable layers:**

	* Embedding: <2% of parameters in Llama2 7B
	* Normalization: ≤0.004% of parameters
	* Empirical results show a large performance gap between LoRA and full fine-tuning for long contexts, even with larger LoRA ranks

**Performance:**
	* Making embedding and normalization trainable closes this gap between standard LoRA and full fine-tuning for long contexts.
	* Achieves comparable results to full fine-tuning with much lower computational cost.


## Pseudocode description (based on the formal algorithms paper style)

Comparison with previous models/approaches



## Impacts

* Significance for efficient LLM adaptation
* Potential applications and use cases
* Future implications for long-context understanding in AI

## Other methodology worth mentioning 
While LongLoRA's primary contributions are S2-Attn and LoRA+, the paper leverages several other important techniques that are crucial for efficient long-context fine-tuning:

* DeepSpeed:
	* Implements Zero Redundancy Optimizer (ZeRO) to optimize memory usage
	* Allows for efficient training of large models by partitioning model parameters, gradients, and optimizer states across GPUs
	* Reduces memory redundancy in data-parallel training, enabling larger models and batch sizes
 	* (Rasley et al., 2020)


* Position Interpolation (PI):
	* Enables LLMs to handle longer context windows without the need for training from scratch
	* Works by interpolating between learned positional embeddings to extend to longer sequences
	* Provides a foundation for LongLoRA to build upon for context extension
 	* (Chen et al., 2023)


* FlashAttention2:
	* Reorders the attention computation and leverages classical techniques (tiling, recomputation) to significantly speed it up and reduce memory usage
	* Optimizes the memory access patterns in attention calculations
	* Crucial for efficient processing of long sequences in both training and inference
 	* (Dao, 2023) 
    

* Gradient Checkpointing:
	* A technique to reduce memory usage during backpropagation by recomputing intermediate activations instead of storing them
	* Allows for training of deeper models or with larger batch sizes at the cost of increased computation time
 	* (Chen et al., 2023)



These techniques work in synergy with LongLoRA's core contributions:

DeepSpeed and gradient checkpointing enable efficient use of GPU memory, allowing for longer sequences to be processed.
Position Interpolation provides a starting point for extending context length, which LongLoRA then fine-tunes efficiently.
FlashAttention2 complements S2-Attn by optimizing attention computations, further improving efficiency for long sequences.

## Resource Links
* This repository is forked from the [LongLoRA orginal repository](https://github.com/dvlab-research/LongLoRA) 
* Quick Links to related papers
	* [LongLoRA](https://arxiv.org/abs/2309.12307)
 	* [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685)
	* [LoRA GitHub Repository](https://github.com/microsoft/LoRA)
 	* [Positional Interpolation](https://arxiv.org/abs/2306.15595)
  	* [FlashAttention2](https://arxiv.org/abs/2307.08691)
  	* [DeepSpeed](https://dl.acm.org/doi/10.1145/3394486.3406703)
  	* [Gradient Checkpointing](https://arxiv.org/abs/1604.06174)
  	
* Relevant blog posts and videos 
	* [How to code long-context LLM: LongLoRA explained on LLama 2 100K](https://www.youtube.com/watch?v=hf5N-SlqRmA)
 	* [HuggingFace Collection of Long Context Articles](https://huggingface.co/collections/stereoplegic/long-context-65389c8f0e9beb3a415b3356)
  	* [HuggingFace Discussion](https://huggingface.co/papers/2309.12307)
  	* LongAlpaca and models with context extension via improved LoRA fine-tuning and full fine-tuning can be found in the original README below 


# References

Shouyuan Chen, Sherman Wong, Liangjian Chen, and Yuandong Tian. Extending context window of large language models via positional interpolation. CoRR, abs/2306.15595, 2023.

Tri Dao. Flashattention-2: Faster attention with better parallelism and work partitioning. CoRR, abs/2307.08691, 2023.

Jeff Rasley, Samyam Rajbhandari, Olatunji Ruwase, and Yuxiong He. Deepspeed: System optimizations enable training deep learning models with over 100 billion parameters. In KDD, pp.3505–3506. ACM, 2020.

Tianqi Chen, Bing Xu, Chiyuan Zhang, and Carlos Guestrin. 2016. Training deep nets with sublinear memory cost. arXiv preprint arXiv:1604.06174 (April 2016).

## Original LongLoRA README:

</p>
</p>

<p align="center" width="100%">
<img src="imgs/LongAlpaca.png" alt="Stanford-Alpaca" style="width: 100%; min-width: 300px; display: block; margin: auto;">
</p>

# LongLoRA and LongAlpaca for Long-context LLMs


[![Huggingface Models](https://img.shields.io/badge/Models-Huggingface%20Models-bron)](https://huggingface.co/Yukang)
[![Data](https://img.shields.io/badge/Data-LongAlpaca%2012k-light)](https://huggingface.co/datasets/Yukang/LongAlpaca-12k)
[![Paper](https://img.shields.io/badge/Paper-Arvix%20Link-green)](https://arxiv.org/abs/2309.12307)

[![Code License](https://img.shields.io/badge/Code%20License-Apache_2.0-yellow.svg)](https://github.com/dvlab-research/LongLoRA/blob/main/LICENSE)
[![Data License](https://img.shields.io/badge/Data%20License-CC%20By%20NC%204.0-orange.svg)](https://github.com/dvlab-research/LongLoRA/blob/main/DATA_LICENSE)
[![Weight License](https://img.shields.io/badge/Weight%20License-CC%20By%20NC%204.0-red)](https://github.com/dvlab-research/LongLoRA/blob/main/WEIGHT_LICENSE)


## TABLE OF CONTENTS
1. [News](#news)
2. [Highlights](#highlights)
3. [How to contribute](#how-to-contribute)
4. [Requirements](#usage-requirements)
5. [Installation and quick guide](#installation-and-quick-guide)
6. [LongAlpaca Data](#longalpaca-data)
7. [Models](#models)
8. [Training](#training)
9. [Evaluation](#evaluation)
10. [Demo](#demo)
11. [Streaming Inference](#streaming-inference)
12. [Data Generation via Pdf2Text](#data-generation-via-pdf2text)
13. [Examples](#examples)
14. [Citation](#citation)
15. [Acknowledgement](#acknowledgement)
16. [License](#license)
      
## News
- [x] [2024.1.17] [LongLoRA](https://arxiv.org/abs/2309.12307) has been accepted by **ICLR 2024** as an **Oral** presentation.
- [x] [2023.11.19] We release a new version of LongAlpaca models, [LongAlpaca-7B-16k](https://huggingface.co/Yukang/LongAlpaca-7B-16k), [LongAlpaca-7B-16k](https://huggingface.co/Yukang/LongAlpaca-13B-16k), and [LongAlpaca-7B-16k](https://huggingface.co/Yukang/LongAlpaca-70B-16k). These models are fine-tuned on a subset LongAlpaca-12k dataset with LongLoRA in SFT, [LongAlpaca-16k-length](https://huggingface.co/datasets/Yukang/LongAlpaca-16k-length). We evaluate the [LongAlpaca-7B-16k](https://huggingface.co/Yukang/LongAlpaca-7B-16k) model on LongBench and L-Eval benchmarks and results can be found [here](https://github.com/dvlab-research/LongLoRA/tree/main/benchmarks).
- [x] [2023.11.2] We have updated our LongAlpaca models from alpaca prompting to llama2 prompting, which is consistent to their pre-trained models. Please refer to the [inference code](https://github.com/dvlab-research/LongLoRA/blob/2345c6d030f61ac3a031906386a103a5b05e0e6f/inference.py#L18) with the llama2 prompting.
- [x] [2023.10.23] We support the combination of [QLoRA](https://github.com/artidoro/qlora) and LongLoRA in the [supervised fine-tuning](supervised-fine-tune-qlora.py), for further reduction of the GPU memory cost. We release the LoRA weights of a 7B model at [LongAlpaca-7B-qlora-weights](https://huggingface.co/Yukang/LongAlpaca-7B-qlora-weights).
- [x] [2023.10.18] We support [StreamingLLM](https://github.com/mit-han-lab/streaming-llm) inference on our LongAlpaca models. This increases the context-length of the multi-round dialogue in StreamingLLM.
- [x] [2023.10.8] **We release the long instruction-following dataset**, [LongAlpaca-12k](https://huggingface.co/datasets/Yukang/LongAlpaca-12k) and **the corresponding models**, [LongAlpaca-7B](https://huggingface.co/Yukang/LongAlpaca-7B), [LongAlpaca-13B](https://huggingface.co/Yukang/LongAlpaca-13B), and [LongAlpaca-70B](https://huggingface.co/Yukang/LongAlpaca-70B).
- (*The previous sft models*, [Llama-2-13b-chat-longlora-32k-sft](https://huggingface.co/Yukang/Llama-2-13b-chat-longlora-32k-sft) and [Llama-2-70b-chat-longlora-32k-sft](https://huggingface.co/Yukang/Llama-2-70b-chat-longlora-32k-sft), *have been deprecated*.)
- [x] [2023.10.3] We add support GPTNeoX models. Please refer to this [PR](https://github.com/dvlab-research/LongLoRA/pull/32) for usage. Thanks for @naubull2 for this contribution.
- [x] [2023.9.22] We release all our fine-tuned [models](https://huggingface.co/Yukang), including **70B-32k models**, [LLaMA2-LongLoRA-70B-32k](https://huggingface.co/Yukang/Llama-2-70b-longlora-32k), [LLaMA2-LongLoRA-7B-100k](https://huggingface.co/Yukang/Llama-2-7b-longlora-100k-ft). Welcome to check them out!
- [x] [2023.9.22] We release [Paper](http://arxiv.org/abs/2309.12307) and this GitHub repo, including training and evaluation code.

**LongLoRA: Efficient Fine-tuning of Long-Context Large Language Models [[Paper](http://arxiv.org/abs/2309.12307)]** <br />
[Yukang Chen](https://scholar.google.com/citations?user=6p0ygKUAAAAJ&hl=en),
[Shengju Qian](https://scholar.google.com/citations?user=QNnWmasAAAAJ),
[Haotian Tang](https://scholar.google.com/citations?user=WxL13BAAAAAJ&hl),
[Xin Lai](https://scholar.google.com/citations?user=tqNDPA4AAAAJ&hl=zh-CN),
[Zhijian Liu](https://scholar.google.com/citations?user=3coYSTUAAAAJ&hl=en),
[Song Han](https://scholar.google.com/citations?user=E0iCaa4AAAAJ&hl=zh-CN),
[Jiaya Jia](https://scholar.google.com/citations?user=XPAkzTEAAAAJ&hl=en)<br />

## Highlights
1. In LongLoRA approach, The proposed shifted short attention is easy to implement, compatible with Flash-Attention, and is not required during inference.
2. We released all our models, including models from 7B to 70B, context length from 8k to 100k, including [LLaMA2-LongLoRA-7B-100k](https://huggingface.co/Yukang/Llama-2-7b-longlora-100k-ft), [LLaMA2-LongLoRA-13B-64k](https://huggingface.co/Yukang/Llama-2-13b-longlora-64k), and [LLaMA2-LongLoRA-70B-32k](https://huggingface.co/Yukang/Llama-2-70b-longlora-32k).
3. We built up a long-context instruction-following dataset, [LongAlpaca-12k](#longalpaca-data). We released the corresponding [LongAlpaca-7B](https://huggingface.co/Yukang/LongAlpaca-7B), [LongAlpaca-13B](https://huggingface.co/Yukang/LongAlpaca-13B) and [LongAlpaca-70B](https://huggingface.co/Yukang/LongAlpaca-70B) models. To our best knowledge, this is the first open-sourced long-context 70B model.


## How to Contribute
- Make sure to have git installed.
- Create your own [fork](https://github.com/dvlab-research/LongLoRA/fork) of the project.
- Clone the repository on your local machine, using git clone and pasting the url of this project.
- Read both the `Requirements` and `Installation and Quick Guide` sections below.
- Commit and push your changes.
- Make a pull request when finished modifying the project.


## Usage Requirements
To download and use the [pre-trained weights](#pre-trained-weights) you will need:
1. Hugging Face (HF) account with valid email. Note, the email used for HF must alse be used for the license agreement.
2. Accept the Meta [license and acceptable use policy](https://ai.meta.com/resources/models-and-libraries/llama-downloads/) 


## Installation and Quick Guide
To install and run the application:
1. [Fork this repo](https://github.com/dvlab-research/LongLoRA/fork) on github
2. Clone the repository on your local machine, using git clone and pasting the url of this project.
3. Run the following code:
```
pip install -r requirements.txt
pip install flash-attn --no-build-isolation
```
4. Use either a [Released model](#released-models) or [Fine tune](#fine-tuning) a model to fit your preferences.
5. Test your model by chat.
6. Deploy your own demo.

## LongAlpaca Data

LongAlpaca-12k contains 9k long QA data that we collected and 3k short QA sampled from the original [Alpaca data](https://github.com/tatsu-lab/stanford_alpaca/blob/main/alpaca_data.json). This is to avoid the case that the model might degrade at short instruction following. The data we collect contains various types and amounts as the following figure.

<p align="center" width="100%">
<img src="imgs/data-distribution-in-longalpaca12k.png" alt="Stanford-Alpaca" style="width: 60%; min-width: 300px; display: block; margin: auto;">
</p>


| Data           | Short QA | Long QA  | Total    | Download |
|:---------------|----------|----------|----------|----------|
| LongAlpaca-12k | 3k       | 9k       | 12k      | [Link](https://huggingface.co/datasets/Yukang/LongAlpaca-12k) |

Following the original Alpaca format, our Long QA data uses the following prompts for fine-tuning:
- `instruction`: `str`, describes the task the model should perform. For example, to answer a question after reading a book section or paper. We vary the contents and questions to make instructions diverse.
- `output`: `str`, the answer to the instruction.

We did not use the `input` format in the Alpaca format for simplicity.

## Models

### Models with supervised fine-tuning
| Model          | Size | Context | Train   | Link                                                       |
|:---------------|------|---------|---------|------------------------------------------------------------|
| LongAlpaca-7B  | 7B   | 32768   | Full FT | [Model](https://huggingface.co/Yukang/LongAlpaca-7B)       |
| LongAlpaca-13B | 13B  | 32768   | Full FT | [Model](https://huggingface.co/Yukang/LongAlpaca-13B)      |
| LongAlpaca-70B | 70B  | 32768   | LoRA+ | [Model](https://huggingface.co/Yukang/LongAlpaca-70B) [(LoRA-weight)](https://huggingface.co/Yukang/LongAlpaca-70B-lora) |


### Models with context extension via fully fine-tuning
| Model                       | Size | Context | Train | Link                                                              |
|:----------------------------|------|---------|-------|-------------------------------------------------------------------|
| Llama-2-7b-longlora-8k-ft   | 7B   | 8192    | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-7b-longlora-8k-ft)  |
| Llama-2-7b-longlora-16k-ft  | 7B   | 16384   | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-7b-longlora-16k-ft)  |
| Llama-2-7b-longlora-32k-ft  | 7B   | 32768   | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-7b-longlora-32k-ft)  |
| Llama-2-7b-longlora-100k-ft | 7B   | 100000  | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-7b-longlora-100k-ft) |
| Llama-2-13b-longlora-8k-ft  | 13B  | 8192    | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-13b-longlora-8k-ft)  |
| Llama-2-13b-longlora-16k-ft | 13B  | 16384   | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-13b-longlora-16k-ft) |
| Llama-2-13b-longlora-32k-ft | 13B  | 32768   | Full FT    | [Model](https://huggingface.co/Yukang/Llama-2-13b-longlora-32k-ft) |

### Models with context extension via improved LoRA fine-tuning
| Model                       | Size | Context | Train | Link                                                                |
|:----------------------------|------|---------|-------|---------------------------------------------------------------------|
| Llama-2-7b-longlora-8k      | 7B   | 8192    | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-7b-longlora-8k) |
| Llama-2-7b-longlora-16k     | 7B   | 16384   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-7b-longlora-16k)       |
| Llama-2-7b-longlora-32k     | 7B   | 32768   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-7b-longlora-32k)       |
| Llama-2-13b-longlora-8k     | 13B  | 8192    | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-13b-longlora-8k)       |
| Llama-2-13b-longlora-16k    | 13B  | 16384   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-13b-longlora-16k)      |
| Llama-2-13b-longlora-32k    | 13B  | 32768   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-13b-longlora-32k)      |
| Llama-2-13b-longlora-64k    | 13B  | 65536   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-13b-longlora-64k)      |
| Llama-2-70b-longlora-32k    | 70B  | 32768   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-70b-longlora-32k)      |
| Llama-2-70b-chat-longlora-32k    | 70B  | 32768   | LoRA+ | [LoRA-weight](https://huggingface.co/Yukang/Llama-2-70b-chat-longlora-32k) |

## Training
### Pre-trained weights
We use LLaMA2 models as the pre-trained weights and fine-tune them to long context window sizes. Download based on your choices.

| Pre-trained weights                                                        |
|:---------------------------------------------------------------------------|
| [Llama-2-7b-hf](https://huggingface.co/meta-llama/Llama-2-7b-hf)           |
| [Llama-2-13b-hf](https://huggingface.co/meta-llama/Llama-2-13b-hf)         |
| [Llama-2-70b-hf](https://huggingface.co/meta-llama/Llama-2-70b-hf)         |
| [Llama-2-7b-chat-hf](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf) |
| [Llama-2-13b-chat-hf](https://huggingface.co/meta-llama/Llama-2-13b-chat-hf)         |
| [Llama-2-70b-chat-hf](https://huggingface.co/meta-llama/Llama-2-70b-chat-hf)         |

This project also supports GPTNeoX models as the base model architecture. Some candidate pre-trained weights may include [GPT-NeoX-20B](https://huggingface.co/EleutherAI/gpt-neox-20b), [Polyglot-ko-12.8B](https://huggingface.co/EleutherAI/polyglot-ko-12.8b) and other variants.

### Fine-tuning
```
torchrun --nproc_per_node=8 fine-tune.py  \
        --model_name_or_path path_to/Llama-2-7b-hf \
        --bf16 True \
        --output_dir path_to_saving_checkpoints       \
        --cache_dir path_to_cache \
        --model_max_length 8192 \
        --use_flash_attn True \
        --low_rank_training False \
        --num_train_epochs 1  \
        --per_device_train_batch_size 1     \
        --per_device_eval_batch_size 2     \
        --gradient_accumulation_steps 8     \
        --evaluation_strategy "no"     \
        --save_strategy "steps"     \
        --save_steps 1000     \
        --save_total_limit 2     \
        --learning_rate 2e-5     \
        --weight_decay 0.0     \
        --warmup_steps 20     \
        --lr_scheduler_type "constant_with_warmup"     \
        --logging_steps 1     \
        --deepspeed "ds_configs/stage2.json" \
        --tf32 True \
        --max_steps 1000
```

- Please remember to change `path_to/Llama-2-7b-hf`, `path_to_saving_checkpoints`, `path_to_cache` to your own directory.
- Note that you can change `model_max_length` to other values.
- You could change `ds_configs/stage2.json` to `ds_configs/stage3.json` if you want.
- Please set `use_flash_attn` as `False` if you use V100 machines or do not install flash attention.
- You can set `low_rank_training` as `False` if you want to use fully fine-tuning. It will cost more GPU memory and slower, but the performance will be a bit better.
- When training is finished, to get the full model weight:
```
cd path_to_saving_checkpoints && python zero_to_fp32.py . pytorch_model.bin
```
Note that the path_to_saving_checkpoints might be the global_step directory, which depends on the deepspeed versions.

### Supervised Fine-tuning
```
torchrun --nproc_per_node=8 supervised-fine-tune.py  \
        --model_name_or_path path_to_Llama2_chat_models \
        --bf16 True \
        --output_dir path_to_saving_checkpoints       \
        --model_max_length 16384 \
        --use_flash_attn True \
        --data_path LongAlpaca-16k-length.json \
        --low_rank_training True \
        --num_train_epochs 5  \
        --per_device_train_batch_size 1     \
        --per_device_eval_batch_size 2     \
        --gradient_accumulation_steps 8     \
        --evaluation_strategy "no"     \
        --save_strategy "steps"     \
        --save_steps 98     \
        --save_total_limit 2     \
        --learning_rate 2e-5     \
        --weight_decay 0.0     \
        --warmup_steps 20     \
        --lr_scheduler_type "constant_with_warmup"     \
        --logging_steps 1     \
        --deepspeed "ds_configs/stage2.json" \
        --tf32 True
```
- There is no need to make supervised fine-tuning upon the fine-tuned context extended models. It is all right to directly use base model as Llama2-chat models, as the amount of long instruction following data is enough for SFT.
- Our long instruction following data can be found in [LongAlpaca-12k.json](https://huggingface.co/datasets/Yukang/LongAlpaca-12k).
- Note that supervised-fine-tune.py can be replaced by supervised-fine-tune-qlora.py if you want to try 4-bit quantized fine-tuning for further GPU memory reduction. This follows [QLoRA](https://github.com/artidoro/qlora).
- If you meet issue for saving pytorch_model.bin after the qlora sft, please refer to this [issue](https://github.com/dvlab-research/LongLoRA/issues/123).

### Get trainable weights in low-rank training
In low-rank training, we set embedding and normalization layers as trainable. Please use the following line to extract the trainable weights `trainable_params.bin` from `pytorch_model.bin`
```
python3 get_trainable_weights.py --checkpoint_path path_to_saving_checkpoints --trainable_params "embed,norm"
```

### Merge LoRA Weight
Merge the LoRA weights of `pytorch_model.bin` and trainable parameters `trainable_params.bin`, save the resulting model into your desired path in the Hugging Face format:
```
python3 merge_lora_weights_and_save_hf_model.py \
        --base_model path_to/Llama-2-7b-hf \
        --peft_model path_to_saving_checkpoints \
        --context_size 8192 \
        --save_path path_to_saving_merged_model
```
For example,
```
python3 merge_lora_weights_and_save_hf_model.py \
        --base_model /dataset/pretrained-models/Llama-2-7b-hf \
        --peft_model /dataset/yukangchen/hf_models/lora-models/Llama-2-7b-longlora-8k \
        --context_size 8192 \
        --save_path /dataset/yukangchen/models/Llama-2-7b-longlora-8k-merged
```


## Evaluation
### Perplexity Validation
To evaluate a model that is trained in the low-rank setting, please set both `base_model` and `peft_model`. `base_model` is the pre-trained weight. `peft_model` is the path to the saved checkpoint, which should contain `trainable_params.bin`, `adapter_model.bin` and `adapter_config.json`. For example,
```
python3 eval.py --seq_len 8192 --context_size 8192 --batch_size 1 --base_model path_to/Llama-2-7b-hf --peft_model path_to_saving_checkpoints --data_path pg19/test.bin
```

Or evaluate with multiple GPUs as follows.
```
torchrun --nproc_per_node=auto eval_distributed.py --seq_len 8192 --context_size 8192 --batch_size 1 --base_model path_to/Llama-2-7b-hf --peft_model path_to_saving_checkpoints --data_path pg19/test.bin
```

To evaluate a model that is fully fine-tuned, you only need to set `base_model` as the path to the saved checkpoint, which should contain `pytorch_model.bin` and `config.json`. `peft_model` should be ignored.
```
python3 eval.py --seq_len 8192 --context_size 8192 --batch_size 1 --base_model path_to_saving_checkpoints --data_path pg19/test.bin
```

Or evaluate with multiple GPUs as follows.
```
torchrun --nproc_per_node=auto eval_distributed.py --seq_len 8192 --context_size 8192 --batch_size 1 --base_model path_to_saving_checkpoints --data_path pg19/test.bin
```

- Note that `--seq_len` is to set the sequence length for evaluation. `--context_size` is to set the context length of the model during fine-tuning. `--seq_len` should not be larger than `--context_size`.

- We have already tokenized the validation and test splits of PG19 and proof-pile dataset into `pg19/validation.bin`, `pg19/test.bin`, and `proof-pile/test_sampled_data.bin`, with the tokenizer of LLaMA. `proof-pile/test_sampled_data.bin` contains 128 documents that are randomly sampled from the total proof-pile test split. For each document, it has at least 32768 tokens. We also release the sampled ids in [proof-pile/test_sampled_ids.bin](https://drive.google.com/file/d/1cnzWODLRQYAd7HeugzLCIhaqzaLZv7J5/view?usp=share_link). You can download them from the links below.

| Dataset    | Split      | Link                                                                                                         |
|:-----------|------------|--------------------------------------------------------------------------------------------------------------|
| PG19       | validation | [pg19/validation.bin](https://drive.google.com/file/d/1rbJvb0qRIf2mQoN2ON7S93TbTzMnlrN6/view?usp=share_link) |
| PG19       | test       | [pg19/test.bin](https://drive.google.com/file/d/1QANDMdctpacPAYgS04adDXqByGEq-Ret/view?usp=share_link)       |
| Proof-pile | test       | [proof-pile/test_sampled_data.bin](https://drive.google.com/file/d/1bUI5lPDvrqzY_XXJJ2sSuvZx0Y9AZClE/view?usp=share_link)         |
 

### Passkey Retrieval
We provide a manner to test the passkey retrieval accuracy. For example,
```
python3 passkey_retrivial.py \
        --context_size 32768 \
        --base_model path_to/Llama-2-7b-longlora-32k \
        --max_tokens 32768 \
        --interval 1000
```
- Note that the `context_size` is the context length during fine-tuning.
- `max_tokens` is maximum length for the document in passkey retrieval evaluation.
- `interval` is the interval during the document length increasing. It is a rough number because the document increases by sentences.

## Demo
### Local Inference
To chat with LongAlpaca models,
```
python3 inference.py  \
        --base_model path_to_model \
        --question $question \
        --context_size $context_length \
        --max_gen_len $max_gen_len \
        --flash_attn True \
        --material $material_content
```
To ask a question related to a book:
```
python3 inference.py  \
        --base_model /data/models/LongAlpaca-13B \
        --question "Why doesn't Professor Snape seem to like Harry?" \
        --context_size 32768 \
        --max_gen_len 512 \
        --flash_attn True \
        --material "materials/Harry Potter and the Philosophers Stone_section2.txt"
```

To ask a question related to a paper:
```
python3 inference.py  \
        --base_model /data/models/LongAlpaca-13B \
        --question "What are the main contributions and novelties of this work?" \
        --context_size 32768 \
        --max_gen_len 512 \
        --flash_attn True \
        --material "materials/paper1.txt"
```
- Note that inference.py can be replaced by inference-qlora.py if you want to try 4-bit quantized fine-tuning for further GPU memory reduction. This follows [QLoRA](https://github.com/artidoro/qlora).

### Online Demo
To deploy your own demo run 
```
python3 demo.py  \
	--base_model path_to_model \
	--context_size $context_size \
	--max_gen_len $max_gen_len \
	--flash_attn True
```
Example 
```
python3 demo.py  \
	--base_model /data/models/LongAlpaca-13B \
	--context_size 32768 \
	--max_gen_len 512 \
	--flash_attn True
```
- Note that `flash_attn=True` will make the generation slow but save much GPU memory.

## Streaming Inference
We support the inference of LongAlpaca models with [StreamingLLM](https://github.com/mit-han-lab/streaming-llm). This increases the context-length of the multi-round dialogue in StreamingLLM.
Here is an example,
```
python run_streaming_llama_longalpaca.py \
	----enable_streaming \
	--test_filepath outputs_stream.json \
	--use_flash_attn True \
	--recent_size 32768
```
- Note that please use a smaller recent_size if you meet OOM issues, for example 8192.
- `test_filepath` is the json file that contains prompts for inference. We provide an example file [outputs_stream.json](https://drive.google.com/file/d/13WGepnamWR8FKQS2UceyhNgV1ALHNx3w/view?usp=share_link), which is a subset of LongAlpaca-12k. You can replace it to your own questions.

## Data Generation via Pdf2text
During our dataset collection, we convert paper and books from pdf to text. The conversion quality has a large influence on the final model quality. We think that this step is non-trivial. We release the tool for the pdf2txt conversion, in the folder `pdf2txt`. It is built upon `pdf2image`, `easyocr`, `ditod` and `detectron2`. Please refer to the [README.md](pdf2txt/README.md) in `pdf2txt` for more details.

## Examples
<p align="center"> <img src="imgs/paper-improvements.png" width="100%"> </p>
<p align="center"> <img src="imgs/paper-review.png" width="100%"> </p>
<p align="center"> <img src="imgs/paper-style-compare-cvpr-iclr.png" width="100%"> </p>
<p align="center"> <img src="imgs/demo-compare-journeytothewest.png" width="100%"> </p>
<p align="center"> <img src="imgs/demo-compare-harrypotter.png" width="100%"> </p>
<p align="center"> <img src="imgs/demo-compare-threebody.png" width="100%"> </p>
<p align="center"> <img src="imgs/economy-comparison.png" width="100%"> </p>
<p align="center"> <img src="imgs/economy-prediction.png" width="100%"> </p>

## Citation
If you find this project useful in your research, please consider citing:

```
@inproceedings{longlora,
  author       = {Yukang Chen and Shengju Qian and Haotian Tang and Xin Lai and Zhijian Liu and Song Han and Jiaya Jia},
  title        = {LongLoRA: Efficient Fine-tuning of Long-Context Large Language Models},
  booktitle    = {The International Conference on Learning Representations (ICLR)},
  year         = {2024},
}
```


```
@misc{long-alpaca,
  author = {Yukang Chen and Shaozuo Yu and Shengju Qian and Haotian Tang and Xin Lai and Zhijian Liu and Song Han and Jiaya Jia},
  title = {Long Alpaca: Long-context Instruction-following models},
  year = {2023},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/dvlab-research/LongLoRA}},
}
```
## Acknowledgement
-  This work is built upon the [LLaMA2](https://ai.meta.com/llama) as the pre-trained models.
-  This work can also be built upon the [GPTNeoX-HF](https://huggingface.co/docs/transformers/model_doc/gpt_neox) which is based upon [EleutherAI/GPTNeoX](https://github.com/EleutherAI/gpt-neox) as the pre-trained model architecture.
- This work is based on [DeepSpeed](https://github.com/microsoft/DeepSpeed), [peft](https://github.com/huggingface/peft), and [Flash-Attention2](https://github.com/Dao-AILab/flash-attention) for acceleration.
- Some evaluation code is modified upon [Landmark Attention](https://github.com/epfml/landmark-attention).
- We use [LongChat](https://github.com/DachengLi1/LongChat) for the retrieval evaluation.
- We follow [StreamingLLM](https://github.com/mit-han-lab/streaming-llm) for streaming inference.
- We combine [QLoRA](https://github.com/artidoro/qlora) with LongLoRA for supervised fine-tuning.


## License
- LongLoRA is licensed under the Apache License 2.0. This means that it requires the preservation of copyright and license notices. 
- Data and weights are under CC-BY-NC 4.0 License. They are licensed for research use only, and allowed only non-commercial. Models trained using the dataset should not be used outside of research purposes.
