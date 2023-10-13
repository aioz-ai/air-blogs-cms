---
last_modified_on: "2023-09-14"
id: MultigraphDFL1
title: Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology (Part 1)
description: An introduction about decentralized federated learning approaches and the definition of multigraph
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
Federated learning is an active research topic since it enables several participants to jointly train a model without sharing local data. Currently, cross-silo federated learning is a popular training setting that utilizes a few hundred reliable data silos with high-speed access links to training a model. While this approach has been widely applied in real-world scenarios, designing a robust topology to reduce the training time remains an open problem. In this paper, we present a new multigraph topology for cross-silo federated learning. We first construct the multigraph using the overlay graph. We then parse this multigraph into different simple graphs with isolated nodes. The existence of isolated nodes allows us to perform model aggregation without waiting for other nodes, hence effectively reducing the training time. Intensive experiments on three public datasets show that our proposed method significantly reduces the training time compared with recent state-of-the-art topologies while maintaining the accuracy of the learned model.

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our code can be found at: https://github.com/aioz-ai/MultigraphFL

## 1. Introduction
Federated learning involves training models using remote devices or isolated data centers while keeping the data localized to respect user privacy policies. According to available literature, there are two prominent training scenarios: the "cross-device" scenario, which includes numerous unreliable edge devices with limited computational capacity and slow connection speeds, and the "cross-silo" scenario, which features a smaller number of reliable data silos with powerful computing resources and high-speed access links. Recently, the cross-silo scenario has gained traction in various federated learning applications.

In practical terms, federated learning represents a promising research avenue that allows us to harness the capabilities of machine learning techniques while upholding user privacy. Key obstacles in federated learning encompass issues like model convergence, communication bottlenecks, and disparities in data distributions across different silos. A commonly employed federated training approach involves establishing a central node responsible for overseeing the training process and aggregating contributions from all clients. However, a drawback of this client-server approach is the potential for communication bottlenecks, especially when dealing with a large number of clients. To mitigate this limitation, recent research has explored the concept of decentralized or peer-to-peer federated learning, where communication occurs via a peer-to-peer network topology, eliminating the need for a central node. Nevertheless, a major challenge in decentralized federated learning remains achieving rapid training while ensuring model convergence and preserving model accuracy.

In federated learning, the structure of communication networks holds significant importance. Specifically, an efficient network design contributes to quicker convergence, resulting in reduced training duration and energy consumption, as measured by worst-case convergence bounds within the topology's framework. Additionally, the topology's design has direct implications for other training-related challenges, including network congestion, overall model accuracy, and energy efficiency. The development of a resilient network structure capable of minimizing training time while preserving model accuracy remains an ongoing challenge in federated learning. Our paper is dedicated to devising a novel network design tailored for cross-silo federated learning, a prevalent scenario in practical applications.

