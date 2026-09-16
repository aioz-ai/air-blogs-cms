---
last_modified_on: "2026-09-16"
title: Efficient Human-Contact Representation for Human-Scene Interaction
description: Lightweight network for efficient human-scene interaction
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
------------------------------------------------------------------

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Human-scene interaction (HSI) is an active research topic with applications in virtual reality, gaming, robotics, and surveillance. Despite significant progress in network architectures for improving prediction quality and reducing model complexity, efficiently representing contact between humans and their environments remains an open challenge.

In this paper, we propose a new efficient representation of human-scene contact. Our primary contribution is the introduction of **sparse contact masks** that selectively preserve essential contact information while reducing redundant data in high-dimensional inputs. Building on this representation, we propose a set of **sparse operators** that replace conventional dense operators within deep network layers, enabling faster computation.

Our approach improves both computational efficiency and interaction modeling by filtering out non-essential contact information. We evaluate the proposed method on three public benchmark datasets and two important HSI tasks: **contact prediction** and **scene synthesis**. Experimental results show that our approach outperforms state-of-the-art methods in reconstruction accuracy while achieving at least a **12× computation speed-up** over recent baselines.

## Introduction

Human-scene interaction studies how people perceive, navigate, and physically interact with their surrounding environments. Recently, increasing attention has been devoted to learning the relationships between human motion, body configuration, and scene geometry.

To better model human poses in diverse environments, researchers have explored problems such as **human-scene interaction**, **human-scene synthesis**, and **human pose contact prediction**. Understanding how humans interact with their surroundings is important for downstream applications including human-robot interaction, realistic virtual environments, game animation, and intuitive interfaces.

A major challenge in HSI is the computational cost of processing complex human and scene representations. Many existing approaches focus on generating high-quality scenes from human contacts and interactions. While increasingly sophisticated architectures can model these interactions effectively, they often require substantial computation and may be difficult to deploy in latency-sensitive applications.

To address this issue, previous studies have investigated lightweight architectures, model pruning, quantization, and other forms of architectural optimization. However, these techniques primarily reduce the complexity of the **network**, while the underlying interaction representation often remains dense.

More importantly, existing methods still need to process complex spatial-temporal interactions between human poses and surrounding objects. In many cases, however, only a small portion of the human body is actually involved in physical contact with the environment.

In this work, we take a different perspective. Instead of focusing only on making the network smaller, we focus on making the **interaction representation itself more efficient**.

We propose **E**fficient **CO**ntact Representation (**ECO**) for human-scene interaction. ECO uses a set of **sparse contact masks** to identify and preserve interaction-relevant information from dense human-scene representations (Figure 1). We then introduce corresponding **sparse operators** that replace conventional dense tensor operations in deep network layers.

The resulting representation reduces redundant computation while retaining the information necessary for accurate interaction modeling. Extensive experiments demonstrate that ECO achieves improved performance on both contact prediction and scene synthesis while providing substantially faster inference.

![Efficient contact representation for human-scene interaction](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944368/intro_kxsxck.png)

<center>

**Figure 1.** Efficient contact representation for human-scene interaction. Given dense input (a), ECO learns and selects useful information to construct an efficient contact representation (b), which is then used for contact prediction (c) and scene synthesis (d).

</center>

## Related Works

### Human-Scene Interaction Modeling

**Human-scene interaction (HSI)** focuses on understanding how people physically interact with objects and environments, including where contact occurs, how humans should be positioned, and how realistic interactions can be generated.

The development of parametric human body models such as **SMPL, SMPL-X, MANO, and FLAME** has provided a strong foundation for representing human bodies and their interactions with surrounding scenes.

Early studies investigated human affordances and plausible human poses from visual observations. The development of larger datasets, including **VirtualHome** and **BEHAVE**, subsequently enabled researchers to investigate human-object and human-scene interactions at larger scales using simulated environments and detailed real-world contact annotations.

Building on these resources, recent research has expanded into several directions, including:

* scene population,
* affordance learning,
* human-object and full-body interaction modeling,
* human-scene generation,
* diffusion-based scene synthesis, and
* interaction tracking.

Despite this progress, many existing approaches still represent human-scene interaction using dense representations.

In a typical human-body representation, all body vertices or features are processed even though only a relatively small subset may be involved in physical contact with the environment. For example, when a person sits on a chair, only regions around the **hips, legs, and torso** may be directly relevant to the interaction.

This observation suggests that conventional HSI representations contain substantial redundancy.

### Efficient Human-Scene Interaction

As HSI models become increasingly complex, computational efficiency has become an important consideration. Existing approaches to efficient deep learning generally focus on reducing the complexity of the neural network itself.

Common strategies include **network pruning, redundancy reduction, quantization, knowledge distillation, and neural architecture search**. Similar techniques have also been explored in applications such as trajectory prediction and dynamic scene generation.

These methods can reduce computation by simplifying the network architecture. However, they generally leave the original input representation unchanged. Consequently, redundant information from the human and scene representations is still propagated through the network, requiring computation on information that may be irrelevant to the current interaction.

This motivates a different question:

> **Instead of only making the network smaller, can we make the interaction representation itself more compact?**

Our work explores this direction by identifying the most informative contact regions before they are processed by the main network. In this way, computational efficiency is improved directly at the **representation level**, rather than relying solely on **architectural optimization**.

### Sparse Representations for Geometric Data

Sparsity provides a natural mechanism for reducing unnecessary computation in geometric data. Instead of processing every point, voxel, or feature, sparse representations retain only locations containing meaningful information.

Sparse tensors and sparse convolution operators have therefore become widely used in 3D vision and geometric learning, particularly for large-scale point clouds and voxelized environments.

Previous research has introduced **efficient sparse convolution operators, optimized sparse matrix computation, and compact geometric representations**. These approaches demonstrate that exploiting sparsity can substantially reduce memory usage and computational cost while retaining useful geometric information.

However, **geometric sparsity** and **interaction sparsity** are not necessarily equivalent.

A point may be geometrically important but irrelevant to the current human-scene interaction. Conversely, a small region containing only a few points may be critical because it corresponds to physical contact between the human and the scene.

This distinction is particularly important for HSI. Simply applying a generic sparse representation may remove geometrically sparse regions without considering whether those regions are important for the interaction.

### From Geometric Sparsity to Contact-Aware Sparsity

ECO extends conventional sparsity by explicitly modeling **semantic contact sparsity**. Rather than treating all human-body regions equally, ECO identifies the regions that are most informative for human-scene contact and represents them using **sparse contact masks** and **sparse operators**.

The key observation is that physical contact in human-scene interaction is inherently sparse. Only a small portion of the human body typically interacts directly with the environment, meaning that processing every body element equally introduces unnecessary redundancy.

By explicitly identifying and preserving these interaction-critical regions, ECO reduces the amount of information that must be processed while retaining the cues needed for accurate contact localization and scene understanding.

This provides a better balance between **computational efficiency** and **interaction modeling accuracy**.

The central idea behind ECO can therefore be summarized as follows:

> **If physical contact is naturally sparse, computation should focus on the interaction-critical regions rather than processing the entire human body equally.**

In the next section, we present the details of the ECO framework, which grounds the action probability of human-body vertices using sparse input representations.

</CodeExplanation>
