---
last_modified_on: "2022-02-19"
title: Representation Learning
description: A review about representation learning and its variants.
series_position: 13
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: representation_learning"]
---
Feature learning, also known as representation learning, is a collection of approaches in machine learning that allows a system to discover the representations needed for feature detection or classification from raw data automatically. This eliminates the need for manual feature engineering by allowing a machine to learn and use features to execute a specific task.

## Generative and Discriminative models

In machine learning, methods can be divided into two big categories: generative and discriminative modeling. With the same data sample $\bold{x}$ and its corresponding explanatory variables $y$ , both approaches try to capture the underlying factors that explain $\bold{x}$ in their own way. 

![**Figure 1: Discriminative and generative models of handwritten digits**](https://vision.aioz.io/f/9f95e06236fa4a3bb412/?dl=1)

For generative modeling, the target is to learn the distribution $p(\bold{x}|y)$. The meaning of this distribution is to learn the representation of the data under an explanatory factor, e.g., $y$ is the human face, then the target distribution will capture the underlying feature that represents the human face. To perform generative modeling on discriminative task, i.e., construct distribution $p(y|\bold{x})$, one can utilize Bayes’ rule to estimate. The classic example is the Naive Bayes classifier. Note that, $p(\bold{x},y)$ and $p(\bold{x})$ are another definition for generative model.

For discriminative modelling, the target is to learn the distribution $p(y|\bold{x})$. The meaning of this distribution is to map the data sample to a explanatory factor, or to distinguish decision boundaries, e.g., for $x$ is a image of a dog, and $y=(\text{dog, cat})$, then $p(y=\text{dog}|x) > p(y=\text{cat}|x)$. Discriminative modelling include two main tasks, learn the representation of $\bold{x}$ by infer distribution of latent variable $\bold{v}$, i.e., $p(\bold{v}|\bold{x})$, then using that representation to make decision on downstream task $p(y|\bold{v})$. 

## Supervised and Unsupervised learning

Supervised learning has gained significant attention nowadays. Especially in common tasks like classification, recognition, or segmentation. In a supervised learning setting, the representation is directly learned from mapping the input to the ground truth label. The most notable example in this setting is pretrained models from the ImageNet dataset, e.g. ResNet, VGG. While supervised learning is effective, the cost is very expensive. To create ground truth labels, labeling by hand is needed. Besides the cost, other harmful factors like privacy threats, biases also exist. Moreover, the demand to use a large amount of data at this time is huge. It is well-known that the performance can scale upwards with data and model size [1]. Therefore, unsupervised learning becomes another important field of interest.

![**Figure 2: Examples for famous supervised learning task**](https://vision.aioz.io/f/874f01eb9c524bba9b08/?dl=1)

**Figure 2: Examples for famous supervised learning task**

Until recently, unsupervised learning methods are mostly followed generative modeling. The path of using discrimination modeling in an unsupervised learning setting is called “self-supervised learning”. The main idea of self-supervised learning is to generate a pseudo label from the input data itself [2]. Under this new idea, newer works have achieved great success, produced better pretrained models from the large raw dataset, and surpassed many downstream tasks.

![**Figure 3: Example for self-supervised learning. There are 4 pseudo-classes corresponding to 4 types of rotation.**](https://vision.aioz.io/f/a3e9fa52d4784838ad14/?dl=1)

**Figure 3: Example for self-supervised learning. There are 4 pseudo-classes corresponding to 4 types of rotation.**

Despite the rapid development of unsupervised learning, it alone can not surpass supervised learning. The highest accuracy for ImageNet with an unsupervised approach is only 43.4% [3]. Recently, OpenAI has to use supervised learning to produce a better language model [4]. 

## Evaluation of representations

In a specific discriminative task, there is no need to measure how good the representation is, as long as the metrics (e.g. accuracy) indicate which method is better, it is better. On the other hand, a good representation is argued that it can capture underlying factors or properties that can be utilized by multiple difference tasks (e.g., classification, recognition, detection, segmentation). However, there is a trade-off between the performance for one task and the ability to generalize for other tasks, hence may cause conflict if the size of the model is not enough.

Modern practice usually trains the model with proxy tasks. In this case, like in a discriminative setting,  the metrics of each proxy task can be used to show which model performs better, since our interest when construct model is representation, the better model also means better representation. For example in a contrastive learning setting, the better model will produce a better representation that separates similar and dissimilar samples.

![**Figure 4: Illustration of the result after contrastive learning process given one positive and one negative** ](https://vision.aioz.io/f/ada913dcb68749158676/?dl=1)

**Figure 4: Illustration of the result after contrastive learning process given one positive and one negative** 

Besides being used in downstream tasks, good representation can also be used to visualize or study the properties of the input data [5].

![**Figure 5: Example visualization from VGG model.**](https://vision.aioz.io/f/602b82e636a24d8a9c41/?dl=1)

**Figure 5: Example visualization from VGG model.**
## References

[1] J. Kaplan, S. McCandlish, T. Henighan, T. B. Brown, B. Chess, R. Child, S. Gray, A. Radford, J. Wu, and D. Amodei, Scaling laws for neural language models, Jan. 2020, arXiv:2001.08361. [Online]. Available: [http://arxiv.org/abs/2001.08361](http://arxiv.org/abs/2001.08361)

[2] A. Dosovitskiy, J. T. Springenberg, M. Riedmiller, and T. Brox, Discriminative unsupervised feature learning with convolutional neural networks, in Proc. Adv. Neural Inf. Process. Syst., 2014, pp. 766-774.

[3] Papers with code - ImageNet benchmark (unsupervised image classification).. Retrieved February 11, 2022, from [https://paperswithcode.com/sota/unsupervised-image-classification-on-imagenet](https://paperswithcode.com/sota/unsupervised-image-classification-on-imagenet)

[4] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C. L., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., Schulman, J., Hilton, J., Kelton, F., Miller, L., Simens, M., Askell, A., Welinder, P., Christiano, P., Leike, J., & Lowe, R. (2022). *Training language models to follow instructions with human feedback*. 68.

[5] Zeiler, M. D., & Fergus, R. (2013). Visualizing and Understanding Convolutional Networks. *ArXiv:1311.2901 [Cs]*. [http://arxiv.org/abs/1311.2901](http://arxiv.org/abs/1311.2901)