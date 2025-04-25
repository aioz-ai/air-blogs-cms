---
last_modified_on: "2021-09-30"
title: Camera pose prediction in 3D dancing.
description: Camera pose prediction in 3D dancing.
series_position: 35
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

## Auto-regressive approaches

### Duolando: FOLLOWER GPT WITH OFF-POLICY REINFORCEMENT LEARNING FOR DANCE ACCOMPANIMENT (ICLR 2024)
Duolando focuses on duet dancing i.e., a follower dancer needs to respond to the given movements of the leader while synchronizing with music. Unlike solo or group dance, duet dancing requires both precise coordination in pose and position. A new dataset tailored to Duet Dancing DD100 was introduced. 
![duolandovqvae](https://vision.aioz.io/thumbnail/dc84d5fc4dd24d5995e0/1024/image-8.png)

Similar to Bailando, Duolando also uses VQ-VAE to embed and quantize movements into a discrete space. Two codebooks were constructed, one for body parts (upper, lower body and hands) and the other for relative translation (the positioning of the follwer relative to the leader). An interaction-coordinated GPT will autoregressively predict the next token, conditioned on the amalgamated information of the music signal, the leader’s motion, and the previous follower sequence. To further tackle skating artifacts (unrealistic foot movements), off-policy RL strategy was employed. It penalizes when inconsistencies between global position shifts and footwork velocity happen. 

*Limits:*
- GPT does not generalize well on out-of-distribution cases without RL. 
- VQ-VAEs only capture hand and body gestures while neglecting more fine-grained expressiveness (facial expression).
- Costly computation during inference.

### ENHANCING EXPRESSIVENESS IN DANCE GENERATION VIA INTEGRATING FREQUENCY AND MUSIC STYLE INFORMATION (ExpressiveBailando) (ICASSP 2024)
This work applies the same paradigm as Bailando with some improvements. Instead of using the regular VQ-VAE, authors introduce FreqVQ-VAE, which adds Focal Frequency Loss in their training objective to mitigate speed homogenization i.e., lack of speed variation. The focal frequency loss indicates the weighted average frequency distance between the real and generated dance movements. It focuses the model on synthesizing hard frequencies by down-weighting easy frequencies.
![XBailando](https://vision.aioz.io/thumbnail/ec496f9140a14da0ab25/1024/image-9.png)
*<center>Fig 2: Overall framework of ExpressiveBailando</center>*
*Strengths:*
- Improve the quality of reconstructed dance movements thanks to the integration of focal frequency loss.

*Limits:*
- Same as Bailando.

## Bonus: Camera pose prediction in 3D dancing
### DanceCamera3D: 3D Camera Movement Synthesis with Music and Dance (CVPR 2024)
This work introduces a brand new task in the realm of 3d dancing. Instead of predicting dance movements given a piece of music, DanceCamera3D tries to predict the camera pose given music and 3D dance movements. 
![DCM](https://vision.aioz.io/thumbnail/d3fe4e6fc93d45838754/1024/image-10.png)
*<center>Fig 3: DCM dataset includes music, dance pose and camera pose</center>*
A new dataset named DCM was constructed to tackle this task. The dataset not only includes dance motion data (in VMD format) but also information on camera pose (i.e., camera position in 3D dimension, rotation and field of view). 

The work also presented a baseline for this task. Diffusion framework was employed to synthesize the camera pose conditioned on Dance pose and music. 
![DanceCamera3D](https://vision.aioz.io/thumbnail/25553b5eb6d34930bd11/1024/image-11.png)
*<center>Fig 4: Overall framework for DanceCamera3D</center>*
