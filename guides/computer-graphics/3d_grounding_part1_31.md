---
last_modified_on: "2024-09-11"
title: 3D Downstream Tasks for Foundation Model (Part 1).
description: 3D Visual Grounding
series_position: 31
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

3D Visual Grounding (3D VG) refers to the process of identifying and localizing specific objects or regions within a 3D space based on natural language descriptions. Unlike traditional 2D visual grounding, which operates on flat images, 3D visual grounding extends this concept into three-dimensional environments. 3D visual grounding has numerous applications, including robotics (e.g., for object manipulation or navigation), augmented reality (AR), virtual reality (VR), and autonomous driving, where understanding objects in complex environments is critical. This blog offers a comprehensive and insightful overview of the topic of 3D Visual Grounding.


## Introduction

Humans can effortlessly identify and locate entities in an image or environment from a brief natural description with just a glance. However, modeling computers to perform such tasks remains challenging.

2D image grounding or 2D video grounding aims to locate objects or moments within an image or video based on a natural description. These tasks integrate computer vision and natural language processing to enhance the understanding of both visual content and textual meaning. Models must interpret spatial relationships, object attributes, and visual context. Methods for 2D image/video grounding play a crucial role in advancing the fields of computer vision and natural language processing.

Similar to 2D image/video grounding, 3D visual grounding seeks to detect one or more objects within a 3D scene that match a given description. Unlike traditional 2D images, 3D scenes are represented by complex point clouds or triangular meshes, offering richer spatial, depth, and multi-view information. With advancements in 3D data collection techniques, 3D visual grounding, and scene understanding have gained increasing attention in the field of 3D deep learning.

3D visual grounding has numerous applications and serves as a foundational step for various tasks such as 3D scene understanding, 3D embodied interaction, and other 3D-text-related tasks. However, challenges remain in understanding spatial relationships, and object attributes, and addressing occlusions and varying perspectives inherent to 3D environments. Existing methods have shown effectiveness in overcoming these challenges. This article is intended to delve deeper into 3D VG, including task definition, used framework, implemented methods, and datasets. 
## Method Overview 

### Task definition

3D VG aims to understand the 3D scene at the object level by detecting the objects that match the given natural descriptions. The input required is a 3D point cloud, which represents the geometric and appearance characteristics of objects in the scene, along with other features such as colors, normal vectors, ...  and a textual description of the scene or local objects in that scene.


The point cloud input data can be represented as a matrix $P \in \mathbb{R}^{N \times F}$ where $N$ denotes the number of points (usually 40,000) and $F$ is the dimensionality of point features, including point coordinates $(x,y,z)$ and other features if needed. 3D VG aims to locate a specified object related to the semantic description. The final output is a vector $d \in \mathbb{R}^6$ which denotes a bounding box with center $\bm{c}_i \in \mathbb{R}^3$ and box size $\bm{s}_i \in \mathbb{R}^3$, and $D_i$ is the corresponding text description. A semantic label (in 18 semantic classes of ScanNet) can also be considered as additional output for further explainability and reasoning.



### Architecture 
Most implemented methods that aim to solve 3D VG can be divided into two categories: two-stage framework and one-stage framework. The **two-stage framework** first defines the object proposals through the 3D object detector, then finds the object that best suits the textual description. Although this strategy achieves significant performance, it heavily relies on the quality of the 3D object detector as a sub-task. Without object proposals, the **one-stage framework** aims to train the model end-to-end. This framework directly regresses the spatial bounding box of the text-guided object after fusing point embedding and text embedding. 

In general, 3D VG first encodes the 3D scene point cloud and text descriptions into semantic features, then fuses them to obtain a unique fused feature, and finally detects the local object.  

**Scene encoder**. After preprocessing the original scene data (mesh subsampling or augmentation, ...), they use an encoder to encode the entire scene. 

- **Two-stage framework**. They utilize a pre-trained 3D object detector or segmentor to extract local objects and encode them at the object level. These detected objects are used to interact with the description query for the best-matched object. Currently, the most used 3D object detectors are VoteNet and PointGroup or some 3D segmentation models which can extract instance features from 3D data. The output of the object detector contains object location $D \in \mathbb{R}^{M \times 6}$ and their feature $C \in \mathbb{R}^{M \times d}$, where $M$ is the number of detected objects and $d$ is the feature dimension. Denote $f_{3D \ enc}$ as a 3D encoder, the feature extraction can be formulated as

$$
D,C = f_{3D \ enc}(P)
$$

- **One-stage framework**. One-stage framework does not detect or segment objects. Instead, it encodes 3D scene data at the point level by a global point cloud encoder such as PointNet, PointNet++, or a 3D convolutional network. These point-level features are more fine-grained and can perceive more relevant background contexts. However, it also introduces much irrelevant background noise for ineffective reasoning. The point cloud extractor can be described as 

