---
last_modified_on: "2023-10-30"
id: MultigraphDFL4
title: Reducing Training Time in Cross-Silo Federated Learning using Multigraph Topology (Part 4)
description: Delay time, Cycle Time, and how to construct multigraph
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
In previous post, we have mentioned multigraph parsing proccess, how to train a multigraph under decentralized federated learning, and experimental  setups for Multigraph. In the post, we will mention the effectiveness and efficiency of multigraph topology design under different configurations.

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our code can be found at: https://github.com/aioz-ai/MultigraphFL
## 1. Cycle Time Comparison

Table 1 shows the cycle time of our method in comparison with other recent approaches. This table illustrates that our proposed method significantly reduces the cycle time in all setups with different networks and datasets. In particular, compared to the state-of-the-art RING, our method reduces the cycle time by $2.18$, $1.5$, $1.74$ times in average in the FEMNIST, iNaturalist, Sentiment140 dataset, respectively. Our method also clearly outperforms MACHA, MACHA(+), and MST by a large margin. The results confirm that our multigraph with isolated nodes helps reduce the cycle time in federated learning.  


From Table1, our multigraph achieves the minimum improvement under the Amazon network in all three datasets. This can be explained that, under the Amazon network, our proposed topology does not generate many isolated nodes. Hence, the improvement is limited. Intuitively, when there are no isolated nodes, our multigraph will become the overlay, and the cycle time of our multigraph will be equal to the cycle time of the overlay in RING.

![Tab-1](https://vision.aioz.io/f/8610cb691da0474889bb/?dl=1)


## 2. Isolated Node Analysis

**Isolated Nodes vs. Network Configuration**. The numbers of isolated nodes vary based on the network configuration (Amazon, Gaia, Exodus, etc.). The parameter $t$ (maximum number of edges between two nodes), and the delay time which is identified by many factors (geometry distance, model size, computational cost based on tasks, bandwidth, etc also affect the process of generating isolated nodes. Table 2 illustrates the effectiveness of isolated nodes in different network configurations. Specifically, we conduct experiments on the FEMNIST dataset using five network configurations (Gaia, Amazon, Geant, Exodus, Ebone). We can see that our cycle time compared with RING is reduced significantly when more communication rounds or graph states have isolated nodes.
![Tab-2](https://vision.aioz.io/f/1718bc572119488eab01/?dl=1)
*<center>**Table 2**: The effectiveness of isolated nodes under different network configurations. All experiments are trained with  6400  communication rounds on FEMNIST dataset.We then record the number of states and rounds that have the appearance of isolated nodes and compare our cycle time with RING.</center>*

**Isolated Nodes vs. RING vs. Random Strategy.** Isolated nodes play an important role in our method as we can skip the model aggregation step in the isolated nodes. In practice, we can have a trivial solution to create isolated nodes by randomly removing some nodes from the overlay of RING. Table 3 shows the experiment results in two scenarios on FEMNIST dataset and Exodus Network: i} Randomly remove some silos in the overlay of RING, and  ii} Remove the most inefficient silos (i.e., silos with the longest delay) in the overlay of RING.  From Table 3, the cycle time reduces significantly when two aforementioned scenarios are applied. However, the accuracy of the model also drops significantly. This experiment shows that although randomly removing some nodes from the overlay of RING is a trivial solution, it can not maintain model accuracy. On the other hand, our multigraph not only reduces the cycle time of the model but also preserves the accuracy. This is because our multigraph can skip the aggregation step of the isolated nodes in a communication round. However, in the next round, the delay time of these isolated nodes will be updated, and they can become normal nodes and contribute to the final model.

![Tab-3](https://vision.aioz.io/f/4c513e90939b4d3c8701/?dl=1)
*<center>**Table 3**: The cycle time and accuracy of our multigraph vs. RING  with different criteria.</center>*


**Isolated Nodes Illustration.** Figure belows shows a detailed illustration of our algorithm with the isolated nodes in a real-world training scenario. The experiment is conducted on Gaia network geometry and their corresponding hardware for supporting link latency computation. The image classification task is chosen for this benchmarking by using FEMNIST dataset and CNN backbone provided by Marfod \etal. Hence, we keep the model transmitted size at $4.62$ Mb, all access links have $10$  Gbps traffic capacity, the number of local updates is set to $1$, and the maximum number of edges $t$ is set to $3$. As shown in this Figure, although there are no isolated nodes in the initialized state, the number of isolated nodes increases in the next consequence states with a vast number (4 nodes). This circumstance leads to a $\sim 4$ times reduction in cycle time compared to the initialized state. The appearance of isolated nodes also greatly reduces the connection between silos by $\sim 3.6$ times, from $11$ down to $3$ connections, and also discarded ones all have high latency. 

![Fig-1](https://vision.aioz.io/f/7f066d28ddc14824b27f/?dl=1)

## 3. Multigraph Ablation Study
**Accuracy Analysis.** In federated learning, improving the model accuracy is not the main focus of topology designing methods. However, preserving the accuracy is also important to ensure model convergence. Table 4 shows the accuracy of different topologies after $6,400$ communication training rounds on the FEMNIST dataset. This table illustrates that our proposed method achieves competitive accuracy with other topology designs. This confirms that our topology can maintain the accuracy of the model, while significantly reducing the training time.

![Tab-4](https://vision.aioz.io/f/e8652c78d99445d2bafc/?dl=1)
*<center>**Table 4**: Accuracy comparison between different topologies. The experiment is conducted using the FEMNIST dataset. The accuracy is reported after $6,400$ communication rounds in all methods.</center>*

**Convergence Analysis**. Figure belows shows the training loss versus the number of communication rounds and the wall-clock time under Exodus network using the FEMNIST dataset. This figure illustrates that our proposed topology converges faster than other methods while maintaining the model accuracy. We observe the same results in other datasets and network setups.


![](https://vision.aioz.io/f/657ace98ac7945669f22/?dl=1)

**Cycle Time and Accuracy Trade-off.** In our method, the maximum number of edges between two nodes $t$ mainly affects the number of isolated nodes. This leads to a trade-off between the model accuracy and cycle time. Table 5 illustrates the effectiveness of this parameter. When $t = 1$, we technically consider there are no weak connections and isolated nodes. Therefore, our method uses the original overlay from RING. When $t$ is set higher, we can increase the number of isolated nodes, hence decreasing the cycle time. In practice, too many isolated nodes will limit the model weights to be exchanged between silos. Therefore, models at isolated nodes are biased to their local data and consequently affect the final accuracy.

![Tab-5](https://vision.aioz.io/f/5a7adf62b7cc4c0a9b48/?dl=1)
*<center>**Table 5**: Cycle time and accuracy trade-off with different value of $t$, i.e., the maximum number of edges between two nodes.</center>*

## 4. Conclusion
We proposed a new multigraph topology for cross-silo federated learning. Our method first constructs the multigraph using the overlay. Different graph states are then parsed from the multigraph and used in each communication round. Our method significantly reduces the cycle time by allowing the isolated nodes in the multigraph to do model aggregation without waiting for other nodes. The intensive experiments on three datasets show that our proposed topology achieves new state-of-the-art results in all network and dataset setups.
