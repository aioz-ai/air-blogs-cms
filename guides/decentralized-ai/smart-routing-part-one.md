---
last_modified_on: "2021-08-16"
title: AI-driven routing (part 1) Centralized Routing.
description: A series of Smart Routing based on AI.
series_position: 5
author_github: https://github.com/Gazeal
tags: ["type: tutorial", "level: advance"]
---

With the introduction of Software-Defined Networking (SDN), Network Function Virtualization (NFV), and 5th-generation wireless technologies, networks throughout the world have recently undergone substantial reorganization and transformation (5G). The dominance of conventional ossified architectures is being eroded by emerging networking paradigms, which are lowering reliance on proprietary hardware. However, the associated increases in network flexibility and scalability pose new problems for network administration. Because network complexity is rising all the time, effective network control is becoming increasingly challenging. Current control techniques, in particular, rely heavily on manual procedures, which have inadequate scalability and resilience for the control of complex systems. As a result, there is an urgent need for more effective techniques of tackling networking issues.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## AI-Driven Traffic Routing
With the remarkable success of machine learning in recent years, applications of Artificial Intelligence and Machine Learning (AI&ML) in networking have gained a lot of attention. When compared to painstakingly developed (white-box) tactics, AI&ML (black-box) approaches offer significant benefits in networking systems. For example, AI&ML provides a generic model and consistent learning approach for diverse network settings without predefined procedures. Furthermore, such strategies can handle complicated issues and high-dimensional circumstances well; fact, AI&ML methods have already achieved great success in many complex system control areas, such as computer games and robotic control. Aside from the huge benefits of AI&ML for networking, the creation of novel network approaches is also fertile ground for AI&ML implementation. In-band Network Telemetry (INT), for example, provided end-to-end network visualization at the millisecond scale in 2015, and Cisco released PNDA, a big data analytics platform for networking, in 2017. As a result, the growing trend of using AI&ML in networking is being driven by both job requirements (increasing network complexity and increasingly demanding QoS/QoE needs) and technical advancements (new network monitoring technologies and big data analysis techniques).

Although AI-driven networking is now a hot research topic, the concept of using ML in traffic routing dates back to the 1990s. In this part, we will look at previous research on AI-driven network routing methods: Centralized Routing and Decentralized Rounting.

In this part, we start with the centralized  routing.
## Centralized Routing

AdaR is an RL-based routing algorithm introduced by Wang et al. for Wireless Sensor Networks (WSNs). Least-Squares Policy Iteration (LSPI) is used in AdaR to accomplish the best possible tradeoff between several optimization goals such as routing path length, load balancing, and retransmission rate. The overhead for centralized AI control, on the other hand, is significant.

![Fig-1](https://vision.aioz.io/f/ca4aeade3f2743b4b304/?dl=1)
*<center>**Figure 1**:  A wireless sensor network model. The dark-shaded nodes are data sources, whereas the square node represents the base station. The darkened nodes show sensors with little residual energy, whereas the dashed lines imply untrustworthy connections [(Source)](https://ieeexplore.ieee.org/document/4019984).</center>*

### RL based Routing
The method is designed by using Reinforcement learning (RL), a framework for learning which control rules based on experience and incentives. The Markov Decision Process is the basic idea of RL (MDP).

The goal of solving an MDP is to find an optimal policy, that maps states to actions such that the cumulative reward is maximized, which is equivalent to finding the following Q-fixed points (Bellma equation):


$$

Q^*(s,a) = r(s,a) + \gamma \sum_{s'}P(s'|s,a) max (Q^*(s',a'))

$$

where the first term on the right side represents the expected immediate reward of executing action $a$ at $s$, and the second term is the maximum expected future reward. The discount factor $\gamma$ models the fact that immediate reward is more valuable than future reward.

In the context of learning routing strategy, each sensor can be considered as a state $s$, and for each of its neighbor $s$ , there is a corresponding action $a$. Executing action $a$ at $s$ means forwarding the packet to the corresponding neighbor s . The fixed $Q$-values of $s$ can be regarded as its routing table, telling to which neighbor to forward the data. This scheme is illustrated in Figure. 2. With the help of this routing table, the optimal routing path can be trivially constructed by a sequence of table look-up operations. Thus the task of learning optimal routing strategy is equivalent to finding the $Q$-fixed points.

![Fig-2](https://vision.aioz.io/f/5e50b138735444e196cb/?dl=1)
*<center>**Figure 2**:  The routing table ($Q$-values) of a sensor $s$. The action $a^?$ with the maximum $Q$-value is selected [(Source)](https://ieeexplore.ieee.org/document/4019984).</center>*

### $Q$-Learning
The $Q$-fixed points can be solved deterministically if the underlying transition model $P$ and reward mechanism $R$ are known. However, in most situations, such as ad-hoc sensor networks, determining the underlying model is difficult, necessitating the use of stochastic reinforcement learning approaches.

Q-learning is a temporal difference (TD) control technique that operates outside of policy and directly approximates the best action-value function. When the agent performs an action a, he or she receives an immediate reward r from the environment. It then utilizes this reward, as well as the predicted long-term reward, to update the $Q$-values, which impacts future action selection. In its most basic form, one-step $Q$-learning is defined as:

$$

Q(s,a) = (1-\alpha)Q(s,a) + \alpha [r + \gamma max (Q(s',a'))]

$$

where $\alpha$ is the learning rate, which models the rate of updating $Q$-values.

Q-learning, being a model-free RL approach, necessitates no knowledge of the underlying reward or transition mechanism, making it suitable to the challenge of learning routing strategies in sensor networks.

## Next section introduction
In the next part of this series, I will introduce another useful  learning approaches for Smart Routing based on Distributed structure: Decentralized Routing.
