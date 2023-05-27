---
last_modified_on: "2023-05-21"
title: Minkowski Engine - part 1
description: 4D Spatio-Temporal ConvNets: Minkowski Convolutional Neural Networks
series_position: 21
author_github: https://github.com/aioz-ai
tags: ["type: Optimization", "level: medium"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight'; 


In many robotics systems and VR/AR applications, 3D-videos are readily-available sources of input (sequence of 3D scans such as a video from a depth camera, a sequence of LIDAR scans, or a multiple MRI scans of the same object or a body part). However, processing each frame of 3D-videos using convolutional networks or 3D percrption algorithms becomes inefficiently in memory and time. It is necessary to research methods for processing these 3-dimension input or high-dimensional tensors efficiently.

## Introduction 
**Sparse convolution:** Challenges in using 3D video for high-level perception tasks: 
-  3D data requires heterogeneous representations and processing those either alienates users or makes it difficult to integrate into larger systems.

- The performance of the 3D convolutional neural networks is worse or on-par with 2D convolutional neural networks.

- Limited number of open-source libraries for fast large-scale 3D data. 


*Solution*: Adopt sparse tensor and generalize sparse convolutional for high dimension tensors: 
- Sparse tensor allows homogeneous data representation within traditional neural network libraries. 
- Sparse convolution performs successful in 2D perception as well as 3D reconstruction, feature learning, and semantic segmentation.
- The sparse convolution is efficient and fast as it only computes outputs for predefined coordinates (whose values are non-zero) and saves them into a compact sparse tensor. Therefore, it saves both memory and computation especially for 3D scans or high-dimensional data where most of the space is empty.



**Hybrid-kernel:** Efficient sparse convolution representation for high-dimensional spaces results in significant computational overhead and memory consumption: 

- The convolution operation with kernel size $k$ requires $k^D$ weights in $D$-dimension kernel. 
- This exponential increase, however, does not necessarily lead to better performance and slows down the network significantly.

Hybrid kernel with non-(hyper)-cubic shapes using the generalized sparse convolution aims to resolve this problem.

## Methodology

### **Sparse Tensor**: 
Input of deep learning model (such as image, text, feature) is commonly represented using tensor. However, for 3-dimensional scans or even higher-dimensional spaces, dense representations are inefficient for these tensors as effective (or entire) information occupies only a small proportion of the space. Instead, we can save information of non-zero elements of the tensors. similar to how information was saved on a sparse matrix. 

The sparse tensor was represented by using the COO (coordinate) format as it is efficient for neighborhood queries. This representation is simply a concatenation of coordinates in a matrix $C$ and associated features  $F$:

$$

C = \begin{bmatrix}
x_1^1  & x_1^2 & \ldots &x_1^D \\ 
\vdots &  \vdots& \ddots & \vdots\\ 
x_N^1  & x_N^2  & \ldots & x_N^D
\end{bmatrix} , F = \begin{bmatrix}
f_1^T\\ 
\vdots\\
f_N^T
\end{bmatrix} 

$$ 

where $x_i \in \mathbb{Z}^D$ is a $D$-dimensional coordinate and $N$ is the number of non-zero elements in the sparse tensor, each with the coordinate $(x_i^1,...,x_i^D)$, and the associated feature $f_i$. The sparse tensor $\Im$ is defined based on $C$ and $F$: 

$$

\Im[x^1_i , x^2_i, ..., x^D_i] = \left\{\begin{matrix}
f_i \ \ \text{if} \ \ (x^1_i , x^2_i, ..., x^D_i) \in \mathcal{C}\\ 
 0 \ \ \text{otherwise}
\end{matrix}\right.

$$

where $\mathcal{C}$ denotes set of non-zero indices of matrix $\Im$

### **Sparse Convolutional**

Let $x_u^{\text{in}} \in \mathbb{R}^{N^{in}}$ be an $N^{in}$-dimensional input feature vector in a $D$-dimensional space at $\textbf{u} \in \mathbb{Z}^D$ (a $D$-dimensional coordinate). The convolution kernel weight $K\in \mathbb{R}^{k^D\times N^{\text{out}} \times N^{\text{in}}}$  was broke down  into spatial weights with $k^D$ matrices of size $N^{\text{out}}\times N^{\text{in}}$ as $K_i$. The conventional dense convolution in $D$-dimension is 

$$

x^{\text{out}}_\textbf{u} = \sum_{\textbf{i} \in \mathcal{V}_D(k)} K_ix^{\text{in}}_{\textbf{u}+\textbf{i}} 

$$

where $\mathcal{V}^D(k)$ is the list of offsets in $D$-dimensional hypercube centered at the origin. The generalized convolution in the following equation relaxes the above equation:

$$

x^{\text{out}}_\textbf{u}=\sum_{\textbf{i} \in \mathcal{N}^D(\textbf{u},S^{\text{in}})}K_\textbf{i}x^{\text{in}}_{\textbf{u}+\textbf{i}} 

$$

where $\mathcal{C}^{in}$ and $\mathcal{C}^{out}$ are predefined input and output coordinates of sparse tensors,  $\mathcal{N}^D$ is a set of offsets that define the shape of a kernel, and $\mathcal{N}^D(\textbf{u}, \mathcal{C}^{\text{in}})=\left\{ \textbf{i} | \textbf{u}+\textbf{i} \in \mathcal{C}^{\text{in}}, \textbf{i} \in \mathcal{N}^D\right\}$ as the set of offsets from the current center $\textbf{u}$ that exist in $\mathcal{C}^{in}$

Dense Tensor            |  Sparse Tensor
:-------------------------:|:-------------------------:
![](https://drive.google.com/uc?export=view&id=1BEnl-q2ZkLXQAW8cN6tOlikKBm3CvVsI)  |  ![](https://drive.google.com/uc?export=view&id=1z1kAclTpMQ6ghqR2xYxOArQnRVIKGjdp)

## References

<a id="1">[1]</a> 
High-dimensional Convolutional Neural Networks for 3D Perception, Stanford University, Chapter 4. Sparse Tensor Networks.

<a id="2">[2]</a> Christopher Choy, JunYoung Gwak, and Silvio Savarese. 4D spatio-temporal ConvNets: Minkowski convolutional neural networks. In CVPR, 2019.
