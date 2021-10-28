---
last_modified_on: "2021-10-30"
title: Smart Transcoding (part 3) three-stage STACKELBERG game.
description: A series of Smart Transcoding based on AI.
series_position: 14
author_github: https://github.com/Gazeal
tags: ["type: tutorial", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In this part, we will investigate three-stage STACKELBERG game approach for content processing.

![Fig-0](https://vision.aioz.io/f/d95290a29cda419bb702/?dl=1)

## Problem formulation

In a three-stage Stackelberg game, the upstage acts as the leader, making the very first action. The follower is the downstage and reacts based on the leader's strategy. During stage I, an SBS $m$ serves as the leader and sets the resource price $z_m$. Based on the resource price, the CP decides how many computing resources and which SBS to purchase in stage II. The CP then determines the content price and sells it to users. In stage III, a user $n$ decides which quality level of content to invest from CP and how much power to acquire from SBSs.

## Stage 1: Resource Allocation Model for SBSs
Assume that each SBS is self-centered and only cares about maximizing revenue. The revenue of each SBS is determined by its own resource cost and price, as well as the prices offered by the other SBSs. To maximize revenue, each SBS must determine an optimal price $z_m$. See Algorithm 1 for more details.

<Highlight name="Resource Allocation Iteration Algorithm" color="#0649c7">

INIT: Initialize the maximum number of iterations $\mathbb{T}$ and set iteration number  $\tau = 0$; Initialize a default resource price $z$ for each SBS<br/>
**while** $\tau < \mathbb{T}$ **do**<br/>
    &emsp;1. The CP determines the content price $y$ to their users and the amount of computing resources based on $z_m$<br/>
    &emsp;2. Each user selects the quality level of required contents $\beta_n$ and decides the amount of power resource $p_{mn}$<br/>
    &emsp;3. SBSs update their prices: $z_m(\tau) = \mathcal{G}_m(z_m(\tau -1))$<br/>
    &emsp;**if** $||z(\tau)-z(\tau-1)||/||z(\tau-1)||\leq \epsilon$ **then**<br/>
        &emsp;&emsp;Output the resource allocation $z$<br/>
        &emsp;&emsp;Break<br/>
    &emsp;**end if**<br/>
    &emsp;$\tau = \tau + 1$<br/>
**end while**

</Highlight>

## Stage 2: Content Price Model for CP
The profit of CP is determined by the quality of content required by users and the amount of computing resources consumed. Let $q_{nf} \in \{0, 1\}$ denotes whether user $n$ selects the content file $f$ or not. If user $n$ selects content file $f$ , $q_{nf} = 1$; otherwise, $q_{nf} = 0$.  To maximize its profit, CP needs to find its content price $y$ and the resource computation cost $f_m$. The CP optimization problem can be written as:

$

max_{f_m \geq 0, y \geq \phi} \mu_{CP}(f_m,y) = max_{f_m \geq 0, y \geq \phi} \alpha_v(y-\phi)\sum^N_{n=1}
\sum^F_{f=1}\sum^L_{l=1}\beta_nq_{nf}B_{fl} = \alpha_f\sum^M_{m=1}z_ma_mf_m+\sum^M_{m=1}min\{f_m,\sum^N_{n=1}\frac{\beta_n\sum^F_{f=1}\sum^L_{l=1}q_{nf}B_{fl}}{T_{n,thr}}\}

$

where $\alpha_f$ and $\alpha_v$ denote weighted factors to represent the trade-off between the resource cost and content revenue.  $\phi$ denotes the content cost of CP, such as caching cost and backhaul cost; $T_{n,thr}$ represents the maximum delay required by user $n$. The final term implies that we should ensure that users enjoy their video streaming service with a tolerable delay.

## Stage 3: Content Demand Model for Users
Taking the transmission rate and content quality level into account, the optimization problem for user $n$ can be stated as follows:

$

max_{p_{mn} \geq 0, \beta_n \geq 0} \mu^n_{user}(p_{mn},\beta_n) =max_{p_{mn} \geq 0, \beta_n \geq 0} - \mu_pz_mp_{mn}-\mu_vy\beta_n\sum^F_{f=1}\sum^L_{l=1}q_{nf}B_{fl}+min\{\mathcal{R}_m,\frac{\beta_n\sum^F_{f=1}\sum^L_{l=1}q_{nf}B_{fl}}{T_{n,thr}}\}

$

where $\mu_p$ and $\mu_v$ are weighted factors representing the power and content costs, respectively. We also assume that $\mu_p$ and $\mu_v$ are greater than zero. The final term implies that we must strike a balance between transmission rate and video content quality levels.

## Discussion
In the first three parts, we have explored the development of content transcoding, how AI can be applied in transcoding and the ste-of-the-art method for dealing with this task. In the next section, we will examine the  effectiveness of introduced method for content transcoding task.
