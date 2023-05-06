---
last_modified_on: "2023-05-01"
title: Populating 3D Scenes by Learning Human Scene Interaction (Part 1).
description:  Introduction to Learning Human Scene Interaction
series_position: 18
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

People constantly interact with their surroundings, and such interactions have semantics, specifically as combinations of actions and object instances. These interactions are becoming more and more diverse, and capture meaningful semantics. Understanding the interaction is crucial for developing intelligent systems that can effectively interact with humans in various contexts, including virtual environments, robotics, and human-computer interfaces. Moreover, through the semantics of these interactions, researchers can further understand how humans contact with difference environments with their pose and body, and capture meaningful scene semantics.


## Introduction
Human constantly interact with 3D space and such interactions involve physical contact between surfaces that is semantically meaningful. Thus, it is important to learn how humans interact with scenes and study their applications.

Despite the importance of the interactions, existing representations of the human body do not explicitly represent, support, or capture them.

SMPL-X model [[1]](#1)  can represent the shape and pose of people. Moreover, this representation includes hand and face, and  it supports reasoning about contact between the body and the world. However, some challanges still remain: 

- SMPL-X does not explicitly model contact .
- Not all parts of the body surface are equally likely to be in contact with the scene.
- The poses of body and scene semantics are highly intertwined.

POSA [[2]](#2) (**P**ose with pr**O**ximitie**S** and cont**A**cts) leverages SMPL-X to capture contact and the semantics of Human-Scene Interactions (HSI) in a body-centric representation. POSA aims to solve challenging problems: 
- Automatic scene population:  Given a 3D scene and a body in a particular pose, where in the scene is this pose most likely?
- Monocular 3D human pose estimation in a 3D scene.

## Dataset 

**Training data:** PROX-E [[3]](#3) dataset was used for this task. It is a set of n pairs of 3D meshes

$$

\mathcal{M} = \left \{ \left \{ M_{b,1}, M_{s,1} \right \}, \left \{M_{b,2}, M_{s,2}\right \},... , \left \{M_{b,n}, M_{s,n}\right \} \right \}

$$

comprising body meshes $M_{b,i}$ and scene meshes $M_{s,i}$ and $i$ is the index of $\mathcal{M}$. 

- $M_b= (V_b, F_b)$: body mesh which has a fixed topology with $N_b = |V_b| = 10475$ vertices $V_b \in \mathbb{R}^{N_b \times 3 }$ and body mesh faces $F_b$ .
- $M_s = (V_s, F_s, L_s)$: scene mesh which has a varying number of vertices $N_s = |V_s|$, triangle connectivity $F_s$ to model arbitrary scenes, and per-vertex semantic labels $L_s$ (e.g chair, bed, sofa,...).

![Fig-1](
https://drive.google.com/uc?id=10_3cRVZgZWUEZefoJ8ZCgxHBcr6WorMj)
*<center>**Figure 1**:   PROX-E Dataset.</center>*

Human meshes are represented by SMPL-X model, i.e. a differentiable function $M(\theta, \beta, \psi) : \mathbb{R}^{|\theta| \times |\beta| \times |\psi|} → \mathbb{R}^{N_b \times 3}$ parameterized by pose $\theta$, shape $\beta$ and facial expressions $\psi$.
- The pose vector $\theta = (\theta_b, \theta_f , \theta_{lh}, \theta_{rh})$ is comprised of body $\theta_b \in \mathbb{R}^{66}$ ,  face parameters $\theta_f \in \mathbb{R}^9$, in axis-angle representation, and $\theta_{lh}, \theta_{rh} \in \mathbb{R}^{12}$ which parameterize the poses of the left and right hands respectively in a low-dimensional pose space.
- The shape parameters, $\beta \in \mathbb{R}^{10}$, represent coefficients in a low-dimensional shape space learned from a large corpus of human body scans. 

- The joints, $J(\beta)$, of the body in the canonical pose are regressed from the body shape.

## Methodology: POSA Representation for HSI 

POSA encodes the relationship between the human mesh $M_b$ and the scene mesh $M_s$ in an **egocentric** feature map $f$ that encodes per-vertex features on the SMPL-X mesh $M_b$: 

$$f : (V_b, M_s) \rightarrow [f_c, f_s] $$
where $f_c$ is the contact label, $f_s$ is the semantic label of the contact point and $N_f$ is the feature dimension.

![Fig-2](
https://drive.google.com/uc?id=1Hij0mFT0QhvglQuNHA6tOQjPdveaB2Qm)
*<center>**Figure 2**:   Illustration of the proposed representation.</center>*

For each vertex $V_b^i$ on the body, find its closest scene point:

$$

P_s = \underset{P_s \in \mathcal{S}_s}{\operatorname{argmin}}|| P_s − V_b^i ||

$$


The distance $f_d$ is calculated:

$$

f_d =|| P_s − V_b^i || \in \mathbb{R}

$$

Given $f_d$, determine whether vertex $V_b^i$ is in contact with the scene or not by comparing $f_d$ with a constant threshold : 

$$

f_c = \left\{\begin{matrix}
1\ \  if \ \ f_d \leq \text{Constant Threshold} \\ 
0\ \  if \ \ f_d > \text{Constant Threshold}
\end{matrix}\right.

$$

The semantic label of the contacted surface fs is a one-hot encoding of the object class:
$$

f_s = \left \{0, 1 \right \}^{N_o}

$$

where $N_o$ is the number of object classes.

<!-- ![Fig-1](
https://drive.google.com/uc?id=1Hij0mFT0QhvglQuNHA6tOQjPdveaB2Qm)
*<center>**Figure 1**:   Illustration of the proposed representation.</center>* -->


## References
<!-- smpl-x -->
<a id="1">[1]</a> 
Georgios Pavlakos, Vasileios Choutas, Nima Ghorbani, Timo Bolkart, Ahmed A. A. Osman, Dimitrios Tzionas, and Michael J. Black. Expressive body capture: 3D hands, face, and body from a single image. In Computer Vision and Pattern Recognition (CVPR), 2019


<!-- posa -->
<a id="2">[2]</a> 
Mohamed Hassan, Partha Ghosh, Joachim Tesch, Dimitrios Tzionas, and Michael J.Black. 2021b. Populating 3D Scenes by Learning Human-Scene Interaction. In Conference on Computer Vision and Pattern Recognition (CVPR).



<!-- spiral_conv -->
<a id="3">[3]</a> 
Shunwang Gong, Lei Chen, Michael Bronstein, and Stefanos
Zafeiriou. SpiralNet++: A fast and highly efficient mesh
convolution operator. In International Conference on Computer Vision Workshops (ICCVw), 20

<!-- 
prox-e dataset -->
<a id="4">[4]</a> 
Mohamed Hassan, Vasileios Choutas, Dimitrios Tzionas, and Michael J. Black. Resolving 3D human pose ambiguities with 3D scene constrains. In International Conference on Computer Vision (ICCV), 2019.