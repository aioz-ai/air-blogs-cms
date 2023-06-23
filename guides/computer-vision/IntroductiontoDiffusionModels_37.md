---
last_modified_on: "2023-04-01"
title: Introduction to Diffusion Models (Part 2)
description: A kind introduction to Diffusion Models and its underlying theory.
series_position: 37
author_github: https://github.com/aioz-ai
tags: ["type: theory", "level: advanced"]
---
In the previous [post](https://ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_31/), we have introduced the fundamental mechanism behind diffusion models: the forward process and the reverse process. Now, we will go into more detail about how to train and sample from the diffusion-based denoising network.

## Training diffusion models
Interestingly, we can see the forward and backward process of diffusion closely resemble that of the well-known Variational Autoencoder (VAE). Recall that in VAE, we leverage an Encoder to map the data into a latent (Gaussian) distribution, and then use a Decoder to reconstruct the original data from the latent. However, in diffusion model, we only learn the reverse process (the decoder) while the forward process has no learnable parameters. In particular, we can use similar derivations of the variational lower bound (VLB) of the VAE to optimize the likelihood:

$$
\log p_\theta({x}_0) 
\geq  \log p_\theta({x}_0) - D_\text{KL}(q({x}_{1:T}\vert{x}_0) \| p_\theta({x}_{1:T}\vert{x}_0) ) \tag{7}
$$

Since the $D_\text{KL}$ is always non-negative, to maximize the log-likelihood of the data $\log p_\theta({x}_0)$, we aim to maximize the VLB (the RHS) of the above equation. This is equivalent to minimizing the negation of the RHS, which will later lead to our $L_\text{vlb}$ loss:


$$\\
-RHS = -\log p_\theta({x}_0) + D_\text{KL}(q({x}_{1:T}\vert{x}_0) \| p_\theta({x}_{1:T}\vert{x}_0)\\
= -\log p_\theta({x}_0) + \mathbb{E}_{{x}_{1:T}\sim q({x}_{1:T} \vert {x}_0)} \Big[ \log\frac{q({x}_{1:T}\vert{x}_0)}{p_\theta({x}_{0:T}) / p_\theta({x}_0)} \Big] \\
= -\log p_\theta({x}_0) + \mathbb{E}_q \Big[ \log\frac{q({x}_{1:T}\vert{x}_0)}{p_\theta({x}_{0:T})} + \log p_\theta({x}_0) \Big] \\
= -\log p_\theta({x}_0) + \mathbb{E}_q \Big[ \log \frac{q({x}_{1:T}\vert{x}_0)}{p_\theta({x}_{0:T})} \Big] + \mathbb{E}_q [\log p_\theta({x}_0)]
$$
Since that $p_\theta(x_0)$ does not contain any learnable parameters, we can define our training loss $L_\text{vlb}$ as:
$$
L_\text{vlb} = \mathbb{E}_q \Big[ \log \frac{q({x}_{1:T}\vert{x}_0)}{p_\theta({x}_{0:T})} \Big] \tag{8}
$$

The training objective can be further decomposed as the combination of several KL terms with respect to the time $t$:
$$
L_\text{vlb} 
= \mathbb{E}_{q({x}_{0:T})} \Big[ \log\frac{q({x}_{1:T}\vert{x}_0)}{p_\theta({x}_{0:T})} \Big] \\
= \mathbb{E}_q \Big[ \log\frac{\prod_{t=1}^T q({x}_t\vert{x}_{t-1})}{ p_\theta({x}_T) \prod_{t=1}^T p_\theta({x}_{t-1} \vert{x}_t) } \Big] \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \sum_{t=1}^T \log \frac{q({x}_t\vert{x}_{t-1})}{p_\theta({x}_{t-1} \vert{x}_t)} \Big] \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \color{green}{\sum_{t=2}^T \log \frac{q({x}_t\vert{x}_{t-1})}{p_\theta({x}_{t-1} \vert{x}_t)}} + \log\frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1)} \Big], \scriptsize(q(x_t | x_{t-1}) = q(x_t | x_{t-1}, x_0)) \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \color{green}{\sum_{t=2}^T \log \Big( \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)}\cdot \frac{q({x}_t \vert {x}_0)}{q({x}_{t-1}\vert{x}_0)} \Big)} + \log \frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1)} \Big], \scriptsize(\text{Bayes rule}) \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \color{green}{\sum_{t=2}^T \log \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)} + \sum_{t=2}^T \log \frac{q({x}_t \vert {x}_0)}{q({x}_{t-1} \vert {x}_0)}} + \log\frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1)} \Big] \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \color{green}{\sum_{t=2}^T \log \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)} + \log \prod_{t=2}^T\frac{q({x}_t \vert {x}_0)}{q({x}_{t-1} \vert {x}_0)}} + \log\frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1)} \Big] \\
= \mathbb{E}_q \Big[ -\log p_\theta({x}_T) + \color{green}{\sum_{t=2}^T \log \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)} + \log\frac{q({x}_T \vert {x}_0)}{q({x}_1 \vert {x}_0)}}+ \log \frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1)} \Big]\\
= \mathbb{E}_q \Big[ -\log \frac{p_\theta({x}_T)}{\color{green}{q({x}_T \vert {x}_0)}} + \color{green}{\sum_{t=2}^T \log \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)}}+ \log \frac{q({x}_1 \vert {x}_0)}{p_\theta({x}_0 \vert {x}_1) \color{green}{q({x}_1 \vert {x}_0)}} \Big]\\
= \mathbb{E}_q \Big[ \log\frac{q({x}_T \vert {x}_0)}{p_\theta({x}_T)} + \sum_{t=2}^T \log \frac{q({x}_{t-1} \vert {x}_t, {x}_0)}{p_\theta({x}_{t-1} \vert{x}_t)} - \log p_\theta({x}_0 \vert {x}_1) \Big] \\
= \mathbb{E}_q [\underbrace{D_\text{KL}(q({x}_T \vert {x}_0) \parallel p_\theta({x}_T))}_{L_T} + \sum_{t=2}^T \underbrace{D_\text{KL}(q({x}_{t-1} \vert {x}_t, {x}_0) \parallel p_\theta({x}_{t-1} \vert{x}_t))}_{L_{t-1}} \underbrace{- \log p_\theta({x}_0 \vert {x}_1)}_{L_0} ]
$$

