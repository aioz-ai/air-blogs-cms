---
last_modified_on: "2026-05-04"
title: AffordMatcher: Affordance Learning in 3D Scenes from Visual Signifiers
description: Semantic Matching, Affordance
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


In the previous part, we examined the detailed architecture of the AffordMatcher method. We now turn to the experimental analysis, covering implementation details, evaluation metrics, and the effectiveness of our approach compared to baseline methods. 




## Experiments \& Evaluations

### Experiment Setup \& Baselines
**Implementation Details.** In our experiments, the **AffordMatcher** model is trained with RGB-point cloud and description text inputs for $100$ epochs on an NVIDIA RTX 3090 GPU with a batch size of $16$ and an initial learning rate of $10^{-4}$ decayed by $0.5$ every $30$ epochs. The RGB images are resized to $224 \times 224$ with visual augmentation, while 3D scenes are voxelized into a $64^{3}$ grid and segmented by a pre-trained 3D model to obtain binary-mask affordance candidates. Our evaluation follows the standardized zero-shot affordance segmentation metrics: mAP@$0.25$, mAP@$0.50$, and mAP averaged over IoU thresholds from $0.50$ to $0.95$.

**Baselines.** We benchmark **AffordMatcher** against state-of-the-art baselines in functional adaptations of 3D instance segmentation methods, including **Mask3D-F**, **SoftGroup-F**, and **OpenMask3D-F**, **AffordPose-DGCNN**, **3DAffordanceNet**, **PIAD**, **LASO**, and **Ego-SAG.** 

### Quantitative Results
Table 3 reports the performance of our method against state-of-the-art methods on functionality affordance segmentation. Specifically, **AffordMatcher** achieves an overall mAP of $53.4$, outperforming the second-best baseline by $7.8$ while exhibiting superior localization of functionally relevant regions among both low and high IoU thresholds. The improvement demonstrates the effectiveness of visually guided reasoning in improving spatial affordance localization in 3D scenes, given visual signifiers, under zero-shot settings.

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
  .italic-text {
    font-style: italic;
  table td:nth-child(3) {
    text-align: center;
}
  }
</style>
<table>
    <tr>
        <td><b>Method</b></td>
        <td align="center">mAP</td>
        <td align="center">mAP@0.25</td>
        <td align="center">mAP@0.5</td>
        <td align="center">No. Params</td>
        <td align="center">Inference Speed (ms / sample)</td>
    </tr>
    <tr>
        <td>Mask3D-F</td>
        <td align="center">41.2</td>
        <td align="center">58.6</td>
        <td  align="center">47.1</td>
        <td  align="center">19.0M</td>
        <td  align="center">126.2</td>
    </tr>
    <tr>
        <td>SoftGroup-F</td>
        <td  align="center">43.9</td>
        <td align="center">60.8</td>
        <td align="center">49.3</td>
        <td align="center">30.4M</td>
        <td align="center">288.0</td>
    </tr>
    <tr>
        <td>OpenMask3D-F</td>
        <td align="center">45.6</td>
        <td align="center">62.1</td>
        <td align="center">51.0</td>
        <td align="center">39.7M</td>
        <td align="center">315.1</td>
    </tr>
    <tr>
        <td>APose-DGCNN</td>
        <td align="center">29.7</td>
        <td align="center">47.6</td>
        <td align="center">34.8</td>
        <td align="center"><b>12.5M</td>
        <td align="center">140.2</td>
    </tr>
    <tr>
        <td>3DAffordanceNet</td>
        <td align="center">34.2</td>
        <td align="center">51.3</td>
        <td align="center">39.6</td>
        <td align="center">15.0M</td>
        <td align="center">180.4</td>
    </tr>
    <tr>
        <td>PIAD</td>
        <td align="center">26.1</td>
        <td align="center">44.7</td>
        <td align="center">30.5</td>
        <td align="center">23.0M</td>
        <td align="center">160.9</td>
    </tr>
    <tr>
        <td>LASO</td>
        <td align="center">37.5</td>
        <td align="center">54.2</td>
        <td align="center">42.6</td>
        <td align="center">21.4M</td>
        <td align="center">130.4</td>
    </tr>
    <tr>
        <td>Ego-SAG</td>
        <td align="center">40.3</td>
        <td align="center">56.7</td>
        <td align="center">45.1</td>
        <td align="center">24.8M</td>
        <td align="center">175.3</td>
    </tr>
    <tr>
        <td><b>AffordMatcher</b></td>
        <td align="center"><b>53.4</b></td>
        <td align="center"><b>69.7</b></td>
        <td align="center"><b>59.5</b></td>
        <td align="center"><b>20.7M</b></td>
        <td align="center"><b>112.5</b></td>
    </tr>
