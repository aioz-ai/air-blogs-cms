---
last_modified_on: "2022-04-02"
title: Human Pose & Motion Capture exploration (part 7)
description: Study 2D-to-3D lifting based methods for 3D Human Pose Estimation
series_position: 7
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In the previous post, we have introduced some 3D Human Pose Estimation methods and shown that they can achieve promising results, making solid progress towards a deeper understanding of human behavior. However, most of the current 3D datasets are typically constructed under studio conditions using complicated high-cost Motion Capture (MoCap) system. Therefore, methods that directly estimate 3D Human Pose from raw images often generalize poorly to unseen, unconstrained scenarios such as outdoor scenes with a large diversity of appearance. One solution is to synthesize data based on some graphical rendering engine. However, there is no guarantee that the characteristics of the synthetic images match the real-world settings, thus the model trained on these synthesized data may fail to achieve desired performance. In this post, we will study a very simple yet effective strategy to deal with this problem.

## A simple baseline model of 2D-to-3D Lifting

In [1], Martinez *et al.* did research and found that error in state-of-the-art 3D Pose Estimation networks mainly comes from the erroneous 2d localization or visual understanding, instead of the failure to map the 2d visual cues into 3d joint positions. This is a typical problem of many deep end-to-end systems that predict 3d joint locations given raw image pixels since there is only a small amount of 3D pose data with limited conditions, as stated above.

