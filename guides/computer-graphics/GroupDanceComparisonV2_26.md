---
last_modified_on: "2023-09-05"
title: Music2Dance and GroupDancer comparison.
description: Music2Dance and GroupDancer comparison.
series_position: 26
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: group_dance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Dance generation driven by music has become an increasingly fascinating domain, with a wide range of applications spanning sectors like entertainment, marketing, and digital performances. The progress in advanced deep learning technologies has paved the way for the creation of diverse approaches to generate dance movements that harmonize with musical rhythms. This summary aims to provide a comprehensive overview of the latest research in the field of dance creation in sync with music, accomplished by comparing and scrutinizing key components across different techniques.

## Introduction
Dance synthesis driven by music has emerged as an intriguing field that has garnered significant attention. However, creating a group dance that aligns with the style, rhythm, and melody of the music presents a greater challenge compared to single-person dance synthesis, which has been the focus of most prior research. After reviewing and analyzing the methodologies proposed in two papers—Music2Dance: DanceNet for Music-driven Dance Generation and GroupDancer: Music to Multi-People Dance Synthesis with Style Collaboration. 

1. **Music2Dance: DanceNet for Music-driven Dance Generation**: In the paper Music2Dance, they propose an autoregressive model called DanceNet, which utilizes the style, rhythm, and melody of music to generate 3D dance movements with high realism and diversity. 

2. **GroupDancer: Music to Multi-People Dance Synthesis with Style Collaboration**: This paper addresses the issue by constructing a 3D Multi-Dancer Choreography dataset (MDC) and introducing a metric SCEU for style collaboration evaluation. To execute on the MDC dataset, they introduce a new framework, GroupDancer, comprising three stages: Dancer Collaboration, Motion Choreography, and Motion Transition. 

To gain a deeper understanding of the operational methods as well as the strengths and weaknesses of these two papers, a comparative review will be conducted that centers on their approaches and techniques.

## Research Objective
- **`Music2Dance:`**
 The main objective of the "Music2Dance" paper is to generate realistic and diverse dance movements that are not only consistent with the style, rhythm, and melody of the accompanying music but also overcome challenges in synthesizing complex human motions in harmony with the music.
 - **`GroupDancer:`**
 The objective of the "GroupDancer" paper is to propose a new framework called GroupDancer, which aims to automatically generate group dances based on given music and the choreographic preferences of multiple dancers.  To support this goal, the paper also introduces a new dataset, MDC, which contains both individual and group dance data, as well as new metrics for evaluating the framework's performance.

 In summary, the two papers aim to achieve distinct objectives. The "Music2Dance" paper focuses on generating individual dance movements that are in harmony with the style and melody of the music. On the other hand, the "GroupDancer" paper aims to create group dances that take into account the diverse preferences of multiple dancers. Additionally, the "GroupDancer" paper introduces a new dataset, MDC, to support this specific goal.

## Contributions
- **`Music2Dance:`**

  - This paper introduce DanceNet, a unique autoregressive generative model that uses musical style, rhythm, and melody as control factors and employs GMM loss. This approach allows for the creation of dance motions that are lifelike, diverse, and synchronized with the music.
  
  - They have created a high-quality dataset of music-dance pairs featuring professional dancers, which is aligned with two types of musical styles: smoothing and fast-rhythm.
- **`GroupDancer:`**

  - In this paper, they propose a novel problem called Music-driven Multi-Dancer Group Dance Synthesis, which aims to synthesize group dance according to the given music and the choreography preference of multiple dancers
  - Construct a fully-annotated 3D Multi-Dancer Choreography dataset (MDC) and propose a new metric, SCEU, for evaluating style collaboration
 
## Dataset
 **`Music2Dance:`**
  
  - Types of Data:
   
    - Music Data: The dataset includes about 35.7 hours of music, divided into two categories: 18.2 hours of smoothing music suitable for modern dance, and the remaining hours are fast-rhythm music suitable for curtilage dance.
    - Music-Dance Pair Data: This is the core of the dataset, featuring high-quality dance motions captured from professional dancers. It includes 26.15 minutes (94,155 frames, FPS=60) of modern dance and 31.72 minutes (114,192 frames, FPS=60) of curtilage dance, totaling 57.87 minutes or 208,347 frames. Each frame contains 55 joints, including fingers, to improve the artistry of the dance motion.
  - Data Collection Process:
   
    - Playing music and having professional dancers dance to it.
    - Capturing human motions using a motion capture system.
    - Repairing, clipping, and registering motions according to the music.


