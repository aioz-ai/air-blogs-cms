---
last_modified_on: "2022-04-24"
title: Vision Transformer
description: A sstudy about Vision Transformer
series_position: 17 
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In computer vision, we usually approach the problem by using convolutional architectures due to how effective they are. From small scale to large scale, CNN-based methods remain dominant. Meanwhile, Transformer is a go-to method to work with natural language processing (NLP) because it is highly computational efficient and scalable. Nowadays we can train 500B parameters with self-attention-based architecture. Inspired from NLP success, Vision Transformer (ViT) [1] is a novel approach to tackle computer vision using Transformer encoder with minimal modifications. While small and middle-size dataset are ViT's weakness, further experiment show that ViT performs well and easily achieve state-of-the-art results. Furthermore, with pre-trained model on ImageNet-21K, ViT now can easily beats SoTA on small dataset like CIFAR10. In this blog, we will introduce the basic idea of Vision Transformer.    

## Vision Transformer (ViT) 

![](https://vision.aioz.io/f/d2990be8752f4bf7b778/?dl=1)

Figure 1: Model overview. An image input is split into multiple patches, then we see them as normal token that will be embedded and feed into the encoder. To perform classification task, an extra learnable parameter is added at the beginning of the sequence, and and MLP head at its output position.

Unlike normal Transformer, which needs encoder and decoder, Vision Transformer only need the encoder part with a little modifications because our main task is image classification. The first modification is to split the input images into several patches, then feed them using multilayer perceptron or a simple CNN model to encode them. The second modification is to attach a [class] token at the beginning of the embedding input sequence, its output will behave like the image representation. Formally, we reshape the image $\mathbf{x} \in \mathbb{R}^ {H \times W \times C} \rightarrow (\mathbf{x}_{p}) \in \mathbb{R}^{N \times\left(P^{2} \cdot C\right)}$, with $\mathbf{x}_{p} \in \mathbb{R}^{\left(P^{2} \cdot C\right)}$, where:

- $(H, W)$ is the resolution of the original image, 
- $C$ is the number of channels, 
- $(P, P)$ is the resolution of each image patch, 
- $N=H W / P^{2}$ is the resulting number of patches

Similar to BERT's `[class]` token, a learnable embedding $z_0^0 = \mathbf{x}_{\text{class}}$ is added at the beginning of the embedded patches sequence. And after $L$ encoder layers $z_0^0 \rightarrow \underbrace{\text{encoder} \rightarrow \text{encoder} \rightarrow \cdots}_{L} \rightarrow z_0^L$, the final $z_0^L$ is the image representation. During pretraining and fine-tuning, a MLP to classify $z_0^L$ is added at its position. Position embedding is the standard 1-D Positional Embedding. Note that all MLP in the encoder consists of 2 layers with a GELU non-linearity. 
Formally, the ViT encoder is: 
$$

\mathbf{z}_{0} &=\left[\mathbf{x}_{\text {class }} ; \mathbf{x}_{p}^{1} \mathbf{E} ; \mathbf{x}_{p}^{2} \mathbf{E} ; \cdots ; \mathbf{x}_{p}^{N} \mathbf{E}\right]+\mathbf{E}_{\text {pos }}, & & \mathbf{E} \in \mathbb{R}^{\left(P^{2} \cdot C\right) \times D}, \mathbf{E}_{\text {pos }} \in \mathbb{R}^{(N+1) \times D} \\
\mathbf{z}_{\ell}^{\prime} &=\operatorname{MSA}\left(\operatorname{LN}\left(\mathbf{z}_{\ell-1}\right)\right)+\mathbf{z}_{\ell-1}, & & \ell=1 \ldots L \\
\mathbf{z}_{\ell} &=\operatorname{MLP}\left(\operatorname{LN}\left(\mathbf{z}_{\ell}^{\prime}\right)\right)+\mathbf{z}_{\ell}^{\prime}, & & \ell=1 \ldots L \\
\mathbf{y} &=\operatorname{LN}\left(\mathbf{z}_{L}^{0}\right) & &

$$

Where $\operatorname{LN} (\cdot)$ is LayerNorm, $\operatorname{MSA}(\cdot)$ is multihead self attention, $\mathbf{E}$ is the embedding matrix, $\mathbf{z}$ is the embedded vector and $D$ is the hidden dimension. The more visual illustration about the model is shown in Figure 1. 

## Vision Transformer's variants 

There are three main variants: Base, Large, and Huge. The detail is shown in Table 1. When we are picking which backbone or variants to use, usually it is denoted as `ViT - <size> / <patch size>`, for example ViT-B/8 means the Base variants with $8 \times 8$ patch size.

| Model       | Layers | Hidden size $D$ | MLP size | Heads | Params |
| ----------- | :----: | :-------------: | :------: | :---: | :----: |
| ViT - Base  |   12   |       768       |   3072   |  12   |  86M   |
| ViT - Large |   24   |      1024       |   4096   |  16   |  307M  |
| ViT - Huge  |   32   |      1280       |   5120   |  16   |  632M  |

Table 1: Detais of ViT model variants.

While increasing the model size usually help increasing the performance on large dataset, it is not the case for Vision Transformer. As shown in Table 2, some variants perform worse than the Base architecture. Therefore, we need to keep in mind about this characteristic when working with ViT. 

| Pretrained data | Data               | ViT-B/16 | ViT-B/32 | ViT-L/16 | ViT-L/32 | ViT-H/14 |
| --------------- | ------------------ | -------- | -------- | -------- | -------- | -------- |
| ImageNet        | CIFAR-10           | 98.13    | 97.77    | 97.86    | 97.94    | -        |
|                 | CIFAR-100          | 87.13    | 86.31    | 86.35    | 87.04    | -        |
|                 | Oxford Flowers-102 | 89.49    | 85.43    | 89.66    | 86.36    | -        |
|                 | Oxford-IIIT-Pets   | 93.81    | 92.04    | 93.64    | 91.35    | -        |
| ImageNet-21k    | CIFAR-10           | 98.95    | 98.79    | 99.16    | 99.13    | 99.27    |
|                 | CIFAR-100          | 91.67    | 91.98    | 93.44    | 93.04    | 93.82    |
|                 | Oxford Flowers-102 | 99.38    | 99.11    | 99.61    | 99.19    | 99.51    |
|                 | Oxford-IIIT-Pets   | 94.43    | 93.02    | 94.73    | 93.09    | 94.82    |

Table 2: Top1 accuracy (in %) of ViT variants on various datasets when pretrained on ImageNet and ImageNet-21k.



## Reference 

[1] Dosovitskiy, A., Beyer, L.,  Kolesnikov, A., Weissenborn, D., & Zhai, X. (2021). Thomas  Unterthiner Mostafa Dehghani Matthias Minderer Georg Heigold Sylvain  Gelly Jakob Uszkoreit and Neil Houlsby. An image is worth 16x16 words:  Transformers for image recognition at scale. In *International Conference on Learning Representations*.