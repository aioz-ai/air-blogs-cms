---
last_modified_on: "2024-09-17"
id: Manifold1
title: Scalable Group Choreography via Variational Phase Manifold Learning (Part 1)
description: Introduction about Scalable Music-Driven Group Choreography
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---
Generating group dance motion from the music is a challenging task with several industrial applications. Although several methods have been proposed to tackle this problem, most of them prioritize optimizing the fidelity in dancing movement, constrained by predetermined dancer counts in datasets. This limitation impedes adaptability to real-world applications. Our study addresses the scalability problem in group choreography while preserving naturalness and synchronization. 

![](https://vision.aioz.io/f/4248eadb89e143479193/?dl=1)*<center>**Figure 1**. We present a new group dance generation method that can generate a large number of dancers within a fixed resource consumption. The illustration shows a generated group dance sample with $100$ dancers. </center>*

In particular, we propose a phase-based variational generative model for group dance generation on learning a generative manifold. Our method achieves high-fidelity group dance motion and enables the generation with an unlimited number of dancers while consuming only a minimal and constant amount of memory. The intensive experiments on two public datasets show that our proposed method outperforms recent state-of-the-art approaches by a large margin and is scalable to a great number of dancers beyond the training data.

## Introduction
The proliferation of digital social media platforms has significantly increased the popularity of creating and sharing dance videos. This surge in interest has led to the daily production and consumption of millions of dance videos across various online platforms, attracting attention from the research community in fields such as computer vision and computer graphics. Recent advancements in these areas have focused on generating authentic dance movements in response to music, with broad applications spanning animation, virtual idols, virtual metaverses, and dance education. These technologies provide powerful tools for artists, animators, and educators, enhancing creativity and enriching the dance experience for both performers and audiences.
![](https://vision.aioz.io/f/fc964846ba634396bed8/?dl=1)*<center>**Figure 2**. An example visualization of a virtual group dancer. </center>*
While considerable progress has been made in generating solo dance motions, creating synchronized and realistic group dance performances remains a complex and unresolved challenge. Existing methods typically face limitations in scalability, either generating dances for a fixed number of performers or suffering from high memory consumption due to architectural constraints. These approaches, often based on collaborative mechanisms such as cross-entity or global attention, struggle to scale up for larger groups of dancers, limiting their practical applicability. Moreover, the reliance on predefined datasets with a fixed number of dancers further restricts these models from being adapted to real-world scenarios requiring larger group choreographies.

To address these challenges, we propose a novel approach to scalable group dance generation using a phase-based variational generative model, termed Phase-conditioned Dance VAE (PDVAE). Unlike traditional variational autoencoders that operate in high-dimensional motion space and often struggle to capture temporal dynamics, PDVAE leverages phase parameters in the frequency domain to represent the latent motion space. This enables the generation of realistic and synchronized group dance performances without increasing computational and memory costs, even as the number of dancers grows. PDVAE provides a flexible and efficient solution for generating crowd-scale dance animations, with potential applications in diverse fields such as entertainment, virtual reality, education, and media production.

In this work, we aim to advance the state-of-the-art in scalable group dance generation by overcoming the limitations of existing methods and demonstrating the feasibility of generating large-scale, natural, and synchronized dance performances efficiently.

To summarize, our key contributions are as follows:
    - We introduce PDVAE, a phase-based variational generative model for scalable group dance generation. The method focuses on generating large-scale group dance under limited resources.
    - To effectively learn the manifold that is group-consistent (i.e., dancers within a group lie upon the same manifold), we propose a group consistency loss that enforces the networks to encode the latent phase manifold to be identical for the same group given the input music.
    - Extensive experiments along with thorough user study evaluations demonstrate the state-of-the-art performance of our model while achieving effective scalability.

## Related Works
### Music-driven Choreography
Crafting natural human choreography derived from music poses a complex challenge, encompassing the need for synchronization, coherence, and expressiveness between movement and musical inputs. Earlier approaches often relied on music-motion similarity constraints to ensure alignment between the generated dance and the accompanying music. Many of these methods employ heuristic algorithms to stitch together pre-existing dance segments from limited music-dance databases, successfully producing extended and realistic dance sequences. However, such techniques are constrained when attempting to generate novel dance fragments, as they depend heavily on pre-defined segments rather than innovative motion generation  .

Recent advancements have focused on utilizing deep learning architectures to map music into dance motions. Various techniques, including Convolutional Networks (CNN)   , Recurrent Networks (RNN)  , Graph Neural Networks (GNN)  , Generative Adversarial Networks (GAN) , and Transformer models  , have been explored for dance generation. These models typically rely on inputs such as the current music and a brief history of previous dance motions to predict the next human poses in a sequence. For instance, innovations in multi-modal feature fusion have enabled the simultaneous integration of music and text, producing dance sequences guided by both musical and textual cues .

Despite the progress made by these methods, generating synchronized and coordinated dance movements for multiple dancers remains a significant challenge. Achieving harmony between multiple dancers requires not only temporal alignment but also the consideration of spatial relationships and interactions between performers. This adds complexity to the generation process, making it difficult for current techniques to manage multiple dancers cohesively. Additionally, these methods are often constrained by the limited number of dancers present in their training datasets .



![](https://vision.aioz.io/f/2f8eece31fa2477c8ad3/?dl=1)*<center>**Figure 3**. Multi-person Tracking with GNN. </center>*

Several recent works have sought to address these limitations. For instance, Perez et al. propose a multimodal transformer combined with a normalizing-flow-based decoder to predict a distribution of possible future poses, offering improved flexibility in motion generation. Feng et al. introduce a motion repeat constraint for long-term generation, allowing their model to generate future frames while taking into account historical dance motions. Le et al. further explore group dance by examining consistency and diversity among dancers, ensuring that generated movements maintain coherence across multiple performers. However, these methods are still limited by the number of dancers depicted in training datasets, restricting their scalability for larger group performances. 

![](https://vision.aioz.io/f/58bb982c59e74a379d88/?dl=1)*<center> **Figure 4**. Local Mesh Fitting process for conducting a dance dataset. </center>*

Overall, while significant strides have been made in music-driven dance generation, further research is needed to overcome the challenges of scalability and synchronization in group dance synthesis.

### Motion Manifold Learning

Motion manifold learning has garnered significant attention in computer vision and artificial intelligence, primarily aiming to understand the fundamental structures underlying human movement and dynamics. This approach offers the capability to generate human motion patterns, providing insights into intrinsic motion dynamics, managing nonlinear relationships within motion data, and learning contextual and hierarchical representations. Consequently, numerous methodologies have emerged, each contributing distinct perspectives to advance the comprehension and synthesis of human motion.

Holden et al. pioneered motion manifold learning by generating character movements from high-level parameters mapped to a motion manifold, eliminating the need for manual preprocessing while enabling natural, smooth post-generation editing of motion sequences. MotionCLIP introduced a 3D human motion autoencoder aligned with CLIP's semantic space, facilitating semantic text-based motion generation, disentangled editing, and abstract language specification. This approach capitalizes on CLIP's rich semantic knowledge, integrating it within the motion manifold for enhanced control and interpretation of human movement. Sun et al. further advanced this field by employing VQ-VAE to learn a low-dimensional motion manifold, effectively refining motion sequences to improve coherence and continuity.

![](https://vision.aioz.io/f/201bbb5fd6fe402fb50e/?dl=1)*<center> **Figure 5**. Manifold conduction. </center>*
In the context of group dance generation, motion manifold learning presents a promising solution to scalability challenges, particularly the restricted number of dancers present in most datasets. By learning a distribution over dance motions within the manifold, this approach enables the generation of synchronized, cohesive group dance sequences, potentially overcoming dataset limitations. This direction of research highlights the potential of manifold-based methods to enhance scalability, allowing for the synthesis of large-scale, realistic group dance performances that maintain temporal and spatial harmony.


## Next
In the next part, we will dive deeply into the main proposal that leveragw manifold learning to handle scalability of group dance generation.