---
last_modified_on: "2022-01-17"
title: Human Pose & Motion Capture exploration (part 5).
description:  explore human pose estimation and motion capturing.
series_position: 6
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
In this blog, we will discuss about Multi-person HPE case.

## Now we move to the multi-person case.
Compared to single-person HPE, multi-person HPE is more difﬁcult and challenging. The model is not only required to accurately localize the joints of all persons, but it also needs to ﬁgure out the number of people and their positions, and how to group keypoints for different people. One typical approach is the **top-down** approach, in which we can take advantage of the excellent performance of current human detection algorithms. As shown in Figure 7, the method consists of two essential stages: a human body detector to obtain person bounding boxes and a single-person pose estimator to predict the locations of keypoints within these bounding boxes.

![Figure-7](https://i.imgur.com/Hel6eTl.png)*<center>
**Figure 7**: An illustration of a typical top-down pose estimation method.</center>*


Although the top-down-like methods can produce good results, they are inefficient and the running time grows linearly with respect to the number of people in the scene. For example, the state-of-the-art method HRNet [3] achieves significant results in several benchmarks, but the high computational complexity prevents it from being considered in realworld applications. Moreover, they depend on the quality of the detected person bounding box and may fail when the boxes are overlapped. Only cropping the image patch containing the person also results in the loss of global context, which might be necessary to estimate the correct pose. To that end, many **bottom-up** approaches are introduced to overcome these issues. In contrast to the top-down, the bottom-up pipeline bottom-up methods ﬁrst predict all body parts of every person in the input image and then assemble them together for each individual person. The speed of bottom-up methods is usually faster than top-down methods since they do not need to apply pose estimation for each person separately. Some methods can even run in realtime on a very crowded scene.

### OpenPose
The most well-known method based on bottom-up approach is OpenPose [4]. In particular, the authors present a very efficient approach to detect and group the 2D pose of multiple persons in the scene by utilizing the Convolution Pose Machine [1] as the backbone feature extractor. In addition to the joint heatmaps, they introduced a nonparametric representation, which is referred to as Part Affinity Fields (PAFs) (see Figure 8) to preserve both location and orientation information across the region of support of the limb.

![Figure-8](https://i.imgur.com/IJxiK7W.png)*<center>**Figure 8**: Part Affinity Fields ([Source](https://arxiv.org/pdf/1611.08050.pdf))</center>*

These are 2D vector fields (or vector maps), where each pixel location $(x,y)$ contains a 2D unit vector encoding the direction from one joint to the other joint in the limb (or bone) such as elbow $\rightarrow$ wrist. The PAFs can be used to associate keypoints with individuals in the image. The architecture first extracts the global context, and then a fast bottom-up keypoints grouping algorithm can be utilized to accelerate the speed substantially while maintaining high accuracy. Note that the model run time is not affected by the number of people in the image, which is one of the advantages to be considered in many practical applications. The figure below presents the overall pipeline to fully predict the poses of multiple people in an image:
* First, the input image is fed into a two-branch CNN to jointly predict heatmaps for joints candidate (b) and part affinity fields for joint grouping \(c\).
* Next, based on the joint candidate locations and the part affinity fields, we compute the association weights between pairs of two detected joints. Then we perform multiple steps of graph bipartite matching (d) to associate these detected joint candidates into persons.
* Finally, we assemble them into full body poses for all people in the image to obtain final results (e)

![Figure-9](https://i.imgur.com/4CTAmAl.png)*<center>**Figure 9**: The overall pipeline of OpenPose ([Source](https://arxiv.org/pdf/1611.08050.pdf))</center>*


Formally, the goal of the network is to predict two separate sets of 2D heatmaps $\mathbf{H} = \{H_1, H_2,...,H_J\}$ and the 2D vector-maps $\mathbf{L} = \{L_1, L_2,...,L_C\}$ for each limb $c$ from $1$ to $C$ (each directly-connected joint pair is refered to as a limb), where $L_c \in \mathbb{R}^{w \times h \times 2}$. 

We note that the heatmaps in the bottom-up method are slightly different from the single-person case.  The heatmap for multiple people should have multiple peaks (e.g. to identify multiple elbows belonging to different people), as opposed to just a single peak for a single person. Therefore, the groundtruth heatmap $H_j$ is computed by an aggregation (pixel-wise maximum) of peaks of all persons in a single map for a specific joint $j$.

To generate the groundtruth vector maps $L^*$ for training, we consider the $\mathbf{x}_{j_1,k}$ and $\mathbf{x}_{j_2,k} \in \mathbb{R}^2$ is the groundtruth locations of two end-points $j_1$ and $j_2$ of limb $c$ for the $k$-th person. Let $\mathbf{p} \in \mathbb{R}^2$ is an arbitrary point in the image. If $\mathbb{p}$ lies on the limb, the value at $L^*_{c,k}(\mathbf{p})$ is a unit vector that points from $j_1$ to $j_2$, for all other points, the vector is zero:

$$
\tag{}
L^*_{c,k}=
    \begin{cases}
      \mathbf{v} & \text{if $p$ on limb$_{c,k}$}\\
      \mathbf{0} & \text{otherwise}
    \end{cases} 
$$

where $\mathbf{v} = \frac{\mathbf{x}_{j_2,k}-\mathbf{x}_{j_1,k}}{\Vert \mathbf{x}_{j_2,k}-\mathbf{x}_{j_1,k} \Vert_2}$  is the unit vector representing the direction of the limb (from location $\mathbf{x}_{j_1,k}$ to $\mathbf{x}_{j_2,k}$). Finally, we obtain the groundtruth PAFs $L^*_c$ for limb $c$ as the pixel-wise average of the $L^*_{c,k}$ for all person $k$: $L^*_c = \frac{1}{K}\sum_{k=1}^KL^*_{c,k}$. Same as the CPM, the training objective at each stage is the sum of the $l_2$ loss between the predicted and groundtruth heatmaps as well as the vector maps.

$$
\tag{}
\mathcal{L} = \sum_{j=1}^J\Vert H_j - H^*_j\Vert_F^2 + \sum_{c=1}^C\Vert L_c - L^*_c\Vert_F^2
$$

During testing,  we feed the input image into the network to obtain a set of detected keypoint candidates for multiple people as well as the association weight for each pair of joints. To obtain the candidate keypoints, we perform non-maximum suppression on each heatmap and identify the local maximas. We then start to decode these outputs into poses for all persons. To achieve realtime performance, we greedily associate the keypoints to persons by performing graph maximum weight bipartite matching for each pair of keypoint as demonstrated in Figure 10 (right) . The experiments show that such greedy strategy is sufficient to produce highly accurate results instead of performing full $K$-dimensional matching (see Figure 10 (middle)), which is very slow as it is an NP-hard problem. 

![Figure-10](https://i.imgur.com/ncGULIs.png)*<center>**Figure 10**: Graph matching to associate detected joints to persons ([Source](https://arxiv.org/pdf/1611.08050.pdf))</center>*

The edge weight (for graph matching) is the measure  between candidate joints by computing the line integral of the dot product between the affinity unit vector $L_c(\mathbf{p})$ and the unit vector pointing from one candidate location $\mathbf{x}_{j_1}$ to another candidate joint $\mathbf{x}_{j_2}$

$$
\tag{}
e = \int_{u=0}^{u=1}L_c(\mathbf{p}(u)) \cdot \frac{\mathbf{x}_{j_2}-\mathbf{x}_{j_1}}{\Vert \mathbf{x}_{j_2}-\mathbf{x}_{j_1} \Vert_2}du
$$

where $\mathbf{p}(u)$ is the linearly interpolated position of between two candidate joint locations $\mathbf{x}_{j_1}$ and $\mathbf{x}_{j_2}$ : $\mathbf{p}(u) = (1-u)\mathbf{x}_{j_1} + u\mathbf{x}_{j_2}$. This can be viewed as the average cosine similarity between the affinity vector and the limb direction vector, along the limb. If $e$ takes on a higher value, it is more likely that the two candidate joints $j_1$ and $j_2$ belong to the same person.

### Associative embedding
To produce the final pose for multiple persons, OpenPose first obtain the network predicted maps and then employs a graph bipartite matching algorithm to find which pair of joints associated to a person. This can be inefficient when the scene is crowded, i.e., the number of people is large, as the size of the graph increases. Furthermore, such two-stage approaches (detection first and grouping second) may be suboptimal because detection and grouping are usually tightly coupled. For example, in multiperson pose estimation, a wrist joint is likely a false positive if there is not an elbow joint nearby to group with. Motivated by the OpenPose [4] and Stacked hourglass network [2], Newell et al. [5] proposed a single-stage deep network to simultaneously obtain both keypoint detections and group assignments. They introduced a novel but very simple representation called associative embeddings that encode the grouping information for each keypoints. Specifically, each detected keypoint is also assigned with a real number (or vector), which is called as a "tag", to identify the group (or person) where that keypoint belongs to. Concretely, detected joints with similar tags should be grouped together to form a person. Figure 11 shows an overview of the associative embedding for multi-person pose estimation. 

![Figure-11](https://i.imgur.com/HtXjJEg.png)*<center>**Figure 11**: Associative embedding network. ([Source](https://arxiv.org/pdf/1611.05424.pdf))</center>*

For each body joint, the model extracts the necessary information and simultaneously produces heatmaps and associative embedding maps. Specifically, in addition to the heat value, the network produces a tag at each pixel location for each joint. Hence, each joint heatmap has a corresponding tag map. Finally, to group the detected joints, we compare the values of the tags and identify the group of tags (for different joints) that has close distances to form a person. Note that the network can assign whatever values to the tags as long as these values are the same for detections belonging to the same group, as only the distances between tags matter. Surprisingly, the experiments show that 1D embedding (real number) is sufficient for multi-person pose estimation, and higher dimensions do not lead to significant improvement. As demonstrated in Figure 12, we can easily group the keypoints as the tags are well separated for each person.

![Figure-12](https://i.imgur.com/EzaZoen.png)*<center>**Figure 12**: An example of the predicted tag values for keypoints ([Source](https://arxiv.org/pdf/1611.05424.pdf))</center>*

We might wonder how the method solves the keypoint grouping task in such a simple way. To do so, we impose a common heatmap detection loss and an extra grouping loss on the output tag maps for training the network. The grouping loss assesses how well the predicted tags agree with the ground truth grouping.   Formally, let $h_j(\mathbb{p})$ is the tag value at the location $\mathbb{p}$ for joint $j$, the grouping loss $\mathcal{L}_g$ is formulated as follow:

$$
\tag{}
\mathcal{L}_g = \frac{1}{K} \sum_k\sum_j\left(\bar{h}_k-h_j(\mathbf{x}^*_{j,k}) \right)^2 \quad +\frac{1}{K^2}\sum_k\sum_{k'}exp\left(-\frac{1}{2\sigma^2}(\bar{h}_k-\bar{h}_{k'})^2\right)
$$

where:
* $\mathbf{x}^*_{j,k}$ is the ground truth pixel location of the $j$-th body joint of the person $k$.
* $\bar{h}_k$ is the reference embedding for person $k$, which is the mean of all embedding $h_j$ across $J$ joints of that person: $\bar{h}_k=\frac{1}{J}\sum_{j=1}^Jh_j(\mathbf{x}^*_{j,k})$.

The left term is the squared distance between the reference and the predicted embedding for each groundtruth joint of person $k$. The right term is the penalty that decreases exponentially to zero as the distance between two reference tags of two different person increases (the larger distance between two different persons, the smaller the loss value is). This loss encourages similar tag values for keypoints from the same group (close to the mean embedding) and different tag values for keypoints across different groups.




## Conclusion
In summary, the performance of 2D HPE has been significantly improved via the power of deep learning (some of them can be deployed in many applications, such as OpenPose). In this blog, we first explored the existing approaches on the single-person scenario. In general, they can be classified into two categories: *direct-regression* and *heatmap-based* methods. We also discussed the limitations of the regression approach and show how the heatmap representations are more preferred in recent studies. As for multi-person scenarios, we introduced the *top-down* and *bottom-up* methods. Since the bottom-up approach is more efficient, we further dive deep into some of the most widely used algorithms, not only in the current field of research but also in many computer vision tasks as the basis component (such as human behavior understanding, action recognition,...).


### References

[1] *S.-E. Wei, V. Ramakrishna, T. Kanade, and Y. Sheikh, "Convolutional pose machines," in CVPR, 2016.*

[2] *A. Newell, K. Yang, and J. Deng, "Stacked hourglass networks for human pose estimation," in ECCV, 2016.*

[3] *K. Sun, B. Xiao, D. Liu, and J. Wang, “Deep high-resolution representation learning for human pose estimation,” in CVPR, 2019.*

[4] *Z. Cao, T. Simon, S.-E. Wei, and Y. Sheikh, "Realtime multi-person 2d pose estimation using part afﬁnity ﬁelds," in CVPR, 2017.*

[5] *A. Newell, Z. Huang, and J. Deng, "Associative embedding: End-to-end learning for joint detection and grouping," in NIPS, 2017.*

