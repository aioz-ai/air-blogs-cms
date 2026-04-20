---
last_modified_on: "2026-03-28"
title: AeroScene: Progressive Scene Synthesis for Aerial Robotics
description: Scene Generation, Aerial Robotics, Diffusion model
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


In the first part, we introduce the scene generation task for aerial robotics. In this section, we take a deeper look at our core method, AeroScene, for producing high-quality and realistic scenes. 

## Methodology

We adopt a *denoising diffusion probabilistic model (DDPM)* to learn the distribution of plausible scene layouts.
Let $\alpha_t = 1 - \beta_t$ and $\bar{\alpha}_t = \prod_{s=1}^t \alpha_s$ for a fixed noise schedule $\{\beta_t\}_{t=1}^T$. The forward process gradually adds Gaussian noise to a clean layout $x_0$:

$$
q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{\alpha_t}\, x_{t-1}, \beta_t I),
$$


The reverse process removes noise step-by-step:

$$
p_\theta(x_{t-1} | x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, t), \Sigma_\theta(x_t,t)),
$$


where $\mu_\theta$ is predicted by a hierarchy-aware pipeline every denoising timestep. Concretely, at each timestep $t$, we apply the Initial Layout and Hierarchy Embedding to convert $x_t$ into hierarchy tokens, then extract coarse and fine features, fuse them with the Cross-scale Progressive Attention, and finally condition the denoising UNet on the fused features. Guidance objectives are applied to adjust $\mu_\theta$ before sampling $x_{t-1}$. This design ensures the denoiser performs hierarchical reasoning at each diffusion step, matching the iterative sampling loop. Figure 2 shows the details of our method.



![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624303/hiescenemethod_gcpbup.png)
*<center>**Figure 2**: An overview of our AeroScene method.</center>*


### Initial Layout and Hierarchy Embedding
At each diffusion step, the noisy layout $x_t$ is represented as

$$
x_t = \{ o_i \mid o_i = (\mathbf{p}_i, \mathbf{q}_i, \mathbf{s}_i, c_i) \}_{i=1}^{N_o},
$$

where $\mathbf{p}_i \in \mathbb{R}^3$ is the position, $\mathbf{q}_i \in \mathbb{R}^4$ is the orientation quaternion, $\mathbf{s}_i \in \mathbb{R}^3$ is the scale, $c_i \in \{1,\dots,C\}$ is the semantic category label represented as a $C$-dimensional one-hot vector which conditioned as learned soft-embeddings, and $N_o$ is the number of objects in $x_t$.

Each element is mapped to a $d_m$-dimensional token:

$$
\mathbf{h}_i = \mathbf{f}_i^{(0)} + \mathbf{e}^{\text{pos}}_i + \mathbf{e}^{\text{dom}}_i,
$$

where $\mathbf{f}_i^{(0)} = \text{MLP}([\mathbf{p}_i, \mathbf{q}_i, \mathbf{s}_i, \text{Emb}(c_i)])$
encodes geometry and semantics, $\mathbf{e}^{\text{pos}}_i$ is sinusoidal positional encoding, and $\mathbf{e}^{\text{dom}}_i$ is a learned indoor/outdoor domain embedding parameterized by a small trainable embedding vector per domain, following domain-adaptive encodings.


We predict a tokenizability score $\tau_i \in [0,1]$ for each object at the same timestep:

$$
\tau_i = \sigma\left( \mathbf{w}_\tau^\top \, \text{MLP}(\mathbf{f}_i^{(0)}) \right),
$$

Then, a coarse- or fine-grained route check is performed on tokens based on their tokenizability scores, by comparing them against a learned gating threshold $\gamma$, which determines whether each token is classified as a coarse token $\mathcal{T}_{\text{coarse}}$ or a fine-grained token $\mathcal{T}_{\text{fine}}$.

$$
\mathcal{T}_{\text{coarse}} = \{\mathbf{h}_i \mid \tau_i < \gamma \}, \qquad
\mathcal{T}_{\text{fine}} = \{\mathbf{h}_i \mid \tau_i \geq \gamma \}.
$$

In our implementation $\gamma$ is a learnable scalar (optimized jointly with network parameters). 

