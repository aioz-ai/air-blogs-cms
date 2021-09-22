---
last_modified_on: "2021-09-20"
title: Smart Caching.
description: Edge based Intelligence for Edge Caching.
series_position: 11
author_github: https://github.com/Gazeal
tags: ["type: tutorial", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


Edge networking is a sophisticated and dynamic computing architecture aimed at bringing cloud services closer to the end user, boosting responsiveness and decreasing backhaul traffic. Edge networks' primary dynamic aspects include user mobility, preferences, and content popularity. Temporal and social characteristics of material, such as the number of views and likes, are used to assess the worldwide popularity of content. Such estimations, however, are not always successful to be mapped to an edge network with specific social and geographic features.  Machine learning techniques is employed in next-generation edge networks to forecast content popularity based on user preferences, cluster users based on comparable content interests, and improve cache placement and replacement tactics. Our work is to examine the use of machine learning techniques for edge network caching within the network itself.

![Fig-1](https://images.viblo.asia/d838c5ca-42a6-4904-9cab-9ba1e3ec0690.png)
*<center>**Figure 0**: A general form of network caching. ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fviblo.asia%2Fp%2Fcaching-la-gi-va-no-hoat-dong-nhu-the-nao-m68Z0QpXlkG&psig=AOvVaw1ohAHx1pymmpXYl-KZP8mQ&ust=1632284725828000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCNiKs_Gcj_MCFQAAAAAdAAAAABAU)).</center>*

## Introduction
Cloud data centers and content delivery networks (CDN) provide back-end data storage that is required by mobile apps, which have low latency, mobility, energy efficiency, and high bandwidth. The latency between mobile users and geographically dispersed cloud data centers/CDNs is greatly influenced by distance.  To deal with this problem, many networking technologies have been proposed and implemented to bring processing and caching capabilities closer to end-users. Mobile Edge Computing (MEC), fog computing, cloudlets, and information-centric networks are examples of these emerging technologies. Using in-network caching, edge networks have reduced the burden on backhaul networks while solving the problem of excessive content delay. Key research questions in the area of caching in mobile networks include: the popularity of multimedia, base stations and mobile users, and proactive and reactive to cache. Although the questions are clear, the solution for caching still needs to be optimized more.

To answer the question above, machine learning in wireless networks is used to estimate future user requests based on time-series dynamic mobility, popularity, and preference information. Given the limitations of user data available in edge networks, ML models that learn without prior information are frequently used to optimize caching options.

## Edge based Intelligence for Edge Caching
Edge caching refers to a user-centric cache on the edge that has limited storage capacity and stores material that is only relevant to edge users. Because of the edge's limited storage capacity and the dynamic features of the user (mobility, content access, etc.), the caching choice differs from CDN caching. In edge networks, **Optimal Caching decision** are based on a variety of inputs which corresponding tasks that need to be optimized by learning, including:
- Content popularity and content popularity prediction task.
- User mobility and user mobility prediction task.
- Wireless channel and joint content identification task.

Although mentioned tasks can also output caching decisions, we further leverage Deep Heuristic to anticipate the mentioned input characteristics to optimum caching issues through Edge-based Intelligence for Edge Caching.

