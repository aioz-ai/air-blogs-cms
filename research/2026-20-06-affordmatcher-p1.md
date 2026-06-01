---
last_modified_on: "2026-05-04"
title: AffordMatcher: Affordance Learning in 3D Scenes from Visual Signifiers
description: Semantic Matching, Affordance
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Affordance learning is a complex challenge in many applications, where existing approaches primarily focus on the geometric structures, visual knowledge, and affordance labels of objects to determine interactable regions. However, extending this learning capability to a scene is significantly more complicated, as incorporating object- and scene-level semantics is not straightforward. In this blog, we introduce our work **AffordBridge**, a large-scale dataset with $291,637$ functional interaction annotations across $685$ high-resolution indoor scenes in the form of point clouds. Our affordance annotations are complemented by RGB images that are linked to the same instances within the scenes. Building upon our dataset, we propose **AffordMatcher**, an affordance learning method that establishes coherent semantic correspondences between image-based and point cloud-based instances for keypoint matching, enabling a more precise identification of affordance regions based on cues, so-called visual signifiers.

## Introduction
Humans interact with the environment as part of their daily routines. Analyzing these interactions can provide valuable insights and is highly beneficial for both humans and robots to perform meaningful actions in the given environment. To better understand the interaction between humans and the environment, Gibson introduced the concept of *affordance*, referring to *opportunities for interaction*.  Yet, these opportunities need to be physically verifiable through successful actions to determine whether they are true *signifiers*; one point was later discussed by **Norman**. Therefore, learning about affordances requires not only predicting types of interactions but also correctly identifying specific points on objects that facilitate these human-object interactions. The concept of affordance from signifiers brings together perception and action, thus opening up applications in robotic manipulation, human-robot interaction, visual navigation, and augmented reality.

Still, the concept of **affordance** standalone is broad. Depending on the context, affordance learning is typically treated as a single-modality learner. Image-based affordance learning predicts the corresponding segmentation maps at the pixel level for intended actions. Meanwhile, spatial affordance learning segments desired affordance masks on object point clouds at the voxel level. The share of the two mentioned approaches leverages text prompts to direct affordance learning. However, the fusion of these two modalities remains unclear. Through this, the lack of multimodal representation learning for affordance conveys the idea of imposing cross-modal learning.

In practice, many challenges coexist in localizing spatial affordance from visual signifiers. First, cross-modal representations require overcoming significant discrepancies in feature distributions between images and point clouds. Second, matching affordance localization in the 3D domain and affordance detection in image space under diverse actions across different scenes entails the dexterity of designing such learning models. Additionally, language instructions, **press here** or **rotate knob**, are indeed semantically ambiguous without explicit geometric context, not to mention that real‐world scans can be further degraded by noises and occlusions, which complicate purely geometry-based methods. Last but not least, most existing datasets lack paired RGB images with annotated 3D affordance regions tied to interaction cues, precluding learning models from end‐to‐end training and evaluation. Without a unified solution to tackle these issues, creating an affordance bridge to localization affordances from visual signifiers is a utopian task. The central question is: **How can we learn representations that match diverse spatial affordances across different scenes from visual signifiers?**

To tackle this problem, we introduce a new benchmarking dataset, namely **AffordBridge**, with RGB image–point cloud affordance annotations, and propose a visual‐guidance reasoning affordance method, so-called **AffordMatcher**, that explicitly grounds visual signifiers into precise spatial affordances. As shown in Table 1, our large‑scale benchmark has $317,844$ high‑resolution paired samples of 2D-3D representations in $685$ indoor scenes, resulting in $291,637$ volumetric masks of functionally interactive elements among $157$ object categories through $61$ actionable affordances. Building upon this dual modality, we develop our affordance learning model, *AffordMatcher*, to semantically align keypoints in visual signifiers with those in point cloud instances, which accurately localize and segment interactable regions from both modalities, as shown in Figure 1. To summarize, our contributions are twofold, as follows: 
-  **Affordance Dataset:** We introduce **AffordBridge**, a large-scale dataset that annotates high-resolution point clouds and RGB images, alongside language-descriptive actions, for spatial affordance localization.
- **Affordance Learning:** We propose **AffordMatcher** for effectively matching affordance regions in point clouds derived from RGB images.



![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953234/introaff_njftu8.png)
*<center>**Figure 1**:  Overview of **AffordMatcher**.</center>*

In the next part, we will present the complete process of constructing the **AffordBridge **dataset.

</CodeExplanation>