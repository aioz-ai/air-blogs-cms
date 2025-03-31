---
last_modified_on: "2025-03-20"
title: Introduction to auto-regressive dance generation.
description:  Introduction to auto-regressive dance generation.
series_position: 34
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

## Overview
Dance generation involves generating a sequence of dance movements, which is difficult because dance movements are expressive and freeform, yet precisely structured by music. The task requires the ability to generate continuous motions with high kinetic complexity and capture nuanced details (e.g. hand animation, physical interaction between dancers and environments or other dancers, etc). The motions will also need to match the given music in the setting of music-driven dance generation.

## Auto-regressive approaches
In order to generate coherent dance sequence, many works choose Transformers and generate dance motions based on the past motions for their superior temporal feature modelling. However, these approaches suffer from overfitting, that is the newly generated motions are overfitted to be identical to the past movements to prevent motion inconsistency. Sometimes, this may cause freezing. 

### Bidirectional Autoregressive Diffusion Model for Dance Generation (BADM) (CVPR 2024)
BADM is a diffusion framework aiming to generate a long dance sequence in an autoregressive manner. It still leverages diffusion process which pertubes the the dance sequence $\hat{x}$ and denoises until getting the full sequence $x$. 
![BADM](https://vision.aioz.io/f/64cfe398b3d8405a82d1/)
*<center>Fig 1: Overview framework for BADM</center>*
To generate outputs in an autoregressive manner, the model treats the dance sequence as K slices. The initial noisy sequence $z_T$ at time $T$ is sliced into $K$ slices, each of which is denoised with the help of previously generated slice $\hat{x}_{k - 1}$ and the subsequent noise part $z_{t(k + 1)}$, alongside with segmented feature $c_k$ and beat information $b_k$. The output of the denoising process is the denoised $k^{th}$ slice $\hat{x}_k$. These slices are concatenated and fed to a Local Information Decoder which is constructed by 1D convolution layers to form the complete sequence $\hat{x}$.

*Strengths:*
- The model is enriched with beat information, while generating current slice based on the past and future slice. This helps generate a more coherent dance sequence. 

### Bailando: 3D Dance Generation by Actor-Critic GPT with Choreographic Memory (CVPR 2022)
Bailando proposes using a finite dictionary of quantized dancing units to ensure spatial consistency of dancing poses. It is made by encoding and quantizing 3D joint sequence to a codebook in an unsupervised manner using VQ-VAE, where each learned code is shown to represent a unique dancing pose. 
![bailando](https://vision.aioz.io/f/516cce1624f9408194b2/)
*<center>Overview framework for Bailando</center>*

Given music features and starting pose codes, a GPT-like model (i.e., Actor-critic motion GPT trained with reinforcement learning and supervised learning) predicts the next pose code sequence in an autoregressive manner. The corresponding quantize features are derived by looking up pose codes in the momory codebook, which are then decoded to a dance sequence.  

*Strengths:*
- Ensure spatial consistency

*Limits:*
- Rely on an established memory, which is sometimes limited and hard to construct. 


## Non-autoregressive approaches
These approaches often employ diffusion models, which learn a data distribution by reversing a scheduled noising process. They are also capable of conditional generation, making them highly versatile and controllable. However, non-autoregressive approaches often employ a simple 1D convolution to smooth out the generated output rather than generating new motion based on past motions. As a result, dance movements generated in this manner usually lack coherence. 

### Beat-It: Beat-Synchronized Multi-Condition 3D Dance Generation (ECCV 2024)
Stemming from the motivation that real-world choreography often assigns sparse key poses to musical beats and then fill in the connecting movements to generate complete dance sequences, Beat-It proposes to explicitly incorporate beat and key pose guidance as conditions to diffusion process.

![beatit](https://vision.aioz.io/f/e5d51a06b9ef43bd90a4/)
*<center> Overview framework for Beat-It </center>*

The condition for diffusion is generated thanks to attention mechanism with a custom mask i.e., Beat-Aware mask ($M_d$). In typical keyframe masks, only few frames are marked as keys. Meanwhile, in Beat-Aware mask, not only the keyframes but their neighboring frames are also marked. Valid neighboring frames are decided with respect to how close their keyframes and how close are they to the music beat. Generated beats are ensured to align with music beat through a specific beat alignment loss, with the help of a pretrained beat distance predictor.

*Strengths:*
- Achieve beat synchronization and flexible keyframe control. 

### Lodge: A Coarse to Fine Diffusion Network for Long Dance Generation Guided by the Characteristic Dance Primitives (CVPR 2024)
The motivation is similar to Beat-It, that is sparse key movements are generated first and then, connecting movements are filled in to generate a smooth dance sequence which fits the music beats and overall genre. 

![lodge](https://vision.aioz.io/f/cdd3870d14df44e391e4/)
*<center>Overview framework for Lodge</center>*

Lodge employs 2-stage coarse-to-fine grain process to generate long sequence conditioned on input music. It consists of 2 diffusion models: global diffusion and local diffusion (LD). The global diffusion model is trained to generate sparse characteristic dance primitives of 8 frames, which possess powerful expressiveness and rich semantic information. These dance primitives are further augmented (e.g., mirrored at where the beats repeat, aligned at the right beat) to align with the beats and structural information of the music. The parallel local diffusion independently generate short dance segments. Some dance primitives are used as the beggining and the end of the noise segments to guide the generation of dance segments. The complete dance sequence is achieved by concatenating them.

*Strengths:*
- Able to generate long dance sequence of any length without compromising coherence. 
- Scalable and computationally effecient during inference thanks to its ability to generate dance segments parallelly.

*Weaknesses:*
- Some dance primitives being mirrored and reused may cause repetition in generated dance.

## Summary
In this blog, autoregressive and non-autoregressve approaches for generating dance sequences are introduced. In general, auregressive approaches can generate more coherent dance movements than non-autoregressive approaches do. However, the former can face freezing and lack of diversity because predict the next movements based on past movements. Moreoever, they take longer to inference. On the other hand, non-autoregressive approaches allow a more flexible controlability, along with diverse generated outcomes.  
