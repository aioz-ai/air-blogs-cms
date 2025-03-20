---
last_modified_on: "2025-04-07"
id: FedEFM3
title: FedEFM - Federated Endovascular Foundation Model with Unseen Data (Part 3)
description: The way to train our FedEFM and integrated the weights to various downstream task. 
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Medical", "Robotic"]
---

In this part, we will deeply dive into the way to train our FedEFM and integrated the weights to various downstream task. 
## Method 

### Notations 

We describe the notations used in the methodology


| Notation | Description   |
|---------|------------|
| $\theta_i(k)$ | Weights in silo $i$ in commuication round $k$ | 
| $\xi_i$ | Mini batch data in silo $i$ | 
| $\theta_{i \rightarrow j}$ | Weights in silo $i$ that is transferred to silo $j$ | 
| $\hat{\theta}_{i \rightarrow j}$ | Successfully transferred weights from silo $j$ back to silo $i$ | 
| $\mathcal{L}_c$ | Foundation loss function | 
| $\alpha$ | Learning rate |
| $\mathcal{N}_i$ | In-neighbors of silo $i$ in the topology |
| $T$ |  Temperature for distillation |
| $\vartheta \in \{0,1\}$ |  Accumulation status |
| $N$ |  Number of silos |

### Federated Distillation 
Figure 1 demonstrate the algorithm used for training a foundation model within a decentralized federated learning process, effectively addressing the issue of the unseen data problem. 

Specifically, in the initial round, local model weights  $\theta_i$ of each $i$-th hospital silo is trained using their respective local data $\xi_i$. Within the next communication round, we first perform overseas training where local model weights $\theta_i$ of each $i$-th silo is transmitted to each of their $j$-th neighbor hospital silo. This process aims to let local weights $\theta_i$ learn knowledge from the data $\xi_j$ of its $j$-th neighbor silo. 

In $(k+1)$-th specific communication round, each transferred weight $\theta_{i\rightarrow j}$ is optimized in $j$-th silo using the following equation: 

$$
\theta_{i\rightarrow j}(k+1)=   \theta_{i\rightarrow j}\left(k\right)-\alpha_{k}\nabla  \mathcal{L}_{c}\left(\theta_{i\rightarrow j}\left(k\right),\xi_j\left(k\right)\right) \ \ \ (1)
$$

Then, we perform knowledge transfer where each learned overseas expert $\theta_{i\rightarrow j}$ from the previous step is transferred back to the $i$-th silo.

In the local silo $i$, the local weight is updated based on both the original weight $\theta_{i}$ and the transferred weights $\hat \theta_{i\rightarrow j}$ that is learned from the neighbour silo $j$. In particular, we aim to find regions that share similarities between two weights using the Earth Mover’s Distance $\text{EMD}( \theta_{i}, \hat \theta_{i\rightarrow j})$. In this way, the distance measures the contribution of transferred weights during distillation, enabling the local silo to learn from its neighbors while avoiding divergence when weight convergence goals differ significantly. Local weights $\theta_{i}$ is then optimized using the following equation:


$$
\theta_{i}(k+1) =  \theta_i(k)-\\ \alpha_{k}\sum_{j \in \mathcal{N}(i)}\text{EMD}( \theta_{i}, \hat \theta_{i\rightarrow j},k)\nabla  \mathcal{L}^i_{\rm MD}\left({\theta}_i\left(k\right),{\hat \theta}_{i\rightarrow j}\left(k\right),\xi_i\left(k\right)\right) \ \ (2)
$$

### Differentiable Earth Mover's Distance
Assume that the input sample $\xi_i$ from $i$-th local silo passes through the foundation architecture $\theta_{i}$ to generate the dense representation $\mathbf{U} \in \mathbb{R}^{H \times W \times C}$, where $H$ and $W$ denote the spatial size of the feature map and $C$ is the feature dimension. In a parallel manner, $\mathbf{V} \in \mathbb{R}^{H \times W \times C}$ also denotes the dense representation when $\xi_i$ passes through $\hat{\theta}_{i\rightarrow j}$.

Under Earth Mover circumstance, $\mathbf{V}$ represents suppliers transporting goods to demanders $\mathbf{U}$. Then, $\text{EMD}$ between two feature sets $\mathbf{U} = \{u_1, u_2, \ldots, u_{HW}\}$ and $\mathbf{V} = \{v_1, v_2, \ldots, v_{HW}\}$ can be computed as:
$$
\text{EMD}(\theta_{i}, \hat \theta_{i \rightarrow j}) = \text{EMD}(\mathbf{U}, \mathbf{V}) = \sum_{p=1}^{HW} \sum_{q=1}^{HW} (1 - c_{pq}) \tilde{x}_{pq} \ \ \ (3)
$$ 


where $\tilde{x}$ is conducted from optimal matching flow $\tilde{X} = \{x_1, x_2, \ldots, x_{pq}\} $ for each sample pair of two sets $\mathbf{U}$ and $\mathbf{V}$; $c_{pq}$ is the cost per unit transported from supplier to demander and is obtained by computing the pairwise distance between embedding nodes $u_p \subset \mathbf{U}$ and $v_q \subset \mathbf{V}$.

The cost per unit $c_{pq}$ is computed as below and also plays a virtual role in computing the optimal matching flow:
$$
c_{pq} = 1 - \frac{u_p^T v_q}{\|u_p\|\|v_q\|}
$$

