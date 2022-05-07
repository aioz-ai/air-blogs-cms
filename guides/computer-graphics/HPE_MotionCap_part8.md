---
last_modified_on: "2022-05-07"
title: Human Pose & Motion Capture exploration (part 7)
description: Study 2D-to-3D lifting based methods for 3D Human Pose Estimation
series_position: 8
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In the previous post, we have investigate the 2D-to-3D lifting model and how to encode human body structure using GCNs. In this post, we will talk about Weight-sharing problem of GCN in pose estimation and how to deal with it.

## Weight-sharing problem of GCN in pose estimation
To tackle the issue of uniform weight-sharing , the authors in [3] assumed each neighboring nodes have different semantic meanings and thus we should use different weight matrices for different neighboring nodes (Figure 2b). Specifically, the nodes are classified into the `center` node itself (blue), `physically-connected` nodes including the one closer (purple) to and the one farther (green) from the **root** node, indirect `"symmetrically"-related` node (dark blue), `time-forward` node (yellow), and `time-backward` node (orange). Based on these, the graph convolution formulation is modified as:

$$

\mathbf{Z} = \sum_k \mathbf{D}_k^{-\frac{1}{2}} \mathbf{W}_k\mathbf{D}_k^{-\frac{1}{2}}\mathbf{X}\mathbf{\Theta}_k \tag{2}

$$

where $k$ is the considering type of the neighboring nodes, $\mathbf{W}_k = \mathbf{M}_k \odot \mathbf{W}$ is the element-wise product between the original adjacency matrix $\mathbf{W}$ and the mask $\mathbf{M}_k$, which is 1 for the adjacent nodes satisfied the considering type $k$ and 0 for other nodes. $\mathbf{\Theta}_k$ is the corresponding weight matrix for the $k$-th type nodes. Note that the graph is sparse and the topology is pre-defined (i.e. the adjacency and degree matrix are pre-computed), thus the graph operation should be very fast to compute. The overall process is shown in Figure 4. 

