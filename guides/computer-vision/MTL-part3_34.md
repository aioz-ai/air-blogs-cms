---
last_modified_on: "2023-04-02"
title: Multi Task Learning and its correlation with Neural Network
description:   Multi Task Learning and its correlation with Neural Network
series_position: 34
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
Through two articles about MTL, you must have somewhat understood the basic knowledge as well as the benefits of the MTL algorithm. In the last article of this MTL topic, we will cover more about MTL in Neural Network.

## Two MTL methods for Deep Learning
MTL has many different uses, but in the context of Deep Learning, I would only like to introduce two methods: Hard Parameter Sharing and Soft Parameter Sharing (See Figure.1 for more details).

![Figure 1](https://drive.google.com/uc?export=view&id=1ULoRqKrDbghNNDrmsznI-jBCgVa8CuV5)*<center>**Figure 1:** Hard Parameter Sharing v.s Soft Parameter Sharing  </center>*

**Hard parameter sharing** is the most commonly used approach to MTL in neural networks and goes back to. It is generally applied by sharing the hidden layers between all tasks, while keeping several task-specific output layers.

Hard parameter sharing greatly reduces the risk of overfitting. In fact, showed that the risk of overfitting the shared parameters is an order N -- where N is the number of tasks -- smaller than overfitting the task-specific parameters, i.e. the output layers. This makes sense intuitively: The more tasks we are learning simultaneously, the more our model has to find a representation that captures all of the tasks and the less is our chance of overfitting on our original task.

**Soft parameter sharing**
Soft Parameter Sharing is completely different, each task will have its own model, as well as its own parameters, however the distance of the parameters between tasks will then be constrained to make these parameters there is a high degree of similarity between the tasks. Here we can use Norms to constrain.

If you have heard of Ridge Regression, this method has a lot of similarities with it, except that Ridge Regression constrains the parameter intensity, and Soft Parameter Sharing will constrain the distance.


## Auxiliary tasks

In many cases, we are only interested in the performance of a particular task, however we want to take advantage of the benefits that MTL brings, in cases like this we can add some The task is related to the main task that we are interested in with the aim of further improving performance on the main task. These tasks are called Auxiliary tasks (See Figure.2).
![Figure 1](https://drive.google.com/uc?export=view&id=1BM8DtBW2pk_w-3Sf6m7MXAc5nGeR_t_g)*<center>**Figure 2:**  Auxiliary task under neural network. </center>*

How the use of Auxiliary tasks compares to the Main task is a matter of long research, but there is no solid theoretical evidence that using Auxiliary tasks will bring an improvement to the Main task. Below I will show some suggestions from the experience of previous studies regarding how to use Auxilirary tasks effectively.

Specific ancillary tasks can be customized differently, but in general all Auxiliary Tasks can be grouped into the following 4 categories.

**Statistical:** These are the simplest ancillary tasks, we will use the tasks that predict the statistical properties of the data such as the Auxiliary Task. for example guessing Log Freqency of a word, Pos Tag, etc

**Selective Unsupervised:** These are tasks that require us to predict part of the input, specifically in Sentiment Analysis we can predict when a sentence contains a Positive Sentiment or a Negative Sentiment Word.

**Supervised:** These tasks can be adversarial tasks or inverse tasks or data-related monitoring tasks (for example, Predict Inputs). .

**Unsupervised:** These are tasks that involve predicting all parts of the input data.

## Conclusion

Because of the increased importance of multi-task learning, a large number of methods have been proposed to automatically learn which tasks should be jointly learned. However, these methods have not been exhaustively evaluated, especially on large numbers of tasks and on domains outside of computer vision. New methods may be needed to scale these methods to tens or hundreds of tasks and on other domains where multitasking is important, such as natural language processing and biomedical data.

I hope you found this overview helpful. If I made any error, missed a reference, or misrepresented some aspect, or if you would just like to share your thoughts, please leave a comment below.

## References

1. Yu, J., & Jiang, J. (2016). Learning Sentence Embeddings with Auxiliary Tasks for Cross-Domain Sentiment Classification. Proceedings of the 2016 Conference on Empirical Methods in Natural Language Processing (EMNLP2016), 236-246. Retrieved from http://www.aclweb.org/anthology/D/D16/D16-1023.pdf

2. Sun, X., Panda, R., Feris, R., & Saenko, K. (2020). Adashare: Learning what to share for efficient deep multi-task learning. Annual Conference on Neural Information Processing Systems, NeurIPS, December 6-12, 2020.

3. An Overview of Multi-Task Learning in Deep Neural Networks

4. How to Do Multi-Task Learning Intelligently