![](https://vision.aioz.io/f/edd1ea291c334b3ab6f3/?dl=1)*<center>**Figure 1**. We conducted a comparative analysis of various network structures using the FEMNIST dataset and the Exodus network. After completing 6,400 communication rounds, we measured and reported both the accuracy and the total wall-clock training time (or overhead time). Notably, our approach resulted in a substantial reduction in training duration while upholding model accuracy. </center>*

Lately, various network configurations have emerged for cross-silo federated learning. For instance, the STAR topology involves an orchestrator averaging all models during each communication round. Another approach, known as MATCHA, divides potential communications into pairs of clients, with random selection for model transmission in each round. Additionally, the RING topology employs max-plus linear systems. Despite progress in this field, challenges persist, including access link congestion, straggler effects, and the establishment of diverse topologies across communication rounds.

In this paper, we introduce a novel multigraph topology inspired by recent advancements in federated learning. Our aim is to enhance the efficiency of cross-silo federated learning. Our approach involves constructing a multigraph based on the overlay of existing network topologies. Subsequently, we decompose this multigraph into simpler graphs, each featuring only a single edge connecting two nodes. These individual graphs are referred to as "states" within the multigraph. Importantly, each state can involve isolated nodes that perform model aggregation independently, reducing the cycle time in each communication round significantly. Our intensive experiments demonstrate that our proposed topology outperforms existing state-of-the-art methods by a wide margin in terms of training time for cross-silo federated learning, as illustrated in Figure.1.

## 2. Overview 

**Federated Learning** is recognized for its capacity to safeguard data privacy. In its modern incarnation, federated learning adopts a centralized network design, where a central node collects gradients from client nodes to update a global model. Early contributions in federated learning research include pioneering work and seminal papers by various researchers. Subsequent extensions and developments in federated learning and related distributed optimization algorithms have been proposed. Federated Averaging (FedAvg), initially introduced by one group, has inspired variations and other recent state-of-the-art model aggregation techniques, addressing convergence and the non-IID (non-identically and independently distributed) data challenge. Despite its simplicity, the client-server approach faces communication and computational bottlenecks at the central node, particularly when dealing with a large number of clients.

**Decentralized Federated Learning** flips the traditional federated learning model, enabling direct interactions between siloed data nodes, eliminating the necessity for a central coordinating node. While this approach mitigates communication congestion at a central point, optimizing a fully peer-to-peer network presents substantial challenges. The decentralized periodic averaging stochastic gradient descent method has demonstrated convergence rates comparable to centralized algorithms, making large-scale model training feasible. Furthermore, previous research has conducted systematic analyses of decentralized federated learning. A recent advancement involves leveraging a knowledge distillation mechanism to facilitate collaboration among silos in decentralized federated scenarios while preserving privacy among neighboring nodes.

**Communication Topology** plays a fundamental role in influencing the complexity and convergence behavior of federated learning. Numerous efforts have been dedicated to improving the efficiency of communication topologies, including star-shaped topologies and optimized-shaped topologies. In particular, a spanning tree topology has been introduced to reduce training time.

The STAR topology is designed for orchestrating the averaging of model updates in each communication round. Meanwhile, the MATCHA approach focuses on accelerating the training process through decomposition sampling. Recognizing the impact of straggler effects on communication round duration, methods for selecting the degree of a regular topology have been explored.

The RING topology is tailored for cross-silo federated learning and leverages the principles of max-plus linear systems. A sample-induced topology has been introduced, capable of effectively recovering the performance of existing SGD-based algorithms and their corresponding convergence rates. In a recent comprehensive survey, various models, frameworks, and algorithms related to network topologies in federated learning have been explored.

**Multigraph** is a concept that originates from traditional mathematics. In conventional terms, a "graph" typically denotes a simple graph without loops or multiple edges between two nodes. In contrast, a multigraph allows for the presence of multiple edges between two nodes. In the realm of deep learning, multigraphs have found utility across various domains, including clustering, medical image processing, traffic flow prediction, activity recognition, recommendation systems, and cross-domain adaptation. In this research, we employ a multigraph construction to facilitate isolated nodes and expedite training in cross-silo federated learning.


## 3. Preliminaries
### 3.1 Federated Learning

In federated learning, silos do not share their local data, but still periodically transmit model updates between them. Given $N$ siloed data centers, the objective function for federated learning is:

$$
\min_{\textbf{w} \in \mathbb R^d}  \sum^{N}_{i=1}p_i E_{\xi_i}\left[ L_{i}\left(\textbf{w}, \xi_i\right)\right],
$$

where $L_{i}(\textbf{w}, \xi_i)$ is the loss of model parameterized by the weight $\textbf{w} \in \mathbb R^d$, $\xi_i$ is an input sample drawn from data at silo $i$, and the coefficient $p_i>0$ specifies the relative importance of each silo. Recently, different distributed algorithms have been proposed to optimize the equation. In this work, DPASGD is used to update the weight of silo $i$ in each training round as follows:

$$
\textbf{w}_{i}\left(k + 1\right) = \\
\begin{cases}
    \sum_{j \in \mathcal{N}_i^{+} \cup{\{i\}}}\textbf{A}_{i,j}\textbf{w}_{j}\left(k\right), \\\qquad\qquad\qquad\qquad\qquad \text{if k} \equiv 0 \left(\text{mod }u + 1\right),\\
    \textbf{w}_{i}\left(k\right)-\alpha_{k}\frac{1}{b}\sum^b_{h=1}\nabla L_i\left(\textbf{w}_{i}\left(k\right),\xi_i^{\left(h\right)}\left(k\right)\right),  \\\qquad\qquad\qquad\qquad\qquad\qquad\qquad\quad\text{otherwise.}
\end{cases}
$$

where $b$ is the batch size, $i,j$ denote the silo, $u$ is the number of local updates, $\alpha_k > 0$ is a potentially varying learning rate at $k$-th round, $\textbf{A} \in R^{N \times N}$ is a consensus matrix with non-negative weights, and $\mathcal{N}_i^{+}$ is the in-neighbors set that silo $i$ has the connection to.

### 3.2 Multigraph for Federated Learning
**Connectivity and Overlay**. We consider the \textit{connectivity} $\mathcal{G}_c = (\mathcal{V}, \mathcal{E}_c)$ as a graph that captures possible direct communications among silos. Based on its definition, the connectivity is often a fully connected graph and is also a directed graph. % whenever the upload and download are set during learning. 
The \textit{overlay} $\mathcal{G}_o$ is a connected subgraph of the connectivity graph, i.e., $\mathcal{G}_o = (\mathcal{V}, \mathcal{E}_o)$, where $\mathcal E_o \subset \mathcal E_c$. Only nodes directly connected in the overlay graph $\mathcal{G}_o$ will exchange the messages during training.

**Multigraph**. While the connectivity and overlay graph can represent different topologies for federated learning, one of their drawbacks is that there is only one connection between two nodes. In our work, we construct a \textit{multigraph} $\mathcal{G}_m = (\mathcal{V}, \mathcal{E}_m)$ from the overlay $\mathcal{G}_o$. The multigraph can contain multiple edges between two nodes. In practice, we parse this multigraph to different graph states, each state is a simple graph with only one edge between two nodes. 

In the multigraph $\mathcal{G}_m$, the connection edge between two nodes has two types: \textit{strongly-connected} edge and \textit{weakly-connected} edge. Under both strong and weak connections, the participated nodes can transmit their trained models to their out-neighbours $\mathcal{N}_i^{-}$ or download models from their in-neighbours $\mathcal{N}_i^{+}$. However, in a strongly-connected edge, two nodes in the graph must wait until all upload and download processes between them are finished to do model aggregation. On the other hand, in a weakly-connected edge, the model aggregation process in each node can be established whenever the previous training process is finished by leveraging up-to-date models which have not been used before from the in-neighbours of that node.

**State of Multigraph** Given a multigraph $\mathcal{G}_m$, we can parse this multigraph into different simple graphs with only one connection between two nodes (either strongly-connected or weakly-connected). We denote each simple graph as a state $\mathcal{G}_m^s$ of the multigraph. 

**Isolated Node**. A node is called isolated when all of its connections to other nodes are weakly-connected edges. Figure.2 shows the graph concepts and isolated nodes.

![](https://vision.aioz.io/f/b2fd561e35904bbdac3a/?dl=1)*<center>**Figure 2**. Example of connectivity, overlay, multigraph, and a state of our multigraph. Blue node is an isolated node. Dotted line denotes a weakly-connected edge. </center>*

## Next
In the next post, we will mention the delay time and cylce time also how multigraph can be constructed.
