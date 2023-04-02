---
last_modified_on: "2023-05-04"
id: GDANCE
title: Music-Driven Group Choreography (Part 1)
description: AIOZ-GDANCE a novel Music-Driven Group Choreography Dataset
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---
Dancing is an important part of human culture and remains one of the most expressive physical art and communication forms. With the rapid development of digital social media platforms, creating dancing videos has gained significant attention from social communities. As a result, millions of dancing videos are created and watched daily on online platforms. Recently, studies of how to create natural dancing motion from music have attracted great attention in the research community. 

![](https://vision.aioz.io/f/1e065962a9b747b3a856/?dl=1)*<center>**Figure 1**. We demonstrate the AIOZ-GDANCE dataset with in-the-wild videos, music audio, and 3D group dance motion. </center>* 

Nevertheless, generating dance motion for a group of dancers remains an open problem and have not been well-investigated by the community yet. Motivated by these shortcomings and to foster research on group choreography, we establish AIOZ-GDANCE, a new largescale in-the-wild dataset for music-driven group dance generation. Unlike existing datasets that only support single dance, our new dataset contains group dance videos as shown in Figure 1, hence supporting the study of group choreography. On the basis of the new dataset, we propose the first strong baseline for group dance generation that can jointly generate multiple dancing motions expressively and coherently.

## Dataset Construction
![](https://vision.aioz.io/f/7212f7db1e394f8a92cd/?dl=1)*<center>**Figure 2**. The pipeline of making our AIOZ-GDANCE dataset. </center>* 
 
In this section, we will elaborate and describe the process to build our dataset from a large variety of videos available on the internet. Because our main goal is to develop a large-scale dataset with in-the-wild videos, setting up a MoCap system as in many classical approaches is not feasible. However, manually creating 3D groundtruth for millions of frames from dancing videos is also an extremely costly and tedious job. To that end, we propose a semi-automatic labeling method with humans in the loop to obtain the 3D ground truth for our dataset. The process to construct the data includes the five following key steps:

1. Video collection
2. Human Tracking
4. Human Pose Estimation
5. Local Fitting for Invidual Motions
6. Global Scene Optimization


### Data Collection and Preprocessing



![](https://vision.aioz.io/f/2f8eece31fa2477c8ad3/?dl=1)*<center>**Figure 3**. Human Tracking. </center>* 


**Video Collection.** We collect the in-the-wild, public domain group dancing videos along with the music from Youtube, Tiktok, and Facebook. All group dance videos are processed at 1920 × 1080 resolution and 30FPS.

**Human Tracking.** We perform tracking for all humans in the videos using the state-of-the-art multi-object tracker [1] to obtain the tracking bounding boxes. Note that although the tracker can produce reasonable results, there are failure cases in some frames. Therefore, we manually correct the bounding box of the incorrect cases. This tracking correction is crucial since we want the trajectory of each person to be accurately tracked in order to reconstruct their motion in latter stages.

**Pose Estimation.** Given the bounding boxes of each person in the video, we leverage a state-of-the-art 2D pose estimation method [2] to generate the initial 2D poses for each person. In practice, there exist some inaccurately detected keypoints due to motion blur and partial occlusion. We manually fix the incorrect cases to obtain the 2D keypoints of each human bounding box.

### Local Mesh Fitting

![](https://vision.aioz.io/f/58bb982c59e74a379d88/?dl=1)*<center> **Figure 4**. Local Mesh Fitting. </center>* 

To construct 3D group dance motion, we first reconstruct the full body motion for each dancer by fitting the 3D mesh. We then jointly optimize all dancer motions to construct the globally-coherent group motion. Finally, we post-process and remove wrong cases from the optimization results.

We use SMPL model [3] to represent the 3D human. The SMPL model is a differentiable function that maps the pose parameters $\mathbf{\theta}$, the shape parameters $\mathbf{\beta}$, and the root translation $\mathbf{\tau}$ into a set of 3D human body mesh vertices $\mathbf{V}\in \mathbb{R}^{6890\times3}$ and 3D joints $\mathbf{X}\in \mathbb{R}^{J\times3}$, where $J$ is the number of body joints. 

Our optimizing motion variables for each individual dancer consist of a sequence of SMPL joint angles $\{\mathbf{\theta}_t\}_{t=1}^T$, a sequence of the root translation $\{\mathbf{\tau}_t\}_{t=1}^T$, and a single SMPL shape parameter $\mathbf{\beta}$. We fit the sequence of SMPL motion variables to the tracked 2D keypoints by extending SMPLify-X framework [4] across the whole video sequence:

$$
E_{\rm local} = E_{\rm J} + \lambda_{\theta}E_{\theta} + \lambda_{\beta} E_{\beta} + \lambda_{\rm S}E_{\rm S} + \lambda_{\rm F}E_{\rm F}
$$
where:
* $E_{\rm J}$ is the 2D reprojection term between the  2D keypoints and the 2D projection of the corresponding 3D poses. 
* $E_{\theta}$ is the pose prior term from the latent space of the VPoser model [4] to encourage plausible human pose.
*  $E_{\beta}$ is the shape prior term to regularize the body shape towards the mean shape of the SMPL body model.
*  $E_{\rm S} = \sum_{t=1}^{T-1}\Vert \mathbf{\theta}_{t+1} - \mathbf{\theta}_{t} \Vert^2 + \sum_{j=1}^J\sum_{t=1}^{T-1}\Vert \mathbf{X}_{j,t+1} - \mathbf{X}_{j,t} \Vert^2$ is the smoothness term to encourage the temporal smoothness of the motion. 
*  $E_{\rm F} =  \sum_{t=1}^{T-1} \sum_{j \in \mathcal{F}} c_{j,t}\Vert \mathbf{X}_{j,t+1} - \mathbf{X}_{j,t} \Vert^2$ is to ensure feet joints to stay stationary when in contact (zero velocity). Where $\mathcal{F}$ is the set of feet joint indexes, $c_{j,t}$ is the feet contact of joint $j$ at time $t$.

### Global Optimization
![](https://vision.aioz.io/f/89f4586e3f1b42d5bd94/?dl=1)*<center> **Figure 5**. Global Optimization. </center>* 

Given the 3D motion sequence of each dancer $p$: $\{\mathbf{\theta}^p_t, \mathbf{\tau}^p_t\}$, we further resolve the motion trajectory problems in group dance by solving the following objective: 

$$
E_{\rm global} = E_{\rm J} + \lambda_{\rm pen}E_{\rm pen} + \lambda_{\rm reg}\sum_{p}E_{\rm reg}(p) +  \lambda_{\rm dep}\sum_{p,p',t}E_{\rm dep}(p,p',t) + \lambda_{\rm gc}\sum_{p}E_{\rm gc}(p)
$$

$E_{\rm pen}$ is the Signed Distance Function penetration term to prevent the overlapping of reconstructed motions between dancers. 

${E_{\rm reg}(p) =\sum_{t=1}^T\Vert \mathbf{\theta}^p_t - \hat{\mathbf{\theta}}^p_t\Vert^2}$ is the regularization term that prevents the motion from deviating too much from the prior optimized individual motion $\{\hat{\mathbf{\theta}}^p_t\}$ obtained by optimizing the local mesh for dancer $p$. 

In practice, we find that the relative depth ordering of dancers in the scene can be inconsistent due to the ambiguity of the 2D projection. To ensure the group motion quality, we watch the videos and manually provide the ordinal depth relation information of all dancers in the scene at each frame $t$ as follows:
$$
r_t(p,p') = 
\begin{cases}
1, &\text{if dancer } p \text{ is closer than } p' \\ 
-1, &\text{if dancer } p \text{ is farther than } p' \\
0, &\text{if their depths are roughly equal}
\end{cases}
$$
Given the relative depth information, we derive the depth relation term $E_{\rm dep}$. This term encourages consistent ordinal depth relation between the motion trajectories of multiple dancers, especially when dancers partially occlude each other:
$$
E_{\rm dep}(p,p',t) =
\begin{cases}
     \log(1+\exp(z^p_t - z^{p'}_t)), &r_t(p,p')=1 \\
     \log(1+\exp(-z^p_t + z^{p'}_t)), &r_t(p,p')=-1 \\
     (z^p_t - z^{p'}_t)^2, &r_t(p,p')=0 \\
\end{cases}
$$
where $z^p_t$ is the depth component of the root translation $\mathbf{\tau}^p_t$ of the person $p$ at frame $t$. Intuitively, for $r(p,p')=1$, $z_p$ should be smaller than $z_{p'}$ and otherwise. 



Finally, we apply the global ground contact constraint $E_{\rm gc}$ to further ensure consistency between the motion of every person and the environment based on the ground contact information. This contact term is also needed to reduce the artifacts such as foot-skating, jittering, and penetration under the ground.
$$
E_{\rm gc}(p) = \sum_{t=1}^{T-1} \sum_{j \in \mathcal{F}} c^p_{j,t}\Vert \mathbf{X}^p_{j,t+1} - \mathbf{X}^p_{j,t} \Vert^2 + c^p_{j,t} \Vert  (\mathbf{X}^p_{j,t} - \mathbf{f})^\top \mathbf{n}^* \Vert^2
$$
where $\mathcal{F}$ is the set of feet joint indexes, $\mathbf{n}^*$ is the estimated plane normal and $\mathbf{f}$ is a 3D fixed point on the ground plane. The first term in Equation~\ref{eq_Egc} is the zero velocity constraint when the feet are in contact with the ground, while the second term encourages the feet position to stay near the ground when in contact. To obtain the ground plane parameters, we initialize the plane point $\mathbf{f}$ as the weighted median of all contact feet positions. The plane normal $\mathbf{n}^*$ is obtained by optimizing a robust Huber objective:
$$
\mathbf{n}^* = \arg\min_{\mathbf{n}} \sum_{\mathbf{X}_{\rm feet}} \mathcal{H}\left((\mathbf{X}_{\rm feet} - \mathbf{f})^\top \frac{\mathbf{n}}{\Vert\mathbf{n}\Vert}\right) + \Vert \mathbf{n}^\top\mathbf{n} - 1 \Vert^2,
$$
where $\mathcal{H}$ is the Huber loss function,  $\mathbf{X}_{\rm feet}$ is the 3D feet positions of all dancers across the whole sequence that are labelled as in contact (i.e., $c^p_{j,t} = 1$) . 


###  How will AIOZ-GDANCE be useful to the community?

We bring up some interesting research directions that can be benefited from our dataset:
* Group Dance Generation
* Human Pose Tracking
* Dance Education
* Dance style transfer
* Human behavior analysis

While single-person choreography is a hot research topic recently, group dance generation has not yet well investigated. We hope that the release of our dataset will foster more this research direction.

## References
[1] Peize Sun, Jinkun Cao, Yi Jiang, Zehuan Yuan, Song Bai, Kris Kitani, and Ping Luo. Dancetrack: Multi-object tracking in uniform appearance and diverse motion. In CVPR, 2022

[2] Hao-Shu Fang, Shuqin Xie, Yu-Wing Tai, and Cewu Lu. RMPE: Regional multi-person pose estimation. In ICCV, 2017.

[3] Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, and Michael J. Black. SMPL: A skinned multiperson linear model. ACM Trans. Graphics, 2015



[4] Georgios Pavlakos, Vasileios Choutas, Nima Ghorbani, Timo Bolkart, Ahmed A. A. Osman, Dimitrios Tzionas, and Michael J. Black. Expressive body capture: 3d hands, face, and body from a single image. In CVPR, 2019.