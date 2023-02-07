---
last_modified_on: "2023-02-04"
title: Supervised representation learning
description: In this article, we will explore Self Supervised Learning (SSL) is a hot research topic in a machine learning community.
series_position: 30
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In the article on [Representative Learning](https://ai.aioz.io/guides/computer-vision/OverviewRepresentatonLearning/#supervised-and-unsupervised-learning), we have introduced to you an interesting learning method. To understand more about Presentation Learning, we will analyze it more carefully

Performing learning can be divided into:
* **Supervised representation learning**: learning representations on task A using annotated data and used to solve task B.
* **Unsupervised representation learning**: learning representations on a task in an unsupervised way (label-free data). These are then used to address downstream tasks and reducing the need for annotated data when learning news tasks. Powerful models like GPT and BERT leverage unsupervised representation learning to tackle language tasks.

In this article, we will learn about Supervised representation learning ( Self-supervised learning).

## Introduction
Maybe you will confuse Self-supervised learning and Supervised Learning, Unsupervised Learning. To understand better let's go back to their concepts.

* **Supervised Learning** are algorithms that use labeled data to model the relationship between input data and their labels. Supervised learning has very high requirements on labeled data, often data sets will have to be labeled by humans, some datasets that require high accuracy and importance often take a lot of resources. , moreover, labeled tuples are not necessarily comprehensive and relevant.
* **Unsupervised Learning** are algorithms that use unlabeled data. These algorithms often aim to model the structure or hidden information in the data, thereby describing the characteristics and properties of the data set. These methods are often used in the process of analyzing and visualizing data.

Obviously it is very expensive to generate a good dataset with full labels, but unlabeled data is always generated. To take advantage of this much larger amount of unlabeled data, one approach can learn key features from within the data. The fact that the model can learn key features from within the data can be used to improve the model of some tasks on that data set, and such approaches are commonly known in this study Self-supervised representation learning.
![Figure 1](https://drive.google.com/uc?export=view&id=1oDeABAIEaerKY_UabF0wYquqebhOoNRi)*<center>**Figure 1** Workflow Classification. </center>*
* **Self-Supervised Learning** is proposed for utilizing unlabeled data with the success of supervised learning. Producing a dataset with good labels is expensive, while unlabeled data is being generated all the time. The motivation of Self-Supervised Learning is to make use of the large amount of unlabeled data. The main idea of Self-Supervised Learning is to generate the labels from unlabeled data, according to the structure or characteristics of the data itself, and then train on this unsupervised data in a supervised manner. Self-Supervised Learning is wildly used in representation learning to make a model learn the latent features of the data. This technique is often employed in computer vision, video processing and robot control.

![Figure 2](https://drive.google.com/uc?export=view&id=1KyzweiPS1KeB0Tp9Wv8nyiZGefIADtZX)*<center>**Figure 2**. Self-supervised models </center>*
The self-supervised models after being trained will be used to finetune (using the parameters of the trained model for the self-supervised problem for the main task model) for downstream tasks. (the tasks the model wants to learn) makes the models more robust, thereby increasing the model's performance. For example, we will use a backbone parameter that is learned from unlabeled data (car, house, cat ...) through the process of training a self-supervised model to train the classification model main.

## Classification 
Self-supervised learning is classified into 2 main categories: Contrasting and Non-Contrasting.

* **Contrastive self-supervised learning**: contrast supervised learning is an alternate learning method that uses both positive and negative data types. The two main functions of this learning are:
1. To reduce the gap between active data to a minimum.
2. To increase the distance between negative data to the maximum.
![Figure 3](https://drive.google.com/uc?export=view&id=1w3ylJtChVMoqUjchrzdZt5oVRNgJDicZ)*<center>**Figure 3**. The idea of Contrastive self-supervised learning </center>*

* **Non-contrastive self-supervised learning**: is a learning paradigm where only positive sample pairs are used to train a model, unlike in Contrastive Learning where both positive and negative pairs are used. This seems counterintuitive, since it appears like only trying to minimize distances between positive pairs may collapse into a constant solution.

* The main idea of **Contrastive Predictive Coding** is to first divide the whole image into a grid and inform the upper rows of the image, the task is to predict the lower rows of the same image. To perform this task, the model must learn the structure of the object in the image (for example, seeing the face of a dog, the model must predict that it will have 4 legs). Having the model trained like this will have a huge effect on the downstream task.


![Figure 4](https://drive.google.com/uc?export=view&id=1WKBixXnKODcRSUPa8gWyXjaSpCOp70-f)*<center>**Figure 4**. The idea of Contrastive Predictive Coding. </center>*
## Application
With the ability to learn on unlabeled data sets, self-supervised learning is widely applied to many problems with large amounts of unlabeled data such as voice recognition, signature detection...

![Figure 5](https://drive.google.com/uc?export=view&id=1Za1uOwYhBEva0Hzl-LpH70BXzG-9Fc5M)*<center>**Figure 5**. Possible fields of application. </center>*

## Conclusion
Self-supervised learning is an emerging technology. It is useful in all aspects of life, with applications in healthcare, editing, forecasting, driving, chatbots, video editing, converting and getting data formats… However, it still exists. There are some limitations to be aware of, but once these barriers are overcome Self-Monitored Learning could become the new standard in the technology sector.

##  References
1. [Representation Learning with Contrastive Predictive Coding](https://arxiv.org/pdf/1807.03748.pdf)
2. [The Beginner's Guide to Self-Supervised Learning](https://www.v7labs.com/blog/self-supervised-learning-guide)
3. [Contrastive Predictive Coding (CPC)](https://arxiv.org/pdf/1807.03748.pdf)