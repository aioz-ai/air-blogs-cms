---
last_modified_on: "2022-01-03"
title: Video recognition and categorization (Part 5 - Applying Deep Learning (cont.))
description: A review about using Deep Learning to solve the video classification task (cont.)
series_position: 10
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: basic", "guides: video_category"]
---
In the previous blog, we had an overview of deep learning-based approach for video classification task. In this blog, we will analysis about its development and summarize some important frameworks.

## Developments In Video Classification
Figure 1 shows a summary of video classification approaches based on their working principle.

![Fig-1](https://vision.aioz.io/thumbnail/11765117e984422db48b/1024/part5-figure1.png)  
*<center>**Figure 1**: Summary of video classification approaches.</center>*

According to the existing research, recently developed state-of-the-art deep learning models outperform earlier handcrafted classical techniques when it comes to video classification. This is mostly owing to the availability of large amounts of video data for training deep learning architectures. Apart from improved classification performance, the newly built models are mostly self-learning and do not require any human feature engineering. This extra benefit makes them more practical for usage in real-world scenarios. However, as compared to previously developed architectures, the higher performing recently developed architectures are deeper, resulting in a tradeoff in the computational complexity of the deep architectures.

Improved Dense Trajectories (iDT) is largely regarded as the state-of-the-art among the hand-crafted representations that were first developed. Many recent competitive research have shown that hand-crafted features, high-level, and mid-level video representations have helped deep neural networks in video classification. Hand-crafted models were one of the first developments in the video categorization challenge.

Later, 2DCNNs for video classification were suggested, in which image-based CNN models are used to extract frame level features, and state-of-the-art classification models (for example, SVM) are learnt to categorize films based on the frame level CNN features. These 2D CNN models do not require any manual feature extraction, and they outperformed hand-crafted techniques. Following the success of 2D CNN models that extract features at the frame level, the same concept was expanded to propose 3D-CNNs for extracting features from videos. In comparison to the 2D CNN models, the suggested 3D CNNs are more computationally costly. However, because these models take into account time variations in feature extraction, they are thought to perform better than 2D CNN models for video classification.

The development of 3D CNN models opened the way for completely automatic video categorization models based on various deep learning architectures.

Spatiotemporal Convolutional Networks are techniques based on the integration of temporal and spatial information using convolutional networks to conduct video classification, which are among the advancements employing deep learning architectures. Convolution and pooling layers are used heavily in these algorithms to capture temporal and geographical data. In two/multi Stream Networks approaches, stack optical flow is employed to identify motions in addition to context frame visuals. Recurrent Spatial Networks, such as LSTM or GRU, employ Recurrent Neural Networks (RNN) to represent temporal information in movies. Mixed convolutional models are built using the ResNet architecture. They are particularly interested in "mixed convolutional" models, which use 3D convolution in the bottom or top layers but 2D in the rest. Methods based on mixed temporal convolution with variable kernel sizes are also included. Aside from these architectures, hybrid approaches based on the integration of CNN and RNN architectures are also available.

## Some Important Frameworks

A summary of some deep learnings architectures for video classification is provided in Table I.

![Fig-2](https://vision.aioz.io/thumbnail/e5e71f606f5a4d17a0d6/1024/part5-table1.png)  
*<center>**Table 1**: Summary and findings of studies based on deep learning models.</center>*

In the next blog, we will conduct a step-by-step deep learning tutorial to build a video classification model.
