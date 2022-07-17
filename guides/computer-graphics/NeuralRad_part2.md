---
last_modified_on: "2022-07-02"
title: Applying Neural radiance field in 3D reconstruction from monocular video (Part 2)
description: a study of 3D reconstruction from monocular video
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: theory", "level: medium"]
---
In previous post, I have introduced the general process of BANMo and its losses, including reconstruction loss and feature matching loss. Besides, the detailed information of reconstruction loss is also discussed. In this post,  I will mention about the feature matching loss and visualization results.

## Feature matching loss 
![](https://vision.aioz.io/f/d475e365c2234ca1a81e/?dl=1)
Figure 1. The general flow of feature matching loss.  

Reconstruction loss only captures the frame feature, which leaves out the temporal information between frames. Feature matching loss constructs a pipeline that captures this information. Given a pixel at $x^{t}$ of frame $t$-th, we need to find the best-matched point $\mathbf{X}^{*}$ in the canonical space. First, we compute the expected ray intersection in the canonical space:
$$
\mathbf{X}^{*}\left(\mathbf{x}^{t}\right)=\sum_{i=1}^{N} \tau_{i}\left(\mathcal{W}^{t, \leftarrow}\left(\mathbf{X}_{i}^{t}\right)\right)
$$

Then we compute the 3D surface point corresponding to the position $\mathbf{x}^{t}$, we apply soft argmax descriptor matching:
$$
\hat{\mathbf{X}}^{*}\left(\mathbf{x}^{t}\right)=\sum_{\mathbf{X} \in \mathbf{V}^{*}} \tilde{\mathbf{s}}^{t}\left(\mathbf{x}^{t}\right) \mathbf{X}
$$
where $\mathbf{V}^{*}$ are sampled points in a canonical 3D grid, and $\tilde{\mathbf{s}}$ is a normalized distribution: $\tilde{\mathbf{s}}^{t}\left(\mathbf{x}^{t}\right)=\sigma_{\text {softmax }}\left(\left\langle\psi_{I}^{t}\left(\mathbf{x}^{t}\right), \psi(\mathbf{X})\right\rangle\right)$, where $\langle., .\rangle$ is the cosine similarity score, and $\psi\left(\mathbf{X}\right)=\mathbf{M L P}_{\psi}\left(\mathbf{X}\right) \in \mathbb{R}^{16}$ is the embedding of a canonical 3D point, and $\psi_{I}$ is computed from DensePose CNN.  

Finally, the feature matching loss is:
$$
\mathcal{L}_{\text {match }}=\sum_{\mathbf{x}^{t}}\left\|\hat{\mathbf{X}}^{*}\left(\mathbf{x}^{t}\right)-\mathbf{X}^{*}\left(\mathbf{x}^{t}\right)\right\|_{2}^{2}
$$

As shown in Figure 1, the feature matching loss is basically the loss between two different ways to derive the 3D point from 2D position in image space. To further enforce the relationship between 2D and 3D space, we constrain them directly on their own space. Specifically, we constrain the 2D space by forcing the image projection after forward warping of $\hat{\mathbf{X}}^{*}\left(\mathbf{x}^{t}\right)$ to land back on its original $2 \mathrm{D}$ coordinates:
$$
\mathcal{L}_{2 \mathrm{D}-\mathrm{cyc}}=\sum_{\mathbf{x}^{t}}\left\|\Pi^{t}\left(\mathcal{W}^{t, \rightarrow}\left(\hat{\mathbf{X}}^{*}\left(\mathbf{x}^{t}\right)\right)\right)-\mathbf{x}^{t}\right\|_{2}^{2}
$$
where $\Pi t^{t^{\prime}}$ is the camera parameters.


And we constrain the 3D space by encouraging a sampled 3D point in the camera space to be backward deformed to the canonical space and forward deformed to its original location:
$$
\mathcal{L}_{3 \mathrm{D}-\mathrm{cyc}}=\sum_{i} \tau_{i}\left\|\mathcal{W}^{t, \rightarrow}\left(\mathcal{W}^{t, \leftarrow}\left(\mathbf{X}_{i}^{t}\right)\right)-\mathbf{X}_{i}^{t}\right\|_{2}^{2}
$$
where $\tau_{i}$ is the opacity we define in reconstruction loss.

## Visualization 
![](https://vision.aioz.io/f/26c4c0848c07470c9cec/?dl=1)
Figure 2. The sample output from BANMo

I have introduced BANMo, a solution to reconstructing a 3D model from a monocular RGB video problem using volume rendering like NeRF.
While BANMo is much more complicated than LASR, BANMo empirically has proven that it performs better than LASR, especially in the case of long video (more than 100 frames). The output in Figure 2 is computed from a 10-minute-long video, I only show the first few frames to demonstrate its ability.

As in the figure, the results validate the effectiveness of BANMo compared to other baselines. However, BANMo computation cost is high which lead to the fact that BANMo is hard to achieve run-time proccess.

## Conclusion
In two part of this series, I have mention about the reconstruction and feature matching losses in using Neural radiance field for reconstructing 3D model from monocular video. The method that I mainly focus is BANMo achieve better results than other baselines. However, since the complex of deep learning model is too high to apply real time computation, compression or decomposition methods are considered to be integrated for dealing with the problem.

## References
[1] Yang, Gengshan, et al. "Lasr: Learning articulated shape reconstruction from a monocular video." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.

[2] Yang, Gengshan, et al. "Banmo: Building animatable 3d neural models from many casual videos." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2022.

[3] Mildenhall, Ben, et al. "Nerf: Representing scenes as neural radiance fields for view synthesis." European conference on computer vision. Springer, Cham, 2020.