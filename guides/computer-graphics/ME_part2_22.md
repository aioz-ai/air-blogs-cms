---
last_modified_on: "2023-05-26"
title: Minkowski Engine - part 2
description: 4D Spatio-Temporal ConvNets: Minkowski Convolutional Neural Networks - Experimental results
series_position: 22
author_github: https://github.com/aioz-ai
tags: ["type: Optimization", "level: medium"]
---


import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight'; 


In this part, we will discover the implementation details of Minkowski Engine and corresponding Experimental Results.

## **Minkowski Engine**

Minkowski Engine is an open-source auto-differentiation library for sparse tensors and the generalized sparse convolution. It is an extensive library with many functions
### **1. Sparse Tensor Quantization**: 
Convert an input into unique coordinates, associated features, and optionally labels using  GPU functions. 

### **2. Generalized Sparse Convolution**:  
- Create this output coordinates dynamically allowing an arbitrary output coordinates $\mathcal{C}^{out}$ given the input coordinates  $\mathcal{C}^{in}$ for the generalized sparse convolution

- Create a kernel map for convolving the input with the kernel. The kernel map identifies which inputs affect which outputs, and is defined as pairs of lists of integers: the in map $\textbf{I}$ and the out map $\textbf{O}$. An integer in an in map $i\in \textbf{I}$ indicates the row index of the coordinate matrix or the feature matrix of an input sparse tensor. Similarly, an integer in the out map $o \in \textbf{O}$ also indicates the row index of the coordinate matrix of an output sparse tensor.

