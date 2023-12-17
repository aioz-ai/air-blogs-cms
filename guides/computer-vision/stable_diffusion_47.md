---
last_modified_on: "2023-04-01"
title: Latent Diffusion Models (Stable Diffusion)
description: Algorithmic and architectural explanations  of Stable Diffusion
series_position: 47
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

Recently, Stable Diffusion has become one of the hottest topics in the domain of generative artificial intelligence (generative AI). Stable Diffusion is a generative AI model that can produce images from text and image prompts. It is well known for its capability of generating very impressive and photorealistic images that precisely adhere to the users' descriptions. The core technique of Stable Diffusion is based on Latent Diffusion Model (LDM) [1], a variant of the foundational Diffusion Model introduced in our [previous series](https://blog.ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_31/). In this post, we will delve deeper into the algorithmic and architectural details of this type of Diffusion Model to gain more understanding of how it works and how it could be able to create such amazing results.


## Learning to generate images in latent space
We have known that Diffusion Models can achieve state-of-the-art results in image synthesis [2]. However, the denoising process is extremely slow and takes up lots of memory especially when generating high-resolution images. This is because the image space is very immense. Let's imagine an image with a size of $512\times 512$ and three color channels (red, green, and blue), which, in total, corresponds to a pixel space with $786,432$ dimensions. And standard Diffusion Networks have to process that amount of input numbers in every single denoising step (not to mention that the network usually contains around tens to hundreds of millions parameters with hundreds of layers). Thus, it is very challenging to train these models as well as utilize them for inference efficiently. 

Latent Diffusion Model (LDM) [1] is proposed to address this problem. Instead of operating in the very high-dimensional pixel space, the model first compresses the image into a lower-dimensional latent space (which is perceptually equivalent to the data space but much more computationally efficient). 


Next, the diffusion process is trained to work on this compressed latent space, thus can significantly reduce computational cost and speed up inference. It is based on the observation that most bits of an image correspond to imperceptible details (semantically meaningless information), whereas the semantic and conceptual composition of the data can be preserved after compression. LDM achieves high efficiency by suppressing this pixel-level redundancy information with an autoencoder and then generating semantic concepts with diffusion model on the learned latent space. 

