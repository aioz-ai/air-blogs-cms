---
last_modified_on: "2022-13-02"
title: Exemplar Geometry-Aware Neural Style Transfer (Part2)
description: Survey some Exemplar Geometry-Aware Neural Style Transfer (Continue)
series_position: 12
author_github: https://github.com/aioz-ai
tags: ["type: tutorial", "level: medium"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In previous blog, I have presented a survey on Geometry-Aware Neural Style Transfer (NST) methods and compare them with other standard NST methods. Furthermore, some weaknesses of current Geometry-Aware NST methods are also analyzed. In this blog, we continue to examine the effectiveness of style aware image translation in terms of different configs of hyper-parameters. Note that, all of the experiments are conducted based on the implementation of DST, NST, AdaIN, and FastDST.

IN this blog, all the images follow the structure: style image - content image with facial landmarks - style image with facial landmarks.

##  1. Result on various style images

In this experiment, I do style transfer using only one content image and various style images
The style-weight is set to 0.5.


![Fig-1](https://vision.aioz.io/f/9736c2271f1d4c9eac0c/?dl=1)
*<center>**Figure 1**: Different style transfer results from a static source image</center>*

## 2. Result on various content images
In this experiment, I do style transfer using only one style image and various content images
The style-weight is set to 0.5.


![Fig-2](https://vision.aioz.io/f/c299ac7f30ec43c0b3f2/?dl=1)
*<center>**Figure 2**: Different style transfer results from different source image on a specific style image</center>*

![Fig-3](https://vision.aioz.io/f/de2d2a849d8c4ef1840d/?dl=1)
*<center>**Figure 3**: Different style transfer results from different source image on a specific style image (Cont)</center>*


##  3.  Result on various style-weights
In this experiment, I do style transfer using only one style image and onecontent image
The style-weight is set in range: 5.0, 4.5, 4.0, 3.5, and so on to 0.5, 0.1
The lower style-weight, the more remaining content

![Fig-4](https://vision.aioz.io/f/cb9d854985a6429d87e6/?dl=1)
*<center>**Figure 4**: Different style transfer results from different style weights</center>*

##  4.  Demo
The implementation based on AAMS
Colab Demo: https://colab.research.google.com/drive/1mGxE3ng8SCYunBpMmiHLA7tdnWhc7iVl?usp=sharing
## 5. Conclusion

### Pros:
- Faster than DST, original NST 
- Well keep semantic regions of content image (e.g. eyes, nose,...) (outperforms AdaIN, NST)
- Do arbitrary style transfer (ASMAGAN can not)

### Cons:
- Slower than AdaIN, ASMAGAN
- Sensitive with style image 

Considered version: DST without deformation loss can improve performance and the sensitive property of AAMS. However, we must suffer from the low inference time.

## 6. Reference
"Attention-Aware Multi-Stroke Style Transfer - CVF Open Access." https://openaccess.thecvf.com/content_CVPR_2019/papers/Yao_Attention-Aware_Multi-Stroke_Style_Transfer_CVPR_2019_paper.pdf. Accessed 7 Oct. 2021.