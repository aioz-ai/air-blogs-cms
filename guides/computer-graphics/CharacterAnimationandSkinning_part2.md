---
last_modified_on: "2022-07-30"
title: Character Animation and Skinning (part 2)
description: Study some animation and skinning techniques
series_position: 12
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---


In the previous post, we studied skeleton and bone, which are the core underlying structure and essential to animate 3D characters in computer graphics. In this post, we will continue studying the process to produce animation of 3D mesh characters. 


## Types of animation

The most basic idea of character animation is to transform the character into desired states (for example moving the leg) and combine different states of the character across time to represent their motion according to the target goal. Speaking of animation techniques, there are three types of animation that are popular in practice: keyframing-based animation, procedural animation and physically-based animation.
* Keyframe-based animation: we only specify the state of the character or scene at some important point of the time (called keyframe). Then the in-between motions are generated automatically (typically by interpolation where the animator manually controls the parameter of the interpolating curve). Figure 1 shows an example of the smooth transition using linear interpolation between two keyframes.
* Procedural animation: we describe the character motion in some algorithmic ways. The most common approach is to express the animation as a function of some small number of parameters. For example, we can create a clock ticking animation by expressing the direction of the clock needle as a function of time.
* Physically-based animation: we assign some physical properties to the objects (such as object's mass, forces, gravity,...). The physics is simulated by solving equations of motion (such as Newton's law $F=ma$). This can result in very realistic and natural animation but very difficult to control.