![image](https://vision.aioz.io/f/e5c11bffe0784aad8095/?dl=1)
*<center>**Figure 1**. Autoencoder as a compression model ([Source](https://towardsdatascience.com/autoencoders-introduction-and-implementation-3f40483b0a85)).</center>* 

The perceptual compression is handled by an autoencoder as illustrated in Figure 1. At first, given an image $\mathbf{x} \in \mathbb{R}^{H\times W\times 3}$ with 3 RGB channels per pixel, we use an encoder $\mathcal{E}$ to encode and compress it into a more compact latent image $\mathbf{z} = \mathcal{E}(\mathbf{x})$  of size $\mathbf{z} \in \mathbb{R}^{h\times w\times c}$. Essentially, the image is downsampled to a smaller size with the downsampling rate $f=H/h=W/w$, and we choose the downsampling rate based on the factor of $f = 2^m, m \in \mathbb{N}$. Next, a decoder $\mathcal{D}$ is leveraged to decode the latent back to the original pixel space with $\mathbf{\hat{x}} = \mathcal{D}(\mathbf{z}) \in \mathbb{R}^{H\times W\times 3}$. Accordingly, the autoencoder is mainly trained to reconstruct the original input image (the authors used a combination of a perceptual loss [4] and a patch-based adversarial objective [5] to enforce local realism and avoid blurriness). In addition, the authors of LDM have investigated two regularization types to avoid high-variance latent spaces (e.g., where the latent may contain some random irrelevant information):

* KL-reg: a mild KL divergence regularization towards a standard normal distribution on the learned latent, similar to [VAE model](https://blog.ai.aioz.io/guides/computer-vision/VAE_39/): $KL(\mathcal{Z} \Vert \mathcal{N}(0,I))$, where $\mathcal{Z}$ is inferred by estimating the mean and variance across multiple samples.
* VQ-reg: apply vector quantization in the decoder (quantize the feature vector by looking up the nearest one in a pre-defined learned vocabulary to filter out redundant information). This is similar to VQ-VAE model [3] but the quantization layer is set at the beginning of the decoder: $\mathbf{z}_q =\mathbf{e}_k$ where $k = \arg\min_j\Vert\mathbf{z} - \mathbf{e}_j\Vert$ , with $\mathbf{e}_j$ is an element vector of the vocabulary $\mathbf{E}$ with pre-defined size.

With the trained autoencoder ($\mathcal{E}$ and $\mathcal{D}$), we are able to actually work on a condensed, low-dimensional latent space where irrelevant and non-useful details are removed. In particular, we can train the Diffusion Model to operate in this latent space (a theoretical explanation of diffusion training objective is provided in [this post](https://blog.ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_37/)), with the following loss function:

$$
\mathcal{L}_{LDM} = \mathbb{E}_{\mathbf{z}_0=\mathcal{E}(\mathbf{x}_0), \epsilon \in \mathcal{N}(0,I), t} \left[ \Vert \epsilon -\epsilon_\theta(\mathbf{z}_t,t) \Vert^2_2\right]
$$

To synthesize a new image, the model starts by sampling a random latent noise (low-dimensional) and repeats the denoising (reverse) process for $T$ steps to obtain a "clean" latent image representation. This latent representation is then finally decoded back to the image space by using the decoder. The whole pipeline is illustrated in the Figure 2 below:


![image](https://vision.aioz.io/f/95348d604ec84e559fe5/?dl=1)*<center>**Figure 2**. Image generation workflow of Latent Diffusion Model ([Source](https://huggingface.co/blog/stable_diffusion)).</center>* 






## Network Architecture and Conditioning Mechanism




Figure 3 depicts the architecture of Latent Diffusion Model. The denoising model backbone $\epsilon(\mathbf{z}_t, t)$ is implemented by a time-conditional U-Net. To handle flexible conditioning information (such as text or category labels) for generating images, the network backbone is also augmented with the cross-attention mechanism. The main purpose of conditioning is to steer the network prediction so that it can output the image according to what we want.

![image](https://vision.aioz.io/f/328efb8c9a264519a3cd/?dl=1)*<center>**Figure 3**. Latent Diffusion Model's architecture ([Source](https://arxiv.org/abs/2112.10752)).</center>* 

For example, in text-to-image generation, the input conditioning text $y$ is first projected into an embedding space via a specific domain (text) encoder $\tau_\theta(y)$. Stable Diffusion (V1) uses a pretrained OpenAI's ViT-L/14 CLIP text encoder model [6] to encode the text prompt into a sequence of 768-dimensional vector tokens that can semantically represent the whole text.

Then, the encoded conditioning data is incorporated into denoising U-Net via a cross-attention layer as follows:

$$
\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\Big(\frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{d}}\Big) \cdot \mathbf{V} 
$$

$$
\text{with }
\begin{cases}
\mathbf{Q} = \mathbf{W}^{(i)}_Q \cdot \varphi_i(\mathbf{z}_t) \\
\mathbf{K} = \mathbf{W}^{(i)}_K \cdot \tau_\theta(y) \\
\mathbf{V} = \mathbf{W}^{(i)}_V \cdot \tau_\theta(y) \\
\end{cases}
$$

where $\tau_\theta(y) \in \mathbb{R}^{M \times d_\tau}$ is the encoded text embedding sequence with $M$ tokens and $d_\tau$ per token dimension. Here, we also need to flatten the intermediate features of the U-Net into a format that is compatible with the conditioning input at the $i$-th layer, by $\varphi_i(\mathbf{z}_t) \in \mathbb{R}^{N \times d^i_\epsilon}$ with $N$ is the flattened 2D spatial dimension of the U-Net's feature map. $\mathbf{W}^{(i)}_Q \in \mathbb{R}^{d \times d^i_\epsilon}$ is a learnable linear projection matrix to transform the U-Net's intermediate features into the query $\mathbf{Q}$. And  $\mathbf{W}^{(i)}_K, \mathbf{W}^{(i)}_V \in \mathbb{R}^{d \times d_\tau}$ are the matrices to transform the encoded condition into the corresponding key $\mathbf{K}$ and value $\mathbf{V}$.




## Summary
In summary, Stable Diffusion has demonstrated a remarkable achievement in text-to-image generative models. The key improvement over existing diffusion approaches is that it learns the diffusion process in the compressed latent space instead of the high-dimensional original pixel space, which has resulted in a huge boost in both training and inference speed, and is also runnable on common consumer GPU devices. Its capabilities enable various creative applications such as text-to-image, image-to-image, image editing, and inpainting. Additionally, we would like to show an example of images generated from the text prompt "a pikachu is walking on the street " on the Hugging Face [demo website](https://huggingface.co/spaces/stabilityai/stable-diffusion), in the figure below:

![image](https://vision.aioz.io/f/7bd9f23d0f4b414e98c8/?dl=1)*<center>**Figure 4**. Images generated from text "a pikachu is walking on the street".</center>* 







## References
[1] Rombach R, Blattmann A, Lorenz D, Esser P, Ommer B. High-resolution image synthesis with latent diffusion models. In CVPR 2022.

[2] Dhariwal P, Nichol A. Diffusion models beat gans on image synthesis. In NIPS 2021.

[3] Van Den Oord A, Vinyals O. Neural discrete representation learning. In NIPS 2017.

[4] Richard Zhang, Phillip Isola, Alexei A. Efros, Eli Shechtman, and Oliver Wang. The unreasonable effectiveness of deep features as a perceptual metric. In CVPR 2018.

[5] Phillip Isola, Jun-Yan Zhu, Tinghui Zhou, and Alexei A.Efros. Image-to-image translation with conditional adversarial networks. In CVPR 2017.

[6] Radford A, Kim JW, Hallacy C, Ramesh A, Goh G, Agarwal S, Sastry G, Askell A, Mishkin P, Clark J, Krueger G. Learning transferable visual models from natural language supervision. In ICML 2021.