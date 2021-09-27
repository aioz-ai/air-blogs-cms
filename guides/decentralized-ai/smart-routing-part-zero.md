---
last_modified_on: "2021-09-27"
title: Smart Routing.
description: Smart Routing based on AI-driven.
series_position: 5
author_github: https://github.com/aioz-ai
tags: ["type: tutorial", "level: advance"]
---

import Highlight from '@site/src/components/Highlight';

With the introduction of Software-Defined Networking (SDN), Network Function Virtualization (NFV), and 5th-generation wireless technologies, networks throughout the world have recently undergone substantial reorganization and transformation (5G). The dominance of conventional ossified architectures is being eroded by emerging networking paradigms, which are lowering reliance on proprietary hardware. However, the associated increases in network flexibility and scalability pose new problems for network administration. Because network complexity is rising all the time, effective network control is becoming increasingly challenging. Current control techniques, in particular, rely heavily on manual procedures, which have inadequate scalability and resilience for the control of complex systems. As a result, there is an urgent need for more effective techniques of tackling networking issues.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## AI-Driven Traffic Routing
With the remarkable success of machine learning, applications of Artificial Intelligence and Machine Learning (AI&ML) in networking have gained a lot of attention. When compared to painstakingly developed tactics, AI&ML approaches offer significant benefits in networking systems. AI&ML provides a generic model and consistent learning approach for diverse network settings without predefined procedures. Furthermore, such strategies can handle complicated issues and high-dimensional circumstances well. Aside from the huge benefits of AI&ML for networking, the creation of a novel network is also fertile ground for AI&ML implementation. The growing trend of using AI&ML in networking is being driven by both job requirements (increasing network complexity and increasingly demanding QoS/QoE needs) and technical advancements (network monitoring technologies and big data analysis techniques).

Centralized and decentralized routing using AI-powered reinforcement learning is recently applied in Smart Routing. However, both of them still remain problems during learning. So as to improve our network routing with the fanciest technique, we take advantage of a three-layer logical functionality design. This design can be understood as a combination between both aforementioned groups of approaches, which leverage the advantages of each approaches to cover the other's weaknesses.

## Three-tier logical design: The closed-loop control paradigm

