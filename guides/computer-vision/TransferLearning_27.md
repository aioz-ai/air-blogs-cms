---
last_modified_on: "2022-12-20"
title: Transfer Learning
description: Techniques to support training model deep learning.
series_position: 27
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

To solve any problem with deep learning, the thing that exhausts us the most is the preparation of the dataset. Because, to be able to train a qualified model requires a large amount of data. However, not all problems can be easily collected.
With a small amount of data, we need some methods to help improve this work. Transfer learning was born to solve this problem by taking advantage of models that have been learned very well before through super large datasets, applying Transfer learning so that the model is still good for the object you want to care about.

![Figure 1](https://drive.google.com/uc?export=view&id=1BGmur-qH4qXaPlLAocMOWSH4tib99cJ6)*<center>**Figure 1**. Comparison diagram of model performance before and after applying transfer learning.</center>*

There are two types of transfer learning:
* **Feature extractor**: After extracting the features of the image using ConvNet of the pre-trained model, we will use the linear classifier (linear SVM, softmax classifier, ..) to classify the image. To put it simply, the image features (ears, nose, hair, etc.) are now the input of a linear regression or logistic regression problem.

* **Fine tuning**: After extracting the features of the image using the ConvNet of the pre-trained model, we will treat this as the input of a new CNN by adding ConvNet and Fully Connected layers.

To make it easier to visualize, we will use a simple flower classification problem with VGG 16 architecture that has been trained on the ImageNet dataset.

![Figure 2](https://drive.google.com/uc?export=view&id=1xXAYnOQwKPxQrzfFLx25vpdVH4afaCZ1)*<center>**Figure 2**. VGG-16 architecture illustration.</center>*


## Feature extractor

With this method, we only keep the ConvNet part of the CNN and remove the FCs. Then use the remaining ConvNet output as input for Logistic Regression with multiple outputs.

![Figure 3](https://drive.google.com/uc?export=view&id=1W1qbvMDIsYtenGMthcZT3gmk61LZNrHV)*<center>**Figure 3**. VGG16 with feature extractor method.</center>*

Logistic regression models with multiple outputs come in two forms:
* The first type is a neural network, there is no hidden layer, the activation function at the output layer is a softmax function, the loss function is a categorical-cross entropy function, like an image classification problem.

![Figure 4](https://drive.google.com/uc?export=view&id=1cy7LwVgR9i_XT-KFNaGLx6oNhwzq0aRC)*<center>**Figure 4**. Stage 1: The output layer is softmax function.</center>*

* The second form is like logistic regression, ie the model only classifies 2 classes. Each time we will classify 1 class with all other classes.

![Figure 5](https://drive.google.com/uc?export=view&id=1IEN9Wf6Tv--BXyrvocio9X7JWQT8Edib)*<center>**Figure 5**. Stage 2: The output layer is a logistic regression.</center>*


## Fine tuning
With this method, we only keep the ConvNet part of the CNN and remove the FCs. Then add new Fully Connected layers to the ConvNet output.

![Figure 6](https://drive.google.com/uc?export=view&id=1i3k2jQjZVHdweZchYJhagQlP-11en-3w)*<center>**Figure 6**.VGG16 with fine tuning method.</center>*


We will divide it into two stages to train the model:
* **Stage 1**: Since the fully connected layers we just added have randomly initialized coefficients, but the layers in ConvNet of the pre-trained model have been trained with the ImageNet dataset, so we will not train (freeze/freeze) on layers in ConvNet of model VGG16. After about 20-30 epochs, the coefficients in the new layers have been learned from the data, then we move on to stage 2.

![Figure 7](https://drive.google.com/uc?export=view&id=1kB-a8TNxN5BCMBwEGOwjER1dHVgeFjYI)*<center>**Figure 7**.Stage 1 of the method.</center>*

* **Stage 2**: We will unfreeze the layers on the ConvNet of the pre-trained model and train on the layers of the ConvNet of the pre-trained model and the new layers. You can unfreeze all the layers in VGG16's ConvNet or just unfreeze the last few layers depending on the time and GPU you have.

![Figure 8](https://drive.google.com/uc?export=view&id=1W3At5e47Q3K3MSMQr7AfAxnP1q9gqB7B)*<center>**Figure 8**.Stage 2 of the method.</center>*


Accuracy of fine-tuning is better than feature extractor, but training time of fine-tuning is also much longer. Simply put, the feature extractor only extracts general features from the pre-trained model of the ImageNet dataset for flowers, so it is not very accurate. However, in the fine-tuning part, we add new layers, as well as retrain some layers in ConvNet of VGG16, so the model now learns the properties and characteristics of flowers, so the accuracy is better.

## Experience when using transfer learning
Strategies for applying transfer learning:

![Figure 9](https://drive.google.com/uc?export=view&id=1f2nSqh3djd1rJB67TdqyAGgT_e4jIW6F)*<center>**Figure 9**. The method evaluation graph.</center>*

* For small data: Retraining all layers will lose the features learned from the pretrained model and lead to inaccurate prediction model. We should only retrain the last fully connected layers.
* For big data and domain-like: Can retrain the model on all layers. But to make the training process faster, we will do a warm up step and then fine tune the model.
* For big data and different domain: We should re-train the model from scratch because pretrain-model does not generate good features for data from different domain.

Note: 
* Because the pre-trained model has been trained with a fixed image size, when using the pre-trained model we need to resize the image to the size required by the ConvNet of the pre-trained model.
* The ConvNet learning rate of the pre-trained model should be set to a small value because it has been learned in the pre-trained model so it needs less updating than the newly added layers.

## References

[1] [Transfer learning for deep learning - Machine learning mastery
](https://machinelearningmastery.com/transfer-learning-for-deep-learning/)

[2] [Transfer Learning - C3W2L07 - Andrew Ng](https://www.youtube.com/watch?v=yofjFQddwHE)

[3] [Data Augmentation in Python: Everything You Need to Know](https://neptune.ai/blog/data-augmentation-in-python)