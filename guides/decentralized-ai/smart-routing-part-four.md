---
last_modified_on: "2021-09-06"
title: AI-driven routing (part 4) Smart Routing Effectiveness.
description: A series of Smart Routing based on AI.
series_position: 9
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance"]
---

In previous parts, we have mentioned about hybrid smart routing using the AI-powered reinforcement learning. In this section, we show its effectiveness through empirical results.  Network Minds and AI Routers are considered to make analysis in this section.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## Network Mind Simulation
In this section, I present simulation data to verify the algorithm's feasibility and correctness. A simulated network with 12 nodes and 20 full-duplex links is used in this demonstration. Ten distinct levels of traffic load intensity are used to evaluate the algorithm's performance under varied network congestion scenarios. A Poisson distribution was used to generate the traffic. The researchers employed a neural network with two connected hidden layers, the first of which had 50 hidden units and the second of which had 40 hidden units. OMNet++ is used to simulate network traffic in our investigation. For DDPG agent construction, both Keras and TensorFlow are used.

Network state and processing delay were used to describe network state, each action was represented by a collection of nodes specifying forwarding paths between source and destination, and the reward was represented by a total forwarding delay from source to destination node.

![Fig-1](https://vision.aioz.io/f/38237dc825a44bd8aef5/?dl=1)
*<center>**Figure 1**: The learning process of the DDPG agent ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

The DDPG agent's learning process is depicted in Figure 1. The DDPG agent eventually converges to the best strategy as the number of training steps increases. In addition, the approach is also compared against the shortest path routing algorithm in the experiment.

![Fig-2](https://vision.aioz.io/f/8f0889ad7e5644329ae3/?dl=1)
*<center>**Figure 2**: The average delivery time versus different network loads ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

Congestion does not arise in the network when the traffic load is minimal, as illustrated in Figure 2. As a result, the shortest path routing method outperforms the AI-based approach. However, as demand intensity increases, network congestion on the shortest forwarding path becomes severe, and AI-based routing outperforms shortest path routing. As a result, it can be inferred that AI-based routing is efficient, particularly in the presence of network congestion.

## Analysis of AI Routers
In the experiment, we need to focuse on the congestion management problem for the distributed routing paradigm, which standard routing algorithms find challenging to solve. A network with four nodes and six unidirectional links is simulated, and 400 data packets are created to be routed across the network. To keep things simple, all of these packets originated at the same source node and were delivered to the same destination node. The AI routers routed each packet in a dispersed way.
![Fig-3](https://vision.aioz.io/f/37a94dddeaac4dbb9154/?dl=1)
*<center>**Figure 3**: The global utility of the whole system ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

The method is compared to a deterministic routing strategy and a single-agent RL algorithm in Figure 3. All data packets were routed along the same path for the deterministic routing technique. This approach is incapable of responding to network status in a timely manner, resulting in severe congestion and extremely poor global utility. RL, on the other hand, may dynamically adjust to the network's congestion situation. However, owing to the MAS's non-stationary environment, the learning process for single-agent RL suffers from significant non-convergence, resulting in a very low global score, as seen in Figure 3. In contrast, in the proposed design, a different incentive is provided to change the reward signal in order to improve the collective behavior of the AI routers, therefore enhancing the overall system utility.

## Next section introduction
In the next part of this series, I will introduce the Challenges and Open Issues, including New Hardware Architectures, Promoting Machine Learning Algorithms, and Advanced Software Systems.
