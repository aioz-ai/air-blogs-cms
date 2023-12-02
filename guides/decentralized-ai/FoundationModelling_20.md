---
last_modified_on: "2023-11-17"
title: Foundation Model and Federated Learning (part 1).
description: Foundation Model and Federated Learning.
series_position: 20
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: foundation_model"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


The combination of Foundation Model and Federated Learning provides many benefits and opportunities to unlock research topics in AI Research and real-world applications. Federated learning expands the data for foundation models, enables computation sharing, reduces communication costs, and preserves data security. Foundation models can be applied to multiple downstream tasks and handle various modalities and distributions of data, achieving faster convergence and better performance. We will investigate the potential, challenges, and research direction of the Foundation Model with the corporation of Federated Learning.


## Introduction

The Stanford Institute for Human-Centered Artificial Intelligence's (HAI) Center for Research on Foundation Models (CRFM) defined the term "foundation model" as "the model that is trained on broad data (generally using self-supervision at scale) that can be adapted to a wide range of downstream tasks". 

Examples of foundation models include BERT(Google), GPT family (OpenAI), Stable Diffusion, DALL-E, ...

The technological foundations of foundation models give rise to the capabilities that determine their potential. To form a foundation model that can be adapted to various tasks, many things need to be considered: 


### Model Architecture

![Fig-1](https://drive.google.com/uc?export=view&id=1RXc1Vzy2HLFia9Ts_K4hVURuz4bqbf0n)
*<center>**Figure 1**: Key properties of foundation model. Image is taken from [here]()</center>*

To model a network into a foundation model, consider the following properties: 

- **Expressivity**: Ability of the network to model the train data distribution and represent the data flexibility. Models with stronger structural priors can improve sample efficiency on the particular tasks that benefit from these assumptions; while conversely, models that integrate weaker inductive biases learn more slowly, but can in turn scale to higher volumes of data and adapt to a diverse set of domains, since they do not rely on restrictive or task-specific suppositions.


- **Scalability**: Due to various data sources and powerful computational resources, foundation models should be scalable across all dimensions (models’ depth and width, training time, number of parameters, amount of data) to perform well on all distribution of data (text, image, ...). 
 
- **Multimodality**: Foundation models should ideally connect together the different modalities (text, image, voice, depth, ...) so as to widely apply to various tasks.
    
    - Generally: The foundation model should be designed by using modules that are able to share structure for each modality.

    - Multimodal Interaction: Do the weight used in the architecture can be applied to all modalities? 

- **Memory**: The foundation model must have a deep understanding of the world and subjects, so it needs to collect and store large amounts of knowledge. Memory Networks, Neural Turing Machines, Neural State Machines, and MAC models help disentangle memory and computation, and the Transformer technique could help the foundation model handle long-term dependencies. After training from gathering data, there are multiple ways to retrieve particular facts or memories necessary (explicit prompt, implicit recollection, and reshaping of the prior knowledge, ) for downstream tasks.

- **Compositionality**: Compositionality is the principle according to which the meaning of the whole is derived from the meaning of its constituent parts, and the rules applied to combine them. Compositionality can help the foundation model resolve the out-of-distribution, and enhance interpretability, controllability, and data efficiency. Compositionality can be available in different forms (model architecture, training data, computational, representation).


### Adaptaion 

![Fig-2](https://drive.google.com/uc?export=view&id=1Vk3Fuxge7g9YKffZcXtqmjckfdgsT61t)
*<center>**Figure 2**: Foundation model is converted into an adapted model for suitable situation. Image is taken from [here]() </center>*

Although foundation models provide a powerful general-purpose engine for processing multi-modal information, this information is highly general and meaningless. Adapting the foundation model to specific tasks is necessary for application. Several methods have been tried to improve the performance of the foundation model in the specific task or application: 

- When dealing with adaption, many proposed methods help reduce the storage for adapting (low-storage adaption) by freezing most of the parameters in the model, and only learning a small number of parameters that are related to the task. They can tune only the final layer of the pre-trained model or the model's bias vector, ... 
- Task specialization is a widely-studied case of foundation model adaptation when we need to determine which field, or which task to adapt and optimize the foundation model. In addition, it is often necessary to specialize a foundation model to a particular domain without limiting the breadth of tasks the foundation model can accomplish (domain specialization). One approach to domain specialization is to include an intermediate adaptation step, where the foundation model continues training on unlabeled data from the specialized domain.


## References: 
1. Bommasani, Rishi, et al. "On the opportunities and risks of foundation models." arXiv preprint arXiv:2108.07258 (2021).
2. Zhuang, Weiming, Chen Chen, and Lingjuan Lyu. "When foundation model meets federated learning: Motivations, challenges, and future directions." arXiv preprint arXiv:2306.15546 (2023).
3. Yu, Zhengxin, et al. "PPFM: An Adaptive and Hierarchical Peer-to-Peer Federated Meta-Learning Framework." 2022 18th International Conference on Mobility, Sensing and Networking (MSN). IEEE, 2022.
4. Hao, Weituo, et al. "Towards fair federated learning with zero-shot data augmentation." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.
5. Sun, Shitong, et al. "Federated zero-shot learning with mid-level semantic knowledge transfer." arXiv preprint arXiv:2208.13465 (2022).