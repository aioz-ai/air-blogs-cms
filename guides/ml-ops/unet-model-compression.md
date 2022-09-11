---
last_modified_on: "2022-09-03"
title: Model Compression Part 1.
description: Explore techniques that can possibly be replaced within the layers of the standard U-Net. 
series_position: 4
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

We will explore techniques that can possibly be replaced within the layers of the standard U-Net. There are three techniques that are widely used in U-Net:

1. Convolution Layer
2. Normalisation
3. Interpolation 

In this post, we will discover the possible replacements of mentioned methods. In particular:
1. We consider to replace Convolution by Depthwise Separable Convolution 
2. We consider different variants of Normalization including BatchNorm,LayerNorm, InstanceNorm, and GroupNorm
3. We consider interpolation with convolution layers and transposed convolution layers

## Depthwise Separable Convolution 

In standard convolutions, we convolve the whole 3D filter with the input, which may lead to a lot of redundant calculations. Instead, we can adopt the depthwise separable convolution (Dep-Sep-Conv) introduced in [MobileNet](https://arxiv.org/pdf/1704.04861.pdf). Dep-Sep-Conv breaks down a normal convolution layer into two parts:
1. **Depthwise convolution**: Convolve each channel separately with its own 2d filter. The number of output channels equals to the number of input channels.  
2. **Point-wise convolution**: The information across the channels will be incorporated using a pointwise convolution, as illustrated in Figure 1. 

![depthwise-sep-conv](https://drive.google.com/uc?id=1ov4mIYGmBZAc8ydkcU45ejtSJUtutjhB)
*<center>**Figure 1**: Depthwise Separable Convolution.</center>*


To understand the efficiency of depthwise separable convolution, we need to investigate the reduction it brings in parameters and computations. First, the computation cost for standard convolution is   

$$

D_k \times D_k \times M \times N \times D_f \times D_f  

$$

where:
- D_k = kernel size,
- M = input channels,
- N = output channels,
- D_f = feature map size 

For depthwise conv, as we convolve one 2D filter for each channel, each channel requires $D_k \times D_k \times D_f \times D_f$ multiplications, and M channels would require  $M \times D_k \times D_k \times D_f \times D_f$.

For pointwise convolution, it is exactly standard convolution with $D_k = 1$, thus requires $M \times N \times D_f \times D_f$ multiplications. 

Hence, Dep-Sep-Conv in total requires 

$$

M \times D_k \times D_k \times D_f \times D_f + M \times N \times D_f \times D_f

$$

corresponds to a reduction of $\frac{1}{N} + \frac{1}{D_K^2}$ compared to standard conv.

Indeed, by replacing every convolution layer in the hourglass module to depthwise separable convolution layer, we reduced the running time by half and the number of parameters more than 10 times. 

| Convolution Methods      | Parameters (M) | Mult-adds (M) | Time (ms) | 
|--------------------------|----------------|---------------|-----------|
| Normal conv              | 14.13          | 989.31        | 13.0184   |
| Depthwise Separable Conv | 1.62           | 120.30        | 6.9928    |

While the gain in efficiency is large, the reduction in parameters is too large to expect the model performance will remain the same. We can make up for the lost of parameters by adding extra layers or having multiple convolution layers every downsample/upsample block.  


## Normalisation method
Another candidate to be replaced is Batch Normalisation. While it only takes up 4% running time in encoder and 2.67% in decoder, it is still useful to see other options available. 


Batch Normalisation was introduced for the purpose of reducing internal covariance shift, lead to stabler training and faster convergence. There are drawbacks to BN however, including increased error when the batch size is small, caused by inaccurate batch statistics during estimation. It is also said that BN brings negative effect in pixel-level task as it blurs the details of a single image. Thus, some variations of norms have been introduced.

1. **Layer Norm**: normalises the activations along the feature dimension (i.e. all features per example). s
   
2. **Instance Norm**: normalise across each channel in each training example. IN layers use instance statistics computed from input data in both training and evaluation mode. Originally proposed for texture stylization.

3. **Group Norm**: GN divides the channels into groups and computes within each group the mean and variance for normalization. Also uses statistics in both training and evaluation mode. It is a trade-off between Layer Norm and Instance Norm.

Due to the very small running time of norm layers, and the variations in running time of the model in general, instead of replacing every norm layers like as we did for conv, we test for efficiency by applying each normalization method on an input of size (1,1024,2,2), repeated 1000 times. For the GroupNorm case, we set the `num_groups` parameter to be `num_channels//2`. 


| Norm         | Params | Batch = 1 (ms) | Batch = 32 (ms) |
|--------------|--------|----------------|-----------------| 
| BatchNorm    | 2,048  | 0.0407         | 0.6520          |
| LayerNorm    | 8,192  | 0.0366         | 0.6376          |
| InstanceNorm | 2,048  | 0.0665         | 1.0005          |
| GroupNorm    | 2,048  | 0.0461         | 0.9654          | 

It appears that with batch size = 1, LayerNorm has the shortest running time despite the higher number of parameters. While for the remaining three, the number of parameters are the same, yet BatchNorm has the shortest running time and InstanceNorm consistently is the slowest. 
When batch size is larger (32 in our case), BatchNorm & LayerNorm have comparable running time ~ 0.64 ms, and InstanceNorm & GroupNorm are on the slower end ~ 1 ms.

The results show that the inference time (when batch size is usually 1) is not drastically different among the four normalisation methods. Batch Norm and Layer Norm are evidently preferred for shorter training time (when batch size is large). We can test more on the affect LayerNorm has on accuracy and potentially replace LayerNorm with BatchNorm.


## Interpolation 
In every upsampling block, the spatial dimension of the feature maps are increased using `F.interpolate()` with `mode='nearest'`, followed by a Conv2d layer. This interpolation method is formulaeic and does not have any trainable parameters. One possible alternative to this method is to replace `Interpolate+Conv` to a `ConvTranspose2d` layer. However, a problem often associate with Transposed Convolutions or Deconvolution layers is the [checkerboard pattern problem](https://distill.pub/2016/deconv-checkerboard/).

![checkerboard-pattern](https://drive.google.com/uc?id=10oMktQ8QbqT1qzzVAMVwNbuWl5_XMTiq)
*<center>**Figure 2**: Checkerboard Pattern due to Transposed Convolution.</center>*


Efficiency-wise, we compare the two methods by applying them on an input of size (1, 1024, 2, 2), with the desired output to be (1, 512, 4, 4).

| Upsample method      | Parameters | Mult-adds | Running time | 
|----------------------|------------|-----------|--------------| 
| Interpolation + Conv | 4,719,104  | 75.51M    | 0.16 ms      |
| Transposed Conv      | 4,719,104  | 75.51M    | 0.23 ms      |

It can be seen that while the number of parameters and operations are the same, the running time of the first method is on average 0.07 ms faster than Transposed Convolution. This result is consistent over multiple runs. 