where nodes with similar representations tend to generate small matching costs between each other. 
Then, the optimal matching flow $\tilde{X}$ is conducted by optimizing $\tilde{x}$ as below: 
$$
\underset{x}{\text{minimize}} \quad  \sum_{p=1}^{HW} \sum_{q=1}^{HW} c_{pq} x_{pq} \\
\text{subject to} \quad  x_{pq} > 0, \quad p = 1, \ldots, HW, \; q = 1, \ldots, HW\\
\sum_{p=1}^{HW} x_{pq} = v_q, \quad q = 1, \ldots, HW \\
\sum_{q=1}^{HW} x_{pq} = u_p, \quad p = 1, \ldots, HW
$$

Here, EMD seeks an optimal matching $\tilde{X}$ between suppliers and demanders such that the overall matching cost is minimized. The global optimal matching flows $\tilde{X}$ can be achieved by solving a Linear Programming problem (LP). For the sake of completeness, we transform the above optimization to a compact matrix form

$$
\underset{x}{\text{minimize}} \quad c(\theta)^T x \\
\text{subject to} \quad G(\theta)x \leq h(\theta),\\
A(\theta)x = b(\theta).
$$


Here $x \in \mathbb{R}^{HW \times HW}$ is our optimization variable. $Ax = b$ represents the equality constraint and $Gx \leq h$ denotes the inequality constraint in our optimization problem. Accordingly, the Lagrangian of the LP problem is given by:

$$
L(\theta, x, \nu, \lambda) = c^T x + \lambda^T (Gx - h) + \nu^T (Ax - b),
$$



where $\nu$ denotes the dual variables on the equality constraints and $\lambda \geq 0$ denotes the dual variables on the inequality constraints.
Following the KKT conditions, we obtain the optimum $(\tilde{x}, \tilde{\nu}, \tilde{\lambda})$ of the objective function by solving $g(\theta, \tilde{x}, \tilde{\nu}, \tilde{\lambda}) = 0$ with primal-dual interior point methods, where

$$
g(\theta, x, \nu, \lambda) = \begin{bmatrix}
\nabla_{\theta} L(\theta, x, \nu, \lambda) \\
\textbf{diag}(\lambda)(G(\theta)x - h(\theta)) \\
A(\theta)x - b(\theta)
\end{bmatrix}.
$$


Then, with the theorem below, we can derive the gradients of the LP parameters.

Suppose $g(\theta, \tilde{\lambda}, \tilde{\nu}, \tilde{x}) = 0$.
Then, when all derivatives exist, the partial Jacobian of $\tilde{x}$
with respect to $\theta$ at the optimal solution $(\tilde{\lambda}, \tilde{\nu}, \tilde{x})$, namely
$J_{\theta}\tilde{x}$, can be obtained by satisfying:
$$
J_{\theta}\tilde{x} = - \left( J_{x} g(\theta, \tilde{\lambda}, \tilde{\nu}, \tilde{x}) \right)^{-1} J_{\theta} g(\theta, \tilde{x}, \tilde{\nu}, \tilde{\lambda}).
$$
Then, applying to the KKT conditions, the (partial) Jacobian with respect to $\theta$ can be defined as:
$$
J_{\theta} g(\theta, \tilde{\lambda}, \tilde{\nu}, \tilde{x}) =
\begin{bmatrix}
J_{\theta} \nabla_{x} L(\theta, \tilde{x}, \tilde{\nu}, \tilde{\lambda}) \\
\textbf{diag}(\tilde{\lambda}) J_{\theta} (G(\theta)x - h(\theta)) \\
J_{\theta} (A(\theta) \tilde{x} - b(\theta))
\end{bmatrix}
$$
After obtaining the optimal $\tilde{x}$, we can derive a closed-form gradient for $\theta$, enabling efficient backpropagation without altering the optimization path.

![](https://vision.aioz.io/f/20e1dffd5e20450a8461/?dl=1)*<center>**Figure 1**. Federated Knowledge Distillation pipeline with EMD Distance</center>*

### Training 


The distillation loss of $i$-th silo $\mathcal{L}^i_{\rm MD}$ based on student model loss is designed as:

$$
\mathcal{L}^i_{\rm MD} = \beta T^2 \sum^{\mathcal{N}(i)}_{j=1}
\left( \mathcal{L}_{c}(Q^\tau_{S_i}, Q^\tau_{T_{i\rightarrow j}}) \right) + (1-\beta)\mathcal{L}_{c}(Q_{S_i},y^i_{true})\ \  (11)
$$

where $Q_S$ is the standard softmax output of the local student; $y^i_{true}$ is the ground-truth labels, $\beta$ is a hyper-parameter for controlling the importance of each loss component; $Q^\tau_{S_i}, Q^\tau_{T_{i\rightarrow j}}$ are the softened outputs of the  $i$-th local student and the $j$-th overseas teachers using the same temperature parameter

$$
Q^\tau_k = \frac{\exp(l_k/T)}{\sum_{k} \exp(l_k/T)}
$$

where the logit $l$ is outputted from the pre-final layers for both teacher and student models. Besides, the objective function computed for each  $j$-th contributed transferrable weights is controlled by the corresponding EMD to ensure the learning convergence.


When the training in all silos is completed in each communication round, local model weights in all silos are aggregated to obtain global weights $\Theta = \sum^{N-1}_{i = 0 }\vartheta_i{\theta}_i$, which are further utilized for downstream fine-tuning.
## Next
In the next part, we will validate the effectiveness of our FedEFM on our Endovascular Intervention Dataset.

