---
last_modified_on: "2023-03-18"
title: A kind introduction to Multi Task Learning
description:  A kind introduction to Multi Task Learning
series_position: 32
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
In Deep Learning, in order to save time as well as the cost of building many models to complete many different tasks, people will use the MultiTask Learning method. Find out the commonalities of the tasks from which to build a model that can solve many tasks at the same time. Although this method is not too new, it is still widely used by people today. In this article, let's learn more about it together.
# I. Introduce
Multi-Task Learning (MTL) is a type of machine learning technique where a model is trained to perform multiple tasks simultaneously. In deep learning, MTL refers to training a neural network to perform multiple tasks by sharing some of the network layers and parameters across tasks. Figure 1 illustrates the differences between single task learning and multi-task learning.

![Figure 1](https://drive.google.com/uc?export=view&id=1aMlF_WQMTlL2OhtsOeKErPrDvd2KocEl)*<center>**Figure 1:** Single-task vs. Multi-task learning </center>*

In MTL, the goal is to improve the generalization performance of the model by leveraging the information shared across tasks. By sharing some of the network parameters, the model can learn a more efficient and compact representation of he data, which can be beneficial when the tasks are related or have some commonalities.

Multi Task Learning also known by some names like Joint Learning, Learning to Learn, Learning with Auxiriary Task, every time we work with an optimization problem with more than one loss function, we will think think we are solving a problem involving Multi Task Learning.

# II. Multi Task Learning
There are different ways to implement MTL in deep learning, but the most common approach is to use a shared feature extractor and multiple task-specific heads. The shared feature extractor is a part of the network that is shared across tasks and is used to extract features from the input data. The task-specific heads are used to make predictions for each task and are typically connected to the shared feature extractor.
## 1. Architecture

The architecture of multitasking learning is basically similar to Transfer Learning and consists of 2 phrases:
* Phrases 1: The base network has the function of doing feature extractor. Note that in the multitask learning algorithm, the feature extractor will produce outputs that are common to all tasks.
* Phrases 2: Perform multiple sorting tasks. The common features extracted from phrase 1 will be used as input for ***N*** different Binary Classification problems. Our output will consist of many units (Multi-heads) that each compute the probability of a binary classification task.

Figure.2 below shows the workflow of the multi-task learning under the classification concept.

![Figure 2](https://drive.google.com/uc?export=view&id=1yUpFquqRGCRa8WJnWe_YKN653BwMM1A9)*<center>**Figure 2:** Workflow Classification. </center>*

Distinguish between Transfer Learning and Multi Task Learning, we can see that Multitask learning is the process of performing multiple binary classification problems simultaneously on the same input. Therefore the probability for each binary classification task will be calculated based on the sigmoid function.

In contrast, Transfer learning is a classification problem with classes, so the probability distribution is a softmax function (See Figure.3).

![Figure 3](https://drive.google.com/uc?export=view&id=1vSLeQU5e421z0Po0nf9UEnoHyIDtFGEl)*<center>**Figure 3:** Transfer Learning process </center>*

In addition to the difference in the activation function that calculates the probability distribution, both machine learning methods also have a difference in the loss function that we will learn in the next section.

## 3. Loss function
Let's review some of the basics:
1. For the binary classification problem, the loss function has the form of :

![Figure 5](https://drive.google.com/uc?export=view&id=1Af3XnkLat6nqsxnGFJAsHpQWvcJ9PFxN)

2. In the case of a classification problem with n labels (n more than 2 labels). At the same time we use the sorfmax function to calculate the output probability distribution, the loss function is a cross entropy function as follows:

![Figure 6](https://drive.google.com/uc?export=view&id=1TWWXnccye69CjxH-5tFr8H9WIqloJRXH)

3. In the multitask learning algorithm, for each classification task, the loss function value will be:

![Figure 7](https://drive.google.com/uc?export=view&id=1jwsPjviBUDKdGi7LbMLvAHFH_5MSoYTI)

when there are ***N*** different classification tasks, their aggregate loss function will be:
![Figure 8](https://drive.google.com/uc?export=view&id=1hP4eGdEaBdXZQCpUM3sLZvZaskKRzb9O)


In there: *i* is the index of the sample, *j* is the index of each task.

In essence, the loss function of multitask learning is the sum of the loss functions (binary cross entropy) of each corresponding binary classification problem of each task.