![Figure 4](https://vision.aioz.io/f/31dbc5644d884253b84b/?dl=1)*<center>**Figure 4**.  Illustration of the local-to-global GCN architecture, which is able to effectively process and consolidate features across scales ([Source](https://openaccess.thecvf.com/content_ICCV_2019/papers/Cai_Exploiting_Spatial-Temporal_Relationships_for_3D_Pose_Estimation_via_Graph_Convolutional_ICCV_2019_paper.pdf))</center>*

To simulate the bottom-up process (downsampling the feature maps) and the top-down process (upsampling the feature maps with the combination of higher resolution features from bottom layers), which is capable of capturing semantic features across multi-scale and efficiently reducing the number of parameters, the authors also use graph pooling and upsampling techniques. The graph pooling (bottom-up process) merges the node features according to each body part (e.g. left-hand, right-hand, left-leg,...) into a single node. The upsampling (top-down process) simply takes a reverse step of the graph pooling, where the features of vertices in the coarser graph are duplicated to the corresponding child vertices in the finer scale. Note that these processes are performed in the intra-structure of the body while the temporal edges remain the same.


Concurrently, another work [4] also investigated the weight-sharing problem in GCN and proposed a general weight-unsharing scheme with the flexibility to support customized node relationships. They also found that the common weight-sharing in GCN will harm its representation capability when it is used for 3D pose estimation because each joint needs to aggregate features from its neighbors in unique ways in order to infer its 3D location. For simplicity, the conventional GCN formulation in Equation 1 can be simplified as: 

$$

\mathbf{Z} = \mathbf{S} \mathbf{X}\mathbf{\Theta} \tag{3}

$$

where $\mathbf{S} \in \mathbb{R}^{N\times N}$ is the structure matrix indicating the relationship and dependency between nodes, $\mathbf{X}\in \mathbb{R}^{N\times M}$ is the node features and $\mathbf{\Theta} \in \mathbb{R}^{M\times M'}$ is the weight matrix.  Note that the Fully Connected Network is also a special case of this formulation (see Figure 6 for illustration), where every elements of the structure matrix are 1 (each node is connected to all other nodes). The weight unsharing scheme is to assign each separate weight matrix $\mathbf{\Theta}^q$ for each $q$-th node considered as the center node. 

$$

\mathbf{Z}_q = \mathbf{S}_q \mathbf{X}\mathbf{\Theta}^q 

$$

where $\mathbf{Z}_q \in \mathbb{R}^{1\times M'}$ is the feature vector of node $q$ after the computation, $\mathbf{S}_q \in \mathbb{R}^{1\times N}$ is the $q$-th row of $\mathbf{S}$ (representing the connectivity of node $q$ to other nodes), and $\mathbf{\Theta}_q \in \mathbb{R}^{M\times M'}$ is the unshared weights for node $q$. In the above equation, each local relationship (i.e. one center node and its neighbors) corresponds to a transformation matrix $\mathbf{\Theta}^q$. We can further extend the operation into a more general formula where every neighbor are transformed by different weight matrices:

$$

\mathbf{Z}_q = \sum_{i=1}^N \mathbf{S}_{q,i}* \mathbf{X}_i \mathbf{\Theta}^{(q,i)}

$$

where $\mathbf{S}_{q,i}$ is a scalar representing the connectivity of node $q$ and node $i$ (0 if there is no edge between $q$ and $i$), $\mathbf{X}_j \in \mathbb{R}^{1\times M}$ is the feature vector of node $i$ ($i$-th row of $\mathbf{X}$), and $\mathbf{\Theta}^{(q,j)} \in \mathbb{R}^{M\times M'}$ denotes the transformation weight for node $j$ when considered as neighbor of $q$. This operation results in the highest capacity of the GCN model and thus enhancing the capability to capture the relationship between each local connection, which leads to better performance. However, the total number of parameters in each layer is $NM\times NM'$ which is $N^2$ times higher than the conventional GCNs. The operation for the whole graph can be written in a fully vectorized manner as shown in Figure 5

![Figure 5](https://vision.aioz.io/f/f9ca7096d73246af9129/?dl=1)*<center>**Figure 5**. Example of the fully vectorization of weight unsharing scheme in a GCN layer. The weight matrix $\mathbf{W}$ and structure matrix $\mathbf{S} \in \mathbb{R}^{3M\times 3M}$ are evenly divided into $3\times3$ blocks ([Source](https://ieeexplore.ieee.org/document/9010332/))</center>*


This model is named Locally Connected Network (LCN) since the relationship of one node to its local neighbors is modeled by a specific weight matrix. Figure 6 illustrates the comparisons of FCN, GCN and LCN. In FCN, the input features of different nodes are mixed by dense connections making every output feature dependent on the input features of all nodes. In GCN, the output features of a node only depend on the neighboring nodes defined by the structure matrix while different nodes share the same filter *T* (blue rectangle). In LCN, each node has a separate filter. The full network is constructed by simply stacking multiple layers based on the formulations.

![Figure 6](https://vision.aioz.io/f/7309f9e57868401abe0e/?dl=1)*<center>**Figure 6**. Conceptual comparisions of Fully Connected Network (FCN), conventional Graph Convolutional Network (GCN), and Locally Connected Network (LCN) ([Source](https://ieeexplore.ieee.org/document/9010332/))</center>*

Note that the structure matrix can also be learned end-to-end from data, which is general and might be useful for several tasks. However, the experiments showed that a pre-defined topology based on the human skeleton can yield better performance than the learnable topology, which suggests the importance of the human prior knowledge in pose estimation.


A main drawback of the weight-unsharing approach is the increase in complexity because each node should have a different set of parameters. However, it is negligible in our task since the number of body joints (nodes) is small in practice. These proposed GCN methods can perform significantly better than both the baseline and the conventional GCN approaches. Nonetheless, it is hard to directly compare different designs of GCN (weight-unsharing scheme) since the performance differences may also come from different training settings and loss functions. The key takeaway is that it has many benefits to exploit different body parts as well as their interaction with each other since each body part plays different roles in human body pose and motion. In addition, we refer readers to [5] for further comprehensive study of the weight-sharing problem in GCN for pose estimation.



## References
[1] Julieta Martinez, Rayat Hossain, Javier Romero, and James J. Little. 2017. A simple yet effective baseline for 3d human pose estimation. In *ICCV*

[2] Ce Zheng, Wenhan Wu, Chen Chen, Taojiannan Yang, Sijie Zhu, Ju Shen, Nasser Kehtarnavaz, Mubarak Shah. 2020. Deep Learning-Based Human Pose Estimation: A Survey.


[3] Y. Cai, L. Ge, J. Liu, J. Cai, T. Cham, J. Yuan, and N. M. Thalmann. 2019. Exploiting Spatial-Temporal Relationships for 3D Pose Estimation via Graph Convolutional Networks. In *ICCV*.

[4] H. Ci, C. Wang, X. Ma, and Y. Wang. 2019. Optimizing Network Structure for 3D Human Pose Estimation. In *ICCV*.

[5] Kenkun Liu, Rongqi Ding, Zhiming Zou, Le Wang, and Wei Tang. 2020. A Comprehensive Study of Weight Sharing in Graph Networks for 3D Human Pose Estimation. In ECCV