In short, the $L_\text{vlb}$ can be written as:
$$
L_\text{vlb} = L_T + L_{T-1} + ... + L_{1} + L_0
$$
where $L_t = D_\text{KL}(q({x}_{t} \vert {x}_{t+1}, {x}_0) \parallel p_\theta({x}_{t} \vert{x}_{t+1})), \forall 1 \leq t \leq T-1$. We can see that these KL terms (except $L_0$) are between two Gaussian and can be computed in closed form, whereas the $L_T$ term can be ignored during training since it has no learnable parameters.

Recall that we need to train a neural network to approximate the conditional distributions in the reverse process $p_\theta({x}_{t-1} \vert {x}_t) = \mathcal{N}({x}_{t-1}; {\mu}_\theta({x}_t, t), {\Sigma}_\theta({x}_t, t))$. Moreover, the posterior mean $\mu_t$ (see Equation 5 and 6 in this [post](https://ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_31/)) of the reverse distribution $q({x}_{t} \vert {x}_{t+1}, {x}_0)$ is:
$$
\color{orange}{\tilde{\mu}_t(x_t,x_0)}  = \frac{\sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})}{1 - \bar{\alpha}_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1 - \bar{\alpha}_t} x_0
$$
Considering the property of Equation 2, i.e., $x_t = \sqrt{\bar{\alpha}_t} x_0  + \sqrt{1-\bar{\alpha}_t}\epsilon_t$, we can re-expressed $\mu_t$ by replacing $x_0$ as:
$$
\color{orange}{\tilde{{\mu}}_t}
= \frac{\sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})}{1 - \bar{\alpha}_t} {x}_t + \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1 - \bar{\alpha}_t} \frac{1}{\sqrt{\bar{\alpha}_t}}({x}_t - \sqrt{1 - \bar{\alpha}_t}{\epsilon}_t) \\
= \frac{1}{\sqrt{\alpha_t}} \Big( {x}_t - \frac{1 - \alpha_t}{\sqrt{1 - \bar{\alpha}_t}} {\epsilon}_t \Big) \tag{9}
$$

