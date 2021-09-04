---
last_modified_on: "2021-07-12"      
title: Network Sytem and data silos for Federated Learning.
description: A detail description about network system for federated learning in reasearch.
series_position: 3
author_github: https://github.com/aioz-ai
tags: ["type: tutorial", "level: advance"]
---

## Introduction for network system in federated learning
In previous parts, we have examined the learning algorithm for distributed data. However, to achieve a successful federated learning process, the network system is essential since it determines the stability of the whole process.
Specifically, a network system is a geo-distributed machine learning system that satisfies the two following requirements:
1. We can employ an intelligent communication mechanism into the network over WANs to efficiently utilize the scarce WAN bandwidth.
2. The network is generic and flexible enough to run a wide range of machine learning algorithms, with minimal required modification to the learning algorithms.

In this part, we explore popular network systems listed below that are widely used for benchmarking in research:
- [Gaia](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-hsieh.pdf): a generic geo-distributed machine learning system that decouples data center communication from data center communication, allowing for distinct communication and consistency models for each.
- [The AWS Global Cloud Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/?nc1=h_ls): touted as the most secure, comprehensive, and dependable cloud platform, with over 200 fully featured services available from data centers all over the world.
- [Géant](https://www.geant.org/Networks): the large-scale, high-speed networks required for sharing, accessing, and analyzing the massive amounts of data created by the research and education sectors, as well as for testing new technologies and applications.
- [Exodus and Ebone](http://www.cs.umd.edu/~nspring/papers/sigcomm2002.pdf):  real topologies from Rocketfuel engine . The Ebone network connect European cities and Exodus network connect American cities.

## Gaia

![Fig-1](https://vision.aioz.io/f/4a31c2c6992b444eaea3/?dl=1)
*<center>**Figure 1**:  Gaia system overview [(Sourcce)](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-hsieh.pdf).</center>*

Each data center in Gaia contains a set of worker computers and parameter servers. To accomplish data parallelism, each worker computer processes a shard of the input data stored in its data center. Each data center's parameter servers retain a version of the global model copy, and each parameter server is in charge of a shard of this global model copy. A worker machine in a data center simply READS and UPDATES the global model copy.

The global model copy in each data center is only approximately correct to decrease the communication overhead over WANs. We may remove negligible, and hence unneeded, communication between data centers with this approach. We create a novel synchronization model called Approximate Synchronous Parallel (ASP) that ensures that each global model copy is approximately accurate even when WAN bandwidth is limited.

Worker machines and parameter servers in a data center, on the other hand, use the traditional BSP (Bulk Synchronous Parallel) or SSP (Stale Synchronous Parallel) architectures to synchronize with one another. These approaches enable worker computers to immediately notice recent modifications in a data center. Additionally, within a data center, worker machines and parameter servers might use more aggressive communication methods, such as delivering updates early and often, to fully exploit the plentiful (and cheap) network traffic on a LAN.

## The AWS Global Cloud Infrastructure
![Fig-2](https://vision.aioz.io/f/d8cae7ebdc664f02af46/?dl=1)
*<center>**Figure 2**:  The AWS Global Cloud Infrastructure [(Sourcce)](https://aws.amazon.com/about-aws/global-infrastructure/?nc1=h_ls).</center>*

The AWS Global Cloud Infrastructure is the most secure, comprehensive, and dependable cloud platform available, with over 200 fully featured services available from data centers around the world. Whether you need to deploy your application workloads around the world in a single click or create and deploy particular apps closer to your end-users with single-digit millisecond latency, AWS can help.

AWS offers the largest and most vibrant ecosystem in the world, with millions of active customers and tens of thousands of partners. Every possible use case is being performed on AWS by customers from almost every industry and of every size, including start-ups, corporations, and public sector organizations.

![Fig-3](https://vision.aioz.io/f/96615aeb2df54af3ad69/?dl=1)
*<center>**Figure 3**:  The AWS Global Cloud Infrastructure statistical [(Sourcce)](https://aws.amazon.com/about-aws/global-infrastructure/?nc1=h_ls).</center>*

## Géant
![Fig-4](https://vision.aioz.io/f/41e004f2994e40cb9364/?dl=1)
*<center>**Figure 4**:  The Ge1ant live map [(Sourcce)](https://www.geant.org/Networks).</center>*

Géant is a virtually lossless network, which means that whatever data it receives is sent to its destination without any packets being dropped. Géant actively controls capacity to handle bursts and periodic variations in traffic flow in order to achieve this. In order to maximize throughput of host data flows, transmission of traffic at line rate should be feasible at all times with minimum or no buffering. As a result, data is transmitted without loss, and protocols operating on hosts may be adjusted to function in a practically lossless environment.

Géant is a high-speed network with lines acquired to provide the shortest feasible latency between Géant PoPs in key European cities. This also provides the fastest reaction time between any two Géant network entry points.

Géant aims to connect all nations with sufficient capacity to enable high-speed networking. In nations where such resources are rare and the market is restricted, this allows the community to bridge the digital gap by providing appropriate connectivity.

## Exodus and Ebone
Both Exodus and Ebone are formed by the original [ISP network](http://www.cs.umd.edu/~nspring/papers/sigcomm2002.pdf) and their comunntation policies follow the [Rocketfuel](http://www.cs.umd.edu/~nspring/papers/sigcomm2002.pdf).

![Fig-5](https://vision.aioz.io/f/6ea6d37b3cfa4ec6b10a/?dl=1)
*<center>**Figure 5**:ISP networks are composed of POPs and backbones. Solid dots inside the cloud represent POPs. A POP consists of backbone and access routers (inset). Each traceroute across the ISP discovers the path from the source to the destination [(Sourcce)](http://www.cs.umd.edu/~nspring/papers/sigcomm2002.pdf).</center>*

As illustrated in Figure 5, an ISP network is composed of many points of presence, or POPs. Each POP is a physical place where an ISP's routers are housed. The ISP backbone connects these POPs, while backbone or core routers are routers linked to inter-POP lines. Access routers serve as an intermediary layer between the ISP backbone and routers in nearby networks within each POP. These neighbor routers are made up of both BGP and non-BGP speakers, with the majority of them being non-BGP speaking small businesses.

![Fig-6](https://vision.aioz.io/f/e1b58b75c9e740e8b5ae/?dl=1)
*<center>**Figure 6**: Architecture of Rocketfuel. The Database becomes the inter-process communication substrate [(Sourcce)](http://www.cs.umd.edu/~nspring/papers/sigcomm2002.pdf).</center>*

Figure 6 depicts the structure of Rocketfuel. All data in the blackboard architecture is stored in a PostgreSQL database, which serves as both a repository for measurement results and a platform for communication between asynchronously executing processes. We can perform SQL queries for basic inquiries and quickly incorporate additional analytic modules thanks to the usage of a database.

# Discussion for decentralized infrastructure in decentralized federated learning.
Although there is a wide range of network systems have been introduced, there is currently no network system that focuses on a  peer-to-peer conversation between high-speed well-hardware silos. Since the effectiveness of these kinds of data centers has been shown off in the different fields, we believe that we need an appropriate network that is special design for them.
