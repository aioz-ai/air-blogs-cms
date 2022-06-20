---
last_modified_on: "2022-03-07"
title: Abnormal Human Activity Recognition (Part 1 - Introduction)
description: A review about Abnormal Human Activity Recognition.
series_position: 14
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: abnormal_activity_recognition"]
---
Computer vision technology that can identify human behavior has gotten a lot of interest in recent years. Human motion recognition is still a difficulty in the field of computer vision, despite its widespread usage in a variety of applications.

Surveillance systems, situation awareness, homeland security, and smart environments have all improved as a result of the notion of intelligent visual recognition of anomalous human behaviour. However, due to factors such as (a) the core concept of anomaly, (b) feature representation of an anomaly, (c) its application, and therefore (d) the dataset, aberrant human behavior is exceedingly different in and of itself. The goal of this series is to provide a list of extant abnormal human activity recognition (AbHAR) handmade and deep techniques that differ in the kind of data available, such as two-dimensional or three-dimensional data.

# Approaches

On the basis of the main type of input provided to the system, abnormal human activity recognition (AbHAR) approaches may be divided into two categories: two-dimensional AbHAR systems and three-dimensional AbHAR systems. Figure 1 depicts the AbHAR system's thorough categorization. In the bulk of the ways outlined, a two-dimensional AbHAR system is fed with 2D silhouettes for single person AbHAR. Three-dimensional AbHAR systems, on the other hand, are supplied depth silhouettes and skeletal structures of the individual. Both approaches have their own set of uses and advantages.

![Fig-1](https://vision.aioz.io/thumbnail/893ff5558b8844aea5ec/1024/part1-figure1.png) *<center>**Figure 1**:  Taxonomy of Abnormal Human Activity Recognition (AbHAR).</center>*

## Two-Dimensional AbHAR

An image's RGB intensity and object detection (spatial information) are two basic observations. Researchers have conducted extensive study on normal and dysfunctional HAR systems employing 2D RGB/intensity/spatial information of a picture. Single person AbHAR and Multiple Persons AbHAR are the two types of 2D-AbHAR techniques discussed in this section. Different sorts of feature representations of action are described in each area to offer a quick overview of available approaches.

### Single person AbHAR

 On the basis of their substantial contributions in comprehending single person action recognition, the former is further divided into:
- Silhouette-based approaches.
- Spatio-temporal based approaches.
- Miscellaneous approaches.

### Multiple persons AbHAR

Multiple Persons AbHAR techniques are divided into three categories:
- Sparse Representation based approaches.
- Spatio-temporal based approaches.
- Miscellaneous approaches.

## Three-dimensional AbHAR
In compared to one-dimensional data processing, multi-dimensional data processing always produces more accurate and exact outcomes. This fact has always piqued academics' interest in 3D and 4D data. Due to the enormous popularity of Microsoft depth sensors in recent years, research on aberrant Human Activity utilizing depth information has exploded. Researchers now have access to a lot of new datasets that allow them to construct unique designs and test them on a larger collection of sequences. We can provide image depth information as well as a skeleton view with each joint position in a three-dimensional coordinate system. Using depth and skeleton representations of an image, some intriguing applications have been created.

### Depth based AbHAR

Depth images not only make low-level image processing easier and faster, but they also produce superior results in terms of background removal, object motion detection, and localisation.

### Skeleton based AbHAR

Skeleton representation of the human body gives acute features of human posture in a compact form, removing the requirement for a good segmentation approach to extract 2D silhouettes and simplifying height centroid computation from depth silhouettes. This has prompted academics to create real-time applications based on the skeleton modality, which makes computations faster, easier, and more efficient.

## Deep features based action description

It has been noticed that the notion of feature engineering has evolved from 2D to 3D features throughout time in order to better action representation. However, the complexity of designing handcrafted features is extremely high, which is one of the primary reasons for shifting features from the shallow to deep regions, which has elevated the practical applicability of action recognition algorithms to a new level of excellence by applying deep learning knowledge to recognition systems. It includes 2 main approaches:
- Single person based abnormal human action recognition.
- Multiple persons based abnormal event detection.

In the next blogs, we will go through certain aspects of the topic in greater depth.