### Coarse and Fine Feature Extraction


Tokens $\mathcal{T}_{\text{coarse}}$ are passed through a lightweight 3D CNN to extract coarse-level features $F_{\text{coarse}}$, which are essential for constructing exteriors (e.g., large-scale structural elements like buildings or terrain):

$$
F_{\text{coarse}} = \text{CNN}_{\text{coarse}}(\mathcal{T}_{\text{coarse}}), \quad F_{\text{coarse}} \in \mathbb{R}^{N_c \times d_m}.
$$


Tokens $\mathcal{T}_{\text{fine}}$ are used to construct a spatial adjacency graph $G_{\text{fine}}=(V,E)$: each node corresponds to a fine token and edges connect pairs whose Euclidean distances do not exceed  $ \delta_f$. A two-layer GNN refines these local features:

$$
F_{\text{fine}} = \text{GNN}_{\text{fine}}(\mathcal{T}_{\text{fine}}, G_{\text{fine}}), \quad F_{\text{fine}} \in \mathbb{R}^{N_f \times d_m},
$$

with node updates

$$
\mathbf{f}_i^{(l+1)} = \text{MLP}\Big( \mathbf{f}_i^{(l)} \,\Vert\, \sum_{j \in \mathcal{A}_i} \text{MLP}(\mathbf{f}_j^{(l)}) \Big).
$$

where $\mathcal{A}_i$ denotes the set of neighboring nodes of token $i$ in $G_{\text{fine}}$, and $\Vert$ indicates vector concatenation.  


### Cross-scale Progressive Attention

Cross-scale Progressive Attention is composed of stacked cross-scale attention blocks that alternate between top-down (coarse $\rightarrow$ fine) and bottom-up (fine $\rightarrow$ coarse) interactions:  

$$
\text{Top-down: } 
Q = F_{\text{fine}}, \quad K,V = F_{\text{coarse}},
$$

$$
\text{Bottom-up: } 
Q = F_{\text{coarse}}, \quad K,V = F_{\text{fine}}.
$$

Each block follows the pattern  
$$
F' = \text{FC}\!\left(\text{Attn}(\text{LN}(Q), \text{LN}(K), \text{LN}(V))\right) + F,
$$
where $F$ is the residual input to the block. By stacking $L$ alternating top-down and bottom-up blocks, coarse tokens propagate structural context while fine tokens inject local detail. The outputs are concatenated and projected to form the final feature:
$$
F_{\text{attn}} \in \mathbb{R}^{(N_c+N_f) \times d_m},
$$
which condition the UNet denoiser at each diffusion step via cross-attention layers in the UNet.

### Guidance Objectives
We ensure the physical plausibility and interactivity of generated scenes by guiding the conditional scene diffusion process with physics-based guidance functions. Additionally, the extensibility of fine-grained layout placement must align with coarse-scale structures while ensuring consistency in object categories and spatial relationships. This motivation led us to implement three guidance objectives:

#### Collision Avoidance Guidance
Penalizes spatial overlap above a small tolerance $\delta_d$ using 3D IoU evaluation:
$$
\mathcal{L}_{\text{col}}(x_t) = \sum_{i \neq j} \max(0, \text{IoU}(B_i, B_j) - \delta_d),
$$
where each $B_i$ is an oriented 3D bounding box parameterized by its center $\mathbf{p}_i$, orientation quaternion $\mathbf{q}_i$, and scale $\mathbf{s}_i$, with $B_j$ defined analogously.

#### Coarse-to-Fine Guidance
Encourages fine-grained placement to remain consistent with the coarse-scale structural plan:
$$
\mathcal{L}_{\text{c2f}}(x_t) = \sum_{o_i \in \text{fine}} \text{dist}\big( \mathbf{p}_i, \mathcal{R}_{\text{coarse}}(o_i) \big),
$$
where $\mathcal{R}_{\text{coarse}}(o_i)$ is the spatial region assigned by the corresponding coarse token (determined via Euclidean distance to coarse centers), and $\text{dist}(\cdot,\cdot)$ denotes the Euclidean distance between the object center $\mathbf{p}_i$ and the region's centroid.


