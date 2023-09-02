---
last_modified_on: "2023-09-02"
title: DanceNet for Music-driven Dance Generation
description: Music2Dance, DanceNet for Music-driven Dance Generation
series_position: 23
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: groupdance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Dance synthesis driven by music has emerged as a captivating field, presenting a myriad of applications in industries such as entertainment, advertising, and virtual performances. Recent advancements in deep learning techniques have led to the development of various methods for synthesizing dance motions that are synchronized with music. This synopsis endeavors to offer an inclusive exploration of the cutting-edge investigations into dance generation with music, achieved through the juxtaposition and analysis of pivotal elements within distinct methodologies.

## 1. Objectives 
The research aims to overcome challenges in generating intricate human movements synchronized with music by introducing the innovative DanceNet autoregressive model. This model is designed to produce diverse and authentic dance motions aligned with music rhythm, style, and melody. The research focuses on two key objectives: enhancing the model's adaptation by incorporating dilated convolutions and gated activation mechanisms for more efficient architecture, and increasing model robustness and diversity through Gaussian Mixture Model (GMM) loss and the introduction of Gaussian noise during training. Ultimately, the research strives to enable automated choreography driven by music, offering a novel approach to generating diverse and music-consistent dance motions.                                                    


## 2. Contributions
The paper introduces DanceNet, a novel autoregressive generative model, which utilizes musical elements like style, rhythm, and melody to generate 3D dance motions. The integration of a Gaussian Mixture Model (GMM) loss during training enhances the model's robustness and diversity in generated motions. Additionally, a high-quality music-dance pair dataset is curated, featuring intricate dance motions including finger movements. Experimental results demonstrate DanceNet's superiority over existing models, achieving state-of-the-art performance in producing realistic and music-consistent dance motions. These contributions collectively advance the field of music-driven dance generation.

  
## 3. Dataset
`Music Data Collection:`
  - The music dataset spans approximately 35.7 hours and encompasses a wide variety of musical styles.
  - Musical styles are categorized into two main types: "smooth music" and "fast-rhythm music."
  - Around 18.2 hours of music fall under the category of smooth music.
  - The remaining portion consists of fast-rhythm music, suitable for folk dance.

`Creating Music-Dance Pairs:`
  - The data collection process unfolds in three primary steps:
    - Performance to Music: Professional dancers perform dance routines while the selected music is played.
    - Motion Capture: Motion capture technology is employed to capture detailed human dance movements, recording the positions of various body joints.
    - Refinement and Alignment: The collected dance motions are subjected to thorough refinement. This involves repairing, clipping, and aligning the motions to match the corresponding music.

This dataset encompasses a substantial volume of dance motions, with a total of 208,347 frames captured. These frames collectively span a duration of 57.87 minutes. Notably, each frame of the dance motion data includes detailed information about the positions of 55 body joints, including intricate finger motions. This level of detail enhances the authenticity and expressiveness of the generated dance motions.


## Modeling
The modeling approaches employed in the paper:
  - `DanceNet`: This is an autoregressive learning model that generates new dance motions. DanceNet utilizes information about the style, rhythm, and melody of the music as control signals to create diverse and realistic 3D dance motions. The DanceNet model employs a temporal convolutional structure combined with gated activations to enhance the model's ability to capture complex, irregular, and diverse dance motions.
  - `Gaussian Mixture Model (GMM) Loss`: During DanceNet training, the paper uses GMM loss instead of the conventional Mean Squared Error (MSE) loss in motion generation models. The GMM loss improves training dynamics and contributes to generating more diverse dance motions.
  - `Dilated Convolution`: An integral part of DanceNet is the utilization of dilated convolutions rather than regular temporal convolutions. Dilated convolutions expand the receptive field of the model without increasing the network depth. This allows the model to effectively handle complex and irregular dance motions.
  - `Contact-Aware Motion Representation`: To capture dance motions that exhibit interactions with the ground, DanceNet employs a motion representation that includes information about the positions of endpoints and foot contacts. This contributes to generating more realistic dance motions that reflect ground interactions.
  - `Control Signals`: The model utilizes musical style, rhythm, and melody as control signals to modulate the dance motion generation process. Musical style determines the type of dance, while rhythm and melody dictate the dance's tempo and other characteristics such as amplitude and velocity.


## Experimental results 
To assess the effectiveness of musical style classification, the paper compares it with Convolution RNN and constructs a baseline FC+Bi-LSTM model:
![Fig-1](https://drive.google.com/uc?export=view&id=17gZxahGiNkhEDfICOWcE6yy8KgjTmJ3k)

  - Convolution RNN[10]: This method achieves an accuracy of 89.6% in classifying musical styles.
  - Baseline (FC + Bi-LSTM): The baseline approach, which involves a combination of fully connected layers and Bidirectional LSTM, achieves an accuracy of 88.0% in classifying musical styles.
  - Temporal Conv + Bi-LSTM:   utilizing a combination of temporal convolution and Bidirectional LSTM, outperforms the other methods with an accuracy of 92.1% in classifying musical styles.


To assess the dance motion generation performance, the study compares it with several cutting-edge motion modeling methods. It contrasts with LSTM-autoencoder, which directly generates 3D dance motions from music, and Temporal Convolution (Temporal Conv), which models gestures from speech. The LSTM model is an autoregressive approach, but using Mean Squared Error (MSE) loss to model dance motions proved challenging. Therefore, they replaced MSE with Gaussian Mixture Model (GMM) loss. The paper generated 20 dance motions (10 modern and 10 curtilage) using DanceNet and evaluated them against the aforementioned methods based on realism, diversity, style-consistency, rhythm-consistency, and melody-consistency criteria.
  
![Fig-2](https://drive.google.com/uc?export=view&id=1_NnsWV1Ru7lgKHaxfUn9dgEaE_jumC8w)

- Comparing FID scores, the authors observe the following evaluations across various methods in the "Real Dance Movements" category: LSTM-autoencoder achieved a score of 82.1, indicating that the dance motions it generated might lack authenticity and accurate capture of the essence of real dance movements. In contrast, Temporal Conv scored 36.7, LSTM scored 27.8, the authors (without GMM loss) scored 32.3, and the proposed method attained the lowest score of 12.5. This demonstrates that the approach consistently outperforms other methods in terms of realism and fidelity when generating dance motions. Additionally, when considering the Diversity metric, the method achieved a score of 52.5, and a Rhythm Hit rate of 58.7%, both of which surpass other methods.