![Figure 1](https://vision.aioz.io/f/f7fbae0e8b804f0a90bf/?dl=1)*<center>**Figure 1**. A simple baseline network to lift the 2D poses into 3D poses, which consists of multiple fully connected layers ([Source](https://arxiv.org/pdf/1705.03098.pdf))</center>*

Motivated by the notable successes of 2D Human Pose Estimation in the wild, they first leveraged an off-the-shelf 2D pose detection model on the input image to obtain the 2D keypoints. Then, a simple lifting network (Figure 1) takes the detected 2D keypoint locations as input and produces a set of 3D joint positions for each keypoint. The network consists of six linear layers with batch normalization, dropout and ReLU nonlinearity, as well as residual connections. The input to the network is simply a $2J$ dimensional vector and the output is a $3J$ dimensional vector, where $J$ is the number of joints. They only train the network using a standard regression loss: $\mathcal{L}=\Vert \mathbf{x} - \mathbf{x}^* \Vert$, where $\mathbf{x}$ and $\mathbf{x}^* \in \mathbb{R}^{3J}$ are the predicted and ground-truth 3D joints vector. 

Surprisingly, the authors showed that although the design is straightforwardly simple, the simple method significantly outperforms many other approaches at that time. This demonstrates the huge advantages of the lifting-based method in addressing the insufficient in-the-wild 3D data since the network is augmented by another detection network trained on a large amount of 2D information. This simple approach also allows for real-time 3D HPE applications, since the main bottleneck is just the 2D detection procedure.

Since then, the vast majority of the 3D pose estimation systems are built upon the lifting-based approach, as they generally outperform direct estimation approaches [2].




## Encode human body structure using Graph Convolutional Networks (GCNs)
Although a simple lifting network can yield good performance, there still exist several challenging yet unsolved problems in predicting the 3D pose from 2D pose such as depth ambiguity, self-occlusion, physical plausibility and temporal coherence. Because predicting 3D information from 2D information is an ill-conditioned problem, prior knowledge of the task structures should be taken into consideration. Given the fact that the human skeleton can be represented as a graph where the joints are the nodes and the bones are the edges, an effective solution is to utilize the Graph Neural Network to learn to encode the human skeleton structure. Graph Neural networks are shown to be very effective in many structure prediction tasks.

In [3], Cai *et al.* first construct a skeleton graph with the pre-defined topology, which encodes both spatial and temporal information. The spatial edges (which can be viewed as bones) connect two joints of a natural human body in one frame, while the temporal edges connect the same joints between consecutive frames (Figure 2a).

![Figure 2](https://vision.aioz.io/f/9df56183c76b443d999a/?dl=1)*<center>**Figure 2**. A pre-defined spatial-temporal topology for the human skeleton graph and the semantic meaning of each node. ([Source](https://openaccess.thecvf.com/content_ICCV_2019/papers/Cai_Exploiting_Spatial-Temporal_Relationships_for_3D_Pose_Estimation_via_Graph_Convolutional_ICCV_2019_paper.pdf))</center>*

Based on the constructed human topology, we can use Graph Neural Network to exploit the spatial configurations and temporal consistencies for 3D pose estimation. The features of one joint (node) can be effectively transmitted to other joints in the graph (interestingly, it is somewhat related to the impulse transmission mechanism inside the biological body to control the movement of our body), which demonstrates the effectiveness of this approach in pose estimation. Mathematically, suppose $\mathbf{X} \in \mathbb{R}^{N\times D}$ is a matrix where each row is a $D$ dimensional feature vector of each node of the total $N$ nodes in the graph. Then, the aggregated features of each node and its neighboring nodes $\mathbf{Z} \in \mathbb{R}^{N\times D'}$ is computed as:

$$

\mathbf{Z} = \mathbf{D}^{-\frac{1}{2}} \mathbf{W}\mathbf{D}^{-\frac{1}{2}}\mathbf{X}\mathbf{\Theta} \tag{1}

$$

where $\mathbf{W} = \mathbf{A} + \mathbf{I}_N$ (adding self-loop), $\mathbf{A} \in \mathbb{R}^{N\times N}$ is the adjacency matrix, $\mathbf{D} \in \mathbb{R}^{N\times N}$ is the degree matrix i.e. $\mathbf{D}_{ii} = \sum_j\mathbf{W}_{ij}$ and $\mathbf{\Theta} \in \mathbb{R}^{D\times D'}$ is the matrix containing transformation weights. The above operation is then pass on to an activation function (i.e. $\mathbf{X}' = \sigma(\mathbf{Z})$) to form a layer in the GCN, and we can basically repeat this process to build a complete Graph Convolutional Network. Finally, we should obtain a feature vector for each node that encodes the holistic information of the graph structure (features of one node may contain information of a distant node). 

![Figure 3](https://vision.aioz.io/f/1b365fef7bd345df8876/?dl=1)*<center>**Figure 3**. Illustration of Graph Convolutional Network ([Source](https://web.stanford.edu/class/cs224w/slides/08-GNN.pdf))</center>*

However, in the current graph convolution, the weight matrix is shared across all nodes. Although it is fine with general graphs, this graph operation with uniform weight-sharing may not suitable for the pose prediction task. Since different edges connecting nodes may have different semantic meanings, and spatial edges and temporal edges represent different correlations. For example, the shoulder may affect the elbow in a different way from the affection of the hip to the knee.  


## References
[1] Julieta Martinez, Rayat Hossain, Javier Romero, and James J. Little. 2017. A simple yet effective baseline for 3d human pose estimation. In *ICCV*

[2] Ce Zheng, Wenhan Wu, Chen Chen, Taojiannan Yang, Sijie Zhu, Ju Shen, Nasser Kehtarnavaz, Mubarak Shah. 2020. Deep Learning-Based Human Pose Estimation: A Survey.


[3] Y. Cai, L. Ge, J. Liu, J. Cai, T. Cham, J. Yuan, and N. M. Thalmann. 2019. Exploiting Spatial-Temporal Relationships for 3D Pose Estimation via Graph Convolutional Networks. In *ICCV*.

[4] H. Ci, C. Wang, X. Ma, and Y. Wang. 2019. Optimizing Network Structure for 3D Human Pose Estimation. In *ICCV*.

[5] Kenkun Liu, Rongqi Ding, Zhiming Zou, Le Wang, and Wei Tang. 2020. A Comprehensive Study of Weight Sharing in Graph Networks for 3D Human Pose Estimation. In ECCV
