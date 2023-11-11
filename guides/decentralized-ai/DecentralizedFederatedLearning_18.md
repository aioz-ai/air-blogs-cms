---
last_modified_on: "2023-11-04"
title: Decentralized Federated Learning and Research Directions (Part 1).
description: Decentralized Federated Learning and Research Directions.
series_position: 18
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: federated_learning"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


Federated Learning (FL) has become a powerful learning technique in Deep Learning. Due to the ability to train deep learning models with a large amount of cross-devices or cross-silos, extracting knowledge from distributed data, user privacy, maintaining privacy and sensitive data, increasing learning efficiency, and saving communication costs, FL has been widely studied and applied to various applications. Centralized Federated Learning (CFL) is the most common approach in research and application, which only requires a server and a connection between the server and each client. However, CFL suffers high risk and low trust when the server has the highest probability of bottleneck or being vulnerable to suspicious attacks. DFL addresses to overcome these limitations by removing the server in the configuration and letting the clients connect to each other. Thus, we aim to investigate the potential of DFL in research and application and find out research, development, and optimization directions for this topic.


## Introduction
Decentralized Federated Learning (DFL) is a decentralized structure in which clients (nodes) communicate and share model parameters with each other. The client may be a personal device (cross-device) or a whole organization (silo).
Unlike Centralized Federated Learning, this setup does not need a server to communicate with the clients and help resolve most of the limitations of CFL: 
- DFL improves fault tolerance because nodes constantly update their knowledge about available nodes or those that have ceased communicating. This enhances the robustness of the network and minimizes the risk of a single point of failure when the bottleneck at the server will harm the whole network.
- DFL improves trust issues by distributing trust between the federation nodes, which determines the trust of the entire network, drastically reducing the single point of attack. 
- DFL reduces the network bottleneck issue by allowing more evenly distributed communication and workload among the nodes, thus reducing the chances of congestion or delays in the overall network performance. 
- DFL makes the network customization more diverse without the server, which can further save communication and computational resources with higher confidence in diverse variants. The pointings and connections between nodes can be configured and changed adaptively depending on the scenario.

Despite the benefits of DFL configures, it also remains challenges. The effectiveness of DFL depends on the network topology design and the aggregation between a group of nodes. Therefore, to maximize the performance of the model as well as reduce the communication cost ahead, choosing the suitable topology and designing an aggregate algorithm is an extreme challenge. 



## Research Directions

### DFL Optimization 
Despite the advantages of DFL compared with CFL, optimization strategies are necessary to fully leverage the efficiency of the DFL approach. DFL contains two main stages repeated: the client trains the model locally and aggregates the models from itself and its neighbors. 

In CFL, [FedAvg](https://arxiv.org/pdf/1602.05629.pdf) is widely used as a baseline (the server aggregates the model parameters by averaging them), and many works have improved FedAvg, while in DFL, no algorithm has risen among the others. Many customizations of FedAvg and new techniques have been focused on decentralized data: 

**Decentralized SGD (D-SGD)**: Provides optimization for a set of objective functions distributed over a network. D-SGD applies stochastic gradient updates, performed locally on each worker, followed by a  consensus operation, where nodes average their values with their neighbors.

![Fig-1](https://drive.google.com/uc?export=view&id=127vg0_KZSbs_Zzblq5YV7bqWSt5xvyl_)
*<center>**Figure 1**: [D-SGD Algorithm.](https://arxiv.org/pdf/2003.10422.pdf)</center>*

**FedPGA**:  Defines a partial gradient at each nodes, which will be exchanged to the neighbors to speed up communication time. At the same time, it reduces the convergence rate by adapting the step size of the stable gradient descent direction

![Fig-2](https://drive.google.com/uc?export=view&id=1WD90IUpuz9crh4g1iaU8YpYK2X3oTppv)*<center>**Figure 2**: [FedPGA Algorithm.](https://ietresearch.onlinelibrary.wiley.com/doi/epdf/10.1049/trit.2020.0082)</center>*
**Dynamic Average Consensus-based FL (DACFL)**:  Each user trains its local model utilizing its own training data. The respective trained model intermediates of different users are treated as different discrete-time reference input sequences. In order to aggregate different users' intermediate models, they tracked the average model estimation. Each user in DACFL averages its neighborhood's models to reinitialize its local model after each training round.

![Fig-3](https://drive.google.com/uc?export=view&id=1nPVnCgLnnMYSxJ7JVGupDdgLB36iEDs2)*<center>**Figure 3**: [DACFL Algorithm.](https://arxiv.org/pdf/2111.05505.pdf)</center>*

**Decentralized FedAvg with Momentum (DFedAvgM)**: It uses SGD with momentum to train the local models of the federation participants, communicating only with their neighbors in an undirected graph. DFedAvgM plays the tradeoff between local computing and communications, which makes DFedAvgM more efficient than D-SGD.

![Fig-4](https://drive.google.com/uc?export=view&id=1qObTafJEhYq3g2Q0E_3KQzWWdYWnPQ4v)*<center>**Figure 4**: [DFedAvgM Algorithm](https://arxiv.org/pdf/2104.11375.pdf).</center>*

Another version of DFedAvgM has been proposed, named Quantized DFedAvgM, to reduce communication cost by lowering the bits used to send the model to that client's neighbors.

![Fig-5](https://drive.google.com/uc?export=view&id=1aVgoJe88SyXaQnEy9hlLqo_hmKmA-tRu)*<center>**Figure 5**: [Quantized DFedAvgM Algorithm.](https://arxiv.org/pdf/2104.11375.pdf)</center>*

**DFedSAM and DFedSAM-MGS**: To solve the question: "*Can we design DFL algorithms that can mitigate the inconsistency among local models and achieve similar performance to its centralized counterpart*", the authors have proposed the two algorithms: DFedSAM and DFedSAM-MGS.
- DFedSAM leverages gradient perturbation to generate local flat models via [Sharpness Aware Minimization (SAM)](https://openreview.net/pdf?id=6Tm1mposlrM), which searches for models with uniformly low loss values and improve local training
- DFedSAM-MGS adopts [Multiple Gossip Steps (MGS)](https://arxiv.org/pdf/2011.10643.pdf) for better model consistency, which accelerates the aggregation of local flat models and better balances communication complexity and generalization.

![Fig-6](https://drive.google.com/uc?export=view&id=1hV6xKKBT5gjHXktTRkrprntalkSS3CdF)*<center>**Figure 6**: [DFedSAM and DFedSAM-MGS Algorithm.](https://arxiv.org/pdf/2302.04083.pdf)</center>*


## Next
We have investigated diiferent accumulation method. In the next post, we will explore another direction for decentralized federated learning, Topology design.


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
