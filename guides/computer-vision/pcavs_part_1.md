---
last_modified_on: "2021-10-17"
title: Pose-Controllable Talking Face Generation (part 1)
description: An overview for the Pose-Controllable Audio-Visual System (PC-AVS) framework.
series_position: 4
author_github: https://github.com/phamtrongthang123
tags: ["type: tutorial", "level: basic"]
---

Accurate lip synchronization has been achieved for arbitrary-subject audio-driven talking face generation, the problem of how to efficiently drive the head pose remains. Previous methods rely on pre-estimated structural information, under extreme conditions, would lead to degradation problems.

## Pose-Controllable Talking Face Generation: Introduction
Pose-Controllable Talking Face Generation is a task where agents try to achieve accurately lip-synced talking faces whose poses are controllable by other videos. The authors of [1] address the problem by defining a typical framework to achieve the goal without using pre-estimated structural information as pose information.

The inputs and outputs of the mentioned task are described as below:
**Input**: 
- Identity reference image $I$.
- Pose source $V$.
- Audio $A$.

**Output**:
-  Talking faces with lip-syncing and head pose.

Figure 1 illustrates the general framework for Pose-Controllable Talking Face Generation.

![image-20210917160302569](https://i.imgur.com/5mjb8rJ.png)

*<center>**Figure 1**: **Illustration of Pose-Controllable Audio-Visual System (PC-AVS)**. The framework takes a reference image as identity and uses audio (from the synced video) to generate a talking face, with a generated pose is controlled by the pose source.</center>*

## Challenges
We have two questions that need to be solved: 
- The first question is how they are gonna fuse the given 3 information?
- The second question is how they are gonna produce the pose information without explicit 3D structural information?

If we do not have any direction, blindly following the encode-decode pipeline, i.e., encode {image, pose, audio} -> concatenate encoded results -> decode, almost does not help us achieve good results. Even if it works, we cannot explain how.

## Solutions
So to solve the aforementioned two questions, the authors [1] try to integrate implicit low-dimension pose code into the typical framework as learnable complementary information. Specifically,

- To generate the target frame, intuitively we need 3 input modalities in a target frame face, including: pose, mouth/lip-synced, identity face. 
- Pose code is free of mouth shape or identity information. We need an identity encoder module to create identity space which is further used to represent the identity face.
- The non-identity space contains some information about the remaining features, including pose and speech. To produce non-identity space, we use a non-identity encoder module.
- With a non-identity space input, we can produce visual information by using a non-identity-to-visual module. 
- We also need an audio encoder module to produce an audio feature. 
- Synchronization module aims to synchronize visual and audio and produce speech content space. Note that, the speech content space is responsible for lip-sync
- The pose code can be learned implicitly from the target frame's generation (reconstruction) loss through a non-identity-to-pose module.
- After we have identity space, pose code, and speech content space, we then use modulated convolution to generate the target frame as the decoder. The overall pipeline from our basic ideas is shown in **Figure 2**.

![basic idea](https://i.imgur.com/35jTdlV.jpg)
*<center>**Figure 2**: **The overall pipeline for Pose-Controllable Talking Face Generation**. </center>*

## References

[1] H. Zhou, Y. Sun, W. Wu, C. C. Loy, X. Wang, and Z. Liu, “Pose-Controllable Talking Face Generation by Implicitly Modularized Audio-Visual Representation,” *arXiv:2104.11116 [cs, eess]*, Apr. 2021, Accessed: Sep. 13, 2021. [Online]. Available: http://arxiv.org/abs/2104.11116