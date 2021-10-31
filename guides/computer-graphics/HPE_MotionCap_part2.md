---
last_modified_on: "2021-10-31"
title: Human Pose & Motion Capture exploration (part 2).
description:  explore human pose estimation and motion capturing in 3D. 
series_position: 2
author_github: https://github.com/minhnhatvt
tags: ["type: tutorial", "level: medium"]
---

In previous post, we have investigated the monocular pose estimation and 2D human pose estimation process. In this part, we explore human modelling, 3D pose estimation, and multi-person 3D pose estimation.

## 3D Human Pose Estimation (3D HPE)
Same as 2D HPE, works on 3D HPE are also categorized into *Single-person* and *Multi-person*. In single-person, it is further divided into *Model-free*  and *Model-based* methods. In multi-person, recent works are also grouped into two categories *Top-down* approach and *Bottom-up* approach. 

* **Single-person**
    * **Model-free:** Predict 3D joint coordinates without using human body models
        * **Direct mapping:** Directly infer the 3D coordinates of body joints end-to-end
        * **Lifting-based:** Infer 3D pose by using the intermediately estimated 2D pose.
    * **Model-based:** Incorporate parametric body models (such as skeleton kinematic or body mesh) to estimate human pose & shape.
* **Multi-person:** This research field is pretty new and only a few methods are proposed
    * **Top-down methods:** Same as 2D HPE, first detect person, then apply single-person 3D pose estimation for each person to predict:
        * Absolute root (center joint) 3D coordinate (relative to the world coordinate)
        * Root-relative 3D coordinate for other joints .
        * Based on these coordinates, all poses of different persons are aligned to the world coordinate.
    * **Bottom-up methods:** we first produce all body joint locations and depth maps, then associate body parts to each person according to the root depth and part relative depth.

## Human Body Models
We explore different human body models that are commonly used in recent works.
![Figure-4](https://i.imgur.com/CXMI5bO.png)*<center>**Figure 4:** Human body models</center>*

* **Kinematics model**
    * Also called  skeleton-based  model. 
    * Includes a set of joint positions and the limb orientations (joint angle) to represent the human body structure.
    * Simple and flexible skeleton graph-representation. Which is very effective for capturing motion.
    * Lacking texture and shape information of the human body.
* **Body mesh (volumetric) model**
    * Represent 3D human body poses and shapes by a volumetric mesh (set of 3D points).
    * Complex and detailed to represent human body (by skin).
    * One of the most widely use is Skinned Multi-Person Linear **(SMPL)** model. It is modeled as
        * A function parameterized by two separate pose and shape low-dimensional vector.
        * Output a set of ~7k 3D points (vertices) as body mesh.
        * Can represent a broad range of realistic human pose and shape.
        * It is easy to deploy and compatible with many rendering engines, therefore is widely used in 3D HPE.


## Single-person 3D HPE - Summary

![Figure-5](https://i.imgur.com/KMJ9U9w.png)*<center>**Figure 5:** Single-person 3D HPE [(Source)](https://arxiv.org/abs/2012.13392)</center>*

![Table-3](https://i.imgur.com/MOJRptC.png)

![Table-3](https://i.imgur.com/bUjq2tM.png)*<center>**Table 3:** Single-person 3D HPE methods [(Source)](https://arxiv.org/abs/2006.01423)</center>*

* **Model-Free** Do not employ human body models
    * **Direct mapping**
        * Usually give sub-optimal performance due to the difficulty of highly non-linear problem.
        * Insufficient training data
    * **Lifting-based**
        * Benefit from the state-of-the-art 2D Pose detector (the amount of training data of 2D pose is significantly larger than 3D pose).
        * Generally outperform direct estimation approaches
        * Some works incorporate body structure prior (e.g. skeleton graph) or temporal information to capture realistic motion.
* **Model-based**
    * Many methods leverage prior of human body model (e.g. joint connectivity and rotation constraints) for plausible pose estimation.
    * Volumetric models can recover high-quality human mesh, providing extra shape information of human body.
    * Can be high complexity.
* 3D HPE datasets are usually collected from controlled environments with selected daily motions. It is difﬁcult to obtain the 3D pose annotations for in-the-wild data.

## Multi-person 3D HPE - Summary
![Figure-6](https://i.imgur.com/sV9oAMM.png)*<center>**Figure 6:** Multi-person 3D HPE [(Source)](https://arxiv.org/abs/2012.13392)</center>*

![Table-4](https://i.imgur.com/R5YtJFH.png)*<center>**Table 4:** Multi-person 3D HPE Methods [(Source)](https://arxiv.org/abs/2006.01423)</center>*

* **Top-down**
    * Usually achieve promising results by relying on the SOTA person detector and single-person pose estimation methods.
    * After detecting each individual person, human body mesh of each person can be easily recovered
    * Computational complexity and the inference time is expensive, increases with the number of person.
    * Global information in the scene might be ignored, since it crops out the person bounding box.
* **Bottom-up**
    * Faster than top-down method
    * A key challenge is how to group human body joints belonging to each person.
    * If the goal is to recover 3D body mesh, it is not straightforward to reconstruct human body meshes 
* **Challenges**
    * Occlusion.
    * Insufficient training data, existing datasets are mainly captured in constrained scenes.
    * Computational efﬁciency.
    * Depth ambiguities, different 3D human poses can be projected to a similar 2D pose projection.

## References
1. Chen *et al.* Monocular Human Pose Estimation: A Survey of Deep Learning-based Methods, 2019
2. Zheng *et al.* Deep Learning-Based Human Pose Estimation: A Survey, 2020