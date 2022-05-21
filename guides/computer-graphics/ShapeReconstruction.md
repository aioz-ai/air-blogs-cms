---
last_modified_on: "2022-05-20"
title: Shape reconstruction from a single RGB video
description: a study to reconstruct Shape from a single RGB video
series_position: 9
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy"]
---
3D reconstruction from an RGB video is a difficult task due to how limited information from RGB alone. Many approaches are using 3D mesh templates to help with the reconstruction (e.g. SMPL-X [2] for humans, SMAL [3] for animals). However, those template-based approaches can not handle novel object categories. In this post, I introduce LASR [1], which can reconstruct a 3D mesh from a single RGB video.  

## Shape reconstruction from a single RGB video
![](https://vision.aioz.io/f/51d1359f23534f2e8138/?dl=1)
Figure 1. The simplified version of LASR. Blue cells are parameters that we need to optimize for the input video. Red cell indicates the loss. Grey cells are values and operators that are not optimized in the process. 

From Figure 1 we can see there are 3 main inputs:  
- An RGB video with a sequence of frames $\{I_t\}$, with $t$ indicates the $t$-th frame. 
- Silhouettes $\{S_t\}$.
- Optical flows from the input video $\{u_t\}$.

and the outputs we want are:
- $\mathbf{S}$: rest shape of the object.
- $\mathbf{G_t}$: the time-varying transformations.
- $\mathbf{K_t}$: the camera intrinsics. 

From those outputs, to compute the reconstruction loss, we need to use a differentiable renderer [4] to render them into camera space. The intermediate outputs for loss computation are:
- Rendered color images $\{\hat{I}_t\}$.
- Rendered silhouettes $\{\hat{S}_t\}$.
- Rendered flows $\{\hat{u}_t\}$.

Then we only need to optimize the difference between the rendered intermediate outputs and the inputs to reconstruct the object of interest.


## Modeling
![](https://vision.aioz.io/f/11ea70421c0d4c8f9cbe/?dl=1)
Figure 1. The detailed version of LASR. Blue cells are parameters that we need to optimize for the input video. Red cell indicates the loss. Grey cells are values and operators that are not optimized in the process. 

**Rest shape:** Object shape $\mathbf{S = \{\bar{V}, F\}}$ with vertices $\mathbf{\bar{V}} \in \mathbb{R}^{N\times 3}$ and a fixed topology (faces) $\mathbf{F} \in \mathbb{R}^{M \times 3}$.

**Linear-blend skinning:** To map a vertex from the rest position to a new position corresponding to each frame, we use linear-blend skinning. Each vertex is transformed by linearly combining the weighted bone transformations in the object coordinate frame and then transformed to the camera coordinate frame.

$$
    \mathbf{V}_{i,t} = \mathbf{G_{0,t}} \left(\sum_b\mathbf{W}_{b,i}\mathbf{G}_{b,t}\right)\mathbf{\bar{V}}_i
$$
with skinning weight matrix $\mathbf{W} \in \mathbb{R}^{B \times N}$, $\mathbf{G_{0,t} = (R_0 | T_0)}$ is the root transformations,  $\mathbf{G_{i,t}} = (\mathbf{R}_{i,t} | \mathbf{T}_{i,t})$ is the transformation matrix for $t$-th frame and $b$-th bone (or joint), with $\mathbf{R}$ denotes rotation parameters, $\mathbf{T}$ denotes translation parameters, $i$ is the vertex index, and $b$ is the bone index. 

To transform $\mathbf{\bar{V}}_i$, we use a weighted transformation matrix to move the vertex into that space, then transform again to follow the root space. So the complexity grows with the number of bones and vertices.


**Skinning weights.** The skinning weights are defined as a mixture of Gaussians with $B$ components. The probability of assigning vertex $i$ to bone $b$ is:
$$
    W_{b,i} = C e^{-\frac{1}{2}(\mathbf{V}_i - \mathbf{J}_b)^T\mathbf{Q}_b(\mathbf{V}_i - \mathbf{J}_b)}
$$
where $\mathbf{J}_b \in \mathbb{R}^3$ is the center of $b$-th Gaussian, $\mathbf{Q}_b \in \mathbb{R}^{3 \times 3}$ is the corresponding precision matrix that determines the orientation and radius of a Gaussian, and $C \in \mathbb{R}$ is a normalization factor. In the implementation, $B$ centers are initialized using K-mean before starting optimization.

Finally, we apply a perspective projection $\mathbf{K_t}$ before rasterization, where the principle point $(p_x,p_y)$ is the same across all frames and the focal length $f_t$ are learnable parameters.

In the implementation, instead of optimizing explicit time-varying parameters, LASR uses ResNet18 with the input image $\mathbf{I}_t$ as shown in Figure 2:
$$
    \psi_w(\mathbf{I}_t) = (\mathbf{K, G_0,...,G_B})_t
$$
$\psi_w(\mathbf{I}_t) \in \mathbb{R}^{1+7(B+1)}$, with 1 for focal length (for $\mathbf{K}$), 4 parameters for each bone rotation, and 3 for each translation.

Note that to compute the flow for rendering we predict the next position from the previous timestep. For example, to compute the forward flow, we take surface positions $\mathbf{V}_t$ in frame $t$ and compute their locations $\mathbf{V}_{t+1}$ in the next frame, then use the different as flow.


## Loss
**Reconstruction losses.** Given a pair of rendered outputs $(\hat{S}_t, \hat{I}_t, \hat{u}_t)$ and ground-truth $(S_t, I_t, u_t)$, the reconstruction loss is 
$$
    L_{re} = \beta_1\|\hat{S}_t-S_t\|^2_2 + \beta_2\|\hat{u}_t - u_t\|_2 + \beta_3\|\hat{I}_t-I_t\|_1 + \beta_4pdist(\hat{I}_t,I_t)
$$
Where $\{\beta_1,...,\beta_4\}$ are weights empirically chosen and $pdist(\cdot, \cdot)$ is the perceptual distance measure by AlexNet pretrained on ImageNet.

**Shape and motion regularization:** To enforce a smoothing surface, LASR uses Laplacian smoothing 
$$
    L_{shape} = \|\mathbf{\bar{V}}_i - \frac{1}{|N_i|}\sum_{j\in N_i}\mathbf{\bar{V}}_j\|^2
$$
where $N_i$ is a set of vertex indices adjacient to $\mathbf{\bar{V}}_i$.

Motion regularization consists of as-rigid-as-possible (ARAP) and least deformation loss. The least deformation term enforces the deformation
from the rest shape to be as small as possible. And ARAP term enforces the deformation between consecutive frames to be small, hence the movement will be more natural.
$$
\begin{align*}
    L_{ARAP} &= \sum_{i=1}^V\sum_{j\in N_i}|\|\mathbf{V}_{i,t}-\mathbf{V}_{j,t}\|_2 - \|\mathbf{V}_{i,t+1}-\mathbf{V}_{j,t+1}\|_2 | \\ 
    L_{least} &= \sum_{i=1}^V\|\mathbf{V}_{i,t}-\mathbf{\bar{V}}_{i,t}\|_2
\end{align*}
$$


## Conclusion
![](https://vision.aioz.io/f/ac20089643784f85a1cd/?dl=1)
Figure 3. The reconstructed output uses camel video as input. 

In this post, I describe a method that by utilizing a differentiable renderer and linear-blending skinning, we can achieve very good reconstruction results given only RGB information as shown in Figure 3. While the results are impressive, the obvious drawback is the occlusion problem. Therefore, we can have a lot of room to enhance it, for example by using information from multiple viewpoints.  

## References 
[1] Yang, Gengshan, et al. "Lasr: Learning articulated shape reconstruction from a monocular video." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.

[2] Pavlakos, Georgios, et al. "Expressive body capture: 3d hands, face, and body from a single image." Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. 2019.

[3] Zuffi, Silvia, et al. "3D menagerie: Modeling the 3D shape and pose of animals." Proceedings of the IEEE conference on computer vision and pattern recognition. 2017.

[4] Liu, Shichen, et al. "Soft rasterizer: A differentiable renderer for image-based 3d reasoning." Proceedings of the IEEE/CVF International Conference on Computer Vision. 2019.