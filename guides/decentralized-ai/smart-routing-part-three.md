---
last_modified_on: "2021-08-30"
title: AI-driven routing (part 3) Hybrid Routing.
description: A series of Smart Routing based on AI.
series_position: 7
author_github: https://github.com/aioz-ai
tags: ["type: tutorial", "level: advance"]
---

In previous parts, we have mentioned about centralized and decentralized routing using the AI-powered reinforcement learning. In this section, we introduce a three-layer logical functionality design for AI-driven networking. Then we talk about how distant the intelligent control plane may be from the forwarding plane (if it should be centralized or dispersed). This introduced design can be understanded as a  combination between both afformentioned group of approaches, which leverage the advantages of each approaches to cover the other's weaknesses.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## Three-tier logical design: The closed-loop control paradigm

![Fig-1](https://vision.aioz.io/f/54137ae358744ce5af1d/?dl=1)
*<center>**Figure 1**:The closed-loop control paradigm ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

The advancing plane, the awareness plane, and the intelligent control plane are the three layers of the paradigm, as shown in Figure 1.

In distributed network, the forwarding plane is in charge of moving data packets from one interface to the next. Its operating logic is totally reliant on the control plane's forwarding table and configuration instructions.

The awareness plane's job is to keep track of the network's state and report back to the control plane. Network awareness and monitoring are required for ML-based control and optimization. As a result, we abstract this new layer, which we call the awareness plane, for the gathering and processing of monitoring data (for tasks like network device monitoring and network traffic identification) in order to deliver network status information.

The intelligent control plane is in charge of supplying forwarding plane control choices. In this aircraft, AI&ML-based algorithms are used to convert current and historical operating data into control rules.

The combination of these three abstract planes creates a closed-loop architecture for AI&ML deployment in networking. The advancing plane serves as the "subject of action," the awareness plane serves as the "subject of observation," and the intelligent control plane serves as the "subject of learning/judgment," analogous to the human learning process. By interacting with the underlying network, an AI&ML agent may constantly learn and optimize network control and management methods based on these three planes for closed-loop control.
## Hybrid architecture
We introduce a hybrid AI-driven control architecture as illustrated in Figure 2, that combines a "network mind" (centralized intelligence) with "AI routers" (distributed intelligence) to support various network services.
![Fig-2](https://vision.aioz.io/f/fd55cfadf5cf464d8114/?dl=1)
*<center>**Figure 2**:The hybrid architecture ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*
### Network Mind
The network mind, as depicted in Figure 3, is in charge of centralized intelligent traffic control and optimization. The network mind uses an upload link to obtain the fine-grained network status and a download link to issue actions.

![Fig-3](https://vision.aioz.io/f/344eeceb90d24dcf9d6a/?dl=1)
*<center>**Figure 3**: The centralized intelligent control scheme ([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*

To capture device statuses, traffic characteristics, configuration data, and service-level information, the upload connection uses a network monitoring protocol like INT, Kafka, or IPFIX; the download link uses a standard southbound interface like OpenFlow or P4 to allow efficient network control. The upload and download links provide an interaction structure that gives the network mind a global view and control capabilities, and the current and historical data from closed-loop operations is fed into AI&ML algorithms for knowledge generation and learning.

### AI Routers
The hybrid AI-based hop-by-hop routing paradigm is introduced in Figure 4. We move the duty for intelligent control to the AI routers and use the network mind to promote global convergence in the design to reduce the overhead imposed by centralized control.
![Fig-4](https://vision.aioz.io/f/6ba56b8d15c34ba7acf0/?dl=1)
*<center>**Figure 4**: The decentralized intelligent control scheme([Source](https://ieeexplore.ieee.org/abstract/document/8870277/)).</center>*


With the duty for intelligent control moved to each router, each router functions as an autonomous intelligent agent, and the dispersed AI agents form a Multi-Agent System (MAS). Each AI agent tries to maximize the predicted cumulative reward by optimizing its local policy by interacting with its uncertain environment. In contrast to a single-agent system, where the environment's state transitions are entirely determined by the activities of the single agent, the state transitions of a MAS are determined by the combined actions of all actors.

The ability to exchange experiences across AI routers is critical for improving the overall value of this MAS. However, in this geo-distributed system, how to perform such information exchange is a crucial challenge for high-efficiency operation. The centralized network mind is included in our design to act as a point of global knowledge convergence and experience exchange. The centralized network mind may use the network monitoring system to obtain global network information and use the download connection to exchange knowledge. When compared to a peer-to-peer design, this centralized architecture enhances information exchange efficiency.

## Next section introduction
In the next part of this series, I will introduce the effectiveness of smart routing methods under different benchmarking situations.
