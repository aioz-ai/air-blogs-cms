---
last_modified_on: "2023-09-14"
id: MultigraphDFL2
title: Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology (Part 2)
description: Delay time, Cycle Time, and how to construct multigraph
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
In previous paart, we how explore decentralized federated learning, how and why multigraph is proposed to improve training process. In this part, we will investigate that how delay time and cycle time is affected by the modification of multigraph in the  design of the topology. Also, we will explore how multigraph can be constructed. 

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our code can be found at: https://github.com/aioz-ai/MultigraphFL

## 1. Delay time in multigraph
A delay to an edge $e(i, j)$ is the time interval when node $j$ receives the weight sending by node $i$, which can be defined by:

$$
d(i,j) = u \times T_c(i) + l(i,j) + \frac{M}{O(i,j)},
$$
where $T_{c}(i)$ denotes the time to compute one local update of the model; $u$ is the number of local updates; $l(i,j)$ is the link latency; $M$ is the model size; $O(i, j)$ is the total network traffic capacity. However, unlike other communication infrastructures, the multigraph only contains connections between silos without other nodes such as routers or amplifiers. Thus, the total network traffic capacity $O(i,j) = \text{min}\left(\frac{C_{\rm{UP}}(i)}{\left|{\mathcal{N}_{i}^{-}}\right|}, \frac{C_{\rm{DN}}(j)}{\left|\mathcal{N}_{i}^{+}\right|}\right)$ where $C_{\rm{UP}}$ and $C_{\rm{DN}}$ denote the upload and download link capacity. Note that the upload and download processes can happen in parallel. 

Since multigraph can contain multiple edges between two nodes, we extend the definition of the delay in the previous  equation to $d_k(i,j)$, with $k$ is the $k$-th communication round during the training process, as:

$$
d_{k+1}(i,j) =
\begin{cases}
    d_k(i,j), \\\qquad \qquad \text{if } e_{k+1}(i,j) = 1\text{ and }e_{k}(i,j) = 1\\
    \text{max}( u \times T_c(j),d_{k}(i,j) - d_{k-1}(i,j)), \\\qquad\qquad\text{if }e_{k+1}(i,j) = 1\text{ and }e_{k}(i,j) = 0\\
    \tau_k(\mathcal{G}_m) + d_{k-1}(i,j)), \\\qquad\qquad\text{if } e_{k+1}(i,j) = 0\text{ and }e_{k}(i,j) = 1\\
    \tau_k(\mathcal{G}_m), \\\qquad\qquad\text{otherwise}
\end{cases}
$$

where $e(i,j)$$=$$\mathbb{0}$ is weakly-connected edge, $e(i,j)$$=$$\mathbb{1}$ is strongly-connected edge; $ \tau_k(\mathcal{G}_m)$ is the cycle time at the $k$-th computation round during the training process. 

In general, using introduced equation, \textit{the delay of the next communication round $d_{k+1}$ is updated based on the delay of the previous rounds} and other factors, depending on the edge type connection.

## 2. Cycle time in multigraph
The cycle time per round is the time required to complete a communication round. In this work, we define the cycle time per round is the maximum delay between all silo pairs with strongly-connected edges. Therefore, the average cycle time of the entire training is:

$$
\tau(\mathcal{G}_m) =\frac{1}{k }\sum^{k-1}_{k=0} \left(\underset{j \in \mathcal{N}^{++}_{i} \cup\{i\}, \forall i \in \mathcal{V}}{\text{max}} \left(d_k\left(j,i\right)\right)\right),
$$

where $\mathcal{N}_{i}^{++}$ is an in-neighbors silo set of $i$ whose edges are strongly-connected.

## 3. Multigraph Construction

Algorithm 1 describes our methods to generate the multigraph $\mathcal{G}_m$ with multiple edges between silos. The algorithm takes the overlay $\mathcal{G}_o$ as input. Similar to~\cite{marfoq2020throughput}, we use the Christofides algorithm to obtain the overlay. In Algorithm 1, we establish multiple edges that indicate different statuses (strongly-connected or weakly-connected). To identify the total edges between a silo pair, we divide the delay $d(i,j)$ by the smallest delay $d_{\min}$ overall silo pairs, and compare it with the maximum number of edges parameter $t$ ($t=5$ in our experiments). \textit{We assume that the silo pairs with longer delay will have more weakly-connected edges, hence potentially becoming isolated nodes}. Overall, we aim to increase the number of weakly-connected edges, which generate more isolated nodes to speed up the training process. Note that, from Algorithm 1, each silo pair in the multigraph should have one strongly-connected edge and multiple weakly-connected edges. The role of the strongly-connected edge is to make sure that two silos have a good connection in at least one communication round. 

![](https://vision.aioz.io/f/09306652964b4e56b26e/?dl=1)

## Next
In the next post, we will mention multigraph parsing proccess and how to train a multigraph under decentralized federated learning.