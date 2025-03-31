---
last_modified_on: "2025-03-10"
title: Introduction to dance generation recent works.
description:  Introduction to dance generation recent works.
series_position: 33
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---
Dance generation involves generating a sequence of dance movements, which is difficult because dance movements are expressive and freeform, yet precisely structured by music. The task requires the ability to generate continuous motions with high kinetic complexity and capture nuanced details (e.g. hand animation, physical interaction between dancers and environments or other dancers, etc). The motions will also need to match the given music in the setting of music-driven dance generation.

## Auto-regressive approaches
In order to generate coherent dance sequence, many works choose Transformers for their superior temporal feature modelling. 

### Full Attention Cross-modal Transformer (FACT) (ICCV 2021)
Given a piece of music and a short (2 seconds) seed motion, the goal of FACT is to generate a long sequence of realistic 3D dance motions. The model can effectively learn music-motion correlation and generate varied dance sequences for each different music input. 

![Fig-1](https://vision.aioz.io/f/df409e03bde94229b99d/?dl=1)
*<center>Fig 1: FACT framework</center>*

FACT employs an audio transformer and motion transformer to encode the two types of input. The encoded inputs are then fused with the cross-modal transformer, which models the distribution between motion and music. All of the transformers employed use full-attention mechanism (as in BERT) instead of causal attention (as in auto-regressive models like GPT) because the full-attention model can access all inputs and thereby, being more expressive. The whole model is trained to predict N motions after the input instead of only one motion, encouraging the focus on temporal context. At test time, the model is applied in an auto-regressive manner, i.e., the first predicted motion serves as the input of the next generation step, the conditioning will be shifted by one in the next step.

*Strengths:*
* The motion sequence generated is coherent and fluid. 

*Limits:*
* No physical interactions between the dancer and the floor were taken into account. 
* The generated dance sequence is deterministic, i.e., cannot generate multiple realistic dances per music.

## Diffusion approaches
Diffusion models are a class of deep generative models which learn a data distribution by reversing a scheduled noising process. They are also capable of conditional generation, making them highly versatile and controllable. 

### POPDG: Popular 3D Dance Generation with PopDanceSet (CVPR 2024)
Given a piece of music, the model aims to generate corresponding 3D dances catering to aesthetic preferences of the youth. This is achieved by training the model on their proposed PopDanceSet.  
![Fig-2](https://vision.aioz.io/f/b99633210141480c802a/?dl=1)
*<center>Fig 2: POPDG framework</center>*
POPDG employs improved DDPM instead of the original DDPM for its rich diversity in generation. The core of the model's blocks lie in four attention mechanisms: MFAttention (Music Feature-Attention), MT-Attention (Music Temporal-Attention), DS-Attention (Dance SpatialAttention) and DT-Attention (Dance Temporal-Attention).
The work owes its improvement in the performance to the DS-Attention, which captures the spatial connections between human body joints. The input dance sequence is $x_{motion}$. The positional encoded $x_{motion}$ is used as $Q,K$ and the original one as $V$. The classic attention is computed: 

$$
Attention(Q, K, V, M) = softmax\left(\dfrac{QK^T + M}{\sqrt{C}}\right)V
$$
where $M$ represents Mask operation. 

An algorithm "Space Augmentation Algorithm" is further introduced in the work to strengthen the relationship between each joint in the upper body and its parent joint. Specifically, the attention weight between the root joint and the parent join will be added to the attention weight between the root and children. 

The Alignment Module (AM) applies 1D convolution on the music and motion features before combining them with the time step t in the diffusion model and feeding the resultant feature through an MLP layer. This module enhances the adaptability of the dance to music while ensuring the quality of dance. 

*Limits:*
* High training cost.
* No physical interactions between the dancer and the floor were taken into account. 

### DISCO: Disentangled Control for Realistic Human Dance Generation (CVPR 2024)
The model focuses on generating dances in real-world scenarios, e.g., social media dance where generalization accross a wide spectrum of poses, intricate human details and background is required. Given a (or a sequence of) pose keypoints, the foreground and background (seen or unseen), the model aims to generate realistic images/video conditioned on the given information. 


To achieve that, DISCO introduces disentangled control to enable accurate alterations to the human pose, while simultaneously maintaining attribute and background stability. 

![fig-3](https://vision.aioz.io/f/75dc85f221a44a23970a/?dl=1)
*<center>Fig 3: DISCO framework</center>*

**Controlling foreground via cross attention** 
The foreground is first seperated from the input with existing human matting method (e.g., SAM) for conditioning. Similar to the original Stable diffusion (SD), only difference is the original text embedding being replaced by the local CLIP image embeddings of the human foreground to serve as the key and value feature in cross-attention layer.

**Controlling background via ControlNets**
ControlNet is built upon SD, manipulates the input to the intermediate layers of the U-Net in SD, for controllable image generation. Specifically, it creates a trainable copy of the U-Net down/middle blocks and adds an additional “zero convolution” layer. The outputs of each copy block is then added to the skip connections of the original U-Net. It can be used for many types of data such as poses, segmentation...


The backgrounds and pose skeletons are encoded into latent space with a pre-trained VQ-VAE encoder in SD and a pose encoder respectively. The pose inputs are encoded via four convolution layers. Two dedicated ControlNets will learn the background and pose control. The outputs of these 2 nets are summed and fed into the middle block and up block of the UNet. 

**Temporal Matching** for better frame continuity, additional 1D convolution and Attention are included in the Transblock and ResBlock. 

**Human-Attribute Pre-training**
DISCO further enhances the ability of the model in generating diverse and faithful human attributes (e.g., clothing, makeups, etc.) by designing a Human Attribute pre-training task. Before fine-tuning to generate dance videos, the model needs to reconstruct the whole image given the foreground and the background features. In this way, model effectively learns the distinguishment between human subject and foreground, as well as diverse human attributes from large-scale images that can be easily collected from the internet. In this task, the participation of poses is neglected and therefore, the pose controlnet is removed while the rest remain unchanged. 

*Strengths:*
* Offers extra controbility over generated images and vides.
* Generate videos with stable background and faithful human attributes.

*Limits:*
* Cannt handle hand posture well without the fine-grained hand pose control.
* Hard to be applied to multi-person scenarios and human-object interactions.

## Comparison of Dance Generation Methods

Dance generation models aim to produce realistic and coherent dance movements that align with given music. The primary approaches include transformer-based and diffusion-based methods.

| **Method** | **Approach** | **Strengths** | **Limitations** |
|------------|-------------|--------------|----------------|
| **FACT (ICCV 2021)** | Transformer-based, full attention model | - Generates fluid and coherent motion  <br /> - Captures music-motion correlations well | - Lacks physical interactions with the environment  <br /> - Deterministic generation (no diversity in outputs) |
| **POPDG (CVPR 2024)** | Diffusion-based (DDPM), spatial-temporal attention mechanisms | - Generates diverse dances  <br /> - DS-Attention improves joint spatial relationships | - High training cost  <br /> - Lacks physical interactions with the floor |
| **DISCO (CVPR 2024)** | Diffusion-based (Stable Diffusion + ControlNet), disentangled control over pose and background | - Enables controllable image/video generation  <br /> - Maintains stable backgrounds and detailed human attributes | - Struggles with fine hand poses  <br /> - Hard to generalize to multi-person interactions and human-object interactions |

### Key Insights
1. **FACT**: Uses transformers for music-motion alignment, offering fluid but deterministic outputs.
2. **POPDG**: Utilizes diffusion models for diversity, incorporating spatial-temporal attention at a high computational cost.
3. **DISCO**: Enhances realism and control through ControlNets but struggles with fine details like hand poses and multi-person interactions.

### Considerations for Future Improvements
- Enhancing physical interactions between the dancer and the environment.
- Improving diversity in dance generation without sacrificing motion realism.
- Addressing fine-grained control over hand poses and multi-person interactions.