![Fig-1](https://drive.google.com/uc?export=view&id=1AvMmnZ7XYqKXaBZfpIOPy7b5JM4oOqL_)
*<center>**Figure 1**: Data Collection Process for Music-Dance Pairs.</center>*

 **`GroupDancer:`**
  
  - Dataset Structure:
    - The MDC dataset is constructed based on two main components:
      1.  Dancer Choreography: This part focuses on collecting music-dance pairs for group dances and annotating dancer activation information.

      2.  Dancer-wise Motion Choreography: This part concentrates on gathering music-dance pairs from individual dances and annotating motion information to understand the motion preferences and dance styles of different dancers.
  - Data Collection:
      - Music: Selected from group and individual dance videos on Youtube, Bilibili, and AIST.

      - Dance Motion: Collected from 10 dancers specializing in Hiphop and Locking styles.

  - Dataset Size:
    - The Dancer Choreography part contains 73 music-dance pairs, totaling 60 minutes.

    - The Dancer-wise Motion Choreography part has 165 individual music-dance pairs.

![Fig-2](https://drive.google.com/uc?export=view&id=1ZZ9CEFpyhpsGuOfpZRI99wMIIhAkDE0T)
*<center>**Figure 2**: Multi-dancer group dance choreography procedure.</center>*

**`Comparison:`**
  - Diversity: GroupDancer has a more diverse set of dance types and focuses on both group and individual dances, while Music2Dance focuses mainly on modern and curtilage dances.
  - Size: Music2Dance has a more extensive collection in terms of the total time of dance motions.
  - Quality: Both datasets are high-quality, but Music2Dance includes finger motions, which might add more detail to the dance motions.
  - Annotations: GroupDancer includes unique annotations like dancer activation, which could be beneficial for specific research questions.

## Modeling

**`Music2Dance:`**

DanceNet structure is organized into four main components, each serving a specialized function:
1. Musical Context-Aware Encoder
   - Objective: To encode musical features effectively.
   - Mechanism: Utilizes a combination of temporal convolution and Bi-LSTM.
   - Functionality: Ensures that the encoded music context fully captures the musical rhythm and melody at each moment.

2. Motion Encoder
    - Objective: To encode past frames of dance motion.
    - Mechanism: Comprises two stacked "Conv1D+ReLU" modules.
    - Functionality: Encodes the past k frames, making each frame's motion code independent. The convolution kernel is set to 1, and it operates in 512 channels.

3. Residual Motion Control Stacked Module
   - Objective: To control generated dance motions based on the music and previous dance moves.
   - Mechanism: Employs dilated convolution and incorporates gated activation units inspired by PixelCNN and WaveNet.
   - Functionality: Increases the model's receptive field and enhances feature interaction and model complexity. It uses two separate dilated convolutions for filter and gate features.
4. Motion Decoder
   - Objective: To decode the features into actual dance motions.
   - Mechanism: Consists of two stacked "ReLU+Conv1D" modules.
   - Functionality: Maps the fused features from the residual motion control stacked module to the predicted distribution of dance motion.

![Fig-3](https://drive.google.com/uc?export=view&id=1dIVBMMvHZe_A_kYk7o-KKS-iVBQ2Pb_D)
*<center>**Figure 3**: The overall workflow of DanceNet.</center>*

**`GroupDancer:`**

The architecture of GroupDancer comprising three stages:
1.  Dancer Collaboration Model: This is the first stage where the model predicts dancer activation sequences. It takes acoustic features of the input music as input and produces temporal dancer activation sequences. It uses a Local Musical Encoder and a GRU Decoder for this purpose.
    - Input Features: Acoustic features including Beat, Onset, and Chroma Spectrum.
    - Output: Temporal dancer activation sequences (e.g., "Dancer1&2" means Dancer1 and Dancer2 dance together at time t).

2.  Multi-Dancer Motion Choreography Model: This stage predicts motion sequences for each dancer in a group dance. It uses a seq2seq model for this purpose.
    - Input: Encoded musical features and motion history.
    - Output: Next optional motions for all possible dancer activations.
  
3.  Motion Transition Model: This is the final stage that fills the gap between motion phrases to synthesize a smooth and natural group dance. It consists of three steps: fill the motion gap, align motion with music, and group dance polish.
    - Techniques: Uses Spherical linear interpolation (Slerp) for joint rotations inpainting and linear interpolation for root translations inpainting.
  
![Fig-4](https://drive.google.com/uc?export=view&id=1M9_9teII6R6dqeHp7_MEouE4-pN3IvNE)
*<center>**Figure 4**: The overall workflow of GroupDancer.</center>*

## Limitations

**`Music2Dance:`**

- The generated dance motions are not as good as those performed by professional dancers.
- There is slight foot sliding in the generated dance motions.
- The intrduced dataset have finger motions but the proposal method instead does not consider using it.
- Single dance specific method, which is hard to examine in down-stream group dance task.


**`GroupDancer:`**
- During the training phase, each group sample employs a set number of dancers, which restricts the ability to add and modify the presence of dancers in dance editing tasks.
- The movements of the joints on the character are not smooth.
- Heaavy training structure which is not feasible in practice.

## Conclusion
Both papers "GroupDancer: Music to Multi-People Dance Synthesis with Style Collaboration" and "Music2Dance: DanceNet for Music-driven Dance Generation" make significant contributions to the field of music-driven dance synthesis. However, they differ substantially in their objectives, techniques, and datasets. "GroupDancer" focuses on group dances and style collaboration, while "Music2Dance" aims to generate diverse and music-consistent solo dance motions through its DanceNet model. In the Music2Dance paper, the character's joint movements were more fluid than those in the GroupDancer research, where the motions appeared less smooth. Nonetheless, both methods have limitations in generating complex dance moves that capture emotions, taking into account both the music and the setting.