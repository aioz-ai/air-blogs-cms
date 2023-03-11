---
last_modified_on: "2022-26-12"
title: Introduction to Diffusion Models
description: A kind introduction to Diffusion Models and its related information.
series_position: 31
author_github: https://github.com/aioz-ai
tags: ["type: theory", "level: advanced"]
---

## Introduction
Recently, diffusion models have demonstrated remarkable performance on several generative tasks, ranging from unconditional or text-guided image generation, audio synthesis, natural language generation, human motion synthesis to 3D pointcloud generation, most of which were previously dominated by GAN-based models. Research also has shown that diffusion models are considerably better than GANs in terms of generation quality while also being able to control the results [1]. In this blog, we will go through the mathematics and intuition behind this compelling generative model.


![](https://vision.aioz.io/f/334f5423de224114963e/?dl=1)*<center>**Figure 1**. Overview of Diffusion Model.</center>* 



## Mechanism behind Diffusion Model
The underlying principle of diffusion model is based on a popular idea of non-equilibrium statistical thermodynamics: we use a Markov chain to gradually convert one distribution into another [2]. Intuitively, it is called "diffusion" because in this model, data points in our original data space are gradually diffused from one position into another until reaching a state that is just a set of noise. Unlike the common VAE or GAN models where the data is typically compressed into a low dimensional latent space, diffusion models are learned with a fixed procedure and with high dimensional latent variable (same as the original data, see Figure 1). Now let's define the forward diffusion process.

### Forward diffusion process

![](https://vision.aioz.io/f/c1abc938af5c4efc938f/?dl=1)*<center>**Figure 2**. Example of forward diffusion</center>* 


Given a sample from our real data distribution $x_0 \sim q(x_0)$, we define the forward diffusion process as a Markov chain, which gradually adds random Gaussian noise (with diagonal covariance) to the data under a pre-defined schedule up to $t=1,2,...,T$ steps. Specifically, the distribution of the noised sample at step $t$ only depends on the previous step $t-1$:
$$
\color{red}{q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{1-\beta_t} x_t, \beta_tI)} \tag{1}
$$
where $\beta_t$ is the noise schedule controlling the step size or how slow the noises are added in the process. Typically, they are between $(0,1)$ and should hold the followings:
$$
0 < \beta_1 < \beta_2 < ... < \beta_T < 1
$$

An interesting property of the forward process, which makes it a very powerful generative model, is that when the noise step size is small enough and $T\rightarrow \infty$, the distribution at the final step $x_T$ becomes a standard Gaussian distribution $\mathcal{N}(0,I)$. In other words, we can easily sample from this (*pure*) noise distribution and then follow a reverse process to reconstruct the real data sample, which we will discuss later. 


Another notable property of this process is that we can directly sample $x_t$ at any time step $t$ without going through the whole chain. Specifically, let $\alpha_t = 1 -\beta_t$, from **Eq. 1**, we can derive the sample using reparameterization trick $(x\sim\mathcal{N}(\mu,\sigma^2) \Leftrightarrow x = \mu + \sigma\epsilon )$ as follow:
$$
x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1 - \alpha_t}\epsilon_{t-1} \\
    = \sqrt{\alpha_t} \left(\sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1 - \alpha_{t-1}}\epsilon_{t-2} \right) +  \sqrt{1 - \alpha_t}\epsilon_{t-1}  \\
    = \sqrt{\alpha_t \alpha_{t-1}}x_{t-2}  + \left[\sqrt{\alpha_t(1 - \alpha_{t-1})}\epsilon_{t-2}  + \sqrt{1 - \alpha_t}\epsilon_{t-1}\right]
$$
where $\epsilon \sim \mathcal{N}(0,I)$. Note that when we add two Gaussian with variances $\sigma_1^2$ and $\sigma_2^2$ , the merged distribution is also a Gaussian and should have variance  $\sigma_1^2 + \sigma_2^2$. Therefore, we obtain the merged variance (the right sum $[...]$ of the above equation) as:
$$
\alpha_t(1 - \alpha_{t-1}) + 1-\alpha_t = 1-\alpha_t\alpha_{t-1}
$$
Thus $x_t$ can be written as:
$$
x_t = \sqrt{\alpha_t \alpha_{t-1}} x_{t-2} + \sqrt{1 - \alpha_t \alpha_{t-1}} \bar{\epsilon}_{t-2}  = \dots  \\
x_t = \sqrt{\alpha_t \alpha_{t-1}\dots\alpha_{1}} x_0 + \sqrt{1 - \alpha_t \alpha_{t-1}\dots\alpha_{1}} \epsilon \\
x_t = \sqrt{\bar{\alpha}_t} x_0  + \sqrt{1-\bar{\alpha}_t}\epsilon \tag{2}
$$
where  $\bar{\alpha}_t = \prod_{i=1}^t \alpha_i$. We can also re-write the forward diffusion as a distribution depended on $x_0$ as:
$$
\color{red}{q(x_t \vert x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t} x_0, (1 - \bar{\alpha}_t)I)} \tag{3}
$$


