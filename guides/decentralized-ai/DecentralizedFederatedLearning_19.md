---
last_modified_on: "2023-11-10"
title: Decentralized Federated Learning and Research Directions (Part 2).
description: Decentralized Federated Learning and Research Directions.
series_position: 19
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: federated_learning"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


Federated Learning (FL) has become a powerful learning technique in Deep Learning. Due to the ability to train deep learning models with a large amount of cross-devices or cross-silos, extracting knowledge from distributed data, user privacy, maintaining privacy and sensitive data, increasing learning efficiency, and saving communication costs, FL has been widely studied and applied to various applications. Centralized Federated Learning (CFL) is the most common approach in research and application, which only requires a server and a connection between the server and each client. However, CFL suffers high risk and low trust when the server has the highest probability of bottleneck or being vulnerable to suspicious attacks. DFL addresses to overcome these limitations by removing the server in the configuration and letting the clients connect to each other. Thus, we aim to investigate the potential of DFL in research and application and find out research, development, and optimization directions for this topic.

In the previous part, we have investigate accumulation method direction. In this part, we will export topology design direction.

## Topology design 
When researchers design an algorithm (e.g aggregation), they usually run benchmark on fixed configuration, such as fully connected network, partically connected network, and node clustering. Each network has its own advantages and disadvantages, which will be shown in Figure 7. 

![Fig-7](https://drive.google.com/uc?export=view&id=1em8L_NTyOnOp4pPfm971T48fm_C0Ysy8)*<center>**Figure 7**: Some network topologies. Image is taken from [this paper](https://arxiv.org/pdf/2211.08413.pdf)</center>*

A robust fully completed network greatly enhances the communication cost, while a partially completed network becomes less trusted and delays in transmitting parameters. Therefore, designing a robust network topology is an extreme problem. 

[**Throughput-Optimal Topology Design for Cross-Silo Federated Learning**](https://arxiv.org/pdf/2010.12229.pdf) The authors found a topology with the largest throughput or with provable throughput guarantees using the theory of max-plus linear systems to compute the system throughput number of communication rounds per time unit. They defined the $M_{CT}$ problem: Given a connectivity graph $\mathcal{G}_c$, they want to find the overlay (subgraph) $\mathcal{G}_o$ to be a strongly directed graph with minimal cycle time. 

When $\mathcal{G}_c$ is an undirected graph, the author suggests using the links in $\mathcal{G}_c$ with the smallest average of delays, This leads to the **minimum weight spanning tree** in an undirected graph, which can be solved using Prim algorithm. In the case that $\mathcal{G}_c$ is directed, the problem is non-deterministic polynomial-time hardness (NP-hard). However, in case that $\mathcal{G}_c$ is a complete Euclidean edge-capacitated graph (the delay is symmetric and satisfies triangle inequality), Christofides’ algorithm can be an approximation algorithm for $M_{CT}$. 

[**Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology**](https://arxiv.org/pdf/2207.09657.pdf): The authors construct the multigraph using the overlay graph, then parse this multigraph into different simple graphs with isolated nodes. The node with a long delay time is chosen as the isolated node (does not participate in aggregating parameters), then after some communication rounds, the isolated note is updated, and later becomes a normal node, which helps reduce training time.


[**D-Cliques: Compensating NonIIDness in Decentralized Federated Learning with Topology**](https://arxiv.org/pdf/2104.07365.pdf): They deal with label distribution at nodes in heterogeneous data (in the case when each node holds data with a simple class only), in contrast to the homogeneous data where the label distribution is same across nodes. They introduced the topology design D-Cliques to compensate for data heterogeneity. D-Cliques aims to construct a topology in which each node is a part of a clique (a subset of nodes whose induced subgraph is fully connected) such that the label distribution in each clique is close to the global label distribution.

- Construct locally representative cliques: For a clique $C$ and a label $y$, they defined the distribution of $y$ in $C \subset N$ as $p_C(y)$, the global distribution $p(y)$ and the skew of $C$ following: 

    $$
    p_C(y) = \dfrac{1}{|C|} \sum_{i \in C}p_i(y)
    $$ 

    $$
    p(y) = \dfrac{1}{n} \sum_{i \in N}p_i(y)
    $$

    $$
    \text{skew}(C) = \sum_{l=1}^{L} =|p_C(y=l) - p(y=l)|
    $$

    They tried to construct a set of cliques with a small skew by the Greedy-Swap Algorithm, to ensure that the distribution of labels in a clique is as close to the global distribution as possible.

- Add Sparse Inter-Clique Connections: To achieve global consensus and convergence. By seeing each clique as a node, they provide various topologies that suited well (ring, fractal, fully connected) by adding an inter-clique connection between two cliques in a pair.

- Optimization: By limiting the number of clique connections, the difference in weight of two nodes in an inter-clique connection biases the gradient of the classes represented in these two nodes. They solved the problem by applying Clique Averaging to D-SGD. They average only the gradients of neighbors within the same clique to reduce bias due to inter-clique edges.

## References 
1. Yuan, Liangqi, et al. "Decentralized Federated Learning: A Survey and Perspective." arXiv preprint arXiv:2306.01603 (2023).
2. Beltrán, Enrique Tomás Martínez, et al. "Decentralized federated learning: Fundamentals, state of the art, frameworks, trends, and challenges." IEEE Communications Surveys & Tutorials (2023).
3. Koloskova, Anastasia, et al. "A unified theory of decentralized sgd with changing topology and local updates." International Conference on Machine Learning. PMLR, 2020.
4. Jiang, Jingyan, and Liang Hu. "Decentralised federated learning with adaptive partial gradient aggregation." CAAI Transactions on Intelligence Technology 5.3 (2020): 230-236.
5. Chen, Zhikun, et al. "Dacfl: Dynamic average consensus based federated learning in decentralized topology." arXiv preprint arXiv:2111.05505 (2021).
6. Sun, Tao, Dongsheng Li, and Bao Wang. "Decentralized federated averaging." IEEE Transactions on Pattern Analysis and Machine Intelligence 45.4 (2022): 4289-4301.
7. Shi, Yifan, et al. "Improving the model consistency of decentralized federated learning." arXiv preprint arXiv:2302.04083 (2023).
8. Marfoq, Othmane, et al. "Throughput-optimal topology design for cross-silo federated learning." Advances in Neural Information Processing Systems 33 (2020): 19478-19487.
9. Do, Tuong, et al. "Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology." Proceedings of the IEEE/CVF International Conference on Computer Vision. 2023.
10. Bellet, Aurélien, Anne-Marie Kermarrec, and Erick Lavoie. "D-cliques: Compensating for data heterogeneity with topology in decentralized federated learning." 2022 41st International Symposium on Reliable Distributed Systems (SRDS). IEEE, 2022.