</table>
<p align="center">
    <b>Table 3:</b>
    <i> <b> Quantitative results:</b> Performance comparisons of <b>AffordMatcher</b> and state-of-the-art methods in terms of mAP, mAP@0.25, mAP@0.50, number of parameters (in millions), and inference speed (in milliseconds per sample).</i> 
</p>


Also shown in Table 3, **AffordMatcher** attains high accuracy while maintaining computational efficiency. The model contains $20.7$ million parameters; for example, fewer than OpenMask3D-F, while achieving faster inference at $112.5$ milliseconds per sample. The balanced trade-off between accuracy and efficiency illustrates the scalability of **AffordMatcher** for large-scale and near real-time 3D scene affordance localization.





### Ablation Study
**Input Guidance Analysis**.  As shown in Table 4, we conduct experiments to assess whether the observed performance gains originate from interaction reasoning rather than object recognition. We found that eliminating the 2D branch leads to a significant decrease in mAP to $37.3$, indicating the critical role of visual cues. Meanwhile, removing humans and hands from RGB images via inpainting lowers the mAP to $40.9$, confirming that action semantics contribute to interaction reasoning. Using raw point clouds of more than $500,000$ points reduces the mAP to $48.7$ due to memory constraints. Fine-tuning on the object-centric PIAD dataset produces $45.3$ of mAP, validating the advantage of modeling scene-level affordance cues over isolated object interactions.

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
  .italic-text {
    font-style: italic;
  table td:nth-child(3) {
    text-align: center;
}
  }
</style>
<table>
    <tr>
        <td>Variant of Inputs</td>
        <td align="center">mAP</td>
        <td align="center">mAP@0.25</td>
        <td align="center">mAP@0.5</td>
    </tr>
    <tr>
        <td>pruning RGB image inputs</td>
        <td align="center">37.3</td>
        <td align="center">52.7</td>
        <td align="center">42.1</td>
    </tr>
    <tr>
        <td>inpainting human-object interactions</td>
        <td align="center">40.9</td>
        <td align="center">56.2</td>
        <td align="center">45.3</td>
    </tr>
    <tr>
        <td>without point cloud downsampling</td>
        <td align="center">48.7</td>
        <td align="center">65.1</td>
        <td align="center">54.2</td>
    </tr>
    <tr>
        <td>fine-tuning with PIAD objects</td>
        <td align="center">45.3</td>
        <td align="center">61.8</td>
        <td align="center">50.6</td>
    </tr>
    <tr>
        <td><b>AffordMatcher</b></td>
        <td align="center"><b>53.4</b></td>
        <td align="center"><b>69.7</b></td>
        <td align="center"><b>59.5</b></td>
    </tr>
</table>
<p align="center">
    <b>Table 4:</b>
    <i> Ablation on different input modalities</i> 
</p>

 

**Loss Component Analysis.** Table 5 reports the contribution of each loss component. Beginning with a standard semantic baseline, the inclusion of $\mathcal{L}_{\text{align}}$ and $\mathcal{L}_{\text{dissim}}$ yields significant improvements, showcasing the importance of cross-instance correspondence modeling. Adding the bidirectional and regularization losses further enhances performance, resulting in a cumulative mAP gain of $16.1$, which shows that the complete objective effectively learns structured and discriminative affordance representations.

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
  .italic-text {
    font-style: italic;
  table td:nth-child(3) {
    text-align: center;
}
  }
</style>
| $\mathcal{L}_{\text{align}}$  | $\mathcal{L}_{\text{dissim}}$ | $\mathcal{L}_{\text{embed}}$ | $\mathcal{L}_{\text{bidir}}$ | mAP              | mAP@0.25    |mAP@0.5    |
|-------------------------------|-------------------------------|------------------------------|------------------------------|------------------|------------------|------------------|
|                               |                               |                              |                              | 37.3             | 52.7             | 42.1             |
| ✔️                    |                               |                              |                              | 40.9             | 56.2             | 45.3             |  
| ✔️                    | ✔️                    |                              |                              | 44.1             | 60.0             | 48.7             |  
| ✔️                    | ✔️                    | ✔️                   |                              | 47.8 | 63.5 | 53.0 |  
| ✔️ | ✔️                    | ✔️                   | ✔️                   | **53.4**    | **69.7**    | **59.5**    |

