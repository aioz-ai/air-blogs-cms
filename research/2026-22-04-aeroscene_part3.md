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


In the previous part, we examined the detailed architecture of the AeroScene method. We now turn to the experimental analysis, covering implementation details, evaluation metrics, and the effectiveness of our approach compared to baseline methods. 

## Scene Synthesis Experiments


### Implementation Details.
Our method is implemented in PyTorch and validated on the 3D-FRONT and our dataset. %Scene layouts are represented as structured graphs, where nodes correspond to objects annotated with bounding boxes, orientations (as quaternions), scales, and semantic categories. %The hierarchy is explicitly modeled by linking fine-scale objects (e.g., utensils, decorations, debris) to their corresponding coarse-scale parents (e.g., furniture groups, buildings). For the diffusion process, we adopt a denoising diffusion probabilistic model with a hierarchy-aware denoising network. 

Training is performed using the Adam optimizer with a learning rate of $2 \times 10^{-4}$, batch size of 64, and a cosine learning rate scheduler. We train for 800 epochs on a single NVIDIA A100 GPU. During training, coarse-to-fine guidance objectives are applied to encourage logical placement of objects across scales, while collision and semantic consistency losses are included to ensure physically plausible and semantically coherent layouts. At inference time, we employ 100 reverse diffusion steps, which provide a balance between synthesis quality and computational efficiency. 


### Baselines.
We compare our framework against established baselines for scene layout synthesis: 
- **ATISS** employs an autoregressive transformer to generate indoor scenes. 
- **Diffusion-SDF** uses a diffusion model with signed distance fields to model object placements. 
- **DiffuScene** is a compositional diffusion model that generates scenes without explicit hierarchical modeling. 
- **PhyScene** uses physical constraints to generate indoor environments with layouts and articulated objects. 

### Evaluation Metrics 
We evaluate generated layouts using different metrics: 
- **FID** and **KID** measure perceptual similarity. 
- **Collision Rate (CR)** quantifies physical plausibility as the percentage of object pairs with $\text{IoU} > 0.01$. 
- **Coarse-to-Fine Consistency (CFC)** evaluates hierarchical alignment by computing the average normalized distance between fine placements and their assigned coarse regions. 
- **Semantic Plausibility (SP)** measures the similarity with spatial category priors, calculated as the negative log-likelihood under empirical pairwise category distributions from the training set. 

Metrics are averaged over 1,000 scenes per method.
<style>
      table {
    margin-left: auto;
    margin-right: auto;
    /* Optional: Set a specific width so there is available space to center within its container */
    /* width: 80%; */ 
  }
  .center-content th,
  .center-content td {
    text-align: center; /* Centers text horizontally */
    vertical-align: middle; /* Centers text vertically */
  }
  .bold-text {
    font-weight: bold; /* or use a numeric value like 700 */
  }
</style>
<table class="center-content">
    <tr>
        <td>Method</td>
        <td class="bold-text"> FID &#x2193;</td>
        <td class="bold-text">KID &#x2193;</td>
        <td class="bold-text">CR(%)&#x2193; </td>
        <td class="bold-text">CFC &#x2193;</td>
        <td class="bold-text">SP &#x2193;</td>
    </tr>
    <tr>
        <td class="bold-text" colspan="6">Aeroscene Dataset</td>
    </tr>
    <tr>
        <td>ATISS</td>
        <td>45.2</td>
        <td>0.032</td>
        <td>12.5</td>
        <td>0.21</td>
        <td>3.8</td>
    </tr>
    <tr>
        <td>Diffusion-SDF</td>
        <td>38.7</td>
        <td>0.028</td>
        <td>10.1</td>
        <td>0.18</td>
        <td>3.5</td>
    </tr>
    <tr>
        <td>DiffuScene</td>
        <td>32.4</td>
        <td>0.025</td>
        <td>8.3</td>
        <td>0.15</td>
        <td>3.2</td>
    </tr>
    <tr>
        <td>PhyScene</td>
        <td>29.8</td>
        <td>0.023</td>
        <td>7.1</td>
        <td>0.13</td>
        <td>3.0</td>
    </tr>
    <tr>
        <td class="bold-text">Ours</td>
        <td class="bold-text">27.3</td>
        <td class="bold-text">0.021</td>
        <td class="bold-text">6.2</td>
        <td class="bold-text">0.12</td>
        <td class="bold-text">2.7</td>
    </tr>
    <tr>
        <td class="bold-text" colspan="6">3D-FRONT Dataset</td>
    </tr>
    <tr>
        <td>ATISS</td>
        <td>42.1</td>
        <td>0.030</td>
        <td>11.8</td>
        <td>0.19</td>
        <td>3.6</td>
    </tr>
    <tr>
        <td>Diffusion-SDF</td>
        <td>35.6</td>
        <td>0.026</td>
        <td>9.4</td>
        <td>0.16</td>
        <td>3.3</td>
    </tr>
    <tr>
        <td>DiffuScene</td>
        <td>30.2</td>
        <td>0.023</td>
        <td>7.6</td>
        <td>0.14</td>
        <td>3.0</td>
    </tr>
    <tr>
        <td>PhyScene</td>
        <td>27.9</td>
        <td>0.021</td>
        <td>6.3</td>
        <td>0.12</td>
        <td>2.7</td>
    </tr>
    <tr>
        <td class="bold-text">Ours </td>
        <td class="bold-text">25.8</td>
        <td class="bold-text">0.019</td>
        <td class="bold-text">5.5</td>
        <td class="bold-text">0.11</td>
        <td class="bold-text">2.5</td>
    </tr>
