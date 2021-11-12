---
last_modified_on: "2021-08-16"
title: AI-driven routing (part 2) Decentralize Routing.
description: A series of Smart Routing based on AI.
series_position: 7
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_routing"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Although AI-driven networking is now a hot research topic, the concept of using ML in decentralized traffic routing dates back to the 1990s.. Decentralized Routing can be splited into two different ground of methods: Single-Agent and Multiagent. However, both group of method are based on the general learning algorithm of Reinforcement Learning technique, which is similar to the previous introduced Centralized Routing.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## Single-Agent Reinforcement Learning
Boyan et al. introduced the Q-routing method in [8] to optimize packet routing control. Based on local information and communication, each router in the Q-routing algorithm modifies its policy according to its Q-function. The studies revealed that Q-routing outperformed the nonadaptive shortest route method, especially under heavy workload. Choi et al. introduced predictive Q-routing, a memory-based Q-learning method, in [9] to enhance the learning rate and convergence speed by preserving prior experiences. In addition, Kumar et al. introduced dual reinforcement Q-routing (DRQrouting) in [10], which leverages knowledge obtained from backward and forward exploration to speed up convergence. Reinforcement Learning (RL) was effectively implemented in wireless sensor network routing in [11, 12], with sensors and sink nodes self-adapting to the network environment. In a multiagent system, however, single-agent RL suffers from significant non-convergence. Instead, using multiagent RL to increase network node cooperation is more realistic, and there have been a number of research on MLdriven routing based on multiagent RL.

In this part, I also introduce the most up-to-date Q-Probabilistic routing (Q-PR) algorithm described in [12] as belows. For more details about notations, please visit [12].

The Q-PR-algorithm identifies the next hop on the actual route (a) in the form of network packets (not previous phases), b) when message is sent (opportunistic approach), c) using location-based route position  and d) taking smart decisions on nodes receiving transmission requests, both based on local information and the importance of these packets. The Q-PR algorithm is summarized as follows,

<Highlight name="Q-PR Algorithm" color="#0649c7">

FORWARDING PHASE ($N_i$ receives a message with a list of other candidates)<br/>
**if** Candidate nodes with higher priority forward the message<br/>
**then**<br/>
    &emsp;$N_i$ discards the message<br/>
**else**<br/>
    &emsp;1. $N_i$ selects the potential forwarding subset $\phi_{i}^{*}$ <br/>
    &emsp;2. $N_i$ evaluates $P(T = 1|x)$ and makes decision $d_i$ whether to transmit or not <br/>
    &emsp;**if** $d_i = 1$ <br/>
    &emsp;**then** <br/>
        &emsp;&emsp;Broadcast the message destined to $\phi_{i}^{*}$ <br/>
        &emsp;&emsp;Broadcast its expected $\hat{Q}_i(\phi_{i}^{*})$ <br/>
        &emsp;&emsp;Listen to retransmissions and update candidate neighbor profiles <br/>
        &emsp;&emsp;**if** no candidate nodes forward <br/>
        &emsp;&emsp;**then** <br/>
            &emsp;&emsp;&emsp;go to step 1 <br/>
        &emsp;&emsp;**end if** <br/>
    &emsp;**end if** <br/>
**end if**

</Highlight>

<CodeExplanation>

> **INFORMATION UPDATING PHASE**
* $N_i$ updates $\hat{Q}_i(j)$ via RL from any neighbor.
* $N_j$ that broadcasts its cost metric.
* $N_i$ updates $\hat{E}_i(j)$ when it hears a retransmission by $N_j$ or it knows $N_j$ receives a message.

</CodeExplanation>