<p align="center">
    <b>Table 5:</b>
    <i> Ablation on loss components</i> 
</p>

**Attention Visualization.** Figure 5 illustrates how the learned attention maps transfer reasoning cues across modalities. For the ``Rest on Pillow`` example, the visual signifier concentrates on the pillow region. Through this, its spatial attention is able to emphasize the corresponding voxels on seating surfaces, confirming consistent cross-modal learning for human-object affordance localization.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953233/attn_axmubz.png)
*<center>**Figure 5**:  **Attention visualization:** From the visual signifier in the RGB image and the text *"Rest on Pillow"*, **AffordMatcher** focuses on the pillow area in the RGB image and correctly localizes the corresponding affordance regions in the high-resolution voxelized indoor scene.</center>*


**Reasoning Analysis.** The t-SNE embeddings in Figure 6 reveal that the reasoning module produces compact and well-separated clusters, indicating improved affordance discriminability. Meanwhile, Figure 7 further demonstrates reasoning adaptability by visualizing distinct interaction cues, such as ``Sit`` and ``Pull``, applied to the same object. For the ``Sit``* action, attention focuses on the seat cushion and the frontal area of the chair; whereas for the ``Pull`` action, the attention then shifts toward the upper back and armrest regions, demonstrating that **AffordMatcher** can dynamically adjust its focus according to given visual signifiers while maintaining a consistent geometric understanding.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953233/tsne_u5keou.png)
*<center>**Figure 6**:  **t-SNE visualization:** Visual reasoning produces more compact and well-separated clusters among different affordance types compared to without visual reasoning.</center>*

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953233/vis_distinct_aff_p8gemt.png)
*<center>**Figure 7**:  **Visualization of distinct affordances**: ``Sit`` and ``Pull`` cues on the same chair activate different 3D regions, showcasing that **AffordMatcher** adapts attention to interaction semantics.</center>*

**Qualitative Results.** Figure 8 presents qualitative comparisons between our **AffordMatcher** and PIAD together with Ego-SAG. PIAD tends to under-segment affordance regions, often missing fine interaction details, while Ego-SAG often over-segments them, producing overly broad or redundant affordance masks that lack spatial precision. Notably, **AffordMatcher** is able to generate compact and accurate affordance masks that suit both coarse and fine-grained interaction parts, such as knobs and prongs, while achieving higher spatial precision.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953233/aff_vis_r0x8gv.png)
*<center>**Figure 8**:  **Qualitative results:** Our affordance‐mask prediction, compared with other baselines. The first column illustrates the inputs, including visual signifiers and 3D scenes. The remaining columns show the affordance segmentation results, including extracted actions over different methods, where **blue texts** indicate correct affordance actions and **red texts** represent wrong ones. In the point clouds, **green areas** denote correct affordance localization,  **red** and **blue areas** denote false positives and false negatives, respectively.</center>*

**Limitations.** Although **AffordMatcher** demonstrates strong cross-modality affordance learning capability, it faces challenges with memory and scalability in highly detailed scenes, resulting in increased computational costs. Some errors also occur, such as overlapping affordances or unclear actions, which reveal limits in disambiguation and spatial reasoning.




## Conclusion 
In this work, we introduce **AffordMatcher**, an affordance learning method for spatial affordance localization in high-resolution voxelized indoor scenes from visual signifiers through sophisticated cross-modal reasoning. By leveraging the **AffordBridge** dataset and a dissimilarity-based match-to-match attention mechanism, **AffordMatcher** achieves robust zero-shot affordance segmentation across diverse scenes. Experimental results demonstrate consistent gains over state-of-the-art methods in both accuracy and efficiency, validating the effectiveness of reasoning-guided affordance learning. We reserve the task of extending **AffordMatcher** to temporal and interactive scenarios, enabling dynamic affordance reasoning in real-world robotic systems and embodiments. 

Across all four parts, we hope this blog offers valuable insights into affordance grounding in 3D scenes with visual signifiers.
</CodeExplanation>