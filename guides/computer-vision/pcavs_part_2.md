---
last_modified_on: "2021-10-25"
title: Pose-Controllable Talking Face Generation (part 2)
description: the Pose-Controllable Audio-Visual System (PC-AVS) using Implicitly Modularized Audio-Visual Representation.
series_position: 5
author_github: https://github.com/phamtrongthang123
tags: ["type: tutorial", "level: medium"]
---

In the previous part, we have discussed Pose-Controllable Talking Face Generation, its challenges, and the general solution. In this part, we explain detailly the state-of-the-art solution for the mentioned task.

## Pose-Controllable Talking Face Generation by Implicitly Modularized Audio-Visual Representation (PC-AVS).

![method.png](https://github.com/Hangz-nju-cuhk/Talking-Face_PC-AVS/blob/main/misc/method.png?raw=true)
*<center>**Figure 3**: **The overall details pipeline of the Pose-Controllable Audio-Visual System(PC-AVS) framework**.</center>*

The detailed architecture of PC-AVS is shown in **Figure 3**. The below sections describe the ideas behind those changes:
 - Pretraining to help the training phase.
 - Identity and non-identity space.
 - Speech Content Space and Pose Code.
 - Generator.

## Pretraining to help training phase

To create the input for the generator, we need identity space, pose code, and speech content space. But raw encoder may not work very well, so they pre-train first before learning to reconstruct the target frame. Specifically,  

- Identity encoder $E_i$: pretrained with classification task.
- Non-identity encoder $E_n$: pretrained with visual-driven task. In this work, the author modified the Face Alignment Network (FAN) model (which is proposed for 2D to 3D face alignment task).
- Visual encoder ($E_n \rightarrow mp_c$) and audio encoder: pretrained with contrastive loss $\mathcal{L}_{c}$.

## Identity and non-identity space

For identity space, the authors follow the common practice, they use a pretrained model in classification task to get this space. For non-identity space, the authors augment the pose source then forward it into a non-identity encoder. The data augmentation step is the key idea to produce non-identity space. In the reconstruction training phase, they need to use the first frame of the pose source video as an identity input image. Hence, they need to transform the pose source such that they do not look like the identity frame. So they use the idea that if they augment the pose video frames enough to make the difference. After encoding, the output latent space will contain only non-identity information. The authors in [1] chose $2$ augment methods to destroy $2$ major aspects (image texture and facial information) which 1 additional augmentation. Through empirical results, the authors show that without data augmentation, the model fails to learn pose.

Unlike identity space, which is only responsible for identity information, non-identity space is leveraged to learn both pose code and speech content space.

##  Speech Content Space and Pose Code

Because non-identity space contains information about speech content space and pose space. They use $2$ FC layers to split the information for each space.

### Speech Content Space

The first FC layers will produce visual information $F_c^v$. With the audio information $F_c^a$ encoded by $E_c^a$ as the input, the model is able to learn to sync the audio and visual. The sync space is speech content space. Later, they use only $F_c^a$ to represent speech content space for reconstruction training.

- To sync the audio and visual, they use a contrastive learning method. The only positive sample is where $F_c^a$ sync with $F_c^v$ (timely aligned). Other negative samples can be drawn from different videos or time shifts from the same source. After that, they transform the task into a classification task with negative log-likelihood loss. Their target is to max the cosine score of $F_c^a$ and $F_c^v$.
- The speech content module is pretrained on video and audio encoding pose task first with a contrastive loss $\mathcal{L}_{c}$ before finetuning in reconstruction task.

### Pose Code

The second FC layers will produce a $12$-dim pose code $F_p$ to express 3D head pose information. Our pose space is the proxy between the Ground Truth pose source and the generated frame.

### Generator

After we have got 3 separated spaces:

- Speech content space $F_c^a$.
- Pose code $F_p$.
- Identity $F_i$.

At frame $k$, we concatenate them and use modulated convolution to generate the target frame. Modulated convolution is chosen because it has been proven to be effective in image-to-image translation [2]. Besides, they cannot apply the generator that leverages skip connection because of the model's structure (the pose information is latent space so skip connection may restrict the possibility of altering poses).

## References
[1] H. Zhou, Y. Sun, W. Wu, C. C. Loy, X. Wang, and Z. Liu, "Pose-Controllable Talking Face Generation by Implicitly Modularized Audio-Visual Representation", *arXiv:2104.11116 [cs, eess]*, Apr. 2021, Accessed: Sep. 13, 2021. [Paper](http://arxiv.org/abs/2104.11116)

[2] T. Park, A. A. Efros, R. Zhang, and J.-Y. Zhu, "Contrastive Learning for Unpaired Image-to-Image Translation", *arXiv:2007.15651 [cs]*, Aug. 2020, Accessed: Sep. 17, 2021. [Paper](http://arxiv.org/abs/2007.15651)
