---
last_modified_on: "2023-09-03"
title: Method comparison between three latest Dance Generation approaches.
description: Summarizing the contents, limitations, and differences of three latest Dance Generation approaches.
series_position: 25
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: dance_generation"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Dance generation with music has emerged as a captivating field, presenting a myriad of applications in industries such as entertainment, advertising, and virtual performances. Recent advancements in deep learning techniques have led to the development of various methods for synthesizing dance motions that are synchronized with music. This summary aims to provide a comprehensive overview of the state-of-the-art research in dance generation with music by comparing and contrasting key aspects of different approaches. 

## Introduction
Generating dance choreography that synchronizes with music is a challenging problem that has been the focus of recent research. In this summary, the focus will be on the three latest works to provide a broader overview of the field.

1. **EDGE: Editable Dance Generation From Music**: This research concentrates on individual dance choreography, introducing the Editable Dance Generation (EDGE) methodology and proposing novel evaluation metrics for assessing choreographic quality.
2. **Music-Driven Group Choreography (AIOZ-GDANCE)**: This study distinguishes itself by introducing AIOZ-GDANCE, a comprehensive dataset tailored for music-driven group choreography, along with its associated algorithm, GDanceR.
3. **Controllable Group Choreography using Contrastive Diffusion**: This paper leverages Group Contrastive Diffusion (GCD) algorithms to produce visually coherent and musically synchronized group choreography.

By comparing these works, the aim is to offer a nuanced understanding of the state-of-the-art research of music-driven dance choreography. Each approach will be evaluated based on its contribution, methodology, dataset, user control, and editability, as well as its proposed metrics for evaluation. Through this analysis, both the advancements and challenges that characterize this exciting field will be identified, setting the stage for future research endeavors.

