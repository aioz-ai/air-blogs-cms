---
last_modified_on: "2023-06-15"
id: FADNet1
title: Deep Federated Learning for Autonomous Driving (Part 1)
description: Deep Federated Learning for Autonomous Driving (methodology)
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
Autonomous driving is an active research topic in both academia and industry. However, most of the existing solutions focus on improving the accuracy by training learnable models with centralized large-scale data. Therefore, these methods do not take into account the user's privacy. In this paper, we present a new approach to learn autonomous driving policy while respecting privacy concerns. We propose a peer-to-peer Deep Federated Learning (DFL) approach to train deep architectures in a fully decentralized manner and remove the need for central orchestration. We design a new Federated Autonomous Driving network (FADNet) that can improve the model stability, ensure convergence, and handle imbalanced data distribution problems while is being trained with federated learning methods. Intensively experimental results on three datasets show that our approach with FADNet and DFL achieves superior accuracy compared with other recent methods. Furthermore, our approach can maintain privacy by not collecting user data to a central server. 

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our source code can be found at: https://github.com/aioz-ai/FADNet

## 1. Introduction
In this paper, our goal is to develop an end-to-end driving policy from sensory data while maintaining the user's privacy by utilizing FL. We address the key challenges in FL to make sure our deep network can achieve competitive performance when being trained in a fully decentralized manner. Fig.1 shows an overview of different learning approaches for autonomous driving. In Centralized Local Learning (CLL), the data are collected and trained in one local machine. Hence, the CLL approach does not take into account the user's privacy. The Server-based Federated Learning (SFL) strategy requires a central server to orchestrate the training process and receive the contributions of all clients. The main limitation of SFL is communication congestion when the number of clients is large. Therefore, we follow the peer-to-peer federated learningto set up the training. Our peer-to-peer Deep Federated Learning (DFL) is fully decentralized and can reduce communication congestion during training. We also propose a new Federated Autonomous Driving network (FADNet) to address the problem of model convergence and imbalanced data distribution. By training our FADNet using DFL, our approach outperforms recent state-of-the-art methods by a fair margin while maintaining user data privacy.

