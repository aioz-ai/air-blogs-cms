---
last_modified_on: "2021-11-08"
title: Human Pose & Motion Capture exploration (part 3).
description:  explore human pose estimation and motion capturing. 
series_position: 3
author_github: https://github.com/minhnhatvt
tags: ["type: insight", "level: medium"]
---


In previous post,we haave explored 3D humman pose estimation. In this part, we discuss about dataset and evaluation protocol for human pose estimation and motion capturing.


## Human Pose Estimation Datasets
We summarize different datasets that are widely used in literature to train and evaluate 2D and 3D HPE system. Most of these datasets have recently been proposed in research (from 2014 up to now).

### 2D HPE Datasets

![](https://i.imgur.com/r4DRDDl.png)*<center>**Table 5:** Summary of 2D HPE Datasets [(Source)](https://arxiv.org/abs/2006.01423)</center>*


### 3D HPE Datasets

![](https://i.imgur.com/aMi1CpS.png)*<center>**Table 6:** Summary of 3D HPE Datasets [(Source)](https://arxiv.org/abs/2006.01423)</center>*


## Evaluation metrics
* **2D HPE**
    * **Percentage of Correct Keypoints (PCK):** measure the accuracy of localization of different keypoints within a given threshold. The threshold is set to 50 percent of the head segment length of each test image and it is denoted as **PCKh@0.5**
    * **Average Precision (AP):**
        * If a predicted joint falls within a threshold of the ground-truth joint location, it is counted as a true positive For multiperson pose evaluation, 
        * For multi-person, all predicted poses are assigned to the ground truth poses one by one based on the PCKh score order, while unassigned predictions are counted as false positives.
        * **Object keypoint similarity (OKS)** plays the same role as the Intersection over Union (IoU) in object detection.
* **3D HPE**
    * **Mean Per Joint Position Error (MPJPE):** Euclidean distance from the estimated 3D joints to the ground truth in millimeters, averaged over all joints in one image. There are some variants of MPJPE:
        * **N-MPJPE:** is the MPJPE is calculated after aligning the depths of the root joints (generally pelvis joint).
        * **P-MPJPE:** is the MPJPE after aligning between the estimated pose and the ground truth pose by a rigid transformation.
    * **3DPCK:** is a 3D extended version of the PCK metric used in 2D HPE evaluation.
    * **Mean Per Vertex Error (MPVE):** Euclidean distances between the ground truth vertices and the predicted vertices. Used in body mesh prediction.

## References
1. Chen *et al.* Monocular Human Pose Estimation: A Survey of Deep Learning-based Methods, 2019
2. Zheng *et al.* Deep Learning-Based Human Pose Estimation: A Survey, 2020