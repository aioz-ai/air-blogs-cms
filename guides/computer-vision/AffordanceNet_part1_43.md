---
last_modified_on: "2023-19-08"
title: Object Affordance Detection
description: A kind introduction to Object Affordance Detection.
series_position: 43
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
![](https://vision.aioz.io/f/1e87d34d0d5e49e19004/?dl=1)
The properties of an object, such as its color, shape, and weight, can be used to identify and classify it. However, these properties do not tell us what actions humans can perform on the object. The ability to understand the functional aspects of objects, or object affordances, has been studied for many years. Unlike visual and physical properties, which describe the object alone, affordances describe the functional interactions between the object and humans. Understanding object affordances is essential for autonomous robots that interact with objects and assist humans in daily tasks. For example, a robot that needs to pick up a cup needs to understand that the cup has a handle that can be grasped.

## Object Affordance Detection 
Object affordance detectionis the task of automatically recognizing the potential actions that can be performed on an object. For example, a chair can be sat on, a cup can be held, and a pen can be written with. Affordance detection is a challenging task, as it requires the ability to understand the physical properties of objects and their relationships to the environment.


![](https://vision.aioz.io/f/d286a3b1eddf4ed8b781/?dl=1)*<center>**Figure 1**. Simultaneous affordance and object detection. Some example results which detects both objects and their multiple affordance classes using an end-to-end architecture. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

There are two main approaches to object affordance detection:

- **Object-centric**: This approach focuses on detecting the affordances of individual objects. For example, a classifier could be trained to recognize the affordance of "sitting" for a chair.
- **Scene-centric**: This approach focuses on detecting the affordances of objects in a scene. For example, a classifier could be trained to recognize the affordance of "eating" for a table and a chair.

## AffordanceNet 
To have a deep understanding about object affordance detection, we introduce  a typical architecture about it: AffordanceNet.

### Problem Statement
Their framework aims to simultaneously find the object positions, object classes, and object affordances in images. Following the standard design in computer vision, the object position is defined by a rectangle with respect to the top-left corner of the image; the object class is defined over the rectangle; the affordances are encoded at every pixel inside the rectangle. The region of pixels on the object that has the same functionality is considered as one affordance. Ideally, they want to detect all relevant objects in the image and map each pixel in these objects to its most probable affordance label.

### AffordanceNet Architecture
![](https://vision.aioz.io/f/c916e068adb44d33adc6/?dl=1)*<center>**Figure 2**. An overview of our AffordanceNet framework ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

The AffordanceNet framework consists of the following components:
- A deep CNN backbone (i.e., VGG) to extract image features.
- A region proposal network (RPN) to propose a set of bounding boxes.
- A RoIAlign layer to extract and pool features from each bounding box.
- A two-layer fully connected network for object detection.
- A sequence of convolutional-deconvolutional layers for affordance detection.
- A softmax layer to output a multiclass affordance mask.

The RPN shares weights with the backbone and outputs RoIs. For each RoI, the RoIAlign layer extracts and pools its features to a fixed size 7x7 feature map. The two-layer fully connected network is used to regress object location and classify object category. The sequence of convolutional-deconvolutional layers is used to upsample the feature map from the object detection branch to the size of the input image. The softmax layer is used to classify each pixel in the image as one of the affordance classes.

## RoIAlign
RoIAlign stands as a key component in recent and successful object detection methods that employ a region-based strategy, such as Faster RCNN. This approach incorporates a Region Proposal Network (RPN) that shares weights with the primary convolutional backbone, generating bounding boxes or object proposals of varying dimensions. For each Region of Interest (RoI), a compact feature map, often 7x7 in size, is extracted from the image feature map using the RoIPool layer. The RoIPool process involves dividing the RoI into a structured grid and applying max-pooling to feature map values within each grid cell. However, this discretization introduces discrepancies between the RoI and the extracted features, stemming from the rigid rounding operations used during RoI coordinate mapping from input image space to image feature map space, and the subsequent grid cell division.

To tackle this challenge, the authors introduced the RoIAlign layer, which addresses feature misalignment concerns by ensuring proper correspondence with the RoI. Unlike traditional rounding methods, RoIAlign employs bilinear interpolation to compute interpolated feature values at four evenly distributed locations within each RoI bin. The resulting values are then aggregated using the max operation. This alignment technique bears significant relevance for tasks that operate at the pixel level, such as image segmentation. 

## Deconvolution for High Resolution Affordance Mask
Utilized in recent instance segmentation advancements like Mask-RCNN and FCIS, the concept of employing compact fixed-size masks (e.g., $14 \times 14$ or $28 \times 28$) to signify object segmentation has proven effective. This approach works because each predicted mask within a Region of Interest (RoI) is binary, representing foreground or background. However, we've observed that this approach doesn't perform optimally for the task of affordance detection, given the presence of multiple affordance classes within each object.

In response to this limitation, we propose an alternative approach involving a series of deconvolutional layers to achieve high-resolution affordance masks. Given an input feature map of size Si, the deconvolutional layer performs the inverse operation of a convolutional layer, producing an enlarged output map with size So. The relationship between Si and So is given by:
$$

S_o = s ∗ (S_i − 1) + S_f − 2 ∗ d

$$

where $S_f$ represents the filter size, and s and d denote the stride and padding parameters, respectively.

In practice, the RoIAlign layer generates a feature map with dimensions 7x7. To enhance this map's resolution, we employ three consecutive deconvolutional layers for upsampling. The initial deconvolutional layer applies padding d = 1, stride s = 4, and kernel size $S_f = 8$, resulting in a map sized $30 \times 30$. Similarly, the second layer uses parameters $(d = 1, s = 4, S_f = 8)$, and the third layer uses $(d = 1, s = 2, S_f = 4)$, eventually producing a final high-resolution map of $244 \times 244$. It's important to note that a convolutional layer (with ReLU activation) precedes each deconvolutional layer, acting as a bridge between consecutive deconvolutional steps. This convolutional layer facilitates feature learning for the subsequent deconvolution. 
 
## Robust Resizing Affordance Mask
Affordance detection branch necessitates a predefined fixed size (e.g., 244x244) target mask for effective training supervision. The authors introduce a resizing technique involving multi-thresholding. Given an original groundtruth mask, without loss of generality, let $P = \{c0, c1, ..., cn−1\}$ represent the set of n distinct labels within that mask. Initially, we linearly map values in $P$ to $P' = \{0, 1, ..., n − 1\}$ and employ this mapping to convert the original mask to a new mask. Following this conversion, we resize the transformed mask to match the predefined target size. 

It's important to note an alternative method to achieve the fixed-size target training mask. This approach involves resizing for each individual affordance label within the original groundtruth mask, treating a specific label as foreground while considering other labels as background. The resized masks for individual labels are then combined to generate the target training mask. However, this strategy can be time-consuming due to the need for multiple resizings for various affordance classes within the RoI.

## Summary
We have introduce Object Affordance Detection and a typical baseline for it. In the next blog, we will introduce the effectiveness of the reviewed method.

##  Reference
Do, Thanh-Toan, Anh Nguyen, and Ian Reid. "Affordancenet: An end-to-end deep learning approach for object affordance detection." 2018 IEEE international conference on robotics and automation (ICRA). IEEE, 2018.















