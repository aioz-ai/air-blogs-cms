---
last_modified_on: "2023-11-23"
title: Foundation Model and Federated Learning (Part 2).
description: Foundation Model and Federated Learning.
series_position: 21
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: foundation_model"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


The combination of Foundation Model and Federated Learning provides many benefits and opportunities to unlock research topics in AI Research and real-world applications. Federated learning expands the data for foundation models, enables computation sharing, reduces communication costs, and preserves data security. Foundation models can be applied to multiple downstream tasks and handle various modalities and distributions of data, achieving faster convergence and better performance. We will investigate the potential, challenges, and research direction of the Foundation Model with the corporation of Federated Learning.


## Introduction

The Stanford Institute for Human-Centered Artificial Intelligence's (HAI) Center for Research on Foundation Models (CRFM) defined the term "foundation model" as "the model that is trained on broad data (generally using self-supervision at scale) that can be adapted to a wide range of downstream tasks". 

Examples of foundation models include BERT(Google), GPT family (OpenAI), Stable Diffusion, DALL-E, ...

## Foundation Model + Federated Learning
In Federated Learning (FL), clients communicate to the server (centralized FL) or other clients without the server (decentralized FL) by sending their local trained model and receiving the updated model. The communication is designed without the leak of data stored locally in each client.


By combining FL with the Foundation Model, the model can access the broader data and can see data in multiple modalities. This results in the boosted performance of the model in various tasks ( prompt engineering, in-context learning, instruction tuning, fine-tuning, pre-training, ...). Moreover, the combination also reduces the communication cost, when clients only share the whole or a proportion of model parameters, without sharing their data. However, adapting the foundation model in DFL configuration remains challenging: 

- Non-IID problem: The distribution of data in each client is different. This case can happen when we try to solve the problem with the same modality and the same output comes from each client. For example, in the heart structure segmentation, each client stores a part of a large-scale dataset. 
- Multi modalities: Each client owns data with different modalities. This case can happen when the modalities of data among clients are different, and the tasks and outputs at each node may be different. For example, in Brain Tumor Classification, a client can store a part of the BRAIN Dataset (4 classes), while another client can store a part of the FGADR Dataset (5 classes).
These factors lead to poor convergence and model performance, and the model at clients does not achieve consensus. 

The foundation model can be adapted to various downstream tasks by leveraging techniques (zero/few/one-shot learning, meta-learning, ...): 

**Zero shot learning**


[**Federated Zero-Shot Learning with Mid-Level Semantic Knowledge Transfer**](https://arxiv.org/pdf/2208.13465.pdf): 

They leveraged Zero-Shot Learning to transfer mid-level knowledge from independent non-overlapping class label spaces for federated learning: 
- In the aggregate phase, they remove the discriminator module since it may contain specific knowledge relevant to classes in each client, and use only the generator. 
- They use CLIP to enrich the mid-level semantic space. CLIP contains word embedding knowledge that can provide information regarding hierarchical relationships among classes. CLIP can help Federated Zero-shot Learning to learn richer external knowledge with the sharable common attribute space.

![Fig-3](https://drive.google.com/uc?export=view&id=1XFwVgF9yYH7GpjMDmIYsLDHvvJjg9I4X)
*<center>**Figure 3**: Overview of federated zero-shot learning with mid-level semantic knowledge transfer. Image is taken from paper  </center>*

[**Towards Fair Federated Learning with Zero-Shot Data Augmentation**](https://openaccess.thecvf.com/content/CVPR2021W/TCV/papers/Hao_Towards_Fair_Federated_Learning_With_Zero-Shot_Data_Augmentation_CVPRW_2021_paper.pdf)

They aim to improve fairness across clients due to bias and high variance caused by statistical heterogeneity. They augmented zero-shot data augmentation under-represented data to reduce statistical heterogeneity and encourage more uniform accuracy performance across clients in networks. 

They want to generate synthetic data (fake data) that matches the distribution of original data stored locally. They suggest the generated data $x$ that: the mean and standard deviation at every batch normalization of the model **when propagating the generated data** are as close to the mean and standard deviation of the mean and standard deviation at every batch normalization **stores at the model**. 



**Meta learning** 

[**PPFM: An Adaptive and Hierarchical Peer-to-Peer Federated Meta-Learning Framework**](https://ssg.lancs.ac.uk/wp-content/uploads/cynthia-PPFM.pdf)

They proposed Peer-to-Peer Federated Meta-learning framework (PPFM) to deal with the heterogeneous and distributed nature of their generated data by multiple learning loops to dynamically self-adapt its own architecture. The key of PPFM is the defragmented meta-learning framework, which can achieve high accuracy of model under limited size on heterogeneous data, and comprises four blocks: 

- Node clustering: Each cluster includes a subset of edge nodes whose training datasets share similar patterns.
- Task-level training: Each edge node in the respective cluster learns a local task-level model
- Cluster-level meta training:  Refine a cluster-level meta model which aims to extract the common features from all task-level models, and upload the model to the global node
- Global-level meta training: Constructs a global-level meta model by extracting common features from different cluster-level meta models.
![Fig-4](https://drive.google.com/uc?export=view&id=1ESxinpO-6LLxNp7R9sdR9Ws7SIRgEJ61)
*<center>**Figure 4**: Symtem architecture of PPFM. Image is taken from the paper </center>*
## References: 
1. Bommasani, Rishi, et al. "On the opportunities and risks of foundation models." arXiv preprint arXiv:2108.07258 (2021).
2. Zhuang, Weiming, Chen Chen, and Lingjuan Lyu. "When foundation model meets federated learning: Motivations, challenges, and future directions." arXiv preprint arXiv:2306.15546 (2023).
3. Yu, Zhengxin, et al. "PPFM: An Adaptive and Hierarchical Peer-to-Peer Federated Meta-Learning Framework." 2022 18th International Conference on Mobility, Sensing and Networking (MSN). IEEE, 2022.
4. Hao, Weituo, et al. "Towards fair federated learning with zero-shot data augmentation." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.
5. Sun, Shitong, et al. "Federated zero-shot learning with mid-level semantic knowledge transfer." arXiv preprint arXiv:2208.13465 (2022).

