---
last_modified_on: "2022-07-02"
title: Applying Neural radiance field in 3D reconstruction from monocular video (Part 1)
description: a study of 3D reconstruction from monocular video
series_position: 10
author_github: https://github.com/aioz-ai
tags: ["type: theory", "level: medium"]
---
Optimizing explicitly on a sphere canonical mesh-like [LASR](https://blog.ai.aioz.io/guides/computer-graphics/ShapeReconstruction/)[1] can easily get stuck due to local minima. To alleviate this issue, BANMo[2] optimizes on implicit space, called canonical space, using volume rendering and an optimizing scheme similar to NeRF[3]. In this post, I introduce the general process of BANMo and its losses, including reconstruction loss and feature matching loss. Besides, the detailed information of reconstruction loss is also discussed. In the next post,  I will mention about the feature matching loss and visualization results.

## Overview
![](https://vision.aioz.io/f/d7cf7fe03ce948a894fa/?dl=1)
Figure 1. The simplified version of BANMo. Blue cells are parameters that we need to optimize from the input video. Red cell indicates the loss. Grey cells are values and operators that are not optimized in the process. 


Similar to [LASR](https://blog.ai.aioz.io/guides/computer-graphics/ShapeReconstruction/)[1], the inputs for BANMo are:
- An RGB video with a sequence of frames $\{I_t\}$, with $t$ indicates the $t$-th frame. 
- Silhouettes $\{S_t\}$.
- Optical flows from the input video $\{u_t\}$.

And the outputs are:
- $\mathbf{S}$: rest shape of the object.
- $\mathbf{G_t}$: the time-varying transformations.
- $\mathbf{K_t}$: the camera intrinsics. 


But because BANMo optimizes the shape implicitly, the real output is the 3D canonical space $\mathbf{X^*}$, typically it is a 3D cube with grid-like 3D points inside it. Then we perform the Marching cube to extract the mesh $\mathbf{S}$. Because our shape is implicit, $\mathbf{G_t}$ and $\mathbf{K_t}$ are transformation matrices on canonical space, not the shape itself. 

From Figure 1, we can see there is a total of 4 main losses, compared to 1 loss (reconstruction loss) of LASR. Despite the fact that BANMo still keeps the same idea of reconstruction loss (compute the discrepancy of image ground truth and rendered shape), the underlying computations are completely different. Details are shown in other sections below.  

## Reconstruction loss 
![](https://vision.aioz.io/f/e11c1835398f47219938/?dl=1)
Figure 2. The general flow of color loss, or RGB loss.  

![](https://vision.aioz.io/f/b8d4bc679d7d48a2ae17/?dl=1)
Figure 3. The general flow of silhouette loss.

![](https://vision.aioz.io/f/e3203efeb88d4cd19eee/?dl=1)
Figure 4. The general flow of optical flow loss.

BANMo's reconstruction loss comprises of 3 sub losses: color loss, silhouette loss, and optical flow loss, and they are shown in Figure 2,3,4. Those three losses share the same idea of computing the $L_2$ distance between the observed value at position (x,y) (or $\mathbf{x^t}$ in the figures) in the image plane and the corresponding rendered value from volume rendering. 

For color loss, from the position (x,y) we sample the 3D points from the ray and use a mapping function from camera space to canonical space $\mathcal{W}^{t,\leftarrow}$. Subsequently, we can compute two components that make up a color in camera space, i.e. opacity and the original color. The opacity is represented by the visibility distribution $\tau$, where its element $\tau_{i}=\prod_{j=1}^{i-1} p_{j}\left(1-p_{i}\right)$, $p_{i}=\exp \left(-\sigma_{i} \delta_{i}\right)$ is the probability that the light is transmitted through the interval $\delta_{i}$, and $\sigma_{i}=\sigma\left(\mathcal{W}^{t, \leftarrow}\left(\mathbf{X}_{i}^{t}\right)\right)$ is the density from the same MLP like color. Specifically, the color and density are computed as 
$$
\begin{aligned}
\mathbf{c}^{t},\sigma &=\mathbf{M L P}_{\mathbf{c}}\left(\mathbf{X}^{*}\right)
\end{aligned}
$$

With the same visibility distribution, we can compute the silhouette loss (Figure 3) by summing all $\tau_i$ of N sampled points from the ray. To compute the optical flow loss, we need to compute the frame $t'$-th position in image space so that we can compute the optical flow. Therefore, the loss uses a projection $\mathcal{W}^{t, \rightarrow}$ to map from canonical space to camera space at frame $t'$-th and combine the same visibility distribution to predict $(x',y')$ in image space, as shown in Figure 4.

## Next time
In the  next part of this series, I will mention about the feature matching loss in using Neural radiance field for reconstructing 3D model from monocular video. Then, I will introduce about the visualization results and comparison with other methods.

## References
[1] Yang, Gengshan, et al. "Lasr: Learning articulated shape reconstruction from a monocular video." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.

[2] Yang, Gengshan, et al. "Banmo: Building animatable 3d neural models from many casual videos." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2022.

[3] Mildenhall, Ben, et al. "Nerf: Representing scenes as neural radiance fields for view synthesis." European conference on computer vision. Springer, Cham, 2020.