![Fig-1](https://vision.aioz.io/f/54137ae358744ce5af1d/?dl=1)
*<center>**Figure 1**: The closed-loop control paradigm ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

The advancing plane, the awareness plane, and the intelligent control plane are the three layers of the paradigm, as shown in Figure 1.

In a distributed network, the forwarding plane is in charge of moving data packets from one interface to the next. Its operating logic is totally reliant on the control plane's forwarding table and configuration instructions.

The awareness plane's job is to keep track of the network's state and report back to the control plane. Network awareness and monitoring are required for ML-based control and optimization. As a result, we abstract this new layer, which we call the awareness plane, for the gathering and processing of monitoring data (for tasks like network device monitoring and network traffic identification) in order to deliver network status information.

The intelligent control plane is in charge of supplying forwarding plane control choices. In this aircraft, AI&ML-based algorithms are used to convert current and historical operating data into control rules.

The combination of these three abstract planes creates a closed-loop architecture for AI&ML deployment in networking. The advancing plane serves as the "subject of action," the awareness plane serves as the "subject of observation," and the intelligent control plane serves as the "subject of learning/judgment," analogous to the human learning process. By interacting with the underlying network, an AI&ML agent may constantly learn and optimize network control and management methods based on these three planes for closed-loop control.
## Hybrid architecture
A hybrid AI-driven control architecture illustrated in Figure 2 is specially designed to fit our system. It combines a "network mind" (centralized intelligence) with "AI routers" (distributed intelligence) to support various network services.
![Fig-2](https://vision.aioz.io/f/fd55cfadf5cf464d8114/?dl=1)

*<center>**Figure 2**: The hybrid architecture ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

### Network Mind
#### Architecture
The network mind, as depicted in Figure 3, is in charge of centralized intelligent traffic control and optimization. The network mind uses an upload link to obtain the fine-grained network status and a download link to issue actions.

![Fig-3](https://vision.aioz.io/f/344eeceb90d24dcf9d6a/?dl=1)

*<center>**Figure 3**: The centralized intelligent control scheme ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

To capture device statuses, traffic characteristics, configuration data, and service-level information, the upload connection uses a network monitoring protocol like INT, Kafka, or IPFIX; the download link uses a standard southbound interface like OpenFlow or P4 to allow efficient network control. The upload and download links provide an interaction structure that gives the network mind a global view and control capabilities, and the current and historical data from closed-loop operations is fed into AI&ML algorithms for knowledge generation and learning.
#### Centralized Learning and Routing paradigm
** RL-based Routing.** Our learning method for Network Mind is designed by using Reinforcement learning (RL), a framework for learning which controls rules based on experience and incentives. The Markov Decision Process is the basic idea of the in-used RL (MDP).

The goal of solving an MDP is to find an optimal policy, that maps states to actions such that the cumulative reward is maximized, which is equivalent to finding the following Q-fixed points (Bellman equation):

$$

Q^*(s,a) = r(s,a) + \gamma \sum_{s'}P(s'|s,a) max (Q^*(s',a'))

$$

where the first term on the right side represents the expected immediate reward of executing action $a$ at $s$, and the second term is the maximum expected future reward. The discount factor $\gamma$ models the fact that immediate reward is more valuable than future reward.

In the context of the learning routing strategy, each node can be considered as a state $s$, and for each of its neighbor $s$ , there is a corresponding action $a$. Executing action $a$ at $s$ means forwarding the packet to the corresponding neighbor $s$. The fixed $Q$-values of $s$ can be regarded as its routing table, telling to which neighbor to forward the data. This scheme is illustrated in Figure 4. With the help of this routing table, the optimal routing path can be trivially constructed by a sequence of table look-up operations. Thus the task of learning optimal routing strategy is equivalent to finding the $Q$-fixed points.

![Fig-4](https://vision.aioz.io/f/5e50b138735444e196cb/?dl=1)

*<center>**Figure 4**: The routing table ($Q$-values) of a state $s$. The action $a^{\ast}$ with the maximum $Q$-value is selected [(Source)](https://ieeexplore.ieee.org/document/4019984).</center>*

**$Q$-Learning.** The $Q$-fixed points can be solved deterministically if the underlying transition model $P$ and reward mechanism $R$ are known. Q-learning is a temporal difference (TD) control technique that operates outside of policy and directly approximates the best action-value function. When the agent performs an action $a$, he or she receives an immediate reward $r$ from the environment. It then utilizes this reward, as well as the predicted long-term reward, to update the $Q$-values, which impacts future action selection. In its most basic form, one-step $Q$-learning is defined as:

$$

Q(s,a) = (1-\alpha)Q(s,a) + \alpha [r + \gamma max (Q(s',a'))]

$$

where $\alpha$ is the learning rate, which models the rate of updating $Q$-values.

### AI Routers
#### Architecture
The hybrid AI-based hop-by-hop routing paradigm is introduced in Figure 5. We move the duty for intelligent control to the AI routers and use the network mind to promote global convergence in the design to reduce the overhead imposed by centralized control.
![Fig-5](https://vision.aioz.io/f/6ba56b8d15c34ba7acf0/?dl=1)
*<center>**Figure 5**: The decentralized intelligent control scheme([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*


With the duty for intelligent control moved to each router, each router functions as an autonomous intelligent agent, and the dispersed AI agents form a Multi-Agent System (MAS). Each AI agent tries to maximize the predicted cumulative reward by optimizing its local policy by interacting with its uncertain environment. In contrast to a single-agent system, where the environment's state transitions are entirely determined by the activities of the single agent, the state transitions of a MAS are determined by the combined actions of all actors.

The ability to exchange experiences across AI routers is critical for improving the overall value of this MAS. However, in this geo-distributed system, how to perform such information exchange is a crucial challenge for high-efficiency operations. The centralized network mind is included in our design to act as a point of global knowledge convergence and experience exchange. The centralized network mind may use the network monitoring system to obtain global network information and use the download connection to exchange knowledge.

#### Collaborative Reinforcement Learning
Collaborative Reinforcement Learning (CRL) is one of the most successful methods belongs to the Multiagent group, which algorithm is then described as below. In a MAS, the CRL technique is used to address system optimization issues, which can be discretized into a collection of DOPs and described as absorbing MDPs in the following schema.

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
