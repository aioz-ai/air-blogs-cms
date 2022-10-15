---
last_modified_on: "2022-10-15"
title: Introduction to Human Body Model
description: Introduction to Skinned Multi-Person Linear Model (SMPL), which is a popular models used in both academia and industry computer graphics
series_position: 14
author_github: https://github.com/aioz-ai
tags: ["type: theory", "level: medium"]
---

In the previous post, we have introduced the concept of Linear Blend Skinning (LBS), which is a highly important technique in graphics and animation. However, LBS is just so simple that it has many problems such as volume collapsing and candy wrapper (see Figure 4 in [this post](https://ai.aioz.io/guides/computer-graphics/CharacterAnimationandSkinning_part2/)), which can lead to unrealistic body deformation. Furthermore, in many practical problems, we may want a realistic body model that has the ability to represent any human body in the real world and the model should not be too complicated to use. In this article, we will go into a human body model called Skinned Multi-Person Linear Model (SMPL) [1], which utilize a very effective data-driven strategy to address all of these concerns. 


## Linear Blend Skinning Recall

To recap, linear blend skinning parameters contain: 
- Mesh vertices in rest pose: $\mathbf{T} \in \mathbb{R}^{3N}$
- Joint locations: $\mathbf{J} \in \mathbb{R}^{3K}$. 
- Joint rotations (pose parameters): $\mathbf{\theta} \in \mathbb{R}^{3K}$
- Blend skinning weights: $\mathbf{W} \in \mathbb{R}^{N \times K}$

where $N$ is the number of vertices, and $K$ is the number of articulated body joints. The local bone transformation matrix $\mathbf{M}_k$ at joint $k$ depends on the joint location and the joint location, which can be computed as:
$$
\mathbf{M}_k = \begin{bmatrix}
    Rot(\theta_k) & \mathbf{j}_k \\
    \mathbf{0}^T & 1
    \end{bmatrix}
$$
The function $Rot(\theta_k)$ transforms the joint rotation vector $\theta_k$ into a $3\times 3$ rotation matrix and $\mathbf{j}_k$ is the offsets of the current joint $k$ from its parent ($\mathbf{J}_k - \mathbf{J}_{p(k)}$). To obtain the world transformation $\mathbf{G}_k(\theta,\mathbf{J})$ at joint $k$, we simply accumulate the local transformation matrix along the kinematic tree up to the root joint: $\mathbf{G}_k(\theta,\mathbf{J}) = \Pi_{j\in A(k)}\mathbf{M}_j$, where $A(k)$ is the ordered list of joints in the path from the root to joint $k$. Finally, the position of the vertex $i$ after LBS is calculated as 
$$
\mathbf{t}_i' = \sum_{k=1}^K w_{ik}\mathbf{G'}_k(\theta,\mathbf{J})\mathbf{t}_i, \tag{1} \\
\mathbf{G'}_k(\theta,\mathbf{J}) = \mathbf{G}_k(\theta,\mathbf{J})\mathbf{G}_k(\theta^0,\mathbf{J})^{-1}
$$

where $\mathbf{G}_k(\theta^0,\mathbf{J})$ is the world transformation of joint $k$ according to the rest pose $\theta^0$; $\mathbf{t}_i$ is the vertex position in the rest pose; $\mathbf{t}'_i$ is the transformed vertex corresponding to the desired pose $\theta$. 

## Pose corrective blendshapes
To deal with the errors of linear blend skinning, in practice, people often artistically sculpt the blendshape, and then add it into the template mesh to correct the posed mesh. **A blendshape is a vector of vertex displacements to the original template mesh.** As shown in Figure 1, after adding the blendshape to the original mesh, the artifacts around the elbow and the hip go away, and the posed mesh is correct.

![](https://vision.aioz.io/f/093eef6177c740dba82d/?dl=1)*<center>**Figure 1**. Skinning results with and without added corrective blendshapes.</center>* 


However, it is quite a tedious and expensive process. Inspired by this, the SMPL's author resolved the problem of LBS by learning the corrective blendshape from a large amount of scanned data of real humans. Specifically, in the SMPL formulation, the pose corrective blendshapes are added to the original LBS (Equation 1) as follows:
$$
\mathbf{t}_i' = \sum_{k=1}^K w_{ik}\mathbf{G'}_k(\theta,\mathbf{J})(\mathbf{t}_i + \mathbf{P}_i(\theta)), \tag{2}
$$
where $\mathbf{P}(\theta) \in \mathbb{R}^{3N}$ is the pose correctives. Let $f(\theta)$ be some function of $\theta$ and $f_n(\theta)$ denote the $n$-th element. The pose correctives depend on the current pose $\theta$ and can be calculated as the combination of the learned blendshapes: 
$$
\mathbf{P}(\theta) = \sum_{n=1}^{\vert f(\theta)\vert} f_n(\theta) \mathbf{P}_n \tag{3}
$$
The authors of SMPL designed $f(\theta)$ to be the function converting $\theta$ into a vectorized version of the concatenated joint rotation matrices $f: \mathbb{R}^{3K} \mapsto \mathbb{R}^{9K}$. Note that although $\mathbf{P}(\theta)$ may be non-linear in pose $\theta$, it is linear in elements of the rotation matrices $f(\theta)$ (the authors also tried to make the model linear, i.e., $f(\theta) = \theta$, but it did not work). Therefore, SMPL is an additive and simple yet very effective model. These blendshape vectors $\mathbf{P}_n \in \mathbb{R}^{3N}$  are the model's parameters and are learned from real-world data.

### Training of pose-related parameters

![](https://vision.aioz.io/f/7d6175003fcc4461a601/?dl=1)*<center>**Figure 2**. The joint location could be estimated as an weighted average of surrounding vertices. The vertex weights for each joint are indicated by the colored lines on the body surface. The joint regressor (matrix containing vertex weights) is sparse since the joint is only influenced by nearby  vertices ([Source](https://files.is.tue.mpg.de/black/papers/SMPL2015.pdf)).</center>* 



The pose-related parameters of the model are trained using a 3D scan dataset when people are in different poses (we called multi-pose dataset). SMPL pose-related parameters in this stage include:
- $\mathcal{J} \in \mathbb{R}^{K\times N}$: Joint regressor matrix, which is used to regress the joint location based of the surrounding vertices (Figure 2): $\mathbf{J} = \mathcal{J}\mathbf{t}$
- $\mathbf{W} \in \mathbb{R}^{N\times K}$: blend skinning weights matrix.
- $\mathbf{P} \in \mathbb{R}^{3N\times 9K}$: pose corrective blendshapes.

To train these parameters, we minimize the surface  reconstruction error (squared euclidean distance) between the ground-truth mesh vertices (template aligned from the scanned human) and the output vertices of the corrective skinning function (Equation 2):
$$\Vert\mathbf{V}_i - \sum_{k=1}^K w_{ik}\mathbf{G'}_k(\theta,\mathbf{J})(\mathbf{t}_i + \mathbf{P}_i(\theta))\Vert^2 \tag{4}
$$

where $\mathbf{V}$ is the ground-truth aligned mesh, $i$ denotes the $i$-th vertex position. In addition, there are many subjects in the dataset, each human subject has a different body configuration (e.g., fat, skinny, tall,...). Therefore, we also need to compute the rest template mesh $\mathbf{t}$ and the joint location $\mathbf{J}$ for each subject. This result in an alternating optimization scheme where we alternate updating between the pose parameter $\theta$, the subject-specific parameters $\{\mathbf{t}$, $\mathbf{J}\}$, and the global parameters $\{\mathbf{P}$, $\mathbf{W}\}$ 


Since the model consists of a large number of parameters, we also apply several regularizations to prevent overfitting:
- $\mathbf{P}$: regularized the blendshapes towards zeros.
- $\mathcal{J}$: regularized towards the predicting joints near the boundaries between the body parts and to be sparse.
- $\mathbf{W}$: regularized towards the initial artist-design LBS blend weights.

## Identity-dependent (Shape) blendshapes
Next, we want the body model to be able represent a wide variety of human body shape in the population.  To do so, the authors also built a dataset for shape training, called multi-shape dataset. This dataset contains several template meshes with high variation of body shape of real human (mostly from US and Europe) . In order to learn the shape space properly (the shape must be completely independent with the pose), we also need to factor out the pose (pose normalization) so that every people in this dataset are in the same rest pose.

![](https://vision.aioz.io/f/72f2d996523b4ffa8c28/?dl=1)*<center>**Figure 3**. Given the shape training dataset with multiple human meshes in the default pose, we first calculate the mean body shape and then subtract it to obtain the data matrix for PCA.</center>* 


Our goal is to learn a statistical model from a body shape space. Note that each body can be considered as a vector in a $3N$-dimensional space (3D positions of $N$ vertices). However, this space is infeasible to deal with since $N$ is typically large, and not all vectors in this space correspond to a valid body shape (they only take the place of a tiny subspace). Therefore, we need to reduce the dimensionality to compress the shape space into a low dimensional space. 

We first start by computing the mean body shape from the training data. We subtract the mean mesh from each of all the meshes in the training dataset (Figure 3). After that, we stack these body mesh vertices into a matrix and perform principal component analysis (PCA). This results in a set of eigenvalues and eigenvectors that describes the major directions of variation in the body shape space. The top ordered eigenvectors give us a low-dimensional linear subspace (typically 10-300D) that captures most of the variance in the $3N$-D space. Consequently, the body shapes of different people are approximated by a linear combination of the shape parameters $\beta$ and the shape-dependent blendshapes $\mathbf{S}$ plus a mean body $\mu$:
$$
\mathbf{T} = \sum_{n=1}^{\vert\beta\vert} \beta_n\mathbf{S}_n + \mathbf{\mu} = \mathbf{S}\beta + \mathbf{\mu}  \tag{5}
$$

![](https://vision.aioz.io/f/fd4ab3a4ce7c4352b940/?dl=1)*<center>**Figure 4**. An arbitrary body shape can be represented by the sum of the mean body and a combination of the shape-dependent blendshapes weighted by some shape coefficient parameters.</center>* 

Figure 5 below shows the variation in the first principal components of the shape PCA subspace. We also note that this procedure works because the non-linearities of the pose are factored out so that the body meshes live in a linear euclidean space of point. Moreover, the joint locations $\mathbf{J}$ is now a function depending on the shape parameters $\beta$ since it is regressed from the body mesh that may vary with the shape. 

![](https://vision.aioz.io/f/dd2dbb8c7b184fdc85cd/?dl=1)*<center>**Figure 5**. The first principal component of the trained shape space ([Source](https://files.is.tue.mpg.de/black/papers/SMPL2015.pdf)).</center>* 



Putting it all together, we obtained the final skinning equation of SMPL:
$$
\mathbf{t}_i' = \sum_{k=1}^K w_{ik}\mathbf{G'}_k(\theta,\mathbf{J}(\beta))(\mathbf{t}_i + \mathbf{S}_i(\beta) + \mathbf{P}_i(\theta)) \tag{6}
$$

where
- $t = \mu$ is the mean body template mesh (in the rest pose).
- $\mathbf{S}(\beta) = \mathbf{S}\beta$ is the shape blendshapes (Equation 5).
- $\mathbf{P}(\theta)$ is the pose blendshapes (Equation 3).

The SMPL model can represent "any" human body in different poses in an effective way thanks to the disentanglement of the shape and the pose. This means we can easily transfer the pose from a person with this body shape to another one with different body shape by just directly copying the pose parameters. We can also see that the SMPL is a simple additive model and is built upon the Linear Blend Skinning. Therefore, it is compatible with existing graphics engines. Another advantage is that it is trained from real data so that it is accurate and can resolve many problems of the traditional skinning method. 

This model cannot just only be used to synthesize artificial characters but can also be used to estimate the pose and the shape of humans precisely even from just a single image, thanks to the differentiability and linearity in the formulation. We can arbitrarily generate a new body shape by randomly sampling the shape vector $\beta$ around a Gaussian distribution with zero means. Moreover, the emergence of this model pushes forward many interesting research directions such as modeling clothing/garments, human reconstruction and motion dynamics, virtual avatars, scene interaction... Since SMPL is simple, it may not fully describe all features of human. There are several extensions of SMPL that can be useful in some specific tasks: DMPL [1] to model body and dynamic soft tissue movements, SMPL-H [2] to model body and hand/finger movement, SMPL-X [3] to model body, hand, and facial expression.


## References

[1] Loper, M., Mahmood, N., Romero, J., Pons-Moll, G., & Black, M. J. SMPL: A skinned multi-person linear model. ACM transactions on graphics (TOG).

[2] Romero, J., Tzionas, D., & Black, M. J. Embodied hands: Modeling and capturing hands and bodies together. ACM transactions on graphics (TOG).

[3] Pavlakos, G., Choutas, V., Ghorbani, N., Bolkart, T., Osman, A. A., Tzionas, D., & Black, M. J. Expressive body capture: 3d hands, face, and body from a single image. In CVPR 2019.