![](https://vision.aioz.io/f/d9d54c672c1045c9b752/?dl=1)*<center>**Figure 1**. Linear interpolation between two keyframes</center>* 



## Animating and Skinning the character 
In practice, animation is typically controlled by using low-dimensional (or high-level) control parameters, rather than modeling the whole geometry of the scene piece by piece for every frame. For example, a common 3D character is composed of thousands of vertices, it is infeasible (for us) to specify the states of these vertices all at once. Instead, we can describe the joint angles of the character's skeleton and find ways to deform the vertices (which is referred to as "skin") according to the skeleton's poses. In other words, our main goal is to first attach the skeleton to a detailed character mesh (which is usually called rigging), and then animate the bone (i.e., change the joint angles over time) so that the skin will move with it. We can achieve that by using a technique called "Skinning", where we need to infer how skin deforms from bone transformations. In particular, the core idea of skinning is basically specifying the weights that describe the *influence* of every bone on the motion for each vertex on the character surface. Each bone in the skeleton defines a deformation of the space around it (e.g., rotation, translation). The most straightforward approach is to attach each vertex (of the skin) to just a single bone, and the skin will be rigidly followed with the bone transformation. Although this is fine for some vertices along the bone,  the skin at the intersection between two bones will result in unnatural stretch (because the skin should be elastic). A solution to this problem is to attach a vertex to different bones. Therefore, the skin at a joint is deformed corresponding to the *weighted combination* of the bones [1]. See the example of the vertex weights in Figure 2, where the colored vertices are attached to only one bone and the black vertices are attached to more than one bone. We can notice that the black vertices are also near the joint regions. 


<!-- ![](https://i.imgur.com/QX7CqIF.png) -->
![](https://vision.aioz.io/f/dc3f97a00a1049d495f0/?dl=1)*<center>**Figure 2**. Example skinning weights of a human mesh</center>*


Mathematically, for each vertex $\mathbf{v}_i$ we assign a weight $w_{ij}$ that corresponds to the influence of the transformation of bone $\mathbf{B}_j$. Intuitively, the weight describes how much vertex $i$ should move with bone $j$. These weights should satisfy the following properties: $w_{ij} \geq 0, \forall i,j$  and $\sum_{j}w_{ij}=1, \forall i$. For many vertices, $w_{ij}=1$ means that $\mathbf{v}_i$ is fully attached to $\mathbf{B}_j$. In practice, the number of bones $N$ that can influence a single vertex are small (typically 4), as skin is not affected by the bone that is far in the hierarchy (for example the leg bone does not influence the hand's skin). Moreover, since $w_{ij}$ are mostly 1, we only need to store pairs of (bone index $j$, weight value $w_{ij}$) per vertex to reduce the storage space.

Given the bone transformation and skinning weights $w_{ij}$, we can compute the vertex position as follows:
* First transform each vertex position $\mathbf{v}_i$ with each bone $\mathbf{B}_j$ as if it was attached to it rigidly

$$
\mathbf{v}'_{ij} = \mathbf{T}_j\mathbf{v}_{i} \tag{1}
$$
where $\mathbf{T}_j$ is the transformation matrix of bone $j$ and $\mathbf{v}'_{ij}$ is the vertex $i$ transformed according to bone $j$.
* Then blend the results by computing the average vertex position based on the skinning weights:
$$
\mathbf{v}'_{i} = \sum_jw_{ij}\mathbf{v}'_{ij}\tag{2}
$$
where $\mathbf{v}'_{i}$ is the final skinned position of vertex $i$.

![](https://vision.aioz.io/f/f335b55444804f47ab86/?dl=1)*<center>**Figure 3**. Examples of mesh deformation with bind pose (left) and the animated pose (right).</center>*

In practice, to do skinning, we are given a skeleton and a skin mesh in default pose (commonly in T-pose) as in Figure 3, which is also called bind pose. However, the undeformed vertex positions $\mathbf{v}_i$ are given in object coordinate space and $\mathbf{T}_j$ is in the local bone coordinate according to skeleton hierarchy. To apply Equation 1 correctly, the coordinate system of the vertices and the transformation must match. In the rigging stage, we link the skeleton up with the undeformed skin mesh by calculating the transformations in **default pose** $\mathbf{B}_j$ from local bone coordinates to global. Specifically, $\mathbf{B}_j$ accumulates all hierarchy matrices from the root joint up to node $j$. When we animate the character, the bone transformations are changed into $\mathbf{T}_j$, which maps from the local coordinate system of bone $j$ to the world space. Therefore, to be able to deform $\mathbf{v}_i$ according to $\mathbf{T}_j$, we must first express $\mathbf{v}_i$ in the local coordinate system of bone $j$. This can be calculated using $\mathbf{B}_j$ as follow:
$$
\mathbf{v}'_{ij} = \mathbf{T}_j \mathbf{B}^{-1}_j \mathbf{v}_{i} \tag{3}
$$
This maps $\mathbf{v}_{i}$ from bind pose to the local coordinate system of bone $j$ using $\mathbf{B}^{-1}_j$, and then to world space using $\mathbf{T}_j$. In summary, the linear blend skinning algorithm is as follow:

1. *Perform forward kinematics to obtain the per bone transformation matrix $\mathbf{T}_j$* 
2. *For each skin vertex $\mathbf{v}_i$, calculate the deformed vertex position: $\mathbf{v}'_{i} = \sum_jw_{ij} \mathbf{T}_j \mathbf{B}^{-1}_j \mathbf{v}_{i}$*

Nevertheless, looking at the equation $\mathbf{v}'_{i} = \left( \sum_jw_{ij} \mathbf{T}_j \mathbf{B}^{-1}_j \right) \mathbf{v}_{i}$, we notice that the total transformation is the linear average of the rotation  $\mathbf{T}_j \mathbf{B}^{-1}_j$. This will not necessarily give us a correct rotation matrix and it can result in some severe artifacts like the candy wrapper effect in Figure 4. There are several methods that are proposed to address the weakness of linear blend skinning by adding corrective blendshape as the function of pose [3], or using quaternion rotation [4] or automatically predicting the blendshape using deep learning [5].

![](https://vision.aioz.io/f/eb8d0aa50ae64cb882a8/?dl=1)*<center>**Figure 4**. Artifacts due to incorrect rotation. The figure is taken from [2].</center>*

In addition, the skinning weights are usually painted manually by the artist (supported by many 3D graphics software such as blender, maya,...) or are automatically inferred by optimization from example meshes data [6] or even by deep networks [5,7].

## References
[1] James & Twigg, Skinning Mesh Animations. In SIGGRAPH 2005.

[2] J. P. Lewis, Matt Cordner, Nickson Fong. Pose Space Deformation: A Unified Approach to Shape Interpolation and Skeleton-Driven Deformation. In SIGGRAPH 2000.

[3] Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, Michael J. Black. SMPL: a skinned multi-person linear model. In SIGGRAPH Asia 2015.

[4] Ahmed A. A. Osman, Timo Bolkart, Michael J. Black. STAR: A Sparse Trained Articulated Human Body Regressor. In ECCV 2020.

[5] Peizhuo Li, Kfir Aberman, Rana Hanocka, Libin Liu, Olga Sorkine-Hornung, Baoquan Chen. Learning Skeletal Articulations with Neural Blend Shapes. In SIGGRAPH 2021.

[6] Xiaohuan Corina Wang, Cary Phillips. Multi-weight enveloping: least-squares approximation techniques for skin animation. In SCA'02.

[7] Lijuan Liu, Youyi Zheng, Di Tang, Yi Yuan, Changjie Fan, Kun Zhou. NeuroSkinning: automatic skin binding for production characters with deep graph networks In SIGGRAPH 2019.

