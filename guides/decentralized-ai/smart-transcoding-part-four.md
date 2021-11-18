---
last_modified_on: "2021-11-10"
title: Smart Transcoding (part 4) Effectiveness Evaluation.
description: A series of Smart Transcoding based on AI.
series_position: 15
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_transcoding"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In this part, we will evaluate the effectiveness of three-stage STACKELBERG game approach for content processing.

![Fig-0](https://vision.aioz.io/f/d95290a29cda419bb702/?dl=1)

## Convergence and Feasibility Analysis

In the duopoly case, the convexity of the follower's reaction function is required for the Stackelberg equilibrium to be unique. We have shown that the Next Estimate (NE) of the proposed three-stage Stackelberg game model exists because each stage has an equilibrium in a NE. Furthermore, because each stage has a unique equilibrium, the existence and uniqueness of the proposed three-stage game's equilibrium are guaranteed.

When the resource prices $z$ converge, the algorithm will come to a halt. Because the entire network's resource allocation is distributed, the CP, SBSs, and users only need to optimize their utility based on the responses of other entities. The computational complexity and signaling overheads of the proposed approach can be significantly reduced when compared to centralized schemes that optimize a unified objective in a centralized manner.

## Simulation
We use Monte Carlo simulation to assess the approach's performance in wireless networks with MEC and blockchains. In an urban setting, we examine a 120m x 120m square area. There are 1000 videos available, with popularity following a Zipf distribution with an exponent of 0.8. The videos are assumed to be distributed at a rate of 200 kbps to 2 Mbps. The channel gain models described in the 3GPP specification are used in this application. Figure I summarizes the important simulation settings.

![Fig-1](https://vision.aioz.io/f/5911b566ae1441fc99c4/?dl=1)
*<center>**Figure 1**: The simulation parameters ([Source](http://vdlt.io/assets/images/TB/Paper5.pdf)).</center>*

For comparison, we've chosen the following three typical schemes: 1) the no-caching scheme, which delivers video files without caching, as labeled "No caching"; 2) the no-transcoding scheme, which delivers video files without transcoding, as labeled "No transcoding"; and 3) the centralized resource allocation scheme, which is attained by method of exhaustion to minimize network costs for video transcoding and delivery, as labeled "Centralized scheme."

## Experimental Details
The entire network cost vs the total frequency band and the number of users is depicted in Figures 2 and 3, respectively. It can be seen in Fig. 2, all of the curves steadily decline as the frequency band is increased. This is due to the fact that spectrum allotment is often a key factor in increasing the transmission rate between customers and SBSs. The network cost of transmission can then be considerably reduced. In Figure 3, the "No caching" plan has the greatest total network cost when compared to the other methods. The reason for this is that without proactive caching, SBSs are forced to transfer content by retrieving it from a remote CP across backhaul networks.

As a result, duplicate content transmissions exist, necessitating massive backhaul resources. Furthermore, when transcoding technology is applied, SBSs may process a variety of content quality levels at the network edge. Duplicated content transmission is minimized even further, and backhaul resource consumption is lowered.

![Fig-2](https://vision.aioz.io/f/9f91fc6cdb524585829c/?dl=1)
*<center>**Figure 2**: The entire cost of the network vs the total frequency band. ([Source](http://vdlt.io/assets/images/TB/Paper5.pdf)).</center>*

![Fig-3](https://vision.aioz.io/f/9c13e23f470d4edbb617/?dl=1)
*<center>**Figure 3**: The total network cost in relation to the number of users. ([Source](http://vdlt.io/assets/images/TB/Paper5.pdf)).</center>*

## Discussion
SBSs with MEC servers make their resources available to allow adaptive video streaming in a distributed, incentivized, and secure way. The optimization problem has been framed as a three-stage Stackelberg game, which includes resource allocation, content pricing determination, and content quality levels. The subgame equilibrium for each stage and the interplays of the three-stage game are investigated. To obtain the result, an iterative approach was used. The approach outperforms other existing schemes in simulations, according to the results.
