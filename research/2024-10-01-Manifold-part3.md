---
last_modified_on: "2024-10-01"
id: Manifold3
title: Scalable Group Choreography via Variational Phase Manifold Learning (Part 3)
description: Training process and Experimental setup
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---
In the previous part,  we introduct our main proposal  Variational Phase Manifold Learning. In this part, we have explore the training process and experimental setup to validate the proposed method.

![](https://vision.aioz.io/f/4248eadb89e143479193/?dl=1)*<center>**Figure 1**. We present a new group dance generation method that can generate a large number of dancers within a fixed resource consumption. The illustration shows a generated group dance sample with $100$ dancers. </center>*

## Training
During training, we consider the following variational lower bound to mainly train our dance generation VAE model:

$$
\log p_\theta(\mathbf{x}|\mathbf{a}) \geq \mathbb{E}_{q_\phi} \left[ \log p_\theta(\mathbf{x}|\mathbf{z},\mathbf{a}) \right] - D_{\text{KL}}(q_\phi(\mathbf{z}|\mathbf{x},\mathbf{a}) \Vert p_\theta(\mathbf{z}|\mathbf{a}))
$$

In practice, we apply the conditional VAE loss, which is defined as the weighted sum $\mathcal{L}_{\text{cvae}} = \mathcal{L}_{\text{rec}} + \lambda_{\text{KL}}\mathcal{L}_{\text{KL}}$. In particular, the reconstruction term $\mathcal{L}_{\text{rec}}$ measures the motion reconstruction error given the decoder output (via a smooth-L1 loss). The KL divergence term $\mathcal{L}_{\text{KL}}$ compares the divergence $D_{\text{KL}}$ between the posterior and the prior distribution to enforce them to be close to each other.

The conditional VAE objective above is calculated for each dancer separately and cannot capture the correlation between dancers within a group. Therefore, it is essential to impose consistency among dancers and avoid strange effects such as unsynchronized dance. To this end, we propose a group consistency loss by  enforcing the latent phase manifold to be similar for the same group, given the input music. Specifically, we first calculate the phase manifold features based on the frequency domain parameters as follows:

$$
    \mathbf{P}_{2i-1} = \mathbf{A}_i\sin(2\pi \cdot \mathbf{S}_i), \qquad \mathbf{P}_{2i} = \mathbf{A}_i\cos(2\pi\cdot \mathbf{S}_i)
$$

where $\mathbf{P}\in\mathbb{R}^{2D}$ is the phase manifold vector that encodes the spatial-temporal information of the motion state. The phase feature may look similar to the positional encodings of transformers in the sense that both use multi-resolution sinusoidal functions. However, the phase feature is much richer in terms of representation capacity since it learns to embed the spatial (body joints) and temporal (positions in time) information of the motion curves, whereas the positional encodings only encode the position of words. Finally, our consistency objective is to constrain phase manifold between dancers within a group to be as close as possible to each other, which is formulated as:

$$
    \mathbf{\mathcal{L}}_{\text{csc}} =  D_{\text{KL}}(q_\phi(\mathbf{z}|\mathbf{x}^m,\mathbf{a}) \Vert (q_\phi(\mathbf{z}|\mathbf{x}^n,\mathbf{a}) ) + \Vert \mathbf{P}^m - \mathbf{P}^n\Vert^2_2
$$

where the first term encourages the network to map different dancers belonging to the same group ($\mathbf{x}^m$ and $\mathbf{x}^n$) into the same set of distributional phase parameters while the second term penalizes the discrepancy in their corresponding phase manifolds. In general, this loss is applied to ensure every dancer is embedded into a single unified manifold that can effectively represent their corresponding group. To summarize, our total training loss is defined as the combination of the VAE loss and the consistency loss $\mathcal{L} = \mathbf{\mathcal{L}}_{\text{cvae}} + \lambda_{\text{csc}}\mathbf{\mathcal{L}}_{\text{csc}}$.



For testing, we can efficiently generate motions for an unlimited number of dancers by indefinitely drawing samples from the learned continuous group-consistent phase manifold. It is noteworthy that for inference, we only need to process the prior network once to obtain the latent distribution. To generate a new motion, we can sample from this latent (Gaussian) distribution and use the decoder to decode it back to the motion space. This approach is much more efficient and has significantly higher scalability than previous approaches that is limited by the number of dancers processed simultaneously by the entire network.
![](https://vision.aioz.io/f/467f4d6312b24242ae3e/?dl=1)*<center>**Figure 2**. An example output. </center>*

## Experiments
### Implementation Details
Our model was trained on 4 NVIDIA V100 GPUs using Adam optimizer with a fixed learning rate of $10^{-4}$ and a mini-batch size of 32 per GPU. For training losses, the weights are empirically set to $\lambda_{\text{KL}} = 5\times 10^{-4}$ and $\lambda_{\text{csc}} = 10^{-4}$, respectively. The Transformer encoders and decoders consist of 5 layers for both encoder, decoder, and prior Network with 512-dimensional hidden units. Meanwhile, the number of latent phase channels is set to 256. To further capture the periodic nature of the phase feature, we also use Siren activation following the initialization scheme. This can effectively model the periodicity inherent in the motion data, and thus can benefit motion synthesis. 

### Experimental Settings
**Dataset.** In our experiments, we utilize the AIOZ-GDance and AIST-M datasets. AIOZ-GDance is the largest music-driven dataset focusing on group dance, encompassing paired music and 3D group motions extracted from in-the-wild videos through a semi-automatic process. This dataset spans 7 dance styles and 16 music genres. For consistency, we adhere to the training and testing split  during our experiments.

**Evaluation Protocol.** We employ several metrics to assess the quality of individual dance motions, including Frechet Inception Distance (FID), Motion-Music Consistency (MMC, and Generation Diversity (GenDiv), along with the Physical Foot Contact score (PFC). Specifically, the FID score gauges the realism of individual dance movements concerning the ground-truth dance. MMC assesses the matching similarity between motion and music beats, reflecting how well-generated dances synchronize with the music's rhythm. GenDiv is computed as the average pairwise distance of kinetic features among motions. PFC evaluates the physical plausibility of foot movements by determining the agreement between the acceleration of the character's center of mass and the foot's velocity.

In assessing the quality of group dance, we adopt three metrics : Group Motion Realism (GMR), Group Motion Correlation (GMC), and Trajectory Intersection Frequency (TIF). Broadly, GMR gauges the realism of generated group motions in comparison to ground-truth data, employing Frechet Inception Distance on extracted group motion features. GMC evaluates the synchronization among dancers within the generated group by computing their cross-correlation. TIF quantifies the frequency of collisions among the generated dancers during their dance movements.

**Baselines.**
Our method is subjected to comparison with various recent techniques in music-driven dance generation, namely FACT, Transflower, and EDGE. These approaches are adapted for benchmarking within the context of group dance generation, considering that their original designs were tailored for single-dance scenarios. Additionally, our evaluation includes a comparison with GDanceR, GCD, and DanY. All of the mentioned works are specifically designed for the generation of group choreography.

![](https://vision.aioz.io/f/b7fde96e07d34f14b6f4/?dl=1)*<center>**Figure 3**. Generated Group Dance from GCD baselines </center>*

## Next
In the next part, we will explore the effectiveness of the proposal through quantitative and qualitative results.