$$
C = f_{point \ enc}(P)
$$

where $C \in \mathbb{R}^{M \times d}$ is defined above and $f_{point \ enc}$ is a point cloud extractor.

**Text encoder**. The textual input contains a sentence that provides information about the scene and local entities. They first preprocess the sentence into text tokens, and then use a feature extractor to obtain text features. 

- **Text preprocessor**. The sentence is segmented into a sequence of fix-length tokens by padding or truncating. Data augmentation can be considered such as masking out words and replacing tokens with specific unknown tokens. 

- **Text feature extractor**. After processing the input, a feature extractor is used to extract the query and map its semantics to embedding space. Common embedding methods used are GloVe (in ScanQA) and RoBERTa. The output of the feature extractor is a matrix containing embedding of each word $T \in \mathbb{R}^{L \times d_t}$ where $d_t$ is the dimension of text feature and $L$ is the fixed number of tokens. 

After extracting the text feature and point feature (or object-level feature) from the original data, recent works further design intra-model encoders to capture dependencies in each modality (e.g. relationship among objects in 3D data or word tokens in text) to obtain rich information. Based on the contextual features, they are later mapped into the same space. Despite the efficiency of these contextual encoders, early works do not consider these steps and directly fuse the extracted rough features. 

**Scene context encoder**. The scene context encoders used can be categorized into two types: *graph neural network* and *self-attention* mechanism. In graph neural networks, methods construct a graph based on object proposals obtained from a 3D object detector. Each node represents an object proposal and each edge represents the relationship between two objects (e.g. the bed is next to the table). After constructing the graph, a Graph Convolution Neural Network (GCN) is implemented to encode relationships between objects into the feature of proposals. Besides, self-attention mechanisms in scene context encoders try to learn the correlation among object proposals without constructing a graph. It can also incorporate attention between scene features and object features to further refine rich information from the point cloud. Taken object feature $C$, scene contextual encoder produces contextual scene feature $C' \in \mathbb{R}^{M \times d_c}$ with contextual dimension length $d_c$. 
$$
C' = f_{3d \ c-enc}(C)
$$


**Text context encoder**. The text context encoders used can also be categorized into two types: *RNNs* and *self-attention mechanisms*. With the first type, recurrent neural networks such as GRU and LSTM are used due to the ability to effectively capture temporal dependencies in the text. As for the second type, self-attention mechanisms can capture long-range dependencies in sentences, but they require more computation cost. Some methods implement large text encoders such as T5 and CLIP for semantic learning. The contextual encoder takes the rough feature $T \in \mathbb{R}^{L \times d_t}$ and produces contextual feature $T' \in \mathbb{R}^{L \times d_w}$ in word level. 


**Multi-modal module.**: After separately obtaining scene features and text features from the contextual encoder, the next step is to align them to the same space for reasoning. This is a crucial step in 3D VG as it requires understanding the complete semantic meaning of the text and complex relationships among multiple objects in the scene. **Transformer-based cross-attention** mechanism is most deployed in existing state-of-the-art methods. In the cross-attention mechanism, object proposals are treated as queries, while text features are treated as keys and values. This configuration enables the evaluation of the alignment between each object/point and the query text. Some methods also incorporate dual-pathway cross-attention mechanisms to further obtain the associations between each work token and each object, where text features serve as queries and object proposals serve as keys and values. Many methods have made modifications to cross-attention modules to further improve performance. such as incorporating additional input or deploying cross-attention modules at various levels. The multi-modal module outputs the fused feature $f_{fus} \in \mathbb{R}^{d_f}$.

**Grounding head**. The final step is to transform the fused feature $f_{fus}$ to the object bounding box. In the two-stage method, after obtaining object proposals $D \in \mathbb{R}^{M \times 6}$, existing methods calculate the matching score of proposals $h \in \mathbb{R}^M$ which represents the matching degree between each proposal and the query text, and the proposal with the highest score will be chosen as the final result. However, in the one-stage method, since object proposals cannot be obtained instantly, methods first regress the positions of multiple objects along with their confidence scores based on $f_{fus}$ with a classifier, then select the one with the highest confidence score as the final output.

![Fig-1](https://vision.aioz.io/f/7d19f6e1a12d462f8211/?dl=1)
*<center>**Figure 1**: General pipeline of two-stage framework</center>*


![Fig-2](https://vision.aioz.io/f/97699704c6744070adcd/?dl=1)
*<center>**Figure 2**: General pipeline of one-stage framework. Unlike two-stage framework, the one-stage framework does not detect bounding box from the 3D encoder. The box is regressed by point feature, then classified to produce the final output</center>*

In the next section, we will discuss state-of-the-art methods, commonly used datasets, and evaluation metrics in 3D VG.

</CodeExplanation>