</table>
<p align="center">
    <b>Table 1:</b>
    <i> Quantitative results on scene synthesis.</i> 
</p>



### Scene Generation Results
Table 1 shows quantitative results of our method. This table shows that our method outperforms baselines in all metrics. Qualitatively, Fig.3  illustrates that the generated layouts in our model produce coherent hierarchies, grouping fine objects (e.g., tables, chairs, sofas) within coarse structures (e.g., room partitions), whereas baselines often exhibit overlaps or implausible placements. On the 3D-FRONT dataset, similar trends hold on both qualitative and quantitative evaluations. Our method yields more realistic indoor arrangements compared with other solutions (Figure 4 ). We also visualize the generation progress of our guided substitution for diffirent objects in Figure 5.

<table>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624316/4_downbaseline_1e_jaqc37.png" width="200"><br>
      <sub>SDF</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624311/4_downbaseline_2_e9zefa.png" width="200"><br>
      <sub>DiffuScene</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624312/4_downbaseline_3_vzsnej.png" width="200"><br>
      <sub>PhyScene</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624312/4_good_case_jpvuc6.png" width="200"><br>
      <sub>Ours</sub>
    </td>
  </tr>
</table>

<p align="center">
    <b>Figure 3:</b>
    <i> An overview of our AeroScene method.</i> 
</p>




<table>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624315/2_bad_case_1_w4ozxg.png" width="200"><br>
      <sub>SDF</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624313/2_bad_case_2_wkoeiq.png" width="200"><br>
      <sub>DiffuScene</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624314/2_bad_case_3_miq8ch.png" width="200"><br>
      <sub>PhyScene</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774624315/2_good_case_luxsqk.png" width="200"><br>
      <sub>Ours</sub>
    </td>
  </tr>
</table>

<p align="center">
    <b>Figure 4:</b>
    <i> Indoor scene generation visual comparison. The red circle shows the collision or incorrect position.</i> 
</p>



<table>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774626498/floorc_kp0uaf.png" width="200"><br>
      <sub>Layout</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774626498/step1c_hnslpu.png" width="200"><br>
      <sub>Tokenization</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774626498/step2c_kxk9i1.png" width="200"><br>
      <sub>Generation w. Constraints</sub>
    </td>
    <td align="center">
      <img src="https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774626499/step4c_u46jig.png" width="200"><br>
      <sub>Finalization</sub>
    </td>
  </tr>
</table>

<p align="center">
    <b>Figure 5:</b>
    <i> The generation sequence of objects in our method.</i> 
</p>

### Ablation Study on Guidance
To assess the impact of guidance objectives, we conduct ablations by removing individual components during training and inference. Table 2 shows results on our dataset. 
- Removing Collision Avoidance increases CR by 40\%, indicating its role in physical plausibility. 
- Without Coarse-to-Fine Guidance, CFC degrades by 25\%, leading to misaligned hierarchies. 
- Semantic Constraint Guidance is crucial for SP, with its removal causing a 30\% worsening. 
- Combining all guidance yields the best performance, confirming their complementary nature. 

Note that our inference time is approximately $2$ minutes per scene on an NVIDIA A100 GPU, comparable to other diffusion baselines. 
<style>
      table {
    margin-left: auto;
    margin-right: auto;
    /* Optional: Set a specific width so there is available space to center within its container */
    /* width: 80%; */ 
  }
  .center-content th,
  .center-content td {
    text-align: center; /* Centers text horizontally */
    vertical-align: middle; /* Centers text vertically */
  }
  .bold-text {
    font-weight: bold; /* or use a numeric value like 700 */
  }
</style>
<table class='center content'>
    <tr>
        <td class='bold-text'>Configuration</td>
        <td class='bold-text'>FID&#x2193;</td>
        <td class='bold-text'>CR(%)&#x2193;</td>
        <td class='bold-text'>CFC&#x2193;</td>
        <td class='bold-text'>SP&#x2193;</td>
    </tr>
    <tr>
        <td class='bold-text'>Ours (full)</td>
        <td class='bold-text'>27.3</td>
        <td class='bold-text'>6.2</td>
        <td class='bold-text'>0.12</td>
        <td class='bold-text'>2.7</td>
    </tr>
    <tr>
        <td>w/o Collision</td>
        <td>32.1</td>
        <td>8.7</td>
        <td>0.13</td>
        <td>2.8</td>
    </tr>
    <tr>
        <td>w/o C2F</td>
        <td>30.5</td>
        <td>6.5</td>
        <td>0.15</td>
        <td>2.9</td>
    </tr>
    <tr>
        <td>w/o Semantic</td>
        <td>31.8</td>
        <td>6.4</td>
        <td>0.13</td>
        <td>3.5</td>
    </tr>
    <tr>
        <td>w/o All Guidance</td>
        <td>35.4</td>
        <td>9.2</td>
        <td>0.17</td>
        <td>3.9</td>
    </tr>
</table>
<p align="center">
    <b>Table 2:</b>
    <i>Ablation on guidance objectives.</i> 
</p>

In the next part, we further investigate the AeroScene dataset and assess the performance of the drone navigation task.

</CodeExplanation>