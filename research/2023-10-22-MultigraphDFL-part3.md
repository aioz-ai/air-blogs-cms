
---
last_modified_on: "2023-10-22"
id: MultigraphDFL2
title: Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology (Part 3)
description: Delay time, Cycle Time, and how to construct multigraph
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
In previous part, we have investigated that how delay time and cycle time is affected by the modification of multigraph in the  design of the topology. Also, we will explore how multigraph can be constructed.  In this post, we will mention multigraph parsing proccess, how to train a multigraph under decentralized federated learning, and experimental  setups for Multigraph.

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our code can be found at: https://github.com/aioz-ai/MultigraphFL
## 1. Multigraph Parsing
In Algorithm~\ref{alg:state_form}, we parse multigraph $\mathcal{G}_m$ into multiple graph states  $\mathcal{G}_m^s$. Graph states are essential to identify the connection status of silos in a specific communication round to perform model aggregation. In each graph state, our goal is to identify the isolated nodes. During the training, isolated nodes update their weights internally and ignore all weakly-connected edges that connect to them.

To parse the multigraph into graph states, we first identify the maximum of states in a multigraph $s_{\max}$ by using the least common multiple (LCM). We then parse the multigraph into $s_{\max}$ states. The first state is always the overlay since we want to make sure all silos have a reliable topology at the beginning to ease the training. The reminding states are parsed so there is only one connection between two nodes. Using our algorithm, some states will contain isolated nodes. During the training process, only one graph state is used in a communication round.  Figure below illustrates the training process in each communication round using multiple graph states.

![](https://vision.aioz.io/f/f16aa3d9d3134c94974a/?dl=1)

## 2. Multigraph Training
In each communication round, a state graph $\mathcal{G}_m^s$ is selected in a sequence that identifies the topology design used for training. We then collect all strongly-connected edges in the graph state $\mathcal{G}_m^s$ in such a way that nodes with strongly-connected edges need to wait for neighbors, while the isolated ones can update their models. We train our multigraph with DPASGD algorithm:

$$
w_{i}\left(k + 1\right) =
\begin{cases}
    \sum_{j \in \mathcal{N}_{i}^{++} \cup{\{i\}}}A_{i,j}w_{j}\left(k - h\right),  \forall  k \equiv 0 \& \left|\mathcal{N}_{i}^{++}\right| > 1 ,\\
   w_{i}\left(k\right)-\alpha_{k}\frac{1}{b}\sum^b_{h=1}\nabla L_i\left(w_{i}\left(k\right),\xi_i^{\left(h\right)}\left(k\right)\right), otherwise.
\end{cases}
$$
where $(k- h)$ is the index of the considered weights; $h$ is initialized to $0$ and 
$h = h + 1 \forall e_{k-h}(i,j) = 0$. Through Equation above, at each state, if a silo is not an isolated node, it must wait for the model from its neighbor to update its weight. If a silo is an isolated node, it can use the model in its neighbor from the $(k-h)$ round to update its weight immediately. The training procedure is described as below:

![](https://vision.aioz.io/f/1adf87be6abb431cb2c1/?dl=1)

## 3. Algorithm Complexity
It is trivial to see that the complexity of training procedure is $\mathcal{O}(n^2)$. In practice, since the cross-silo federated learning setting has only a few hundred silos ($n<500$), the time to execute our algorithms is just a tiny fraction of training time. Therefore, our proposed topology still can significantly reduce the overall wall-clock training time.


## 4. Experimental Setups

**Datasets.** We use three datasets in our experiments: Sentiment140, iNaturalist, and FEMNIST. All datasets and the pre-processing process are conducted by following recent works. Table below shows the dataset setups in detail.
![](https://vision.aioz.io/f/e2113d1fdde244b39c0f/?dl=1)

**Network**. We consider five distributed networks in our experiments: Exodus, Ebone, Géant, Amazon and Gaia. The Exodus, Ebone, and Géant are from the Internet Topology Zoo. The Amazon and Gaia network are synthetic and are constructed using the geographical locations of the data centers. 

**Baselines.**
We compare our multigraph topology with recent state-of-the-art topology designs for federated learning: STAR, MATCHA, MATCHA(+), MST, and RING.

**Hardware Setup**. Since measuring the cycle time is crucial to compare the effectiveness of all topologies in practice, we test and report the cycle time of all baselines and our method on the same NVIDIA Tesla P100 16Gb GPUs. No overclocking is used. 

**Time Simulator**. We adapted PyTorch with the MPI backend to run DPASGD and DPASGD++ on a GPU cluster. We take advantage of the network simulator, the Time Simulator, which uses an arbitrary topology and computation times of silos as input to calculate the time instants at which local models are computed. The wall-clock time is reconstructed by this time simulator needs thorough understanding of the topology, including all factors mentioned in Delay Equations in each network configuration. The related configuration information is already provided in GAIA Network, and the simulator is created by Marfod \etal.

## Next
In the next post, we will mention the effectiveness and efficiency of multigraph topology design under different configurations.