![](https://drive.google.com/uc?id=1BY31bZIBK90B_dPw-xP3ZiiGxmi0Acbh)*<center>**Figure 1**. An overview of different learning methods for autonomous driving. (a) Centralized Local Learning, (b) Server-based Federated Learning, and (c) our peer-to-peer Deep Federated Learning. Red arrows denote the aggregation process between silos. Yellow lines with a red cross indicate the non-sharing data between silos. </center>*

Our contributions can be summarized as follows:
- We propose a fully decentralized, peer-to-peer Deep Federated Learning framework for training autonomous driving solutions.
- We introduce a Federated Autonomous Driving network that is well suitable for federated training.
- We introduce two new datasets and conduct intensive experiments to validate our results.

## 2. Problem Formulation
We consider a federated network with $N$ siloed data centers (e.g., autonomous cars) $\mathcal{D}_{i}$, with $i \in [1,N]$. Our goal is to collaboratively train a global driving policy $\theta$ by aggregating all local learnable weights $\theta_i$ of each silo. Note that, unlike the popular centralized local training setup, in FL training, each silo does not share its local data, but periodically transmits model updates to other silos.

In practice, each silo has the training loss $\mathcal{L}_i(\xi_i, \theta_i)$. $\xi_i$ is the ground-truth in each silo $i$.  $\mathcal{L}_i(\xi_i, \theta_i)$ is calculated as the regression loss. This regression loss is modeled by a deep network that takes RGB images as inputs and predicts the associated steering angles.



## 3. Deep Federated Learning for Autonomous Driving
A popular training method in FL is to set up a central server that orchestrates the training process and receives the contributions of
all clients (Server-based Federated Learning - SFL). The limitation of SFL is the server potentially represents a single point of failure in the system. We also may have communication congestion between the server and clients when the number of clients is massive. Therefore, in this work, we utilize the peer-to-peer FL to set up the training scenario. In peer-to-peer FL, there is no centralized orchestration, and the communication is via peer-to-peer topology. However, the main challenge of peer-to-peer FL is to assure model convergence and maintain accuracy in a fully decentralized training setting.

![](https://drive.google.com/uc?id=1kyNnPwDRQDSy7du5_ybD5KMJPmieYsaj)*<center>**Figure 2**. An overview of our peer-to-peer Deep Federated Learning method. (a) A simplified version of an overlay graph. (b) The training methodology in the overlay graph. Note that blue arrows denote the local training process in each silo; red arrows denote the aggregation process between silos controlled by the overlay graph; yellow lines with a red cross indicate the non-sharing data between silos; the arrow indicates that the process is parallel. </center>*

Fig.2 illustrates our Deep Federated Learning (DFL) method. Our DFL follows the peer-to-peer FL setup with the goal to integrate a deep architecture into a fully decentralized setting that ensures convergence while achieving competitive results compared to the traditional Centralized Local Learning or SFL approach. In practice, we can consider a silo as an autonomous car. Each silo maintains a local learnable model and does not share its data with other silos. We represent the silos as vertices of a communication graph and the FL is performed on an overlay, which is a sub-graph of this communication graph.

### Designing the Overlay
Let $\mathcal{G}_c = (\mathcal{V}, \mathcal{E}_c)$ is the connectivity graph that captures the possible direct communications among $N$ silos. $\mathcal{V}$ is the set of vertices (silos), while $\mathcal{E}_c$ is the set of communication links between vertices. $\mathcal{N}_i^{+}$ and $\mathcal{N}_i^{-}$ are in-neighbors and out-neighbors of a silo $i$, respectively. As in~\cite{marfoq2020throughput}, we note that it is unnecessary to use all the connections of the connectivity graph for FL. Indeed, a sub-graph called an overlay, $\mathcal{G}_o = (\mathcal{V}, \mathcal{E}_o)$ can be generated from $\mathcal{G}_c$. In our work, $\mathcal{G}_o$ is the result of Christofides’ Algorithm~\cite{monnot2003approximation}, which yields a strong spanning sub-graph of $\mathcal{G}_c$ with minimal cycle time. One cycle time or time per communication round, in general, is the time that a vertex waits for messages from the other vertices to do a computational update. 

In practice, one block cycle time of an overlay $\mathcal{G}_o$ depends on the delay of each link $(i, j)$, denoted as $d_o(i, j)$, which is the time interval between the beginning of a local computation at node $i$, and the receiving of $i$'s messages by $j$. Furthermore, without concerns about access links delays between vertices, our graph is treated as an edge-capacitated network with:

$$

d_o(i,j) = s \times T_c(i) + l(i,j) + \frac{M}{B(i,j)}

$$

where $T_c(i)$ is the time to compute one local update of the model; $s$ is the number of local computational steps; $l(i,j)$  is the link latency; $M$ is the model size; $B(i,j)$ is available bandwidth of the path $(i,j)$. As in~\cite{marfoq2020throughput},  we set $s=1$.

### Training Algorithm

At each silo $i$, the optimization problem to be solved is:

$$

\theta_i^{*} = \underset{\theta_i}{\arg\min} \underset{\xi \sim \mathcal{D}_i}{\mathbb{E}}[\mathcal{L}(\xi_i, \theta_i)]

$$

We apply the distributed federated learning algorithm, DPASGD, to solve the optimizations of all the silos. In fact, after waiting one cycle time, each silo $i$ will receive parameters $\theta_j$ from its in-neighbor $\mathcal{N}_i^{+}$ and accumulate these parameters multiplied with a non-negative coefficient from the consensus matrix $\mathbf{A}$. It then performs $s$ mini-batch gradient updates before sending $\theta_i$ to its out-neighbors $\mathcal{N}_i^{-}$, and the algorithm keeps repeating. Formally, at each iteration $k$, the updates are described as: 

$$

\theta_{i}\left(k + 1\right) = \begin{cases}
    \sum_{j \in \mathcal{N}_i^{+} \cup{i}}\textbf{A}_{i,j}{\theta}_{j}\left(k\right), \textit{ if k} \equiv 0 \pmod{s + 1},\\
    {\theta}_{i}\left(k\right)-\alpha_{k}\frac{1}{m}\sum^m_{h=1}\nabla \mathcal{L}\left({\theta}_{i}\left(k\right),\xi_i^{\left(h\right)}\left(k\right)\right),  \text{otherwise.}
\end{cases}

$$

where $m$ is the mini-batch size and $\alpha_k > 0$ is a potentially varying learning rate.

### Federated Averaging

To compute the prediction of models in all silos, we compute the average model $\theta$ using weight aggregation from all the local model $\theta_i$. The federated averaging process is conducted as follow:

$$

\theta = \frac{1}{\sum^N_{i=0}{\lambda_i}} \sum^N_{i=0}\lambda_{{i}} \theta_{{i}}

$$

where $N$ is the number of silos; $\lambda_i = \{0,1\}$. Note that $\lambda_i = 1$ indicates that silo $i$ joins the inference process and $\lambda_i = 0$ if not. The aggregated weight $\theta$ is then used for evaluation on the testing set $\mathcal{D}_{test}$.

## 4. Network Architecture
One of the main challenges when training a deep network in FL is the imbalanced and non-IID (identically and independently distributed) problem in data partitioning across silos. To overcome this problem, the learning architecture should have an appropriate design to balance the trade-off between convergence ability and accuracy performance. In practice, the deep architecture has to deal with the high variance between silo weights when the accumulation process for all silos is conducted. To this end, we design a new Federated Autonomous Driving Network, which is based on ResNet8, as shown in Fig.3. 

![](https://drive.google.com/uc?id=1lrCsZG-T_3BtnGnEFb0kUTu9PFmkwtjV)*<center>**Figure 3**. Human Tracking. </center>*

In particular, our proposed FADNet first comprises an input layer normalization to improve the stability of the abstract layer. This layer aims to handle different distributions of input images in each silo. Then, a convolution layer following by a max-pooling layer is added to encode the input. To handle the vanishing gradient problem, three residual blocks are appended with a following FC layer to extract ResBlock features. However, using residual blocks increases the variance of silo weights during the aggregation process and affects the convergence ability of the model. To address this problem, we add a Global Average Pooling layer (GAP) associated with each residual block. GAP is a non-weight pooling layer which sums out the spatial information from each residual block. Thus, it is not affected by the weighted variance problem. The output of each GAP layer is passed through an Accumulation layer to accrue the Support feature. 
The ResBlock feature and the Support feature from GAP layers are fed into the Aggregation layer to calculate the model loss in each silo.

In our design, the Accumulation and Aggregation layers aim to reduce the variance of the global model since we need to combine multiple model weights produced by different silos. In particular, the Accumulation layer is a variant of the fully connected (FC) layer. Instead of weighting the contribution of input nodes as in FC, the Accumulation layer weights the contribution of multiple features from input layers. The Accumulation layer has a learnable weight matrix $w \in \mathbb{R}^\text{n}$. Its number of nodes is equal to the \text{n} number of input layers. Note that the support feature from the Accumulation layer has the same size as the input. Let $F = \{f_\text{1}, f_\text{2}, ..., f_\text{n}\}, \forall f_\text{h} \in \mathbb{R}^\text{d}$ be the collection of $\text{n}$ number of the features extracted from $\text{n}$ input GAP layers; $\text{d}$ is the unified dimension. The Accumulation outputs a feature $f_\text{c} \in \mathbb{R}^\text{d}$ in each silo $i$, and is computed as:

$$

f_\text{c} = Accumulation(F)_i = \sum^{\text{n}}_{\text{h}=1}(w_\text{h}f_\text{h})_i

$$

The Aggregation layer is a fusion between the ResBlock feature extracted from the backbone and the support feature from the Accumulation layer. For simplicity, we use the Hadamard product to compute the aggregated feature. This feature is then averaged to predict the steering angle. Let $f_\text{s} \in \mathbb{R}^\text{d}$ be the ResBlock features extracted from the backbone. The output driving policy $\theta_i$ of silo $i$ can be calculated as:

$$

\theta_i = Aggregation(f_\text{s}, f_\text{c})_i = \bar{(f_\text{s} \odot f_\text{c})}_i

$$

where $\odot$ denotes Hadamard product; $\bar{(*)}$ denotes the mean and we set $\text{d} = 6,272$.

## Next
In the next post, we will show the effectiveness and efficiency of FADNet during Federated Learning proccess.