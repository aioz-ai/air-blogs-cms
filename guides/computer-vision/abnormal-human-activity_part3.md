---
last_modified_on: "2022-05-16"
title: Abnormal Human Activity Recogition (Part 3 - Two-Dimensional AbHAR (cont.))
description: A review about Abnormal Human Activity Recognition (2D AbHAR) (cont.)
series_position: 18
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: abnormal_activity_recognition"]
---

# Two-Dimensional AbHAR (continue)

# Multiple persons AbHAR
Multiple Persons AbHAR is a generalized instance of Crowd Activity analysis, which are terms that are used interchangeably in this article. It is a newer area of study in computer vision that might lead to new applications such as automated detection of riots or chaotic acts in crowds and localisation of aberrant spots in scenes for high resolution analysis. Some of the issues that must be addressed for crowd activity analysis include occlusions and ambiguities in crowded scenes with complicated behaviors and scene semantics.

It is thoroughly researched in the fields of transportation and community security. However, the level of complexity required to characterize the entire crowd activity with the help of a feature is quite high. There are two methods for analyzing crowd activity.
* recognize group activity patterns – aberrant activity localization represents an individual's conduct in a crowd, i.e. a trajectory-based approach.
* shape dynamics based method.

[Table 1 in the previous blog](https://ai.aioz.io/guides/computer-vision/abnormal-human-activity_part2/#single-person-abhar) describes numerous techniques for multiple people AbHAR specifies feature abstraction methods, along with their major parts and limits.

Multiple persons AbHAR based approaches are classified into three types: spatiotemporal techniques, sparse representation approaches, and miscellaneous approaches.

## Spatio-temporal based approaches

In this part, we will sumarize some recent spatio-temporal based approaches.
*  (Zhan et al., 2016). Effective encryption of intricate aspects of the dynamic backdrop and various changes of the foreground in a crowded or public setting need both geographical and temporal information of the scenes.
* (Han et al., 2016). The authors search for latent action patterns in crowd behaviors by employing the notion of visual attention mechanism with spatiotemporal saliency-based representation and grouping them into important groups using affinity matrix based N-cut.
*  (Wang et al., 2017b). The author of one of the researchattempts to detect and pinpoint the abnormal act in a crowd utilizing the notion of visual attention analysis. For spatiotemporal feature extraction, the reported method employed two sparse coding strategies: offline long-term sparse representation and online short-term sparse representation.
*  (Saini et al., 2017). A recent paper defined block-based scene representation to emphasize the need of entire scene awareness in addition to motion detection. It used a non-overlapping block-based model of the surveillance scene to extract high-level information from object trajectories. And the Hidden Markov Model (HMM) discovered the parameters that explain the dynamics of a certain surveillance scenario.

## Sparse Representation based approaches

Sparse models give descriptors with unique discrimination capabilities, making sparse representation a preferred strategy for anomalous behavior identification.
* Li et al. (2016) created the Histogram of Maximal Optical Flow Projection (HMOFP) descriptor and utilized it for dictionary optimization and estimating the 'l1' norm of the sparse reconstruction coefficient (SRC) in the test frame for crowd anomaly identification.
* Liu et al. (2017) observed crowd movements using a double sparse representation with a dynamic dictionary updating approach that dynamically increases the size of the dictionary for a limited amount of training samples.

## Miscellaneous

* (Guo et al., 2016). The work uses a pixel-wise technique with the Weighted Quaternion Discrete Cosine Transform (QDCT) to generate an anomalous saliency map. QDCT cannot leverage information for long-term events, which is a shortcoming of this technique.
* (Stephens and Bros, 2016). The streaklines approach is used to identify and differentiate particular human actions in a video. Each streakline is made up of numerous optical flow vectors that follow local movement in the scene.
* (Zhang et al., 2016). Locality Sensitive Hashing Filters (LSHF) were developed by the authors to filter out aberrant actions by hashing ordinary activities into various feature buckets. The Particle Swarm Optimization (PSO) approach is used to improve the efficiency of anomaly identification in video. An online updating technique is also used to update dynamic scene variations in the movie to this framework.

In the next blog, we will discuss about Three-Dimensional AbHAR in details.
