---
last_modified_on: "2022-09-11"
title: Model Compression Part 5.
description: Explore techniques used in a specific task, the Deep Keypoint Detection for Motion Driving task (Cont).
series_position: 8
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: advance"]
---
In this post, we will explore the specific method to lightweight the deep keypoint detection model: encoder-decoder for replacing the U-net like architecture.

## Goal

To find lightweight encoder-decoder architectures, with no residual connections between encoder and decoder. 

## Findings 

1. The majority of lightweight encoder-decoder (LED) networks was introduced in 2016-2019, followed the success of FCN and ResNet.
2. Some pioneering works in LED is [ENet](https://arxiv.org/pdf/1606.02147.pdf) and [ERFNet](http://www.robesafe.uah.es/personal/eduardo.romera/pdfs/Romera17iv.pdf).
3. From 2016-2019, the main difference in LED lays in the encoder, as the decoder is mostly just multiple blocks of ConvTransposed2D, or even just simply interpolation. Some use multiscale pyramid methods, e.g. [LEDNet](https://arxiv.org/pdf/1905.02423.pdf).
4. The encoder is often made up of "core" blocks. These blocks are fairly similar to each other and inspired by the convolution decomposition methods introduced in:
   1. Residual Bottleneck Block as introduced in ResNet
   2. Inception Module as introduced in InceptionNet
   3. Depthwise Separable Conv as used extensively in XceptionNet and MobileNet
   4. Shuffle Unit as introduced in ShuffleNet
5. From 2020 onwards, attention mechanism is applied more in decoders to recover details, and connections between encoder and decoder became a stable, e.g. [LAEDNet](https://www.sciencedirect.com/science/article/pii/S0045790622000799), [LRDNet](https://www.sciencedirect.com/science/article/pii/S0925231221010535)


## Main techniques used in encoder 
### [Asymmetric Convolutions](https://arxiv.org/abs/1412.5474)
- One of the most often used convolution decomposition method is to factorise a standard nxn conv by an nx1 conv followed by an 1xn conv, effectively an approximation of standard conv.
- Used in LEDNet, EDANet, ESNet, ERFNet. 
- When kernel size = 3, #parameters and computation cost is saved by 33%. 
 ![asymmetric-conv](https://drive.google.com/uc?id=1xfG90spuC2DqykhL0RdMeDgPRQZIPIge)
 *<center>**Figure 1**: a) Standard Convolution b) Asymmetric Convolution.</center>*

### (Inverted) Residual bottleneck connection
Given the tremendous success of ResNet in 2015, residual bottleneck blocks have been applied extensively in LED networks. The overall structure of the block follows closely to that in Figure 2. With the 3x3 conv replaced by 3x1 and 1x3.

![bottleneck-resnet](https://drive.google.com/uc?id=1Xk1jemcU__cHdfX5_hQ7AkHs6RqSZO7z)
   *<center>**Figure 2**: (a) Non-bottleneck residual (b) Bottleneck residual (c,d) Non-bottleneck-1D</center>* 
    
### Inception module 
The premise for Inception module is that salient parts in the image can have large variations in size, e.g. a dog can be very close to the camera and takes up most of the image, or very far and appears in a small corner. Thus, to capture details of varying sizes, we can have filters of multiple sizes operate on the same level in parallel, in order to provide multiscale features. 

Another variant of this is to have multiple dilated convolutions with different dilated rates on different branches, in order to broaden receptive field.   

![inception](https://drive.google.com/uc?id=18rmIYMcqOOSSk2KKD-gHERzIfBpsv6-g)
*<center>**Figure 3**: Inception Module.</center>*

### Shuffle unit in [ShuffleNet](https://arxiv.org/pdf/1707.01083v2.pdf)
Created to address the problem of stacked group conv: outputs from a certain group only relate to the inputs within the group, blocks info flow between channels groups and weakens representation.

![shuffle-unit](https://drive.google.com/uc?id=1F-DM9cWXNzuHrly_3TWLQPpYe_5uGDkt)
   *<center>**Figure 4**: (a,b) ShuffleNetV1 (c,d) ShuffleNetV2.</center>*

### Encoder results 

In this section, we will discuss the encoder of four LED networks, ERFNet (2017), EDANet (2018), ESNet (2019), and LEDNet (2019). They are all strictly encoder-decoder architectures with no residual connections between encoder and decoder. They also have learning in the decoder, not just simply interpolation. 

Regarding the feature extraction path, we have ERFNet and EDANet main blocks take on the non-bottleneck-1D approach. On the other hand, ESNet and LEDNet, written by the same authors, have multiple branches in their modules. Both uses dilated convolutions with different rates for different branches. LEDNet uses a split-transform-merge strategy, inspired by that of ShuffleNetV2. We can see that a combination of the above four techniques were used in constructing these blocks. 

![es_led](https://drive.google.com/uc?id=133gelEhqhlM0wMSPsfJ1BVOiYWIE3GDt)
*<center>**Figure 5**: (left) ESNet bottleneck module, (right) LEDNet bottleneck module</center>*


[Comparison results](https://docs.google.com/spreadsheets/d/1VRSiacvzy1_RqgylJod5b1760nr-h1IkVA6Aw5YhRpo/edit?usp=sharing)
