---
last_modified_on: "2021-09-30"
title: Motion generation.
description: Motion generation.
series_position: 36
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---


## Motion generation
Human motion generation involves creating high-quality 3D human motion based on given conditions, such as a text prompt, which generates Euler angles for each joint in a human skeleton. To emulate human motion effectively, virtual characters must respond to the conditional context, exhibit natural movement, and perform motion accurately. Several challenges may arise when solving this problem, such as the reliance on 3D data, which is often expensive to obtain.
### MAS: Multi-view Ancestral Sampling for 3D Motion Generation Using 2D Diffusion (CVPR 2024)
MAS attemtps to overcome the scarcity of 3D motion capture data by leveraging widely available 2D videos to synthesize realistic 3D motion sequences. It uses a 2D diffusion that is trained to predict the clean 2D motion at each denoising step. This training step is conducted with in-the-wild 2D videos scraped from the internet. 

When sampling, a noisy 3D motion is initialized, which is then projected on $V$ camera views, resulting in noisy $V$ 2D motions $x_t^{1:V}$. The trained 2D diffusion model will try to predict the clean 2D motions $\hat{x}_0^{1:V}$ from the noisy 2D motions. The final 3D motion $X$ is reconstructed from multiple clean 2D motions by optimizing the difference between projections of $X$ to all views $\tilde{x}_0^{1:V} = P(X, 1), P(X, 2),...P(X, V)$ and the multiview motion predictions $\hat{x}_0^{1:V}$. 
$$
X=\arg\min_{X^{\prime}}\|P\left(X^{\prime},1{:}V\right)-\hat{x}_0^{1:V}\|_2^2= 
\arg\min_{X^{\prime}}\sum_{v=1}^V\lVert P\left(X^{\prime},v\right)-\hat{x}_0^v\rVert_2^2
$$

## Human object interaction generation
Scene-aware motion generation aims to synthesize 3D human motion given a 3D scene model to enable virtual humans to naturally wander around scenes and interact with objects, which has a variety of applications in AR/VR, filmmaking, and video games.
### Hierarchical Generation of Human-Object Interactions with Diffusion Probabilistic Models (ICCV 2023)
Given the starting position and the target object, the model tries to generate the motion of the human interacting with the object in the scene.
![fig1](https://vision.aioz.io/thumbnail/a18a0a7374894578920e/1024/image-12.png)
*<center>Fig 1: Overview framework of HGHOI</center>*
Instead of generating a long motion, the method first predicts the goal pose that reflects how the human would interact with the object. Using this goal pose as the condition, a milestone generation module which is a transformer DDPM will estimate the length of the milestones, produces the trajectory of the milestones from the starting point to the goal, and places milestone pose. Finally, DDPM process is employed to fill in the in-between motions between the milestones. 

### InterDiff: Generating 3D Human-Object Interactions with Physics-Informed Diffusion (ICCV 2023)
InterDiff aims at generating human object interacion in which the object is non-static. 
![fig2](https://vision.aioz.io/thumbnail/29e047ae3215462792a5/1024/image-13.png)
*<center>Fig 2: Overview framework of InterDiff</center>*

After predicting the human-object interaction (HOI) $\tilde{x}$ in the reverse diffusion step, an extra interaction correction step will be applied if there happens penetration or floating. In this step, a spatialtemporal graph neural network (STGNN) specifically predicts the motion of the object alone $\hat{x}$ in a new reference system where the origin aligns with the contact point between the human and the object. The information on the object motion after transformed back to the ground reference system will be blended with HOI through a blending function: 
$$\tilde{\boldsymbol{x}}\leftarrow\tilde{\boldsymbol{x}}\times\frac{t}{T}+\hat{\boldsymbol{x}}\times(1-\frac{t}{T})$$


This way, the model can mitigate floating and penetration issue. 