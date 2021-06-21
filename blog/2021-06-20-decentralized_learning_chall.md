---
last_modified_on: "2021-06-20"           
id: FL-trends     
title:Federated Learning, Challenges and Hot Trends for research.
description:  Discussion about Challenges and Hot Trends in Federated Learning for research.      
author_github: https://github.com/Gazeal  
tags: ["type: post", "AI", "Federated Learning"]
---

## Federated learning and its remaining challenge

Federated learning is the process of training statistical models via a network of distant devices or siloed data centers, such as mobile phones or hospitals, while keeping data locally. In terms of federated learning, there are five major obstacles that have a direct impact on the paper publishing trend.

#### 1. Expensive Communication
Due to the internet connection, huge number of users, and administrative costs, there is a bottleneck in communication between devices and server-devices.
    
#### 2. Systems Heterogeneity
Because of differences in hardware (CPU, RAM), network connection (3G, 4G, 5G, wifi), and power, each device in federated networks may have different storage, computational, and communication capabilities (battery level).
    
#### 3. Statistical Heterogeneity
Devices routinely create and collect data in non-identically dispersed ways across the network; for example, in the context of a next word prediction, mobile phone users employ a variety of languages. Furthermore, the quantity of data points on different devices may differ greatly, and an underlying structure may exist that describes the interaction between devices and their related distributions. This data generation paradigm violates frequently-used independent and identically distributed (I.I.D. problem) assumptions in distributed optimization, increases the likelihood of stragglers, and may add complexity in terms of modeling, analysis, and evaluation.
    
#### 4. Privacy Concerns
In federated learning applications, privacy is a crucial problem. Federated learning takes a step toward data protection by sharing model changes, such as gradient information, rather than the raw data created on each device. Nonetheless, transmitting model updates during the training process may divulge sensitive information to a third party or the central server.
    
#### 5. Domain transfer
Not any task can be applied to the federated learning paradigm to finish their training process due to the aforementioned four challenges.

## Hot trends
#### Data distribution heterogeneity and label inadequacy.
-   Distributed Optimization    
-   Non-IID and Model Personalization    
-   Semi-Supervised Learning    
-   Vertical Federated Learning    
-   Decentralized FL    
-   Hierarchical FL    
-   Neural Architecture Search    
-   Transfer Learning    
-   Continual Learning    
-   Domain Adaptation    
-   Reinforcement Learning    
-   Bayesian Learning 
#### Security, privacy, fairness, and incentive mechanisms:
-   Adversarial-Attack-and-Defense    
-   Privacy    
-   Fairness    
-   Interpretability    
-   Incentive Mechanism    
#### Communication and computational resource constraints, software and hardware heterogeneity, the FL system
-   Communication-Efficiency    
-   Straggler Problem    
-   Computation Efficiency    
-   Wireless Communication and Cloud Computing    
-   FL System Design
#### Models and Applications
-   Models    
-   Natural language Processing    
-   Computer Vision    
-   Health Care    
-   Transportation    
-   Recommendation System    
-   Speech
-   Finance    
-   Smart City    
-   Robotics    
-   Networking    
-   Blockchain    
-   Other
#### Benchmark, Dataset, and Survey
-   Benchmark and Dataset
-   Survey