This nice property can make the training much more efficient since we only need to compute the loss at some random timesteps $t$ instead of a whole process, which we will see later. 




### Reverse diffusion process
We have seen that the forward diffusion essentially pushes a sample off the data manifold and slowly transforms it into noise. The main goal of the reverse process is to learn a trajectory back to the manifold and thus reconstruct the sample in the original data distribution from the noise.

Feller et. al [3] demonstrated that for Gaussian diffusion process with small step size $\beta$, the reverse of the diffusion process has the identical functional form as the forward process. This means that  $q(x_t|x_{t-1})$ is a Gaussian  distribution, and if $\beta_t$ is small, then $q(x_{t-1}|x_t)$ will also be a Gaussian distribution.

Unfortunately, estimating reverse distribution $q(x_{t-1}|x_t)$ is very difficult because in early steps of the reverse process ($t$ is near $T$), our sample is just a random noise with high variance. In other words, there exist too many trajectories for us to arrive at our original data $x_0$ from that noise, which we may need to use the entire dataset to integrate. And ultimately, our final goal is to learn a model to approximate these reverse distributions in order to build the generative reverse denoising process, specifically:
$$
p_\theta(x_{0:T}) = p(x_T) \prod^T_{t=1} p_\theta(x_{t-1} | x_t), \quad
p_\theta(x_{t-1} | x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(\mathbf{x}_t, t), \Sigma_\theta(\mathbf{x}_t, t))
$$
where $\mu_\theta(\mathbf{x}_t, t)$ is the mean predicted by the model (a neural network with parameters $\theta$), and $\Sigma_\theta(\mathbf{x}_t, t)$ is the variance that is typically kept fixed for efficient learning.

Fortunately, by conditioning on $x_0$, the posterior of the reverse process is tractable and becomes a Gaussian distribution [4]. Our target now is to find the posterior mean $\tilde{\mu}_t(x_t,x_0)$ and the variance $\tilde{\beta}_t$. Following Bayes rule, we can write:
$$
q(x_{t-1} | x_t, x_0) = \frac{ q(x_t | x_{t-1}, x_0) q(x_{t-1} | x_0) }{ q(x_t | x_0) }
$$
where
$$
\begin{cases}
q(x_t | x_{t-1}, x_0) = q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{\alpha_t} x_{t-1}, \beta_t I), \\
q(x_{t-1} | x_0) = \mathcal{N}(x_{t-1}; \sqrt{\bar{\alpha}_{t-1}}x_0, (1-\bar{\alpha}_{t-1})I), \\
q(x_t | x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t} x_0, (1 - \bar{\alpha}_t)I)
\end{cases}
$$

$q(x_t | x_{t-1}, x_0) = q(x_t | x_{t-1})$ because the process is a Markov chain, where the future state ($x_t$) is conditionally independent with all previous states ($x_0,\dots,x_{t-2}$) given the present state ($x_{t-1}$). We also leverage the fact that both of the distributions on the right hand side are Gaussian, which means we can apply the density function ($\mathcal{N}(x;\mu,\sigma) \propto \exp(-\frac{(x-\mu)^2}{2\sigma^2})$) as follow:


