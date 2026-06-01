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

In the first part, we introduced the task of affordance grounding in 3D scenes using visual cue images. In this section, we examine the construction of the AffordBridge dataset in greater detail.

## AffordBridge Dataset


In Figure 2, we outline our three-stage annotation flow, which includes 3D scene processing, visual signifier processing, and affordance annotation. 

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953234/method_uv9tuh.png)
*<center>**Figure 2**:  Construction of the **AffordBridge** dataset. Our **AffordBridge** dataset is built through a semi-supervised pipeline linking visual signifiers with 3D affordances. The building process includes (i) 3D scene processing via voxelized point clouds with object-view filtering through visual scanning, (ii) visual signifiers processing with human-object interaction extraction with fine-grained captioning, and (iii) affordance annotation by matching key views to 3D instances for spatial action labeling..</center>*

Let the colored scene point cloud be $\mathcal{P} = \{(p_{i}, f_{i})\}_{i=1}^{N}$, where $p_{i} \in \mathbb{R}^{3}$ denotes the 3D coordinates, and $f_{i} \in \mathbb{R}^{6}$ represents per-point features, including RGB colors and surface normals. From raw scans in **SceneFun3D** work, the point clouds are downsampled to $100,000$ points via voxelization with a voxel size of $0.05$ meters, while preserving the details of scene representations and maintaining frugality for the affordance learning model. Thus, instance segmentation masks are applied to extract object regions within the scene.

### 3D Scene Processing

**Visual Scan.** Each colored scene point cloud is temporally aligned with the corresponding RGB video sequence $\mathcal{V} = \{ v_{k} \}_{k=1}^{K}$, where each frame $v_{k}$ represents a visual observation captured from a calibrated camera pose $\left[ R_{k} \mid t_{k} \right]$ using the camera's intrinsic matrix, denoted as $K_{c}$. Through SLAM-based trajectory estimation, each frame $v_{k}$ is projected onto its associated 3D segment through depth alignment, ensuring geometric and temporal consistency along the trajectory. Multiview checks are used to remove occlusions and misalignments, yielding clean 2D-3D correspondences, denoted as $(P_{k}, v_{k})$, for downstream annotation. 

**Object-View Filtering**. Each visual scan may contain multiple objects with potential affordances. We detect candidate objects using MobileNet and rank them by spatial location, scale, and contextual relevance. Each detected instance $v_{k}^{(l)}$ is therefore aligned with its 3D counterpart $P_{k}^{(l)}$, and inconsistent matches of approximately $15\%$ are manually discarded. To maintain reliability, we ensure inter-annotator agreements with a Kappa score higher than $0.75$ among three human experts reviewing the annotations.

### Visual Signfier Processing

**Annotation of Visual Signifiers.** We adopt interaction images from **PIAD**, retaining only those that depict direct human-object contact. Each image is annotated with three bounding boxes $b = (b_{H}, b_{O}, b_{I})$ for human, object, and interaction regions. We refine the interaction box $b_{I}$ by inspecting model-predicted class scores to precisely localize the contact region. To further address potential ambiguities in visual signifiers, as static images may miss dynamics, we incorporate human pose estimation via OpenPose to annotate keypoint-based hand-object contacts, enhancing the bounding boxes for humans, objects, and contact regions.

**Fine-grained Captioning.**
Next, we generate fine-grained captions using visual signifiers as inputs. Specifically, using the Object Relation Transformer (ORT), we fuse the three bounding boxes $b$ to produce coherent, templated descriptions. As shown in Figure 3, the caption **A man opens the black door** describes the RGB image input. All captions are manually verified for semantic and spatial accuracy.

### Affordance Annotation.

To associate textual descriptions with corresponding scene views, we align image-text embeddings using CLIP encoders and retrieve the most relevant key-view by maximizing cosine similarity. The embedding space is refined through contrastive learning to achieve higher similarity between positive image-text pairs and vice versa. 