![Fig-2](https://vision.aioz.io/f/b9ba67d871f141569594/?dl=1)
*<center>**Figure 2**:  Edge-based Intelligence for Edge Caching. ([Source](https://arxiv.org/pdf/2006.16864.pdf)).</center>*

A high-level conceptual framework of ML-based edge caching is depicted in Figure 2. Social awareness, mobility patterns, and user preferences are fed into an ML framework, which may be federated across multiple nodes and stored at dispersed MEC servers, C-RAN, or CDN. The social networks, D2D communications, and V2I communications are used to provide the inputs for the ML-based caching decision framework. Intelligent and optimum decisions are sent back to caches handled by network virtualization methods. Caching placement strategies are being innovated with the aid of UAV-mounted caches that could also be directed towards user populations and real-time events. For more details, let examine four optimum tasks in the Smart Edge Caching System that play an important role in giving out caching decisions.

### 1. Content Popularity Prediction for Caching.
In our system, we use the cloud-based architecture (master node) which leverages supervised and deep learning in two phases to estimate video popularity. Then, the system automatically pushes the learned cache decisions to the BSs (slave nodes). To begin, a data collector module gathers information on the number of video IDs that have been requested to make predictions about the future popularity and class of videos based on supervised learning. Deep learning algorithms are used to estimate future video content request numbers. See Figure 3 and Figure 4 for more details about the system model and the architecture of the master node.

![Fig-3](https://vision.aioz.io/f/89bd0de473af433ca5d7/?dl=1)
*<center>**Figure 3**: System model of learning-based caching at the edge. ([Source](https://ieeexplore.ieee.org/document/8576500)).</center>*

![Fig-4](https://vision.aioz.io/f/513fca1d7ba84c08acc2/?dl=1)
*<center>**Figure 4**: Master Node Design. ([Source](https://ieeexplore.ieee.org/document/8576500)).</center>*

The Algorithm below conducts the training processes for the estimation model at the cloud data center in order to improve the prediction accuracy. This method takes as inputs the best model $m^*$ and a subset of raw data $d[t=1:t=j]$. This method produces an optimal trained model for predicting popularity scores. First, the accuracy measurement metrics $m_{tacc}$, $m_{vacc}$, and $m_{pacc}$ are set to zero. The learning rate lm and the regularization rate $m$ are then set to fixed values of $0.001$.

<Highlight name="Alg.1 Training process">

**INPUT** model $m^*$ , train dataset $d[t=1:t=j]$<br/>
**OUTPUT** Trained model $m^*$<br/>
**INITIALIZED** $m_{tacc}, m_{vacc},m_{pacc} = 0 , l_m,\gamma_m = 0.001$<br/>
**for** $i$ **in** range($n$) **do**<br/>
    &emsp;$l_{m,i},\gamma_{m,i} \leftarrow rand(0,1)$<$br/>
    &emsp;Train the model $m$ with small dataset by minimizing loss<br/>
    &emsp;**if** $m^{i-1}_{vacc}<m^i_{vacc}$ **then**<br/>
        &emsp;&emsp;$l^*_m,\gamma^*_m \leftarrow l_{m,i},r_{mi}$<$br/>
    &emsp;**end if**<br/>
    &emsp;Reset model $m$<br/>
Train model $m$ with $l^*_m,\gamma^*_m$<br/>
**while** $m_{tacc}< \epsilon$ or dataset is not fully utilized **do**<br/>
    &emsp;Minimize the training loss of model $m$ with the dedicated optimizer<br/>
    &emsp;**if** Current validation accuracy of model $m_{tacc}$ is better than the previous **then**<br/>
        &emsp;&emsp;Update and store the parameters of m<br/>
    &emsp;**end if**<br/>
    
</Highlight>

Alg. 2 depicts the caching procedure at the BS in order to optimize cache hit. This algorithm's inputs include user requests, BS log files, and learned models from the cloud data center, which are utilized to forecast popularity scores. This algorithm's result is a decision on whether to save the predicted popular content.

<Highlight name="Alg.2 Cache Decision Process">

**INPUT** Contents request history<br/>
**OUTPUT** Contents list to store<br/>

**MASTER NODE PROCESS** Predict the future $(t+1)$ popularity class and request counts of video contents by using trained model $m$ for $BS$<br/>
**if** $m_{pacc}<\epsilon$ **then**<br/>
    &emsp;Repeat while loop of **Alg. 1** with daily data $d^{t+1}_{bs}$ to update parameters of model $m$ such as weight $w$<br/>
**end if**<br/>
Send the predicted information to **Slave Node**<br/>

**SLAVE NODE PROCESS** Received the predicted information<br/>
**if** Total size of Class 1 contents $< s$ **then**<br/>
    &emsp;Store all of Class 1 contents<br/>
    &emsp;**if** Cache space $s$ still has free space **then**<br/>
        &emsp;&emsp;1. Sort Class 2 contents in descending ordered based on predicted request count<br/>
        &emsp;&emsp;2. Store the contents from the beginning until the cache space is full<br/>
        &emsp;&emsp;**if** Cache space $s$ still has free space **then**<br/>
            &emsp;&emsp;&emsp;1. Sort Class 3 contents in descending ordered based on predicted request count<br/>
            &emsp;&emsp;&emsp;2. Store the contents from the beginning until the cache space is full<br/>
        &emsp;&emsp;**end if**<br/>
    &emsp;**end if**<br/>
**else**<br/>
    &emsp;1. Sort Class 1 contents in descending ordered based on predicted request count<br/>
    &emsp;2. Store the contents from the beginning until the cache space is full<br/>
**end if**<br/>

</Highlight>

### 2. Mobility Prediction for Caching.
Caching for mobile users is important since wearable high-tech devices are more and more popular. Thus, mobility prediction is essential for optimizing the cache hit of mobile devices. Here, we utilize Temporal Dependency, Spatial Dependency, and Geographic Restriction to forecast user locations in mobile networks. 

The mobility models depict the movement of mobile nodes as well as the changes in position, velocity, and acceleration over time. Based on the Temporal Dependency mobility model, prediction systems presume that mobile node trajectories may be restricted by physical properties such as acceleration, velocity, direction, and movement history. The estimation is based on the premise that mobile nodes tend to travel in a correlated way and that the mobility pattern of one node is influenced by the mobility pattern of other adjacent nodes.

For Geographic Restriction, node trajectories are affected by the environment, and mobile node mobility is restricted by geographic constraints. Similarly, buildings and other barriers may obstruct pedestrians. To address this issue, we employ a completely cloudified mobility prediction service that enables on-demand mobility prediction life-cycle management. The mobility prediction method is based on the Dynamic Bayesian Network (DBN) model, and the rationale for adopting DBN is that the next place visited by a user is determined by its present position, (ii) the movement duration, and (iii) the day that user is in motion. Figure 5 illustrates how to relocate content for mobile users. The output relocation contents are considered to access cache hit.

![Fig-5](https://vision.aioz.io/f/155fb0f9dc134898ab32/?dl=1)
*<center>**Figure 5**: Content relocation architecture for cache hit. ([Source](https://ieeexplore.ieee.org/document/7335295)).</center>*

### 3.Joint Content Identification for Caching.
Optimization for the joint content caching and mode selection for slice instances is essential in fog radio access networks (F-RANs). A fog-computing layer is established at the network's edge in F-RANs, and a portion of the service needs may be met locally without engaging with the cloud computing center via the fronthaul connections. Hotspot and vehicle-to-infrastructure situations, in particular, are addressed, and related network slice instances are coordinated in F-RANs. When various users' expectations and restricted resources are taken into account, there is a substantial high complexity in addressing the initial optimization issue using standard optimization techniques. We adopt a deep reinforcement learning-based method because of the benefits of deep reinforcement learning in tackling complicated network optimizations. In this case, the cloud server uses intelligent operations to enhance the hit ratio and total transmission rate.

As shown in Figure 6, we describe a system model of the F-RAN RAN's slice instances. Customized RAN slice instances are deployed to enable $K 0$ hotspot UEs and $K 1$ V2I UEs. Set $mathbbK 0$ to represent hotspot UEs and $mathbbK 1$ to represent V2I UEs. High rates are required by the hotspot UEs in the hotspot slice instance. As a result, $M 0$ remote radio units (RRUs) are spread across $K 0$ UEs and linked to the cloud server through fronthaul. $M 1$ F-APs with popular content stored are situated in specific zones to reduce the strain on fronthaul. F-APs communicate with the cloud server through the $X n$ interface. In the V2I slice instance, the F-AP mode and RRU mode are also accessible when V2I UEs require delay assured data transfer.

![Fig-6](https://vision.aioz.io/f/2d730e1d1bde4b97a12e/?dl=1)
*<center>**Figure 6**: A system model of RAN slice instances in F-RANs, wherein the deep Q-network (DQN) is utilized to make a decision to perform the content caching in F-APs and mode selection for UEs. ([Source](https://ieeexplore.ieee.org/document/8891508)).</center>*

Here, we have employed a deep reinforcement learning (DRL)-based approach, in which a DQN is built using historical data (visit Algorithm below for the process of DRL).

<Highlight name="Alg.3 DRL process">

1. To create the system state $s$, the cloud server obtains the channel state of each UE and the cache status of the F-AP  $s(t)$<br/>
2. The cloud then transmits the system state $s(t)$ to the agent DQN in order to receive the action $a(t)$<br/>
3. To balance exploration and exploitation, a classic-greedy approach is used. The action $a(t)$ requires users to specify communication modes and F-APs to perform content caching<br/>
4. Following the completion of an activity, the rewards may be acquired using the reward function specified in the following, and the system will transition to a new state<br/>
5. The DRL-based method would result in a maximum discounted cumulative reward<br/>
6. The current system state $s(t)$ is defined as an vector with $\{(4 + M1) * (K0 + K1) + K1 + 4C + 3\}$ dimensions:

$

s(t) =\{||h_{m,k}(t)||^2|m = 0, 1, ..., M_1, k = 1, 2, ..., K_0 + K_1\} 
\times \{B^{res0}_k(t), B^{res1}_k(t)|k = 1, 2, ..., K_0 + K_1\} 
\times \{T_k(t)|k = 1, 2, ..., K_1\} \times C(t) \times Req(t)
\times \{f_{z,c}(t)|z = 0, 1, 2, c = 0, 1, ..., C\}

$

Here, the cache features $f_{z,c}$ represents the total number of requests for content $A_c$ in a specific short (z = 0), medium ($z = 1$), long ($z = 2$)-term, respectively.

7.  System Action: the system action space is denoted as $\mathbb{A}$, wherein its element $a(t)$ is defined as follows:

$

a(t) = \{s_k(t)|k = 1, 2, ..., K_0 + K_1\} \times \{c(t)\}

$

8. The reward function is defined as weighted reward sum of hotspot UEs and V2I UEs:

$

r(t) = g(t) + \alpha_1 \sum_{k \in \mathbb{K}_0}r^0_k(t) + \alpha_2\sum_{k \in \mathbb{K}_1}r^1_k(t),

$

$

g(t) = g^s(t) + \rho g^l(t)

$

</Highlight>

<CodeExplanation>

> **NOTATION**
* $r^0_k(t)$ and $r^1_k(t)$ are slice specific rewards.
* $\alpha_1$ and $\alpha_2$ are their weights ranging from 0 to 1.
* $g(t)$ is a cache-related function.
* $\rho$ is the weight to balance the short and long-term cache hit ratios, namely $g^s(t)$ and $g^l(t)$.
* $g^s(t)$ is the number of requests for local content within the next request while $g^l(t)$ is within the next 10 requests.

</CodeExplanation>
