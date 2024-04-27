---
last_modified_on: "2024-07-12"
id: Style3
title: Style Transfer for 2D Talking Head Generation (Part 3)
description: Style Transfer
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In the previous part, we have presented our general pipeline to generate a stylized talking head animation given an input audio signal, which consists of Motion Generator, Style Mapping, and Style-aware Generator modules. Here, let's investigate the style-transferring process of how to learn and capture specific talking styles of individuals to achieve personalized motion synthesis.


![image](https://vision.aioz.io/f/c1a222c66dec4a26b3dc/?dl=1)


## Style Transfer

The style transfer phase focuses on transferring the styles to a new character by re-weighting the Motion Generator given the input audio. In our transferring phase, we assume that the talking or singing styles are encoded in both the audio stream and reference images, and should be distinguishable from the visual information of the character. Therefore, this style information is learnable and can be transferred from one to another character. Practically, we also mainly rely on the pre-trained models from the training phase to perform the style transfer. Since Style-Aware Generator can cover the visual information generated from different styles, our goal in this phase is to make sure the style encoded in the Intermediate Audio-driven Motions can be adjusted to different styles rather than just the neutral one (i.e., the styles in the training data). We capture both the audio stream and reference images as the input in this stage. Figure 1 shows the details of our style transfer process (the light blue box) that performs appropriate adjustments on the Generator's weights to take into account a specific style.

![fig3](https://vision.aioz.io/f/42f083f365864c97995d/?dl=1)*<center>**Figure 1**. A detailed illustration of our Style-aware Talking Head Generator and Style Transferring process.</center>*



Given the reference images and an audio stream (e.g., *opera*, *rap*, etc.), we first use the pre-trained audio encoding to extract the audio feature and apply the Motion Generator to reconstruct the audio-driven motion $\mathbf{\phi}_{\rm mg}$. The reference images are fed through a pre-trained landmark detector to extract their corresponding facial landmarks $\mathbf{\phi}_{\rm s}$. The generated motions and facial landmarks are vectorized into $(68 \times 3)$-dimensional vectors. Both $\mathbf{\phi}_{\rm mg}$ and $\mathbf{\phi}_{\rm s}$ are then passed through a style transfer network to extract the mean features. A style transfer loss $\mathcal{L}_{\rm transfer}$ is then optimized through back-propagation. The mean features are the latent encoded vector containing both information from the audio-driven landmarks and the facial landmarks.

 
### Style Transfer Network

The style transfer network $f(\cdot)$ is a discriminator that aims to learn the differences between motions of the input reference images and audio-driven motions extracted from the Motion Generator. Thanks to the style transfer loss $\mathcal{L}_{\rm transfer}$, the network is optimized to lower the gap of both mentioned motions, and then re-weight the parameters of Motion Generator to generate output motions that is similar to the target style. After re-weighting, the Motion Generator can produce style-aware audio-driven motions which are then passed into Style-Aware Generator to generate 2D animation with style. The style transfer network has three multilayer perceptrons (MLP), each MLP layer has $1024$, $512$, and $256$ neurons, subsequently. The final layer produces the mean features used in the style transfer loss.

### Style Transfer Loss
The style transfer loss is proposed to ensure the generated motions take into account the target style. This loss is in-cooperated with the Motion Generator loss $\mathcal{L}_{\rm mg}$ for fine-tuning the Motion Generator module during the transferring process.  The style transfer loss $\mathcal{L}_{\rm transfer}$ is contributed by the constraint loss $\mathcal{L}_{\rm sc}$ and the regularization loss $\mathcal{L}_{\rm r}$. The constraint loss is introduced to learn the style from the source motion and then transfer it into the generated one through the style transfer network.  
$$
   \mathcal{L}_{\rm sc} = \left\lVert f(\mathbf{\phi}_{\rm{mg}}) - f(\mathbf{\phi}_{\rm s}) \right\lVert_2^{2} \tag{1}
$$
where $f(\cdot)$ is the style transfer network. 

The regularization loss $\mathcal{L}_{\rm r}$ aims to increase the generalization of the style transfer process. Besides, it can deal with extreme cases of the generated motions that may break the manifold of valid styles and negatively affect the generated images. This loss is computed as:
$$
    \mathcal{L}_{\rm r} = \bigg(\left\lVert \nabla_{\mathbf{\hat\phi}_{\rm{mg}}}f(\mathbf{\hat\phi}_{\rm{mg}}) \right\lVert_2 - 1\bigg)^{2} \tag{2}
$$
where $\mathbf{\hat\phi}_{\rm{mg}}$ is the joint representation that controls the contribution of source motion $\mathbf{\phi}_{\rm s}$ during the style learning process.  $\mathbf{\hat\phi}_{\rm{mg}}$ is computed from $\mathbf{\phi}_{\rm s}$ and $\mathbf{\phi}_{\rm{mg}}$ as follows:
$$
    \mathbf{\hat\phi}_{\rm {mg}} = \gamma \mathbf{\phi}_{\rm s} + (1 - \gamma) \mathbf{\phi}_{\rm {mg}} \tag{3}
$$
where $\gamma$ controls the amount of leveraged style information. 

The final transferring loss $\mathcal{L}_{\rm {transfer}}$ is computed as:
$$
\mathcal{L}_{\rm {transfer}} = \mathcal{L}_{\rm mg} + \mathcal{L}_{\rm sc} + \mathcal{L}_{\rm r}    \tag{4}
$$
where $\mathcal{L}_{\rm mg}$ is the Motion Generator reconstruction loss. So as to control the style, both reference images and the audio stream are required during the transferring process. 