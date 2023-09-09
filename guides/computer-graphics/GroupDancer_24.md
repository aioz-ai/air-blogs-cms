---
last_modified_on: "2023-08-25"
title: Music to Multi-People Dance Synthesis with Style Collaboration.
description: GroupDancer, the Music to Multi-People Dance Synthesis with Style Collaboration
series_position: 24
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: groupdance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Dance synthesis driven by music has emerged as a captivating field, presenting a myriad of applications in industries such as entertainment, advertising, and virtual performances. Recent advancements in deep learning techniques have led to the development of various methods for synthesizing dance motions that are synchronized with music. This synopsis endeavors to offer an inclusive exploration of the cutting-edge investigations into dance generation with music, achieved through the juxtaposition and analysis of pivotal elements within distinct methodologies.

## 1. Objectives 
The research objective of the paper "GroupDancer: Music to Multi-People Dance Synthesis with Style Collaboration" is to tackle the challenge of generating group dance performances that involve multiple dancers, each with their distinct choreographic preferences, in response to provided music inputs. To accomplish this goal, the paper introduces a richly annotated 3D Multi-Dancer Choreography dataset (MDC) and presents a three-stage framework known as GroupDancer. This framework encompasses the stages of Dancer Collaboration, Motion Choreography, and Motion Transition. 

## 2. Contributions
This paper introduces a new research direction in dance synthesis – the synthesis of group dance performances with style collaboration among multiple dancers in response to music. The key contributions include the creation of a rich-annotated 3D Multi-Dancer Choreography dataset (MDC), the development of the GroupDancer framework consisting of three stages for synthesizing collaborative group dances and the proposal of a novel evaluation metric for style collaboration. The framework is demonstrated to outperform baseline methods in terms of group dance diversity and dancer collaboration through comprehensive evaluations and user studies on the MDC dataset. 
  
## 3. Dataset
The authors use a dataset called the 3D Multi-Dancer Choreography dataset (MDC). This dataset is specifically constructed to facilitate the research objective of synthesizing group dance performances with style collaboration. The MDC dataset comprises two main parts:
- Individual Dance Annotations: The first part of the dataset includes annotations from 10 professional dancers. These annotations involve 725 3D dance motions from two dance styles, namely Hiphop and Locking. For each dancer and dance type, there are motion phrase sequences created for each of the music tracks, totaling 15 minutes of music. This part provides the basis for individual dancers' choreography preferences and motion sequences.
- Group Dance Choreography: In the second part, the dataset contains 73 music pieces (totaling 60 minutes) from pop music. Each music piece is accompanied by temporal dancer activation sequences. Professional dancers are invited to annotate the activation sequences, which represent the moments when different dancers start their motions during group dances. This part of the dataset captures the essence of style collaboration, where dancers synchronize their actions while maintaining individuality.

## Modeling
In this article, three main stages are introduced to facilitate the generation of dance steps for multiple individuals simultaneously with style collaboration. Here is a description of these stages:
  - Dancer Collaboration Model: This stage focuses on determining the timing and participants for collaboration in dance movements. The model learns how to map from input music to activation sequences of dancers. This helps decide when and who should collaborate in creating dance steps.

  - Motion Choreography Model: In this stage, the model identifies sequences of specific dance steps for each dancer based on the input music and the dancer activation sequences from the previous stage. The goal is to generate step sequences that fit each dancer's style as well as the playing music.

  - Motion Transition Model: This stage is responsible for filling the gaps between dance steps to create seamless and natural transitions for the entire group. This model helps connect the generated dance steps into a complete dance performance.

![Fig-3](https://drive.google.com/uc?export=view&id=1RH-_yKy-MOs0p25rG9S46Bpm9BBL6eSq)

## Experimental results 

![Fig-4](https://drive.google.com/uc?export=view&id=1jbGmV7abi6kVVm1ucGD4QHjt1GwyUFY9)

Looking at the table, several important observations can be made:
  - Data Scope: MDC is the first rich-annotated 3D dance dataset that encompasses both individual music-dance pairs and collaborated group dance music-dance pairs. This unique characteristic of MDC allows it to address both individual and group dance synthesis.

  - Integration of Music and Dance: In MDC, both individual and group dance data come with corresponding music. This advantageous integration of music and dance facilitates a strong correlation between the two elements, ensuring coherence and alignment.

  - Diversity: MDC includes a variety of dance styles and genres, enhancing the diversity of the dataset compared to other datasets.

  - Detailed Annotations: MDC provides detailed annotations of dancer activations for each music segment. These annotations establish a clear link between dancers and music, creating a conducive environment for high-quality dance synthesis.

- The results are compared between different methods based on the mean value of the SCEU𝑛 index. (Style Collaboration Evaluation Understudy). SCEU𝑛 index. measures the degree of stylistic cooperation in the creation of dance moves. The values compared in the table are the result of three different methods, including:
  - DBD: A method utilizing the DBD mixed training strategy.
  - PBP: A method arranging music-dance pairs piece by piece.
  - RS: A method randomly shuffling music-dance pairs.
  - 
![Fig-5](https://drive.google.com/uc?export=view&id=15cPygQDMNrsnXVPYhFYZHE5wxFTD_8gD)
Looking at the table above, it is evident that the DBD method consistently outperforms the other methods in terms of effectively promoting style collaboration for dance motion synthesis, as evidenced by its consistently higher SCEU𝑛. scores across all indices. While the PBP and RS methods also display promising results, the DBD method stands out with a stronger emphasis on generating balanced and collaborative dance motions.