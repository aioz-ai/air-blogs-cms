---
last_modified_on: "2026-09-09"
title: Efficient Human-Contact Representation for Human-Scene Interaction
description: Lightweight network, human-scene interaction
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';





In the first part, we introduced the contact prediction and synthesis task. In this section, we will dive into the core architecture of ECO method. 

## Efficent Contact Representation

### Contact Representation with Sparse Mask


We follow the POSA work to represent the human-scene contact. In particular, the human-scene input tensor $\bm I$ is defined as $\bm I =(\bm V, \bm F)$, where $\bm V \in \mathbf{R}^{N_v \times 3}$ is body vertices and $\bm F\in \mathbf{R}^{N_v \times N_c}$ is the contact label of the vertices. $N_v$ is the number of vertices, and $N_c$ is the number of labels.


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944368/method_fpn2ij.png)
*<center>**Figure 2**:  Efficient Contact Representation method overview. We adopt Graph and Spiral Blocks from POSA as contact predictor backbone, replacing original layers with our sparse layers. Red cells denote non-zero kernel weights and mask values, blue cells represent coordinates, green cells indicate non-zero contact values, and white cells denote zeros.</center>*

**Sparse Mask.** Our goal is to convert the human-scene input tensor $\bm{I}$ into a sparse tensor $\bm{I'}  \in \mathbf{R}^{N_v \times N_S}$ for a more efficient contact representation ($N_S = N_c + 3$). We define a sparse mask $\bm{M} \in \mathbf{R}^{N_v \times N_S}$ and calculate $\bm{I'}$ as:
$$
    \bm{I'} = \bm{M} \circ \bm{I}\ \ \  (1)
$$

where $\circ$ denotes element-wise multiplication. Each element in the sparse mask $\bm{M}$ is sampled from a binomial distribution. The sparsity of $\bm{M}$ is controlled via a *sparsity ratio* parameter which indicates the non-zero value ratio of the mask. Intuitively, $\bm{M}$ is a matrix with only $0$ or $1$ values to mask out the unnecessary information from the input.


In practice, applying only a *single* high-sparsity mask $\bm{M}$ to the input causes significant information loss hence heavily affecting the effectiveness of the model. To overcome this limitation, we apply $K$ *multiple* sparse masks $\left\{\bm{M}_1, \bm{M}_2, ..., \bm{M}_K \right\} $ to the input with the expectation that each sparse mask $\bm{M}_k$ would learn different important information from the input. We note that each sparse mask $\bm{M}_k$ is applied independently to the input to obtain the sparse tensor $\bm{I'}_k$, and $K$ is the hyper-parameter that indicates how many sparse masks we use during training. 



After applying the sparse mask $\bm{M}_k$ to the input tensor $\bm{I}$, we obtain a sparse tensor $\bm{I'}_k = \bm{M}_k \circ \bm{I}$ which has a high proportion of zero values. Consequently, the conventional dense representation is inefficient for representing the sparse tensor $\bm{I'}_k$ during the learning process. Additionally, effectively storing only non-zero values in the sparse tensor facilitates computation. To this end, we *decompose* the sparse tensor $\bm{I'}_k$ into two tensors to remove the zero values as in. This decomposition results in a coordinate matrix $\bm{C'}_k \in \mathbf{R}^{N'_k \times 2}$ and an associated feature matrix $\bm{S'}_k \in \mathbf{R}^{N'_k \times N'_S}$  where $N'_k$ denotes the number of non-zero values in $\bm{I'}_k$. This strategy not only saves memory by removing zero-values from the sparse tensor but also streamlines the computation process for $\bm{I'}_k$. In practice, the sparse tensor $\bm{I'}_k $ is represented as $\bm{I'}_k = \left (  \bm{C'}_k | \bm{S'}_k \right)$, where $\bm{C'}_k$ and $\bm{S'}_k$ are defined as: 
$$
 \bm{C'}_k = \begin{bmatrix}
 b_1  & x_1 \\ 
 \vdots &  \vdots\\ 
  b_{N'_k}  & x_{N'_k}  
\end{bmatrix} , \bm{S'}_k = \begin{bmatrix}
\bm{s}_1^{\intercal}\\ 
\vdots\\
\bm{s}_{N'_k}^{\intercal}
\end{bmatrix} 
$$
where $\left (b_i, x_i\right )$ is the frame index and coordinate of $i$-th feature $\bm{s}_i \in \mathbf{R}^{N'_S}$. 







**Sparse Mask Selection.** Although using a list of sparse masks preserves the model's performance compared to using a single mask, it leads to the fact that some sparse masks capture duplicate information or unnecessary features in the input which may have a negative effect on the results or slow down the inference. To resolve this problem, we define the learnable *mask score* $\bm{\alpha} \in \mathbf{R}^K$ to *indicate the importance* of each sparse mask. This mask score is calculated based on the contribution of each mask to the final results and the similarity between corresponding masks as follows: 
$$
\bm{\alpha}_{(t+1,k)}
= \bm{\alpha}_{(t,k)} + \frac{1}{K-1}\sum_{i \neq k, 1 \leq i \leq K} \left(1-  \frac{\left \| {\bm{O}^{\intercal}_{(t,i)}}\bm{O}_{(t,k)} \right \|_\text{F}^2}{\left \| \bm{O}_{(t,k)}^{\intercal}\bm{O}_{(t,k)} \right \|_\text{F} \left \| \bm{O}_{(t,i)}^{\intercal}\bm{O}_{(t,i)} \right \|_\text{F}}\right)
$$
where $\left \| . \right \|_{\text{F}}$ is the Frobenius norm; $t$ corresponds to iteration during learning; $\bm{O}_k$ is the output tensor corresponding to mask $\bm{M}_k$. Our goal is to compare the differences in distribution between features outputted from different sparse masks to identify which masks mostly produce the same outputs and then discard the redundant ones during the inference process. We note that during training, we utilize $K$ sparse masks and calculate the associated mask scores, while *during testing*, we select $\kappa$ masks} ($\kappa << K$) based on the mask score $\bm \alpha$ to use only the useful masks. The selected useful masks are then applied to the input human-scene representation (Equation 1) to produce the efficient contact presentation for human-scene interaction.

### Contact Network with Sparse Operations

Traditional contact networks such as POSA learn the human-scene contact using the full dense input tensor $\bm{I}$ with conventional layers such as convolution, group normalization, ReLU, etc. This leads to two problems: *i)* the dense input $\bm{I}$ contains unnecessary information (e.g., non-contact points) which may decrease the accuracy of the network, and *ii)* learning on a dense input $\bm{I}$ reduces the inference time as whole tensor $\bm{I}$ is used and subsequently increases the number of parameters of the network. To utilize our efficient contact representation $\bm{I'}$, we propose to replace the conventional matrix operations with our designed sparse operations, utilizing input from our sparse mask. As illustrated in Figure 3, the convolutional layer in dense tensor must loop through all the elements in the input tenser. However, on a sparse tensor, we compute  convolution outputs on a few specified points.

This strategy can be applied across different layers, including convolution, batch normalization, pooling, and more, all without necessitating changes to the network architecture. Next, we describe sparse operations in popular deep network layers.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788949312/sparse_tensor_ah9ekx.gif)
![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788949411/sparse_input_1_wtwtog.gif)
*<center>**Figure 3**:  Efficient Contact Representation method overview. We adopt Graph and Spiral Blocks from POSA as contact predictor backbone, replacing original layers with our sparse layers. Red cells denote non-zero kernel weights and mask values, blue cells represent coordinates, green cells indicate non-zero contact values, and white cells denote zeros.</center>*


**Convolution Layer**
The $k$-th reformatted inputs $\bm{S}_k$ and $\bm{C}_k$ are passed through the network and interact with sparse kernels $\bm{W} \in  \mathbf{R}^{m \times m}$ via a mapping function. In the convolutional layer, kernel weights with an indexing matrix $\mathcal{M}^n_k$ of a $k$-th mask at the $n$-th stride can be calculated as follows:
$$
\mathcal{M}^n_k = \begin{bmatrix} \hat{\bm{W}}[c] \ | \  \bm{C}_k[c'] \\\hat{\bm{W}}[i] \ | \ \bm{C}_k[i'] \end{bmatrix}, \ \ \begin{matrix}
c = (m^2-1)/2, \  c' = i' \  \forall i = c \\ i' = \text{idx}(\bm{C}_k[\bm{W},n,i]), \  \hat{\bm{W}}[i] \neq 0
\end{matrix}
$$
where $\hat{\bm{W}}$ is the flattened vector of the kernel $\bm{W}$ and $\bm{C}_k[\bm{W},n,i]$ is the value when kernel $\bm{W}$ is applied to the $k$-th sparse input $\bm{C}_k$ over $n$-th stride corresponding to the $i$ element. $\mathcal{M}^n_k$ is then retrieved in $\bm{C}_k$ and $\bm{S}_k$ to compute the sparse output  $\bm{C'}_k$ and  $\bm{S'}_k$ using the below equations.
$$
\begin{gathered}
\bm{C}^{'n}_k =  \left [ \mathcal{M}_k^n[0][1:] \right], \\ 
\bm{S}^{'n}_k = \sum_{i=1} \mathcal{M}_k^n[i][0] \bm{S}_k[\text{idx}\left ( \mathcal{M}_k^n[i][1] \right)]
\end{gathered} 
$$

**Linear Layer**
Linear layers applied to sparse tensors only change the number of channels in the feature matrix and do not affect the coordinate matrix. With $\bm{W}_l \in \mathbf{R}^{N_S \times N_S'}$ and $\bm{b} \in \mathbf{R}^{N_S'}$ are the weight matrix and bias vector of the linear layer, respectively, the linear operator output is:
$$
    \bm{S'}_{k} = \bm{W}_l\bm{S}_{k} + \bm{b}, \ \ \bm{C'}_k = \bm{C}_k 
$$



**Group Normalization Layer**
In group normalization, we divide the features into $G$ group, each group has $N_v / G$ feature values. Then the values in each group are normalized: 
$$
    \bm{\mu}_{k,g}^b = \dfrac{1}{N_k^b} \sum_{\substack{i:\bm{C}_k[i][0] = b}} \bm{S}_{k,g}[i]
$$
$$
    (\bm{\sigma}_{k,g}^b)^2 =  \dfrac{1}{N_k^b}\sum_{i:\bm{C}_k[i][0] = b} \left (  \bm{S}_{k,g}[i] - \bm{\mu}_{k,g}^b \right )^2 
$$
$$
    \bm{S}^{'b}_{k,g}
    = \dfrac{\bm{S}^b_{k,g} - \bm{\mu}^b_{k,g}}{\sqrt{(\bm{\sigma}^b_{k,g})^2 + \epsilon}}, \ \ \bm{C'}_k = \bm{C}_k
$$
where $g$ is the group index. When $G=1$, the group normalization becomes layer normalization instead.

**ReLU Layer**
For the ReLU and any other non-linearly layers, sparse operations only change each value in the feature matrix and do not affect the coordinate matrix. 
With  the activation function $\bm{f_{\text{act}}}$, the output is calculated as:
$$
    \bm{S'}_k = \bm{f_{\text{act}}}\left( \bm{S}_k \right) , \ \ \bm{C'}_k = \bm{C}_k 
$$




In the next part, we will validate the effectiveness of our ECO method on various datasets.

</CodeExplanation>