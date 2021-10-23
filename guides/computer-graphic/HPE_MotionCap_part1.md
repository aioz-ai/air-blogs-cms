---
last_modified_on: "2021-10-25"
title: Human Pose & Motion Capture exploration (part 1) Defenition and 2D Human Pose Estimation.
description:  explore human pose estimation and motion capturing. 
series_position: 1
author_github: https://github.com/minhnhatvt
tags: ["type: insight", "level: medium"]
---

**Human Pose Estimation (HPE)** is a crucial task in computer vision to study and understand human behavior and interaction in the real world. HPE provides geometric and motion information of the human body which is important for various applications such as *Human-computer interaction, Augmented reality (AR), Virtual reality (VR), Healthcare and Character Animation.* Additionally, motion capture is the key component to present characters with complex movements and realistic physical interactions in industries of animation, movie, and gaming. Nevertheless, The motion capture devices are usually expensive and complicated to set up.

Fortunately, the recent developments of deep learning techniques have made signiﬁcant progress and remarkable breakthroughs in HPE. This allows us to build a highly accurate system, even only with a single cheap RGB camera, while alleviating the demand for professional high-cost equipment. Although there are many works have utilized extra sensors such as depth sensor, inertial measurement units (IMUs), but these approaches are usually not cost-effective and require special setup. In this blog, we explore and summarize the overview of a monocular Human Pose Estimation system, which only requires RGB images as input.

In this part, we first explore the defenition of pose estimation and the 2D human pose estimation task (2D HPE).

## Monocular Human Pose Estimation 
**Problem definition**: Given an *input RGB image* (or video) containing persons. The goal of HPE is to: 
* **2D Pose estimation**: predict 2D spatial coordinates $(x,y)$ for each joint or keypoint (typically 11-23 joints per human) of each person. 
* **3D Pose estimation**: predict 3D coordinates $(x,y,z)$ for each joint.

![Figure-1](https://i.imgur.com/4ToTAUR.png)*<center>**Figure 1**: Human Pose Estimation</center>*

## 2D Human Pose Estimation (2D HPE)
Existing 2D HPE methods are categorized into *Single-person* and *Multi-person* HPE. In single-person, it is further divided into *Regression-based* approach and *Detection-based* approach. In multi-person, we consider the *Top-down* approach and *Bottom-up* approach. 

* **Single-person**
    * Localize human body joint positions when the input is a single-person image.
    * If there are more than one person, the input image is cropped so that there is only one person in each cropped patch.
    * Deep learning techniques for single-person are classified into two categories
        1. **Regression-based methods**: directly map the input image to the 2D coordinates of body joints
        2. **Detection-based methods**: Most recent works treat the body parts as detection targets based on Heatmap.
* **Multi-person**
    * Is more difﬁcult than single-person. 
    * Need to ﬁgure out people and keypoint location, then assign keypoints for different people.
    * Deep learning techniques for multi-person are classified into two categories:
        1. **Top-down methods**: First detect person, then apply single-person pose estimation for each person
        2. **Bottom-up methods**: Jointly estimate pose for people all at once, then group them to the corresponding subjects.
        

### Single-person 2D HPE - Summary

![Figure-2](https://i.imgur.com/n4u7lsX.png)*<center>**Figure 2**: Single-person 2D HPE [(Source)](https://arxiv.org/abs/2012.13392)</center>*

![Table-1](https://i.imgur.com/rwDdT9a.png)*<center>**Table 1**: Single-person 2D HPE methods [(Source)](https://arxiv.org/abs/2006.01423)</center>*

* **Regression-based**	
    * Learn a nonlinear mapping from input images to keypoint coordinates with an end-to-end framework.
    * Easy to implement & training, sub-pixel level prediction accuracy.
    * However, they usually give sub-optimal performance due to the difficulty of highly non-linear problem.
* **Detection-based**
    * Heatmap representations are widely use in recent works
    * Provide richer supervision information by preserving the spatial information and probabilistic prediction.
    * Give better & more robustness results than regression-based methods.
    * However, the performance is dependent on the resolution of heatmaps.
    * High resolution heatmaps require more memory & computational cost while low resolution limit the accuracy.
    * Obtaining joint coordinates from heatmap is normally a non-diﬀerentiable process that prevent the network from end-to-end training. 
    * Luvizon *et. al* proposes a soft-argmax function to convert heatmap into 2D coordinates in a differentiable way, which can be trained end-to-end with regression loss. [[Paper]](https://arxiv.org/pdf/1710.02322.pdf)

### Multi-person 2D HPE - Summary
![Figure-3](https://i.imgur.com/uxb6hSE.png)*<center>**Figure 3**: Multi-person 2D HPE [(Source)](https://arxiv.org/abs/2012.13392)</center>*

![Table-2](https://i.imgur.com/JKRQ36Y.png)*<center>**Table 2**: Multi-person 2D HPE Methods [(Source)](https://arxiv.org/abs/2006.01423)</center>*

* **Top-down**	
    * Gives better results by break down the problem into person detection and single-person pose estimation.
    * The predicted poses heavily depend on the accuracy of the person detection.
    * Slow inference time, usually grows proportionally with the number of person.
* **Bottom-up**
    * Directly and jointly detect all the keypoints and group them into individual poses using keypoint association strategies.
    * Correct grouping of joint points in a complex environment is a challenging task.
    * Faster than top-down methods (some methods can run in real-time such as OpenPose).
    * Typically less accurate results than top-down methods.
* **There exist some challenges in 2D HPE**
    * Occlusion (self-occlusion, object-occlusion, crowded scene, overlapped person).
    * Limited data of rare poses (uncommon activities, infant pose), which makes existing methods are not generalized well to such poses.

### References
1. Chen *et al.* Monocular Human Pose Estimation: A Survey of Deep Learning-based Methods, 2019
2. Zheng *et al.* Deep Learning-Based Human Pose Estimation: A Survey, 2020
