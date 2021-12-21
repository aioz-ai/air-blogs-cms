---
last_modified_on: "2021-12-20"
title: Video recognition and categorization (Part 4 - Applying Deep Learning)
description: A review about using Deep Learning to solve the video classification task
series_position: 9
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: basic", "guides: video_category"]
---
In recent years, the work of video categorization has seen a lot of success. The topic has gotten a lot of interest recently, especially because deep learning models have shown to be a viable tool for automatically identifying videos. This blog provides a very detailed and succinct assessment of the topic in recognition of the relevance of the video classification challenge and to describe the effectiveness of deep learning models for this task.

## A Succinct Review about Convolutional Neural Network.

A Convolutional Neural Network (CNN) is a Deep Learning method that can take an input image, give relevance (learnable weights and biases) to various aspects/objects in the image, and distinguish between them. When compared to other classification algorithms, the amount of pre-processing required by a CNN is significantly less. While filters are hand-engineered in basic approaches, CNN can learn these filters/characteristics with enough training.
The design of a CNN is inspired by the arrangement of the Visual Cortex and is akin to the connection pattern of Neurons in the Human Brain. Individual neurons can only respond to stimuli in a small area of the visual field called the Receptive Field. A number of similar fields can be stacked on top of each other to span the full visual field.

![Fig-1](https://vision.aioz.io/thumbnail/e51037fc54b64ad6baba/1024/part4-figure1.jpeg)  
*<center>**Figure 1**: A image classification framework using CNN [(Source)](https://towardsdatascience.com/a-comprehensive-guide-to-convolutional-neural-networks-the-eli5-way-3bd2b1164a53).</center>*

## Convolutional Neural Network for Video Classification Task.

This section provides a very thorough and succinct overview of deep learning models used in video classification tasks. This section discusses video data modalities, breakthroughs in video classification, and state-of-the-art deep learning models for video classification.

### Video Data Modalities.

Due to the complicated structure of the temporal material, videos are more difficult to analyze and classify than images. However, to classify videos, three separate modalities, namely visual information, audio information, and text information, may be provided, as opposed to image classification, which can only use one visual modality. The task of classification may be classified as Uni-modal video classification or Multi-modal video classification based on the availability of distinct modalities in movies, as shown in Fig. 2. For the video classification task, the current research has used both of these models, and it is widely accepted that models based on Multi-modal data perform better than models based on Uni-modal data. Furthermore, the visual representation of a video performs better than the text and audio description in terms of video categorization.

![Fig-2](https://vision.aioz.io/thumbnail/ec4dc5271033497a92bf/1024/part4-figure2.png)  
*<center>**Figure 2**: Different modalities used for classification of videos.</center>*

### Deep Learning Frameworks for Video Classification Task.

The trend for video classification tasks has shifted from traditional handcrafted approaches to completely automated deep learning approaches in recent years, as more powerful deep learning architectures have been developed. A 3D-CNN model is one of the most widely used deep learning architectures for video categorization. Figure 3 shows an example of a 3D-CNN architecture for video categorization. 3D blocks are used in this architecture to collect the video information needed to identify the video material. A multi-stream architecture, in which spatial and temporal information is analyzed separately and features extracted from distinct streams are subsequently merged to generate a decision, is another highly prevalent framework.

![Fig-3](https://vision.aioz.io/thumbnail/2fefca0763b44b8d968d/1024/part4-figure3.png)  
*<center>**Figure 3**: An example of 3D-CNN architecture to classify videos ([Source](https://ieeexplore.ieee.org/document/8695806)).</center>*

Different approaches are utilized to process temporal data, with the two most prominent methods being: (i) RNN (mostly LSTM) and (ii) optical flow. Figure 4 shows an example of a multi-stream network model in which the temporal stream is handled using optical flow.

![Fig-4](https://vision.aioz.io/thumbnail/725faa9e21514a5a882b/1024/part4-figure4.png)  
*<center>**Figure 4**: An example of two stream architecture with optical flow ([Source](https://ieeexplore.ieee.org/document/7899964)).</center>*

 Figure 5 depicts a high-level overview of the video categorization process. Where the phases of feature extraction and prediction are depicted with the most often used strategies in the literature. The breakthrough in video classification and research related to video classification especially using deep learning frameworks are discussed in the following sections, along with the success rate and limits of employing deep learning architectures.
 
![Fig-5](https://vision.aioz.io/thumbnail/d92c9ddbfce747c2b251/1024/part4-figure5.png)  
*<center>**Figure 5**: An overview of video classification framework.</center>*

### Basic Deep Learning Architectures for Video Classification.

Convolutional Neural Network (CNN) and Recurrent Neural Network (RNN) are the two most extensively utilized deep learning architectures for video classification (RNN). RNNs are used to learn the temporal information from videos, whereas CNNs are used to learn the spatial information. The capacity to comprehend temporal information or data in sequences is the main difference between these two architectures. As a result, in general, each of these network architectures are employed for fundamentally different purposes. However, the nature of video data, which contains both spatial and temporal information, requires the usage of both of these network architectures to handle the two-stream data properly.

To transform data, the architecture of a CNN uses different filters in the convolutional layers. RNNs, on the other hand, utilize the activation functions to produce the next output in a series from the previous data points. However, using only 2D CNNs limits video interpretation to only the spatial domain. RNNs, on the other hand, are capable of comprehending a sequence's temporal content. Several research have used both these fundamental architectures and their upgraded versions for video categorization.

In the next blog, we will discuss about "Developments In Video Classification Over Time" and "Summary about Some Important Frameworks".