Our ultimate goal is to train the network $\theta$ to approximate the mean $\color{blue}{\mu_\theta({x}_t, t)}$, and the loss term $L_t$ measured the KL Divergence between two Gaussian. Furthermore, Ho et. al [1] suggested predicting the noise $\epsilon_\theta(x_t,t)$ using the re-parameterization as in Equation 9, and keeping the variance $\Sigma$ fixed to the schedule. Therefore, we can calculate the loss as follows:
$$
L_t 
= \mathbb{E}_{{x}_0, {\epsilon}} \Big[\frac{1}{2 \| \Sigma_t \|^2_2} \| \color{orange}{\tilde{{\mu}}_t} - \color{blue}{{\mu}_\theta({x}_t, t)} \|^2 \Big] \\
= \mathbb{E}_{{x}_0, {\epsilon}} \Big[\frac{1}{2  \|\Sigma_t \|^2_2} \| \color{orange}{\frac{1}{\sqrt{\alpha_t}} \Big( {x}_t - \frac{1 - \alpha_t}{\sqrt{1 - \bar{\alpha}_t}} {\epsilon}_t \Big)} - \color{blue}{\frac{1}{\sqrt{\alpha_t}} \Big( {x}_t - \frac{1 - \alpha_t}{\sqrt{1 - \bar{\alpha}_t}} {{\epsilon}}_\theta({x}_t, t) \Big)} \|^2 \Big] \\
= \mathbb{E}_{{x}_0, {\epsilon}} \Big[\frac{ (1 - \alpha_t)^2 }{2 \alpha_t (1 - \bar{\alpha}_t) \| \Sigma_t \|^2_2} \| \color{orange}{\epsilon_t} - \color{blue}{\epsilon_\theta({x}_t, t)} \|^2 \Big]
$$
[1] also suggested the simplification of this loss (i.e., removing the weighting term) as they found the training results are better, which leads to the "simple" objective:
$$
L_\text{simple} = \mathbb{E}_{{x}_0, {\epsilon}}  \Big[ \| \color{orange}{\epsilon_t} - \color{blue}{\epsilon_\theta({x}_t, t)} \|^2 \Big] \tag{10}
$$

In summary, the training process iteration is as follows:
* **Repeat**
    * Sample data $x_0 \sim q(x_0)$
    * Sample $t\sim\text{Uniform}[1,T]$ 
    * Sample noise $\epsilon \sim \mathcal{N}(0,I)$
    * Compute loss $L_\text{simple} = \mathbb{E}_{{x}_0, {\epsilon}}  \Big[ \| \epsilon_t - \epsilon_\theta({x}_t, t) \|^2 \Big]$
    * Backprop and update network parameters $\theta$
* **Until convergence**

## Sampling diffusion models
To obtain a sample from the original data distribution, we start by sampling from the noise distribution $q(x_T)$ and then gradually remove the noise until we reach $x_0$, following the reverse process. At each step, we sample from the approximated reverse distribution:
$$
p_\theta(x_{t-1}|x_{t}) = \mathcal{N}(x_t; \mu_\theta(x_t,t), \Sigma_t )= \mathcal{N}(x_t; \frac{1}{\sqrt{\alpha_t}} \big( {x}_t - \frac{1 - \alpha_t}{\sqrt{1 - \bar{\alpha}_t}} {\epsilon}_\theta(x_t,t) \big) , \Sigma_t)
$$

The sampling process can be summarized as follows:

1. **Firstly** sample $x_T \sim \mathcal{N}(0,I)$
2. **For** $t = T \rightarrow 1$ **do** sample from posterior distribution:
    * $z \sim \mathcal{N}(0,I)$ if $t>1$, else $z=0$
    * $x_{t-1} = \mu_\theta(x_t,t) + \Sigma_tz$, (reparameterization trick)
4. **Return** $x_0$


In the original Denoising Diffusion Probabilistic Model (DDPM), the sampling process is usually quite slow to obtain a sample. This is because we need to follow the whole chain of the reverse diffusion process from $T$ to $0$, where the number of steps is mostly up to $T=1000$. For example, it takes around 20 hours to sample 50k images of size 32 × 32 from a DDPM, but less than a minute to do so from a GAN model [2]. Many recent works have proposed several strategies to overcome this limitation to speed up the sampling process [2],  [3], [4]. They can generate samples within only a few steps (e.g., 50 steps) while the sample quality can be as high as the full sampling process. 


## References
[1] Ho J, Jain A, Abbeel P. Denoising diffusion probabilistic models. In NIPS 2020.
 
[2] Song J, Meng C, Ermon S. Denoising diffusion implicit models. in ICLR 2021.

[3] Liu L, Ren Y, Lin Z, Zhao Z. Pseudo numerical methods for diffusion models on manifolds. ICLR 2022.

[4] Salimans T, Ho J. Progressive distillation for fast sampling of diffusion models. ICLR 2022.