#### Semantic Constraint Guidance
Ensures object categories and spatial relations follow learned semantic priors:
$$
\mathcal{L}_{\text{sem}}(x_t) = - \sum_{(o_i, o_j)} \log P_{\text{sem}}(c_i, c_j, r_{ij}),
$$
where $P_{\text{sem}}$ is an MLP estimated from pairwise statistics of the training set and 
$r_{ij}$ denotes the spatial displacement between objects $o_i$ and $o_j$.


We define the combined guidance loss as
$$
\mathcal{L}_{\text{guide}}(x_t) = 
\mathcal{L}_{\text{col}}(x_t) + 
\mathcal{L}_{\text{c2f}}(x_t) + 
\mathcal{L}_{\text{sem}}(x_t).
$$

During training, $\mathcal{L}_{\text{guide}}$ enters the total loss with a small weight (see Algorithm 1). During inference, $\nabla_{x_t}\mathcal{L}_{\text{guide}}$ is normalized and scaled before adjusting $\mu_\theta$ (Algorithm 1). This is depicted in Fig 2 as a feedback arrow from Guidance Objectives to the denoiser output.

### Training Algorithm
Training proceeds by sampling a batch of ground-truth layouts and a noise timestep $t$, forming the noisy input $x_t$. For each $x_t$, we compute hierarchy tokens and extract multi-scale conditioning via the coarse/fine branches and the Cross-scale Progressive Attention, then predict $\epsilon_\theta(x_t,t)$ and minimize the reconstruction loss
$$
\mathcal{L}_{\text{rec}}=\|\epsilon-\epsilon_\theta(x_t,t)\|^2.
$$

In parallel, we compute differentiable guidance losses $\mathcal{L}_{\text{guide}}$  on object parameters (using smooth surrogates such as soft IoU, signed distance functions, or soft adjacency). The total training loss is
$$
\mathcal{L}=\mathcal{L}_{\text{rec}}+\lambda_{\text{guide}} \mathcal{L}_{\text{guide}},
$$
where $\lambda_{\text{guide}}$ is a scalar value controlling the influence of guidance. Guidance is applied softly (i.e., small $\lambda_{\text{guide}}$ value) during training to stabilize optimization, while at inference (Algorithm 2), it is always enforced with step-size scaling.  

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624517/algo1_mm1vov.png)
*<center>**Algorithm 1**:  Training procedure</center>*



### Inference Algorithm
Inference performs the reverse diffusion starting from $x_T\sim\mathcal{N}(0,I)$ and iterating $t=T,\dots,1$. At each step we compute hierarchy conditioning for the current $x_t$, predict $\epsilon_\theta(x_t,t)$, and obtain the denoising mean via the epsilon-parameterization
$$
\mu_\theta(x_t,t)=\frac{1}{\sqrt{\alpha_t}}\Big(x_t-\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon_\theta(x_t,t)\Big).
$$

We then evaluate the differentiable guidance loss $\mathcal{L}_{\text{guide}}$ on object parameters, compute its gradient 
$$
g = \nabla_{x_t}\mathcal{L}_{\text{guide}}(x_t),
$$
normalize it $\tilde g=g/(\|g\|+\varepsilon)$, and scale it by a timestep-dependent step-size
$$
\eta_t = \eta_0\sqrt{1-\bar\alpha_t}.
$$
The denoising mean is adjusted as
$$
\mu'_\theta = \mu_\theta - \eta_t \tilde g,
$$
and the next state is sampled
$$
x_{t-1} \sim \mathcal{N}(\mu'_\theta,\Sigma_\theta(x_t,t)).
$$


After sampling, normalize quaternions $\mathbf{q}_i \leftarrow \mathbf{q}_i / \|\mathbf{q}_i\|$ is applied for each object. Unlike training, guidance is always applied during inference to enforce physical plausibility and semantic consistency.


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624517/algo2_nnkuhl.png)
*<center>**Algorithm 2**:  Inference procedure</center>*

In the next part, we will evaluate the effectiveness of our approach on the AeroScene dataset, comparing it with baseline methods.




</CodeExplanation>