![Fig-2](https://drive.google.com/uc?export=view&id=1ETXj0zX9B6anzAvqFM5APpsgVlZgsWrE)
*<center>**Figure 2**:  Convolution Kernel Map.</center>*

For example,  $3\times 3 $ kernel requires 9 kernel maps. Due to sparsity in tensor, some kernel maps do not have elements. The kernel maps are extracted: 

- Kernel map $B: 1 \mapsto 0$ 
- Kernel map $B: 0 \mapsto 2$ 
- Kernel map $H: 2 \mapsto 3$ 
- Kernel map $I: 0 \mapsto 0$ 

### **3. Max Pooling and Global Pooling**
Max pooling layer selects the maximum element within a region for each channel. For a sparse tensor input, define it as

$$

x_{\textbf{u},i}^{\text{out}} = \max_{\textbf{k} \in \mathcal{N}^D(\textbf{u}) \cap \mathcal{C}^{in}} x_{\textbf{u}+\textbf{k},i}^{\text{in}}

$$

where $x_{\textbf{u},i}$ indicates the $i$-th channel feature value at $\textbf{u}$. The region to pool features from is defined as $\mathcal{N}^D(\textbf{u}) \cap \mathcal{C}^{in}$. The global pooling is similar to the max pooling except that features from all non-zero elements in the sparse tensor are pooling:

$$

x_{i}^{\text{out}} = \max_{\textbf{k} \in  \mathcal{C}^{in}} \ x_{\textbf{u}+\textbf{k},i}^{\text{in}}

$$
### **4. Normalization**
First, instance normalization computes batch-wise statistics and whiten features batch wise. The mean and standard deviations are: 
$$ 

\mu_b = \dfrac{1}{|\mathcal{C}^{in}_b|} \sum_{\textbf{k} \in \mathcal{C}^{in}_b}x_{\textbf{u},b}^{\text{in}}

$$

$$

\sigma_{bi}^2 = \dfrac{1}{|\mathcal{C}^{in}_b|} \sum_{\textbf{u} \in \mathcal{C}^{in}_b} (x_{\textbf{u},bi}^{\text{in}} - \mu_{bi})^2

$$
where $x_{\textbf{u},bi}$ indicates the $i$-th channel feature at the coordinate $\textbf{u}$ with batch index $b$. $\mathcal{C}^{in}_b$ is the set of non-zero element coordinates in the $b$-th batch. $\mu_b$ indicates the $b$-th batch batch-wise feature mean and $\sigma_{bi}$ is the $i$-th feature channel standard deviation of the $b$-th batch.

$$

x_{\textbf{u},bi}^{\text{out}} = \dfrac{x_{\textbf{u},bi}^{\text{in}} - \mu_{bi}}{\sqrt{\sigma_{bi}^2 + \epsilon}} 

$$

Batch normalization is similar to the instance normalization except that it computes statistics for all batch:
$$ 

\mu = \dfrac{1}{|\mathcal{C}^{in}|} \sum_{\textbf{k} \in \mathcal{C}^{in}}x_{\textbf{u}}^{\text{in}}

$$

$$

\sigma_{i}^2 = \dfrac{1}{|\mathcal{C}^{in}|} \sum_{\textbf{u} \in \mathcal{C}^{in}} (x_{\textbf{u},i}^{\text{in}} - \mu_{i})^2

$$

$$

x_{\textbf{u},i}^{\text{out}} = \dfrac{x_{\textbf{u},i}^{\text{in}} - \mu_{i}}{\sqrt{\sigma_{i}^2 + \epsilon}} 

$$

### **5. Non-linearity Layers**
Most of the commonly used non-linearity functions are applied independently element-wise. Thus, an element wise function $f(·)$ can be a rectified-linear function (ReLU), leaky ReLU, ELU, SELU, etc: 
$$ 

x_{\textbf{u},i}^{\text{out}} = f(x_{\textbf{u},i}^{\text{in}}) \ \text{for}\ \ \textbf{u} \in \mathcal{C} 

$$

## **Minkowski Convolutional Neural Networks**

Problems of high-dimensional convolutions: 
- Computational cost and memory consumption increase exponentially due to the dimension increase, and they do not necessarily lead to better performance.
- The networks do not have an incentive to make the prediction consistent throughout the space and time with conventional cross-entropy loss alone.

**Hybrid kernel:** The hybrid kernel is a combination of a cross-shaped kernel a conventional cubic kernel. 
- Spatial dimensions: Use cubic kernel to capture the spatial geometry accurately.

- Temporal dimension: Use cross-shaped kernel to connect the same point in space across time.

The hybrid kernel experimentally outperforms the tesseract kernel both in speed and accuracy.
![Fig-1](https://drive.google.com/uc?export=view&id=1lo-7sXV6Q1A6VInwqtfcc2-ElyfQtpac)
*<center>**Figure 1**:  Various kernels in space-time. The red arrow indicates the temporal dimension and the other two axes are for spatial dimensions</center>*

## Experiment

**ScanNet:** The ScanNet 3D segmentation benchmark consists of 3D reconstructions of real rooms. It contains 1500 rooms, some repeated rooms captured with different sensors. They feed an entire room to a MinkowskiNet fully convolutionally without cropping.

![Fig-2](https://drive.google.com/uc?export=view&id=1OQVnPgAsTozHCMSV6uVx8hM7vVe5SIN5)
*<center>**Figure 2**:   3D Semantic Label Benchmark on ScanNet.</center>*


Visualization: 
3D input point cloud         |  Predictions | Ground truth
:-------------------------:|:-------------------------:|:-------------------------:
![](https://drive.google.com/uc?export=view&id=14Rz6NR1ZIk-ah-iEGPEhBMGTAE3UfO8X)  |  ![](https://drive.google.com/uc?export=view&id=1sNu2dgBL0ksIazpkOGGHxI8MljkKLl-u)|![](https://drive.google.com/uc?export=view&id=1cLu4C1OfqmaNKTqTZt7kmDFbzAQfyw81) 
![](https://drive.google.com/uc?export=view&id=1N32BTVc9k7wFnrhP8ilFl9RoLl5FjfI6)  |  ![](https://drive.google.com/uc?export=view&id=1fyh9K88dC2YwUS1WG4HStO0guSmgv69l)|![](https://drive.google.com/uc?export=view&id=1C4TFkYRVbsnUTdGKGdPIQ6wbu9kCEXju)


**Synthia 4D:** They use the Synthia dataset to create 3D video sequences of driving. Each sequence consists of 4 stereo RGB-D images taken from the top of a car. They back-project the depth images to the 3D space to create 3D videos. The Synthia 4D dataset has an order of magnitude more 3D scans than Synthia 3D dataset.

![Fig-3](https://drive.google.com/uc?export=view&id=1JWF97ZuTOGKrs8ZI94bA1GNhkkklia4b)
*<center>**Figure 3**:  Segmentation results on the 4D Synthia dataset without noise addition for the input point cloud.</center>*

Visualization: 

3D network           |  4D network
:-------------------------:|:-------------------------:
![](https://drive.google.com/uc?export=view&id=1BKVdphoD_IB3LO8dlMWx0lqfVCoyd36Q)  |  ![](https://drive.google.com/uc?export=view&id=1SAZxy2UiuDMdB_tRcRNYAUPiUAbeYfi1)
![](https://drive.google.com/uc?export=view&id=1KTRgmOI10-v_N5SZ-THrRQ-TXV2FrCgJ)  |  ![](https://drive.google.com/uc?export=view&id=11Vusoeqxc2KYiBXJrs-6eok2EfIc-a-r)

**Stanford 3D Indoor:** The ScanNet and the Stanford Indoor datasets are one of the largest non-synthetic datasets, which make the datasets ideal test beds for 3D segmentation. They have achieved $+19\%$ mIOU on ScanNet, and $+7\%$ on Stanford compared with the original works.

![Fig-4](https://drive.google.com/uc?export=view&id=1WdX-9LVdnDoZ1BFb7ku4lmB3rFdTk8pV)
*<center>**Figure 4**:  Stanford Area 5 Test.</center>*

Visualization: 

RGB input        |  Predictions | Ground truth
:-------------------------:|:-------------------------:|:-------------------------:
![](https://drive.google.com/uc?export=view&id=1FvZlec-3bQpnrfWqxg84DqNwTJHLW2_h)  |  ![](https://drive.google.com/uc?export=view&id=1X1B1VxKzPejgpSGLMmyw07krYGVLTvqo)|![](https://drive.google.com/uc?export=view&id=1kpJNka8TpAf3tr22StCEcSDLk6dmCWpq) 
![](https://drive.google.com/uc?export=view&id=1_NCAj7ImcUnhcjxWITTbJenns681--WW)  |  ![](https://drive.google.com/uc?export=view&id=12q7szIeRcA5sEfo41x8wpuhnvIm2JP6w)|![](https://drive.google.com/uc?export=view&id=1_9uhAfaaburEMR2ZS7rtvI6gicpdagbp) 
## References

<a id="1">[1]</a> 
High-dimensional Convolutional Neural Networks for 3D Perception, Stanford University, Chapter 4. Sparse Tensor Networks.

<a id="2">[2]</a> Christopher Choy, JunYoung Gwak, and Silvio Savarese. 4D spatio-temporal ConvNets: Minkowski convolutional neural networks. In CVPR, 2019.