After filtering, each key-view $I_{i}$ is paired with an affordance action $a_{i}$ and its corresponding region within $\mathcal{P}$. We use a web-based annotation interface, where annotators are able to identify the 3D instance segmentation mask $M_{i}$ that corresponds to the 2D view with the affordance mask $A_{i}$ for the affordance action $a_{i}$. The mapping between 2D affordances and 3D instances is 
$$
M_{i} = \arg \max_{M_{j} \in \mathcal{P}}\phi_{\text{sim}}\left(I_{i}, M_{j}\right)
$$

where $\phi_{\text{sim}}(I_{i}, M_{j})$ measures the visual-geometric similarity between the key-view and the 3D object. To avoid one-to-many ambiguities, where a single visual signifier may correspond to multiple 3D instances, we allow multi-instance projection by retaining top-$3$ matches based on CLIP similarity scores. Subsequently, annotators verify and refine these candidates to obtain the most accurate correspondence. The resulting annotations yield object-level affordance masks embedded within full-scene geometry, with approximately $5\%$ of samples re-annotated to resolve ambiguous cases.


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
        <td>Dataset</td>
        <td>Total Samples</td>
        <td>Environment</td>
        <td>Implicit Interaction</td>
        <td>Explicit Interaction</td>
        <td>Affordance Annotation</td>
        <td>No. Affordance</td>
        <td>No. Categories</td>
        <td>No. Aff. Action</td>
    </tr>
    <tr>
        <td>EPIC-Aff</td>
        <td>38,876</td>
        <td>2D Images</td>
        <td>--</td>
        <td>--</td>
        <td>2D masks</td>
        <td>--</td>
        <td>304</td>
        <td>43</td>
    </tr>
    <tr>
        <td>AGD20k</td>
        <td>23,816</td>
        <td>2D Images</td>
        <td>--</td>
        <td>--</td>
        <td>2D boxes</td>
        <td>--</td>
        <td>50</td>
        <td>36</td>
    </tr>
    <tr>
        <td>PartNet</td>
        <td>26,671</td>
        <td>3D Objects</td>
        <td>--</td>
        <td>--</td>
        <td>3D masks</td>
        <td>573,585</td>
        <td>--</td>
        <td>24</td>
    </tr>
    <tr>
        <td>AffordPose</td>
        <td>641</td>
        <td>3D Objects</td>
        <td>--</td>
        <td>Grasp hand poses</td>
        <td>3D masks</td>
        <td>26,712</td>
        <td>13</td>
        <td>8</td>
    </tr>
    <tr>
        <td>3DAffordanceNet</td>
        <td>22,949</td>
        <td>3D Objects</td>
        <td>--</td>
        <td>--</td>
        <td>3D masks</td>
        <td>56,307</td>
        <td>23</td>
        <td>18</td>
    </tr>
    <tr>
        <td>PIAD</td>
        <td>7,012</td>
        <td>3D Objects</td>
        <td>Single interactions</td>
        <td>--</td>
        <td>3D masks</td>
        <td>7,012</td>
        <td>23</td>
        <td>17</td>
    </tr>
    <tr>
        <td>LASO</td>
        <td>8,434</td>
        <td>3D Objects</td>
        <td>Commands on objects</td>
        <td>--</td>
        <td>3D masks</td>
        <td>19,751</td>
        <td>23</td>
        <td>17</td>
    </tr>
    <tr>
        <td>Scenefun3D</td>
        <td>--</td>
        <td>3D Scenes</td>
        <td>--</td>
        <td> Commands on scenes</td>
        <td>3D masks</td>
        <td>14,279</td>
        <td>--</td>
        <td>9</td>
    </tr>
    <tr>
        <td>PIADv2</td>
        <td>38,889</td>
        <td>3D Objects</td>
        <td>Single interaction</td>
        <td>--</td>
        <td>3D masks</td>
        <td>38,889</td>
        <td>43</td>
        <td>24</td>
    </tr>
    <tr>
        <td>AED</td>
        <td>--</td>
        <td>2D Images</td>
        <td>--</td>
        <td>--</td>
        <td>2D masks</td>
        <td>--</td>
        <td>13</td>
        <td>8</td>
    </tr>
    <tr>
        <td>SeqAfford</td>
        <td>18,371</td>
        <td>3D Objects</td>
        <td>Commands on objects</td>
        <td>--</td>
        <td>3D masks</td>
        <td>183,233</td>
        <td>23</td>
        <td>18</td>
    </tr>
    <tr>
        <td>MIPA</td>
        <td>7,012</td>
        <td>3D Scenes</td>
        <td>Multiple interactions</td>
        <td>--</td>
        <td>3D masks</td>
        <td>7,012</td>
        <td>23</td>
        <td>17</td>
    </tr>
    <tr>
        <td><b>AffordBridge</b></td>
        <td>317,844</td>
        <td>3D Scenes</td>
        <td>Visual signifiers
        <td>Descriptions</td>
        <td>3D masks</td>
        <td>291,637</td>
        <td>157</td>
        <td>61</td>
    </tr>
