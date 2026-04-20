---
last_modified_on: "2026-03-28"
title: AeroScene: Progressive Scene Synthesis for Aerial Robotics
description: Scene Generation, Aerial Robotics, Diffusion model
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


Generative models have shown substantial impact across multiple domains, their potential for scene synthesis remains underexplored in robotics. This gap is more evident in drone simulators, where simulation environments still rely heavily on manual efforts, which are time-consuming to create and difficult to scale. In this blog, we introduce **AeroScene**, a hierarchical diffusion model for progressive 3D scene synthesis. Our approach leverages hierarchy-aware tokenization and multi-branch feature extraction to reason across both global layouts and local details, ensuring physical plausibility and semantic consistency. This makes **AeroScene** particularly suited for generating realistic scenes for aerial robotics tasks such as navigation, landing, and perching. We demonstrate its effectiveness through extensive experiments on our newly collected dataset and a public benchmark, showing that AeroScene significantly outperforms prior methods. Furthermore, we use AeroScene to generate a large-scale dataset of over 1000 physics-ready, high fidelity 3D scenes that can be directly integrated into **NVIDIA Isaac Sim**. Finally, we illustrate the utility of these generated environments on downstream drone navigation tasks. 

## Introduction

Drones are increasingly applied in delivery, inspection, and surveillance, requiring them to navigate and operate within complex 3D environments. Synthesizing realistic scenes for such applications necessitates hierarchical layout generation, where coarse-scale structures (e.g., rooms, terrains, building layouts) establish navigable flight corridors, while fine-scale details (e.g., obstacles, landing areas) ensure task-specific feasibility. However, existing scene creation methods for drone simulators are designed primarily by humans, making them challenging to scale, and often fail to accommodate both physical and fidelity requirements such as unobstructed aerial navigation, accessible interaction areas (e.g., landing pads, inspection points), and coherent indoor-outdoor transitions.


While several drone simulators have been developed to support research in navigation and control, most remain limited in providing diverse and realistic environments for evaluating aerial tasks. Many simulators focus on accurate physics and sensor fidelity, yet often rely on static, handcrafted environments, hindering scalability, diversity, and the realism necessary for advanced testing. Moreover, interaction areas critical to aerial robotics, such as landing zones, cluttered corridors, and inspection surfaces, are frequently simplified or entirely absent, limiting full task realism. As a result, existing platforms offer limited support for benchmarking higher-level autonomy, where navigation, task execution, and environment understanding must be jointly evaluated in realistic, hierarchical settings.


In this blog, we introduce **AeroScene**, a hierarchical diffusion-based framework for 3D scene generation tailored to drone tasks. Our approach operates across scales: coarse-scale synthesis generates high-level structures that preserve airspace and navigability, while fine-scale synthesis refines object placement and specifies drone-interaction areas for task execution. AeroScene includes *Cross-scale Progressive Attention*, which explicitly models dependencies across scales, ensuring that fine-scale details remain consistent with coarse-scale spatial structures. In addition, we design task-aware guidance functions that encourage collision-free plausibility, maintain semantic correlations in hierarchical orders, and handle relationships between indoor and outdoor objects, thereby aligning generated layouts with real-world aerial operation requirements. To support downstream tasks, scenes created by our method are directly embedded into **NVIDIA Isaac Sim** for physics-ready simulation. 


Our main contributions are as follows:
- We introduce a new framework that generates realistic and high-fidelity scenes for aerial robotics. 
- We contribute a large-scale dataset with more than 1000 scenes and embed them into Isaac Sim to serve as a benchmark for drone-related tasks.


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774623930/intro_puhygs.png)
*<center>**Figure 1**:  We introduce AeroScene, a progressive scene synthesis method and dataset for aerial robotics.</center>*

In the next part, we will discuss the method for generating realistic hierarchical scenes that incorporate both indoor and outdoor objects.

</CodeExplanation>