![Fig-1](https://vision.aioz.io/f/ee20e0f1dbf047309d63/?dl=1)
*<center>**Figure 1**: Illustration of network topology and routing path between node N14 and the sink node (node N54). Discontinuous links show neighborhood relationships. Dead nodes have no discontinuous connection lines [12].</center>*
## Multiagent Reinforcement Learning
Stone et al. introduced the Team-Partitioned Opaque-Transition RL (TPOT-RL) routing algorithm in [13], [14], which allows a group of network nodes working toward a common objective to learn how to accomplish a collaborative job. Wolpert et al. created the Collective Intelligence (COIN) algorithm in [15], a sparse reinforcement learning method in which a global function is used to change the behavior of each network agent. In contrast, the author of [16] presented a routing algorithm based on Collaborative RL (CRL) that does not have a single global state. In [17], the CRL method was successfully utilized for delay-tolerant network routing.

Collaborative Reinforcement Learning [16] is one  of the most successive method belongs to the Multiagent group, which algorithm is then decribed as belows. In a multiagent system, the CRL technique may be used to address system optimization issues, which can be discretized into a collection of DOPs and described as absorbing MDPs in the following schema.

<Highlight name="CRL Algorithm" color="#0649c7">

1. A dynamic set of agents $N = \{n_1, n_2, ..., n_M\}$, often corresponding to nodes in a distributed system.
2. Each agent $n_i$ has a dynamic set $V_i$ of neighbors.
3. Each agent $n_i$ has a fixed set $S_i$ of states, where $S$ is the system-wide set of states.
4. Agents have both internal and external states.
$Int: N \rightarrow P(S)$is the function that maps from the set of agents to a nonempty set of internal states that are not visible to neighboring agents.
$Ext: N \rightarrow P(S)$  is the function that maps from the set of agents to a set of externally visible states.
5. Each agent $n_i$ has a dynamic set of actions
$A_i = A_{d_i} \cup A_{p_i} \cup  \{discovery\}$
where $A_i$ and $A_{d_i}$ are the set of delegation actions, $A_{p_i}$ are the set of DOP actions. The $discovery$ action updates the set of neighbors $V_i$ for agent $n_i$ and queries if discovered neighboring agent $n_j$ provides the capabilities to accept a delegated MDP from $n_i$. If it does, $A_{d_i}$ is updated to include a new delegation action that can result in a state transition to $s$ and the delegation of a MDP from $n_i$ to $n_j$.
6. There are a fixed set of state transition models, $\mathbf{P}_i$, that model the probabilities of making a state transition from state $s$ to state $s'$under action $a$.
7. $D_i$ is the connection cost function that observes the cost for the attempted use of a connection in a distributed system. $D_i$ is the connection cost model at agent $n_i$ that describes the estimated cost of making a transition from state $s$ to state $s'$ under delegation action $a$.
8. We define a cache at $n_i$ as $Cache_i  = \{(Q_i(s,a_j),r_j)\}$. The value $r_j$ in the pair $(Q_i(s,a_j),r_j)$ corresponds to the last advertised $V_j(s)$ received by agent $n_i$ from agent $n_j$. For each $n_j$, $Cache_j$ is updated by a advertisement $V_j$for a causally connected state. The update replaces the $r_j$ element of the pair $(Q_i(s,a_j),r_j)$ in $Cache_i$ with the newly advertised $V_j$ value.
9. $Decay(r_j)$ is the decay model that updates the element $r_j$ in $Cache_i$. The distributed model-based reinforcement learning algorithm is:

$$

Q_i(s,a) = R(s,a) + \sum_{s'}P_i(s'|s,a) \dot (D_i(s'|s,a) + Decay(V_j(s')))

$$

10. The value function at agent $n_i$, $V_j$, can be calculated using the Bellman optimality equation:

$$

V_i(s) = max_{a}[(Q_i(s,a)]

$$
</Highlight>

## Next section introduction
In the next part of this series, I will introduce a three-layer logical functionality architecture for AI-driven networking. This is the most up-to-date group of methods that is a combination between Centralized and Decentralized Routing to maximize the transmition.
## References
[8] J. A. Boyan and M. L. Littman, “Packet routing in dynamically changing networks: A reinforcement learning approach,” in Proc. Int. Conf. Neural Information Processing Systems, Denver, 1993, pp. 671–678.

[9] S. P. Choi and D.-Y. Yeung, “Predictive Q-routing: A memory-based reinforcement learning approach to adaptive traffic control,” in Advances in Neural Information Processing Systems, Denver, Dec. 2–5, 1996, pp. 945–951.

[10] S. Kumar and R. Miikkulainen, “Dual reinforcement Q-routing: An on-line adaptive routing algorithm,” in Proc. Artificial Neural Networks in Engineering Conf., 1997, pp. 231–238.

[11] A. A. Bhorkar, M. Naghshvar, T. Javidi, and B. D. Rao, “Adaptive opportunistic routing for wireless ad hoc networks,” IEEE Trans. Netw., vol. 20, no. 1, pp. 243–256, Feb. 2012.

[12] R. Arroyo-Valles, R. Alaiz-Rodriguez, A. Guerrero-Curieses, and J. Cid-Sueiro, “Q-probabilistic routing in wireless sensor networks,” in Proc. Int. Conf. Intelligent Sensors, Sensor Networks and Information, Melbourne, Dec. 3–6, 2007, pp. 1–6.

[13] P. Stone, “TPOT-RL applied to network routing,” in Proc. Int. Conf. Machine Learning, California, June 29–July 2, 2000, pp. 935–942.

[14] P. Stone and M. Veloso, “Team-partitioned, opaque-transition reinforcement learning,” in Robot Soccer World Cup, Springer, 1998, pp. 261–272.

[15] D. Wolpert, K. Tumer, and J. Frank, “Using collective intelligence to route internet traffic,” in Advances in Neural Information Processing Systems, Denver, Nov. 29–Dec. 4, 1999, pp. 952–960.

[16] J. Dowling, E. Curran, R. Cunningham, and V. Cahill, “Using feedback in collaborative reinforcement learning to adaptively optimize MANET routing,” IEEE Trans. Syst., Man, Cybern., vol. 35, no. 3, pp. 360–372, May 2005.

[17] A. Elwhishi, P. H. Ho, K. Naik, and B. Shihada, “ARBR: Adaptive reinforcement based routing for DTN,” in Proc. IEEE Int. Conf. Wireless and Mobile Computing, Networking and Communications, Ontario, Oct. 10–13, 2010, pp. 376–385.
