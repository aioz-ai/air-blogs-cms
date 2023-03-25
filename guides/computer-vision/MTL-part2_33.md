---
last_modified_on: "2023-03-25"
title: Multi Task Learning and its effectiveness
description:   Multi Task Learning and its effectiveness
series_position: 33
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
In the last article, we learned the basics of MultiTask Learning together. In this article, we will take a look at the advantages and disadvantages that MultiTask Learning brings and how to use this method effectively. 

## I. The benefits of the MTL method.
Let's assume that we have 2 related tasks A and B, I will show some benefits of Multi Task Learning when learning both A and B simultaneously which relies on a common hidden layer representation F.

**1. Implicit data augmentation**

 MTL effectively increases the sample size that we are using for training our model. As all tasks are at least somewhat noisy, when training a model on some task A, our aim is to learn a good representation for task A that ideally ignores the data-dependent noise and generalizes well. As different tasks have different noise patterns, a model that learns two tasks simultaneously is able to learn a more general representation. Learning just task A bears the risk of overfitting to task A, while learning A and B jointly enables the model to obtain a better representation F through averaging the noise patterns.
 
**2. Attention focusing**

If a task is very noisy or data is limited and high-dimensional, it can be difficult for a model to differentiate between relevant and irrelevant features. MTL can help the model focus its attention on those features that actually matter as other tasks will provide additional evidence for the relevance or irrelevance of those features.

**3. Eavesdropping**

Some features G are easy to learn for some task B, while being difficult to learn for another task A. This might either be because A interacts with the features in a more complex way or because other features are impeding the model's ability to learn G. Through MTL, we can allow the model to eavesdrop, i.e. learn G through task B. The easiest way to do this is through hints, i.e. directly training the model to predict the most important features.

**4. Representation bias**

MTL biases the model to prefer representations that other tasks also prefer. This will also help the model to generalize to new tasks in the future as a hypothesis space that performs well for a sufficiently large number of training tasks will also perform well for learning novel tasks as long as they are from the same environment.

**5. Regularization**

Finally, MTL acts as a regularizer by introducing an inductive bias. As such, it reduces the risk of overfitting as well as the Rademacher complexity of the model, i.e. its ability to fit random noise

## II. How should I use MTL to get the most out of it?
**Tasks have a common classification feature**: In Multitask learning, tasks will use a common feature to distinguish them. Therefore, if the features that help classify these tasks are not related and support each other in the classification, the model will not achieve high accuracy.

**The data size between classes is similar:** Suppose they need to simultaneously classify 1000 different tasks, each of which recognizes a class and consists of 100 images. Thus, when using multitask learning, to recognize a single task T we will benefit from 99900 features learned from the remaining 999 tasks. 99900 images is quite a large number so the learned features will be more diverse and help improve the single task T.

![Figure 1](https://drive.google.com/uc?export=view&id=1DgC2GIyYrqswCr5BrJVcHeqIHxhk9-39)

On the contrary, if there is a data imbalance. Task T takes up 99,000 images and the remaining tasks take up 1000 images. Thus, most of the features learned from the network will mainly have specific characteristics of task T and easily lead to a poor predictive model on the remaining tasks.

**Training on a large neural network:** As the number of classes increases, the probability of wrong class prediction is greater, so the prediction accuracy decreases and is inversely proportional to the number of classes. This has been verified in object detection models. Multitask learning models will train on more classes than individual classification models. Therefore we need to use a larger neural network size to learn more features. Thereby helping to improve accuracy on each task.