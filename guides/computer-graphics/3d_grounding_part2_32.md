---
last_modified_on: "2024-09-18"
title: 3D Downstream Tasks for Foundation Model (Part 2).
description: 3D Visual Grounding
series_position: 32
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import Highlight from '@site/src/components/Highlight';



In the previous section, we introduced the 3D VG task and the model architecture designed to tackle it. Now, we will explore the implementation of recent state-of-the-art methods, along with the datasets and metrics used to evaluate their effectiveness.

## Method

Most implemented methods in 3D VG follow the fully supervised learning strategy. Early and most recent methods follow the two-stage framework and utilize the pre-trained 3D object detector. However, there exist works that follow the one-stage framework to avoid dependence on the quality of the 3D detector.

**3D-SPS.** Luo et al. introduce a groundbreaking grounding model called 3D-SPS (**S**ingle-stage 3d visual grounding via referred point **p**rogressive **s**election - CVPR 2022), which directly regresses the bounding box of the target object as output. The model begins by encoding coarse-grained point-level features from pre-trained PointNet++ and then interacts with these features with textual semantics to eliminate text-irrelevant points. Next, a fine-grained cross-modal reasoning module is applied to identify the key points that aid in final grounding. Finally, regression heads are used to produce high-quality bounding boxes based on the representations of these points. Unlike previous detection-based frameworks, this model is more efficient as it avoids the complex reasoning process across multiple object proposals.

![Fig-3](https://vision.aioz.io/f/5a297c21b3a34680b3c1/?dl=1)
*<center>**Figure 3**: 3D-SPS architecture. 3D-SPS follows a one-stage framework that avoids relying on a detection module. They coarsely sample language-keypoint relevant keypoint $P_0$ from the point feature with the DKS module, then finely select keypoint $P_T$ using $T$-layer TPM, and finally regress the box. The blue box denotes the ground truth, the green box denotes the prediction while yellow boxes denote objects of the same category as a target. </center>*

**Uni3DL.** Li et al. present Uni3DL a query transformer that leverages U-Net to learn task-agnostic semantics and generate bounding box/mask outputs by attending to 3D point-level visual features. Specifically, the proposed Uni3DL model captures both cross-attended and self-attended contexts from latent and text queries. It also incorporates additional information from various object-level datasets and utilizes multi-task learning to enhance grounding performance.

![Fig-4](https://vision.aioz.io/f/cedf633c850440ed85ae/?dl=1)
*<center>**Figure 4**: Uni3DL architecture. A Query Transformer (3) module is implemented with cross-attention and self-attention operations between latent queries, text features, and voxel features. A Task Router (4) consists of multiple heads for multiple 3D Downstream tasks.</center>*

**VPP-Net.** VPP-Net (**V**iew**P**oint **P**rediction **Net**) is introduced by Shi et al. at CVPR 2024. VPP-Net learns viewpoint prediction to transform 3D scenes to reduce ambiguity in spatial-relation grounding by estimating the viewpoints of expression point cloud pairs and rotating the 3D scenes accordingly before performing expression grounding. They use a Multimodal Encoder which consists of multiple layers of attention-based modules to align scene and text features. To enable viewpoint (location/perspective) prediction, they modify the multi-modal encoder by adding two transformer-based into each multimodal encoder layer to learn the location feature and perspective feature. In each layer in the encoder, they predict a distribution over locations and perspectives and supervise them with synthetic viewpoint supervision using cross-entropy loss. 

![Fig-5](https://vision.aioz.io/f/eb5977fddf60478eb706/?dl=1)
*<center>**Figure 5**: VPP-Net architecture. Left: The overall framework. Right: The detailed structure of the multi-modal encoder. Two transformer-based layers are added to learn the location and perspective information</center>*

**3D VLP.** 3D VLP (**3D V**ision-**L**anguage **P**re-training with object
contrastive learning - AAAI 2024) is a vision-language pre-training framework, which takes a scene point cloud and a sentence (not a question) as input and takes visual grounding as a proxy task for training. After training, they can flexibly transfer to other 3D downstream tasks. During training, they design different object contrastive learning strategies to better enhance scene understanding.

- **Object-level IoU-guided Detection Loss.**
They propose Object-level IoU-guided Detection Loss to enhance the object detector, therefore obtaining a high-quality proposal. by leveraging the Euclidean distance between the predicted box and the ground-truth box.


- **Object-level Cross-contrastive Alignment.**
The Object-level Cross-Contrastive alignment (OCC) task is introduced to align the distribution of cross-modal data (between bounding box proposals and text description) when contrastive learning can handle embedding alignment across different distribution (from point cloud and text) better than cross-model attention. The loss aims to align the features of positive samples with the language embedding and push the features of negative samples away:

- **Object-level Self-contrastive Learning.**
They design an object-level self-contrastive loss to effectively differentiate between objects and improve the model’s semantic understanding of the scene.

![Fig-6](https://vision.aioz.io/f/5fbc6138f84c481ebb79/?dl=1)
*<center>**Figure 6**: 3D VLP pipeline. 3DVLP takes visual grounding as the proxy task and utilizes multiple contrastive learning losses to enable the model to better differentiate the objects.</center>*




## Dataset 



The availability of large-scale datasets greatly enhances the performance of image grounding tasks, as well as other vision and language tasks. However, collecting and annotating 3D datasets remains a challenge, often resulting in relatively small RGB-D datasets. The ScanNet dataset, introduced by *Dai et al.*, is a richly annotated 3D indoor scan dataset of real-world environments, captured using a custom-built RGB-D capture system.
 
The ScanNet dataset consists of 2.5M views obtained from 1513 scans, making it large in scale. However, ScanNet is designed for 3D object classification and 3D object retrieval with instance-level category annotation in a 3D point cloud, and it may not be suitable for 3D VG and other related language-integrated 3D tasks. To overcome this limitation, ScanRefer, Nr3D, and MultiRefer are developed as datasets specifically tailored for 3D VG, building on top of ScanNet. 

### ScanRefer 
ScanRefer is widely used for 3D VG. ScanRefer consists of 51,583 descriptions for objects in 800 ScanNet scenes. On average, there are 13.81 objects, 64.48 descriptions per scene, and 4.67 descriptions per object after filtering. The descriptions are complex and diverse, covering over 250 types of common indoor objects, and exhibiting interesting linguistic phenomena. With each object of each scene, there are approximately 5 descriptions, providing a rich understanding of the 3D scene at the instance level. 


![Fig-7](https://vision.aioz.io/f/d8a815cb03674c9f873f/?dl=1)
*<center>**Figure 7**: Overview of ScanRefer dataset.</center>*

### Nr3D

The 3D Natural Reference (Nr3D) dataset is one of the datasets from ReferIt3D, along with the synthetic language descriptions dataset Sr3D. Nr3D focuses on a challenging setup where the referred object belongs to a fine-grained object class and the underlying scene contains multiple object instances of that class (a scene that contains many chairs belonging to the same type). Nr3D consists of 41,503 natural, free-form utterances describing objects belonging to one of 76 fine-grained object classes within 5,878 communication contexts. 


## Metric

It is necessary to compare the predicted object bounding box from the model with the ground-truth bounding box. Intersection over Union (IoU) is a common metric in object detection to measure the similarity between two bounding boxes. In 3D VG, the most used metric is Acc@kIoU, which determines the proportion of correct boxes that exceed the specified IoU@$k$. Commonly, $k$ is set at 0.25 or 0.5. 
