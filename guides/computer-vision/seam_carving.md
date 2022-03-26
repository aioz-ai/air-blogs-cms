---
last_modified_on: "2022-03-26"
title: Relationship between energy map and seam carving
description: Relationship between energy map and seam carving.
series_position: 15
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advantage", "guides: carving"]
---
Seam carving [1] is a content-aware image scaling method. The algorithm reduces the image size (width, height, or both) while maintaining the regions of interest. While seam carving is an ideal method for image resizing, it need an good energy map (importance map). Furthermore, images with too many details cause the seam carving to fail to give a good outcome. However, for the vast majority of our applications, in which we are only interested in a few important areas of the image, we still need to utilize seam-carving with an appropriate energy computing method.

In this post, we will go through two energy algorithms, i.e., the center importance method (CI) (computed by summing the difference in the value of a pixel with its surrounding pixels) and Lab saliency (LS) [2]. 

## Seam carving

![Figure 1](https://vision.aioz.io/f/c2c7a19226114ce1bce9/?dl=1)
*<center>**Figure 1**: Overview pipeline for seam carving.</center>*

Formally, for an $n \times m$ image, we define a vertical seam to be:
where $x$ is a mapping $x:[1, \ldots, n] \rightarrow[1, \ldots, m]$. That is, a vertical seam is an 8-connected path of pixels in the image from top to bottom, containing one, and only one, pixel in each row of the image. 

$$

\mathbf{s}=\{(x(i), i)\}_{i=1}^{n} \text {, s.t. } \forall i,|x(i)-x(i-1)| \leq 1

$$

As shown in Figure 1, the image is used to compute an energy map $E$, the we find the top $k$ - optimal seam lines, where an optimal seam line $s^*$ is

$$

s^{*}=\min_{{s}} E({s})=\min_{{s}} \sum_{i=1}^{n} E\left(V\left(s_{i}\right)\right)

$$

Where $V(p)$ is the pixel value at position $p$.

## Center importance (CI)

![Figure 2](https://vision.aioz.io/f/1d36d770c5704b238195/?dl=1)
*<center>**Figure 2**: Illustration of Center important method.</center>*
Following Figure 2, we can formulate the energy map as: 

$$

E(x,y) = \sum_{i=0}^{3}|V(x,y)-V(x+{F_1}_i,y+{F_2}_i)|

$$

Where $(x,y)$ is the position, $E(p)$ is the energy value at position $p$,  $V(p)$ is the pixel value at position $p$, $F_1=(-1,1,0,0)$, $F_2=(0,0,1,-1)$. For each position, its energy depends on how different is its value to its neighbors value. Therefore, only edges and lines are highlighted in the energy map. 

## Lab saliency (LS)

![Figure 3](https://vision.aioz.io/f/cbb1f255680f4285b2fb/?dl=1)
*<center>**Figure 3**: Original images and their saliency map using LS.</center>*

To produce the saliency map in Figure 3, LS method first apply Gaussian blur on the original image, then convert that blurred image to L$\alpha\beta$ color space, let call this converted image $I = (I_L, I_\alpha, I_\beta) \in \mathbb{R}^{n\times m \times 3}$. For each channel of $I$, we compute its mean value $I_\mu = (I_{L\mu}, I_{\alpha\mu}, I_{\beta\mu}) \in \mathbb{R}^{3}$. The saliency map $S \in \mathbb{R}^{n \times m}$ is computed as: 

$$

S(x,y) =\sum_{i\in\{L,\alpha,\beta\}}(I_i(x,y)-I_{i\mu})^2

$$

## Results and conclusion

![Figure 4](https://vision.aioz.io/f/656738b74a0c41109e13/?dl=1)
*<center>**Figure 4**: Results of seam carving using CI and LS methods. The final results (green area) are separated with the first column is discarding 25% of width, and the second column is discarding 55% of width.</center>*


The CI method emphasizes the pixels which have different colors than their neighbors. That means edges and lines that construct object shapes will be highlighted. Therefore, when we only want to reduce a small number of seam lines, CI is stable enough to use. But when re-scaling to a much smaller size, like more than 50% of the original width, the results are guaranteed to be bad, as shown in Figure 4 green area second column . 

In theory, the saliency technique is more promising. We can ensure that only insignificant regions are excluded in the seam carving procedure if we can determine the essential object in the image. However, in practice, developing a suitable saliency detection method is difficult; if it succeeds in detecting the right regions, we are good; however, if it fails, the seam carving becomes useless. In our case, we can see that the LS has the tendency to make the pixels at edges have low value, which means they are the first thing seam carving wants to discard. As shown in Figure 4 first row, the right pillar of the tower gets eaten horribly. But in case the saliency detection succeeds, as we can see in Figure 4 second row, the results are impressive. 

After comparing the two algorithms qualitatively, we can see that both methods have their own advantages and disadvantages. Therefore, there is no superior one between the two. 

## References

[1] Avidan, Shai, and Ariel Shamir. Seam carving for content-aware image resizing.In *ACM SIGGRAPH 2007 papers*. 2007. 10-es.

[2] Radhakrishna Achanta, Sheila Hemami, Francisco Estrada, and Sabine Susstrunk. Frequency-tuned salient region detection. In *2009 IEEE Conference on Computer Vision and Pattern Recognition*, pages 1597-1604, 2009