---
last_modified_on: "2021-09-20"
title: Smart Transcoding (part 2) Video Transcoding and Delivery with Blockchain.
description: A series of Smart Transcoding based on AI.
series_position: 13
author_github: https://github.com/Gazeal
tags: ["type: tutorial", "level: advance"]
---


In this part, we will investigate how to deal with the Blockchain-Enabled video transcoding using deep reinforcement learning, including the network model and the current sota three-stage STACKELBERG game approach for video transcoding and delivery.

![Fig-0](https://vision.aioz.io/f/d95290a29cda419bb702/?dl=1)

## Network Model

![Fig-1](https://d3i71xaburhd42.cloudfront.net/e87768672ee1ccc4d31d02fe158b68d9b6a5bd0b/3-Figure1-1.png)
*<center>**Figure 1**: The architecture of video transcoding and delivery with blockchain and mobile edge computing. ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.semanticscholar.org%2Fpaper%2FVideo-Transcoding-and-Delivery-with-Blockchain-and-Liu-Li%2Fe87768672ee1ccc4d31d02fe158b68d9b6a5bd0b&psig=AOvVaw3-12T5ro7HCjqNOW3lSPNP&ust=1631763295276000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCIDf7o-GgPMCFQAAAAAdAAAAABAN)).</center>*

As shown in Figure 1, we envision a virtual P2P blockchain network to be the backbone of the distributed content delivery network. There are $M$ SBSs equipped with MEC servers, which have minimal caching and processing resources but are sufficient to fulfill the CP's expectations. We suppose that the $N$ consumers serviced by these SBSs enjoy video material from a single CP (e.g. Youtube or Netflix). All content files $f \in \mathbb{F}$ are chunked into $l$ fixed-length chunks, and each segment has distinct bitrate versions in different formats. Then, we examine a rate $n$, where $0 < \beta_n \leq 6$ represents the multiple video quality levels requested by user $n$, to scale down the original movies. Each single user can only pick one quality level of a movie at a time. Finally, the total bandwidth of the accessible spectrum is $W$. During packet transmission, the channel does not change, and perfect instantaneous Channel Quality Information (CQI) is provided. Assume that SBSs have cached a collection of popular items proactively. It charges the CP for allocating computing resources $f_m$ (Mbps) and the user for allocating power pmn ($W$) for an arbitrary SBS $m$. We examine a split spectrum method, in which the same spectrum is divided among its neighboring interfering SBSs and spectrum inside one SBS is orthogonally allotted to each user. Assume that SBS $m$ sends the needed video content to user $n$ through wireless access connections. The data rate of user $n$ when connected to the $m$-th SBS is indicated as follows:

$

\mathbf{R}_{mn} = w_{mn} log_2 \left( 1 + \frac{\rho_{mn}|h_{mn}|^2}{\delta^2}\right)

$

where $|h_{mn}|^2$ denotes the channel gain between SBS $m$ and user $n$; $w_{mn}$ denotes the allocated bandwidth of SBS $m$ to user $n$; $\delta^2$ denotes the power spectrum density of the additive white Gaussian noise (AWGN).

## Video Transcoding and Delivery Process

We use a video transcoding and distribution strategy that includes a number of tiny smart contracts to ensure that the content trade is executed correctly. Each entity (for example, a CP, SBS, or user) translates as a node in the blockchain network and is identifiable by a unique transaction address based on the hash-code of its public key.

During the video transcoding and distribution process, consumers initially request a collection of materials based on their preferences. When a request comes on the blockchain, a content brokering contract (CBC) is produced and broadcast that defines the needed contents. The CP is then alerted of the new CBC, which is used to construct a content licensing contract (CLC). The CLC defines the user's content pricing, a reference to the CBC, and the maximum payment to SBSs for transcoding and delivery. When the CLC becomes public on the blockchain, SBSs respond by posting content delivery contracts (CDCs) that define the resource pricing for providing content as well as a reference to the CLC. Finally, the original CBC gathers all relevant CDCs, chooses the cheapest one to carry out the content distribution, and cancels all other contracts. Following content trading, a user pays for the content transaction using a CP's address and the delivery transaction using an SBS's address. CP pays a group of SBSs for a content processing transaction through SBS addresses. The payees (CPs or SBSs) then produce transaction records, which the payers (CPs or users) check and digitally sign before uploading for audit.

We use a proof-of-stake (PoS) consensus mechanism, with the CP and SBSs acting as consensus nodes to participate in the consensus process. Users act as user nodes, sending just requests for content delivery. To validate transactions and produce blocks, CP and SBSs who function as forgers first ‘stake' their own currencies before participating in the consensus process. We calculate each SBS's stake based on the amount of transcoding and delivery rewards obtained, and we consider the CP's stakes to be proportionate to the amount of unconfirmed payment for contents in the most recent time slot. They are typically encouraged to authenticate the proper transactions since they have staked their own money. If they verify a fraudulent transaction, they forfeit their holdings as well as their future rights to participate as a forger.
This prevents CPs and SBSs from abusing the content transcoding and delivery environment.

## Next section introduction
In the next section, we will investigate the algorithm of three-stage STACKELBERG game approach for video transcoding and delivery.
