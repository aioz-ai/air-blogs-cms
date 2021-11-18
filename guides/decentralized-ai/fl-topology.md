---
last_modified_on: "2021-07-20"      
title: Topologies in Decentralized Federated Learning.
description: A detail description about  topologies in Decentralized Federated Learning.
series_position: 4
author_github: https://github.com/aioz-ai  
tags: ["type: tutorial", "level: advance", "guides: decai_fl"]
---

Data silos are nearly always available in the Cross-Silo configuration, have high-speed connectivity similar to the orchestrator, and exchange information with other silos more quickly than with the orchestrator. An inter-silo communication architecture focused on orchestrators would be ineffective since it overlooks rapid communication possibilities and makes the orchestrator a congestion candidate. A recent tendency is to replace communication between separate silos and an orchestrator with peer-to-peer communications. This setup conducts part of the local model update aggregations. In the section, we examine the mentioned scenario and how to develop the topology of communications.

Currently, their are three main types of topologies are leveraged in Decentralized Federated learning, including: Underlay, Connectivity Graph and Overlay.

## Underlay

![Fig-1](https://vision.aioz.io/f/c37e2fbfe9d04eaba63e/?dl=1)
*<center>**Figure 1**:  Underlay $\mathcal{G}_u = (\mathcal{V} \cup \mathcal{V}', \mathcal{E}_u)$.</center>*

FL silos are connected by a so-called underlay, i.e., a communication infrastructure such as the Internet or some private network. The underlay can be represented as a directed graph (digraph). $\mathcal{G}_u = (\mathcal{V} \cup \mathcal{V}', \mathcal{E}_u)$, where $\mathcal{V}$ denotes the set of silos, $\mathcal{V}'$ is the set of other nodes (e.g., routers) in the network, and $\mathcal{E}_u$ the set of communication links. For simplicity, we consider that each silo $i \in \mathcal{V}$ is connected to the rest of the network through a single link $(i,i')$, where $i' \in \mathcal{V}'$, with uplink capacity $C_{UP}(i)$ and downlink capacity $C_{DN}(i)$ (See the example in Figure 1 which illustrates the underlay).

## Connectivity Graph
![Fig-2](https://vision.aioz.io/f/a0b7f671471442fea0e0/?dl=1)
*<center>**Figure 2**:  Connectivity Graph $\mathcal{G}_c = (\mathcal{V}, \mathcal{E}_c)$.</center>*

Connectivity Graph denotes by $\mathcal{G}_c = (\mathcal{V}, \mathcal{E}_c)$ captures the possible direct communications among silos. Often
the connectivity graph is fully connected, but specific NAT or firewall configurations may prevent some pairs of silos to communicate. If transmission is allowed, the messageexperiences a delay that is the sum of two contributions: 1) an end-to-end delay accounting for link latencies, and queueing delays long the path, and 2) a term depending on the model size and the available bandwidth. We assume that in the stable cross-silo setting these quantities do not vary or vary slowly, so that the topology is recomputed only occasionally, if at all.

## Overlay
![Fig-3](https://vision.aioz.io/f/b2da510e8fe146f190dd/?dl=1)
*<center>**Figure 3**:  Overlay $\mathcal{G}_o = (\mathcal{V}, \mathcal{E}_o)$.</center>*

Thank to the development of decentralized training algorithm, we do not need to use all potential connections. Hence, the orchestrator can select a connected subgraph of $\mathcal{G}_c$, the so-called Overlay $\mathcal{G}_o = (\mathcal{V}, \mathcal{E}_o)$. Only nodes directly connected in $\mathcal{E}_o$ will exchange messages.
