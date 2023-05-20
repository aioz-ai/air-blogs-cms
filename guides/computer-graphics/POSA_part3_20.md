---
last_modified_on: "2023-05-14"
title: Populating 3D Scenes by Learning Human Scene Interaction (Part 3).
description:  Learning Human Scene Interaction - The inference pipeline and effectiveness analysis.
series_position: 20
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In this post, we will discover the inference phase of POSA framework in the human-scene interaction task and how to evaluate its effectiveness.

## Inference phase

**Putting people into scenes**: Given a scene $M_s$, semantic labels of objects present, and a body mesh $M_b$, POSA finds where in $M_s$ this given pose is likely to happen:

First, given the posed body, use the decoder of cVAE to generate a feature map by sampling $P(f_{Gen}|z, Vb)$ with $f_{Gen} = [\hat{f_c}, \hat{f_s}]$ .

Second, optimize the objective function: 
$$

E(\tau, \theta_0, \theta) = \mathcal{L}_{\text{afford}} +  \mathcal{L}_{\text{pen}} + \mathcal{L}_{\text{reg}}

$$

where $\tau$ is the body translation, $θ_0$ is the global body orientation, $\theta$ is the body pose and: 

- The afforance loss $\mathcal{L}_{\text{afford}}$ finds position in the scene where given pose is likely to happen.

$$

\mathcal{L}_{\text{afford}} = \lambda_1 ||f_{Gen_c} . f_d||_2^2 + \lambda_2 \sum_i CCE(f_{Gen_s}^i, f_s^i)

$$

- The penetration penalty $\mathcal{L}_{\text{pen}}$ discourages the body from penetrating the scene.

$$

\mathcal{L}_{\text{pen}} = \lambda_{\text{pen}} \sum_{f_d^i < 0} (f_d^i)^2 

$$

- The regularizer $\mathcal{L}_{\text{reg}}$ that encourages the estimated pose to remain close to the initial pose $\theta_{\text{init}}$ of $M_b$.

$$\mathcal{L}_{\text{reg}} = \lambda_{\text{reg}} ||\theta - \theta_{\text{init}}||_2^2$$

![Fig-6](
https://drive.google.com/uc?id=1d--C7Y_kMTi3wadFEl7i8mM9PeraORNq)
*<center>**Figure 1**: Putting realistic people in scenes.</center>*

**Locating Clothed Bodies**:

![Fig-7](
https://drive.google.com/uc?id=1w13c-b45aHS-wLP3RdfIgz6Elb1dO16k)
*<center>**Figure 2**: Locate clothed bodies in scenes.</center>*
Using SMPL-X fits to clothed meshes from the AGORA dataset. The optimization objective now is defined: 
$$

E(\tau, \theta_0) = \mathcal{L}_{\text{afford}} +  \mathcal{L}_{\text{pen}} 

$$

**Monocular Pose Estimation with HSI**: Fit SMPL-X to RGB image features such that the contacts are consistent with the 3D scene and its semantics, in order to minimize an objective function of multiple terms: the re-projection error of 2D joints, priors and physical constraints on the body: 

$$

E_{\text{SMPLify-X}}(\beta, \theta, \psi, \tau ) = E_J + \lambda_{\theta}E_{\theta} + \lambda_{\alpha}E_{\alpha} + \lambda_{\beta}E_{\beta} +\lambda_{\mathcal{P}}E_{\mathcal{P}}

$$

To get a pose matching the image observations and roughly obeying scene constraints, sample features from $P(f_{Gen}|z, V_b)$ from body pose, then minimize

$$

E(\beta, \theta, \psi, \tau, M_s ) = E_{\text{SMPLify-X}} + ||f_{Gen_c} \cdot f_d|| + \mathcal{L}_{\text{pen}} 

$$

## Evaluation

**Comparison to PROX ground truth**: They take 4 real scenes from the PROX test set, 100 SMPL-X bodies from the AGORA dataset, corresponding to 100 different 3D scans from Renderpeople, and take each of these bodies and sample one feature map for each using cVAE. Then, automatically optimize the placement of each sample in all the scenes, one body per scene. The pose is changed slightly to fit the scene for unclothed bodies and kept fixed for clothed bodies.

For each variant, the optimization results in 400 unique body-scene pairs. They render each 3D human-scene interaction from 2 views so that subjects are able to get a good sense of the 3D relationships from the images.

![Fig-8](
https://drive.google.com/uc?id=1Hgn4a0UiS-DZq0ZxVagAdAZozPGvz1sN)
*<center>**Figure 3**: Comparison to PROX ground truth. Subjects
are shown pairs of a generated 3D human-scene interaction
and PROX ground truth.</center>*


**Comparison between POSA and PLACE**  directly compare POSA and PLACE using the above protocol. Adding semantics to POSA improves realism.




![Fig-9](
https://drive.google.com/uc?id=1i7oZkz_l_0Ao6OtLynFnMKow137QencW)
*<center>**Figure 4**: POSA compared to PLACE for 3D human-scene
interaction generation.</center>*


**Physical Plausibility:** They take 1200 bodies from the AGORA  dataset and place all of them in each of the 4 test scenes of PROX. They compute the following scores: 
- Non-collision score for each body mesh $M_b$, which is the ratio of body mesh vertices with positive scene signed distance field (SDF)  values divided by the total number of SMPL-X vertices. 
- The contact score for each $M_b$, which is 1 if at least one vertex of $M_b$ has a non-positive value. 

Experiment shows that POSA and PLACE are comparable under these metrics.

![Fig-10](
https://drive.google.com/uc?id=1bLfGRojByPTeOJczqZkWlKyYOTXEssyx)
*<center>**Figure 5**:  Evaluation of the physical plausibility metric.</center>*

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