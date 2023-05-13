---
last_modified_on: "2023-05-07"
title: Populating 3D Scenes by Learning Human Scene Interaction (Part 2).
description:  Learning Human Scene Interaction - The training pipeline.
series_position: 19
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In the previous post, we have studied about POSA Representation for HSI and the corresponding dataset for Learning Human Scene Interaction. In this post, we will discover the prediction of egocentric feature map as well as how to train POSA framework in the human-scene interaction task.

## Learning to predict egocentric feature map

**Goal:** To learn a probabilistic function from body pose and shape to the feature space of contact and semantics. Given a body, sample labelings of the vertices corresponding to likely scene contacts and their corresponding semantic label. 

<!-- Note that this function, once learned, only takes the body as input and not a scene it is a body-centric representation.

To train this we use the PROX [22] dataset, which contains bodies in 3D scenes. We also use the scene semantic annotations from the PROX-E dataset. For each body mesh $M_b$, we factor out the global translation and rotation $R_y$ and $R_z$ around the $y$ and $z$ axes. The rotation $R_x$ around the $$ axis is essential for the model to differentiate between, e.g., standing up and lying down -->

<!-- Given pose and shape parameters in a given training frame, we compute a $M_b = M(\theta, \beta, \psi)$. This gives vertices $V_b$ from which we compute the feature map that encodes whether each $V_b^i$ is in contact with the scene or not, and the semantic label of the scene contact point $P_s$. -->


## Training conditional Variational Autoencoder
Train a conditional Variational Autoencoder (cVAE), where they condition the feature map on the vertex positions, $V_b$, which are a function of the body pose and shape parameters. 

![Fig-3](
https://drive.google.com/uc?id=1DicWs5WiN2ZDSaPpzuDfulUknUscVm6G)
*<center>**Figure 1**: cVAE architecture.</center>*

## Encoder
Learn to approximate posterior $Q(z| f, V_b)$: 
- **Input:** vertice coordinate $(x_i,y_i,z_i)$, contact label $f_c$ and semantic label $f_s$.
- **Output:** Latent vector $z \sim N(0,I)$ for simpler randomness control (sampling)

The loss $\mathcal{L}_{KL}$ encourages approximate posterior $Q(z|f,V_b)$ to match a distribution $p(z)$: 
$$\mathcal{L}_{KL} = KL(Q(z|f, V_b)\ || \ p(z))$$


- **Spiral convolution** [[4]](#4): Since $f$ is defined on the vertices of the body mesh $M_b$, graph convolution can be used as building block for VAE. It acts directly on the 3D mesh and becomes efficient at encoding the ordering relationship between nodes. 

![Fig-4](
https://drive.google.com/uc?id=1vsQLzyU39MLTouB25U7ocxeII4DP1lMq)
*<center>**Figure 2**: Spiral sequence contain vertices around the red star vertex.</center>*

## Decoder
Learn to maximize the log-likelihood and reconstructs original per-vertex feature: 
- **Input:** vertice coordinate $(x_i,y_i,z_i)$, latent vector $z$ from the encoder
- **Output:** reconstructed contact label $\hat{f_c}$ and reconstructed contact semantic $\hat{f_s}$ 

The reconstruction loss $\mathcal{L}_{rec}$  encourages the reconstructed samples to resemble the input: 

$$

L_{rec}(f,\hat{f}) = \lambda_c * \sum_{i}BCE(f_c^i, \hat{f_c}^i)  + \lambda_s *\sum_{i}CCE(f_s^i, \hat{f_s}^i)

$$ 


Training optimizes the encoder and decoder parameters to minimize $\mathcal{L}_{total}$ using gradient descent:

$$

\mathcal{L}_{total} = \alpha * \mathcal{L}_{KL} + \mathcal{L}_{rec}

$$

![Fig-5](
https://drive.google.com/uc?id=18a-qqFORtIoFfD5jrs75UU2Q3k6-5nPu)
*<center>**Figure 3**: Random samples from trained cVAE.</center>*

## References
<!-- smpl-x -->
<a id="1">[1]</a> 
Georgios Pavlakos, Vasileios Choutas, Nima Ghorbani, Timo Bolkart, Ahmed A. A. Osman, Dimitrios Tzionas, and Michael J. Black. Expressive body capture: 3D hands, face, and body from a single image. In Computer Vision and Pattern Recognition (CVPR), 2019


<!-- posa -->
<a id="2">[2]</a> 
Mohamed Hassan, Partha Ghosh, Joachim Tesch, Dimitrios Tzionas, and Michael J.Black. 2021b. Populating 3D Scenes by Learning Human-Scene Interaction. In Conference on Computer Vision and Pattern Recognition (CVPR).



<!-- spiral_conv -->
<a id="3">[3]</a> 
Shunwang Gong, Lei Chen, Michael Bronstein, and Stefanos
Zafeiriou. SpiralNet++: A fast and highly efficient mesh
convolution operator. In International Conference on Computer Vision Workshops (ICCVw), 20

<!-- 
prox-e dataset -->
<a id="4">[4]</a> 
Mohamed Hassan, Vasileios Choutas, Dimitrios Tzionas, and Michael J. Black. Resolving 3D human pose ambiguities with 3D scene constrains. In International Conference on Computer Vision (ICCV), 2019.