</table>
<p align="center">
    <b>Table 1:</b>
    <i> Comparisons between AffordBridge and other datasets</i> 
</p>

### Dataset Statistics

Our **AffordBridge** dataset is organized into training, validation, and test sets across three modalities: visual signifiers, 3D scenes, and affordance areas, as shown in Table 2. The visual signifiers subset contains $9,870$ samples, while the 3D scenes subset includes $689$ samples. Affordance areas form the largest component, with $291,637$ samples. In total, the dataset comprises $317.8$K samples, with $206.6$K for training, $63.6$K for validation, and $47.7$K for testing, providing a sufficient scale for learning spatial affordance.

We also compare our AffordBridge dataset with available datasets that annotate affordance labels in diverse environments as shown in Table 1.


<table>
    <tr>
        <td><b>Criteria</b></td>
        <td><b>Train</b></td>
        <td><b>Validate</b></td>
        <td><b>Test</b></td>
        <td><b>Total</b></td>
    </tr>
    <tr>
        <td>Visual Signifiers</td>
        <td>6,416</td>
        <td>1,974</td>
        <td>1,480</td>
        <td>9,870</td>
    </tr>
    <tr>
        <td>3D Scenes</td>
        <td>448</td>
        <td>138</td>
        <td>103</td>
        <td>689</td>
    </tr>
    <tr>
        <td>Affordance Areas</td>
        <td>189,564</td>
        <td>58,327</td>
        <td>43,746</td>
        <td>291,637</td>
    </tr>
    <tr>
        <td>Total Samples</td>
        <td>206.6K</td>
        <td>63.6K</td>
        <td>47.7K</td>
        <td>317.8K</td>
    </tr>
</table>
<p align="center">
    <b>Table 2:</b>
    <i> Train/validation/test split of the AffordBridge dataset</i> 
</p>


Figure 3 illustrates the object distribution across $689$ indoor 3D scenes. More specifically, chairs account for $28.2\%$, cups for $18.9\%$, and buttons for $8.7\%$, representing the most frequently interacted objects in the dataset. Lights contribute $7.5\%$, books $7.8\%$, tables $8.1\%$, and boxes $6.1\%$, each providing essential diversity for modeling functional affordances. The **Others** category covers $14.7\%$ of the dataset. In Figure 5, the object distribution demonstrates the diversity and balance of our **AffordBridge** dataset, supporting both object-centric and scene-level affordance learning.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953232/stat_xidxrv.png)
*<center>**Figure 3**:  Dataset statistics: Statistics of objects in human-object interactions yielding affordances in our **AffordBridge** dataset.</center>*



In the next part, we will discuss the method for grounding affordance in 3D scene with the assist of visual cue images.




</CodeExplanation>