![Fig-1](https://drive.google.com/uc?export=view&id=1BLzKzL9jnM1Fq9SZbcS-_714P6CGmOI4)
*<center>**Figure 1**: Comparison between the three works.</center>*

## 1. EDGE: Editable Dance Generation From Music - CVPR 2023
**Research objectives**: The primary objective of the paper is to introduce a method for generating editable dance sequences that are both realistic and physically plausible, aligned with the input music, and equipped with powerful editing capabilities.

**Contributions**:
- Introduction of the Editable Dance Generation (EDGE) method, which allows for the creation and editing of individual dance sequences.
- Proposal of new metrics to evaluate the quality and physical plausibility of generated dances, filling a gap in the existing literature.

**Dataset**: AIST++ Dataset

- `Data Collection`:

    - Derived from the AIST Dance Database, which contains videos without 3D information.
    - Requires substantial effort and compute to recover camera calibration parameters and 3D human motion using SMPL parameters.
    - Provides 3D data, camera parameters, and annotations for each frame.
    - Contains multi-view synchronized image data.
    - Largest 3D human dance dataset with 1408 sequences, 30 subjects, and 10 dance genres.

- `Annotations`:

    - 9 views of camera intrinsic and extrinsic parameters.
    - 17 COCO-format human joint locations in 2D and 3D.
    - 24 SMPL pose parameters along with global scaling and translation.
    - Contains 10 dance genres with basic and advanced choreographies.
    - Equally distributed motions among genres, covering various music tempos.

**Modeling**:
- `Diffusion-Based Neural Network Model`: The paper employs a diffusion-based model to generate dance motion sequences. This model learns and reproduces the complex relationship between music and dance motion.

- `Utilizing Audio Representations from Jukebox`: To enhance performance compared to previous methods, the paper utilizes audio representations from Jukebox, a deep learning-based music generation model. Incorporating these audio representations enables the EDGE model to create dance motion sequences that are well-aligned with the input music.

- `Robust Editing Capabilities`: An essential aspect of the modeling process is its strong editing capabilities. EDGE allows users to flexibly edit dance motion sequences to create various performances. This feature can be used to modify musical interpretations, change dance genres, or create new and innovative performances.


**Limitations**:
- The paper focuses on individual dance choreography, which may limit its applicability to group performances.
- Advanced machine learning models often require significant computational resources, which could be a limitation for some users.
- The training setup exclusively permits the generation of dance sequences with a fixed duration, thereby hindering the ability to engage in long-term dance generation.
- The training procedure is constrained to solo dance inputs, thereby impeding the capacity to incorporate and edit the addition of dancers in dance editing tasks. 

## 2. Music-Driven Group Choreograph - CVPR 2023
**Research objectives**: Creating consistent and coherent group dacing motions. To address this issue, the paper introduces a new dataset called AIOZ-GDANCE, aimed at supporting the creation of dance steps for a group based on music.

**Contributions**:
- Introduction of AIOZ-GDANCE dataset.
- Introduction of GDanceR Method. This method is designed to generate group dance motions efficiently by utilizing the provided audio input as a guide.

**Dataset**: AIOZ-GDANCE dataset
- `Data Collection and Preprocessing`:
​
    - Collected from in-the-wild group dancing videos on platforms like YouTube, TikTok, and Facebook.
    - In-the-wild videos covering 7 dance styles and 16 music genres
    - Videos are processed at 1920x1080 resolution and 30FPS.
    - Human tracking performed using a multi-object tracker, with manual correction for failures.
    - 2D pose estimation used to generate initial 2D poses, with manual correction.
    - Group dance motion reconstructed by fitting 3D mesh for each dancer, followed by global optimization.
    - Addressing depth ordering challenges by manually providing depth relations.
​
- `Annotations`:
    - Provides 3D group dance motion reconstructed through optimization.
    - Uses the SMPL model for 3D human representation.
    - Involves complex global optimization to ensure coherent group motion.
    - Focuses on group dance generation, human pose tracking, and related research directions.

**Comparison between AIOZ-GDANCE dataset and AIST++ dataset:**
- `Size & Scope`: AIOZ-GDANCE is larger and focuses on group choreography, while AIST++ is smaller and focuses on solo dance.
- `Dance Styles & Genres`: AIOZ-GDANCE covers a broader range of dance styles and music genres compared to AIST++.
- `Data Collection and Labeling Method`: AIOZ-GDANCE uses a semi-autonomous labeling method and in-the-wild videos, whereas AIST++ uses  Motion Capture (MoCap) and multi-view videos with known camera poses.
- `Validation`: AIST++ employs MPJPE-2D (Mean Per Joint Position Error in 2D) to assess the quality of 3D motion recovery. While AIOZ-GDANCE utilizes Local Mesh Fitting and Global Optimization to evaluate and enhance the data quality.



**Modeling**:

The GDanceR model consists of two main parts: the Initial Pose Generator and the Group Motion Generator.

- `Initial Pose Generator`: This component is used to generate initial poses for the dancers. It combines audio features with the initial positions of the dancers to create these starting poses. The audio features are synthesized by taking the average of audio features across the music sequence. Subsequently, the synthesized audio features are concatenated with the initial positions and fed into the model.

- `Group Motion Generator`: This part is employed to generate group dance steps based on the music and initial poses. The model utilizes a Long Short-Term Memory (LSTM) recurrent neural network to encode the local motion representation of each dancer, and an attention mechanism to capture the spatial relationships between dancers. At each time step, the pose of each dancer is fed into an LSTM unit to encode the local motion representation. Concurrently, the model employs an attention mechanism to encode the spatial relationships among dancers. Finally, the local motion representation and the global motion representation are combined to create the final motion representation for each dancer.

**Limitations**:
- GDanceR is the first method to solve the group dance problem; however, the use of LSTM could lead to vanishing and exploding gradients during training, which may affect the training results.
- The model uses an MLP to predict the initial pose for each dancer based on the average audio representation and initial positions. This could be sensitive to the quality of the initial positions provided, affecting the overall choreography.
- The model is specifically designed for group dance choreography, making it less versatile for other types of motion generation.
- In GDanceR, a set number of fixed dancers is utilized in each group sample during training. Consequently, the quantity of generated dancers in each group is constrained and strictly adheres to the number of dancers present in the training samples.
- Encounter limitations associated with a fixed duration during training, akin to what is observed in the EDGE paper.

## 3. Controllable Group Choreography using Contrastive Diffusion - SIGGRAPH Asia 2023

**Research objectives**: The objective is to address the demand for generating high-quality and customizable group dance sequences as desired.

**Contributions**:
- Introduction of Contrastive Diffusion.

- High-Fidelity and Diverse Group Dancing Motions.

- Trade-off Method for Consistency and Diversity.

- State-of-the-Art Performance in Synthesizing Group Choreography.

- Availability of Source Code and Model.

**Dataset**: AIOZ-GDANCE dataset.

**Modeling**: 

- `Diffusion-Based Generative Approach`: The authors utilize a diffusion-based generative approach as the foundation of their modeling strategy. This approach is designed to synthesize group dance motions while considering both the number of dancers involved and the duration of the dance sequence. By leveraging diffusion processes, the model is capable of generating dance motions that are coherent with the input music.

- `Group Contrastive Diffusion (GCD)`: A pivotal contribution in the modeling phase is the introduction of the Group Contrastive Diffusion (GCD) strategy. This strategy enhances the connection and coherence among the dancers within the group. The GCD strategy allows for fine-tuned control over the level of consistency or diversity in the generated group dance animation. It leverages a classifier-guidance sampling technique to enable this control.

- `Consistency and Diversity Control`: The modeling approach ensures that the generated dance motions maintain the desired level of consistency with the input music while still being diverse enough to provide an engaging visual experience. This is achieved through careful design and incorporation of the GCD strategy, allowing users to adjust the balance between consistency and diversity.

- `Synthesis of Long-Term Group Dances`: The diffusion-based approach facilitates the generation of long-term group dance sequences, which is a significant advantage over some existing methods that struggle with producing high-fidelity long-duration motions. This capability contributes to the practicality and versatility of the proposed modeling technique.

**Limitations**: 
- Limitation on physical contact between dancers: Due to the absence of clear interactions between dancers in the AIOZ-GDANCE group dance dataset.

- Limitation in achieving alignment between "high diversity" and music: While GCD provides a trade-off between diversity and consistency, perfect alignment between high diversity and music remains a challenging task.

- Fine-tuning parameters and significant computational resources are required for training and testing.

- Over-controlling consistency and diversity might restrict creative freedom or consensus among dancers.

- The subjective nature of evaluating consistency and diversity.

## Conclusion
The three papers, **EDGE: Editable Dance Generation From Music,Music-Driven Group Choreography, and Controllable Group Choreography using Contrastive Diffusion**, each offer unique approaches to the problem of generating dance choreography that is synchronized with music. The second and third papers focus on the complexities of group choreography, employing diffusion-based generative approaches and large-scale datasets, respectively. In contrast, the first paper hones in on individual dance generation, offering powerful editing capabilities and introducing new evaluative metrics. Overall, all three papers effectively address the issues they have raised. However, it's worth noting that all three methods exhibit limitations when it comes to generating intricate dance movements that authentically reflect emotions, a task that draws influence from both the musical context and the surrounding scene factors. These limitations underscore the ongoing challenges in achieving a nuanced and emotionally resonant group dance generation process, including emotions of dancers in different dance styles.