$$
q(x_{t-1} | x_t, x_0)  \propto\exp \left[ -\frac{(x_t - \sqrt{\alpha_t} x_{t-1})^2}{2\beta_t} -  \frac{(x_{t-1} - \sqrt{\bar{\alpha}_{t-1}} x_0)^2}{2(1-\bar{\alpha}_{t-1})} +  \frac{(x_t - \sqrt{\bar{\alpha}_t} x_0)^2}{2(1-\bar{\alpha}_t)} \big)\right] \\
= \exp \left[ -\frac{1}{2} \left(  
\frac{x_t^2 - 2\sqrt{\alpha_t} x_t x_{t-1} + \alpha_t x_{t-1}^2 }{\beta_t} +
\frac{ x_{t-1}^2 - 2 \sqrt{\bar{\alpha}_{t-1}} x_0 x_{t-1} + \bar{\alpha}_{t-1} x_0^2}{1-\bar{\alpha}_{t-1}} - 
\frac{(x_t - \sqrt{\bar{\alpha}_t} x_0)^2}{1-\bar{\alpha}_t} 
\right) \right]\\
= \exp \left[ -\frac{1}{2} \left(  
\frac{\alpha_t x^2_{t-1}}{\beta_t} + 
\frac{x^2_{t-1}}{1-\bar{\alpha}_{t-1}} -
\frac{2\sqrt{\alpha_t} x_t x_{t-1} }{\beta_t} -
\frac{2 \sqrt{\bar{\alpha}_{t-1}} x_0 x_{t-1}}{1-\bar{\alpha}_{t-1}}
\right)  + O(x_0,x_t) \right]
$$

where $O(x_0,x_t)$ is a function only depending on $x_0$ and $x_t$ but not $x_{t-1}$, so it can be effectively ignored. Therefore we have:
$$
q(x_{t-1} | x_t, x_0)  \propto 
\exp\left[ -\frac{1}{2} \left( 
(\frac{\alpha_t}{\beta_t} + \frac{1}{1 - \bar{\alpha}_{t-1} } )\color{green}{x^2_{t-1}} -
2 (\frac{\sqrt{\alpha_t}}{\beta_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1 - \bar{\alpha}_{t-1}} x_0)\color{orange}{x_{t-1}}
\right) \right]
$$

Comparing with the Gaussian density function $\exp(-\frac{(x-\mu)^2}{2\sigma^2}) = \exp(-\frac{1}{2} (\frac{\color{green}{x^2}}{\sigma^2} -\frac{2\mu \color{orange}{x}}{\sigma^2} + \frac{\mu^2}{\sigma^2}))$, we can see some similaries. Recall that $\bar{\alpha}_t = \prod_{i=1}^t\alpha_i = \alpha_t\bar{\alpha}_{t-1}$ and $\beta_t + \alpha_t = 1$. Matching the variance term, we have the posterior variance:
$$
\color{green}{\tilde{\beta}_t}  = \frac{1}{\frac{\alpha_t}{\beta_t} + \frac{1}{1 - \bar{\alpha}_{t-1}}}
= \frac{1}{\frac{\alpha_t - \bar{\alpha}_t + \beta_t}{\beta_t(1 - \bar{\alpha}_{t-1})}}
= {\frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t} \cdot \beta_t} \tag{4}
$$

We also obtain the mean:
$$
\frac{\color{orange}{\tilde{\mu}_t(x_t,x_0)}}{\color{green}{\tilde{\beta}_t}} = \frac{\sqrt{\alpha_t}}{\beta_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1} }}{1 - \bar{\alpha}_{t-1}} x_0 \\
\rightarrow \tilde{\mu}_t(x_t,x_0)
= (\frac{\sqrt{\alpha_t}}{\beta_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1} }}{1 - \bar{\alpha}_{t-1}} x_0)\beta_t\\
= (\frac{\sqrt{\alpha_t}}{\beta_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1} }}{1 - \bar{\alpha}_{t-1}} x_0) {\frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t} \cdot \beta_t} 
$$

$$
\rightarrow \color{orange}{\tilde{\mu}_t(x_t,x_0)}  = \frac{\sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})}{1 - \bar{\alpha}_t} x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1 - \bar{\alpha}_t} x_0 \tag{5}
$$

Now we obtain the final formula for the reverse distribution:
$$
\color{blue}{q(x_{t-1} | x_t, x_0) = \mathcal{N}(x_{t-1}; \tilde{\mu}_t(x_t, x_0), \tilde{\beta}_t I)} \tag{6}
$$


So far we have provided the core concepts behind the two essential constituents of diffusion model: forward process and reverse process. In the next post, we will go into detail about the training loss as well as the sampling algorithm of diffusion model.

## References
[1] Dhariwal P, Nichol A. Diffusion models beat gans on image synthesis. In NIPS 2021
 
[2] Sohl-Dickstein J, Weiss E, Maheswaranathan N, Ganguli S. Deep unsupervised learning using nonequilibrium thermodynamics. In ICML 2015.

[3] Feller W. On the theory of stochastic processes, with particular reference to applications. In Selected Papers I 2015 (pp. 769-798). Springer, Cham.

[4] Ho J, Jain A, Abbeel P. Denoising diffusion probabilistic models. In NIPS 2020