---
last_modified_on: "2022-09-11"
title: Model Compression Part 2.
description: Explore techniques used in a specific task, the Anti-alias
series_position: 5
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';  
import Highlight from '@site/src/components/Highlight';

Hourglass module is used as an anti-alisas function for The [First Order Motion](https://papers.nips.cc/paper/2019/file/31c0b36aef265d9221af80872ceb62f9-Paper.pdf).In  this post, we aim at finding possible methods for optimizing the efficiency of metioned task. Note that the CPU running time was measured on Intel Core i7-8700K CPU @ 3.70GHz.

As the hourglass module accepts images of size (64, 64, 3), an interpolation module is needed to reduce the spatial dimension of the input image. In an earlier version of the paper, Monkey-Net, only an `F.interpolate()` function with `mode='nearest'` was used to bring down the image size.  This, however, appeared to have produced aliasing artifacts. Thus, the authors have changed to use anti-alias interpolation in the First Order Motion paper. 

## Gaussian + Subsampling

The paper pre-filters the input image with a Gaussian filter before subsampling, inducing a blurring effect, and at the same time averaging out and reducing the sudden changes in intensity values, thus preserving these changes in the resized image in a smoother manner.
  
As the Gaussian kernel is explicitly coded out, there is **no learnable parameter** in this module. 

## Other methods

Regarding downsampling, the easiest technique is to select every other pixel according to the desired size. For instance, if we want to reduce the size of an image by half, we can simply skip every 2nd pixel. This technique, which is called subsampling, can lead to severe aliasing. 

We can also collapse every nxn window by taking the average of the window or the median, which is equivalent to Average Pooling or Median Pooling.

Finally, we can pre-filter the image, similar to the technique used in the paper, before subsample the image. More sophisticated filters can be used at the expense of computation costs.

## Experiments

The techniques tested include:
1. Baseline: Gassian filter + Subsampling
2. No prefilter: Just subsampling, i.e. select every 4th pixel
3. Method done in Monkey-Net: `F.interpolate()` with `mode=nearest` 
4. `F.interpolate()` with `mode=bicubic` and `antialias=True`
5. `F.interpolate()` with `mode=bilinear` and `antialias=True`
6. `F.interpolate()` with `mode=area`: Equivalent to apply an Adaptive Average Filter + Subsampling
4. Average Pooling: Equivalent to apply an Average Filter + Subsampling
5. [Median filter](https://gist.github.com/rwightman/f2d3849281624be7c0f11c85c87c1598) + Subsampling 
6. [Bilateral filter](https://gist.github.com/etienne87/af4210586a1b5316e287479d512fc5e5) + Subsamling

Let the keypoints predicted by the baseline model be $(b_1, b_2, ..., b_{10})$ and another interpolation technique be T $(t_1, t_2, ..., t_{10})$, and N the number of samples in the vox test dataset. The errors are calculated as follows:  

$$

\frac{1}{N} \sum_{i=1}^10 || b_i - t_i ||_2   

$$


## Results

| Interpolation methods                          | Error in keypoints | Average CPU Running time | 
|------------------------------------------------|--------------------|--------------------------| 
| Gaussian + Subsampling (Baseline)              | 0                  | 3.2464 ms                | 
| Subsampling                                    | 0.0653             | 0.1136 ms                |
| `F.interpolate` with `mode=nearest`            | 0.0653             | 0.3729 ms                | 
| `F.interpolate` with `mode=bicubic`            | 0.1728             | 1.1172 ms                | 
| `F.interpolate` with `mode=bilinear`           | 0.1711             | 0.8454 ms                | 
| `F.interpolate` with `mode=area`               | 0.1729             | 0.4388 ms                | 
| Average Pooling | 0.0786             | 0.3062 ms                |
| Median + Subsampling                           | 0.0689             | 56.9150 ms               | 
| Bilateral + Subsampling                        | 0.0395             | 95.5080 ms               | 

![anti-alias](https://drive.google.com/uc?id=1ZGRxK_CUj-i-QRBEiCvNRlN3LoKXnyBw)
*<center>**Figure 1**: Downsample results.</center>*

Visually, the results that are most similar to the baseline result are ones that are produced by bicubic, bilinear, and area. However, the keypoints predicted after using these techniques are the furthest away from the original technique. The bilateral fitler, on the other hand, didn't give the smoothest result, as the jagged lines along the left jaw and at the eyes are quite prominent. However, it gave the smallest deviance from the original keypoints. It also took the longest to run.  
