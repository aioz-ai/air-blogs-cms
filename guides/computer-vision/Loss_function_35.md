---
last_modified_on: "2023-04-10"
title: Summarization of common loss functions
description:  Introduction to loss functions in Deep learning
series_position: 35
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
Today, AI application models are more and more widespread. The knowledge related to building an AI model is also widespread. If you have tried to learn about them, you will not be unfamiliar with the concept of **Loss Function**. 
However, the reality is that many people still do not understand what it does?. So, in this article, let's learn more about this **Loss Function** concept. 

## Introduct

The loss function in a neural network quantifies the difference between the expected outcome and the outcome produced by the machine learning model. From the loss function, we can derive the gradients which are used to update the weights. The average over all losses constitutes the cost. 

A machine learning model such as a neural network attempts to learn the probability distribution underlying the given data observations. In machine learning, we commonly use the statistical framework of maximum likelihood estimation as a basis for model construction. This basically means we try to find a set of parameters and a prior probability distribution such as the normal distribution to construct the model that represents the distribution over our data. If you are interested in learning more, I suggest you check out my post on maximum likelihood estimation.

## Loss Function 

**1. Loss Function in Regression**

* **Mean Squared Error (MSE)**

Mean Squared Error (also called L2 loss) is almost every data scientist preference when it comes to loss functions for regression. This is because most variables can be modeled into a Gaussian distribution.

Mean Squared Error is the average of the squared differences between the actual and the predicted values. For a data point $y_i$ and its predicted value $\hat{y}_i$, where n is the total number of data points in the dataset, the mean squared error is defined as:

![Figure 1](https://drive.google.com/uc?export=view&id=1HZh-jeZTsgHkeNpf4jXLY_NKfyLI4MUi)


```
import numpy as np
 
# Mean Squared Error
 
def mse( y, y_pred ) :
     
    return  np.sum( ( y - y_pred ) ** 2 ) / np.size( y )
```

* **Mean Bias Error (MBE)**

Mean Bias Error is used to calculate the average bias in the model. Bias, in a nutshell, is overestimating or underestimating a parameter. Corrective measures can be taken to reduce the bias post-evaluating the model using MBE.

Mean Bias Error takes the actual difference between the target and the predicted value, and not the absolute difference. One has to be cautious as the positive and the negative errors could cancel each other out, which is why it is one of the lesser-used loss functions.

The formula of Mean Bias Error is:
![Figure 2](https://drive.google.com/uc?export=view&id=185GDy6Prh1WRC4ZoHJjc8FdPcnpUoSEY)

```
import numpy as np

# Mean Bias Error
 
def mbe( y, y_pred ) :
     
    return np.sum( y - y_pred ) / np.size( y )
```

* **Mean Absolute Error (MAE)**

Mean Absolute Error is the mean of the sum of absolute differences between predictive values and actual values. The similarity it shares with MSE is that even this cost function cares only about the magnitude and not the direction. A slight disadvantage compared with MSE is that calculating gradients is a bit more complicated as we need to leverage linear programming methods to achieve that.

The formula of Mean Absolute Error is:
![Figure 3](https://drive.google.com/uc?export=view&id=1o35xlNwuiENfPYsjtoTnADevfiHs8tEo)
```
import numpy as np

# Mean Absolute Error
 
def mae( y, y_pred ) :
     
    return np.sum( np.abs( y - y_pred ) ) / np.size( y )
```

* **Huber Loss**

A comparison between L1 and L2 loss yields the following results:

1. L1 loss is more robust than its counterpart.

On taking a closer look at the formulas, one can observe that if the difference between the predicted and the actual value is high, L2 loss magnifies the effect when compared to L1. Since L2 succumbs to outliers, L1 loss function is the more robust loss function.

2. L1 loss is less stable than L2 loss.

Since L1 loss deals with the difference in distances, a small horizontal change can lead to the regression line jumping a large amount. Such an effect taking place across multiple iterations would lead to a significant change in the slope between iterations.

On the other hand, MSE ensures the regression line moves lightly for a small adjustment in the data point.

Huber Loss combines the robustness of L1 with the stability of L2, essentially the best of L1 and L2 losses. For huge errors, it is linear and for small errors, it is quadratic in nature.

Huber Loss is characterized by the parameter delta. For a prediction f(x) of the data point y, with the characterizing parameter, Huber Loss is formulated as:
![Figure 4](https://drive.google.com/uc?export=view&id=1XnprI42xLlnbJq-cTUx9hfjCmYh3cszH)

```
import numpy as np

# Huber Loss

def Huber(y, y_pred, delta):
 
    condition = np.abs(y - y_pred) < delta
 
    l = np.where(condition, 0.5 * (y - y_pred) ** 2,
                 delta * (np.abs(y - y_pred) - 0.5 * delta))
 
    return np.sum(l) / np.size(y)
```
**2.  Loss Functions for Classification**

* **Binary Cross Entropy Loss**
This is the most common loss function used for classification problems that have two classes. The word  cross-entropy, seemingly out-of-place, has a statistical interpretation.

Entropy is the measure of randomness in the information being processed, and cross entropy is a measure of the difference of the randomness between two random variables.

If the divergence of the predicted probability from the actual label increases, the cross-entropy loss increases. Going by this, predicting a probability of .011 when the actual observation label is 1 would result in a high loss value. In an ideal situation, model would have a log loss of 0. 

When the number of classes is 2,the binary classification:
![Figure 5](https://drive.google.com/uc?export=view&id=1p9MBDCHD5_7OzzFrTt4dGltZMM7BRqpL)

When the number of classes is more than 2, the multi-class classification:
![Figure 5](https://drive.google.com/uc?export=view&id=1lw7g41xD8jjhz9Ne859CZ121mS_4TpNG)

```
import numpy as np

# Binary Cross Entropy Loss
 
def cross_entropy(y, y_pred):
 
    return - np.sum(y * np.log(y_pred) + (1 - y) * np.log(1 - y_pred)) / np.size(y)
```

* **Hinge Loss**

Another commonly used loss function for classification is the hinge loss. Hinge loss is primarily developed for support vector machines for calculating the maximum margin from the hyperplane to the classes.

Loss functions penalize wrong predictions and does not do so for the right predictions. So, the score of the target label should be greater than the sum of all the incorrect labels by a margin of (at the least) one.

The mathematical formulation of hinge loss is as follows:
![Figure 6](https://drive.google.com/uc?export=view&id=1VyuDnLcOIc1rtj1J-O4MHlTeoaNht9PC)

```
import numpy as np

# Hinge Loss
 
def hinge(y, y_pred):
 
    l = 0
 
    size = np.size(y)
 
    for i in range(size):
 
        l = l + max(0, 1 - y[i] * y_pred[i])
 
    return l / size
```

**3. Loss Function for Object Detection**

* **Focal Loss**

A Focal Loss function addresses class imbalance during training in tasks like object detection. Focal loss applies a modulating term to the cross entropy loss in order to focus learning on hard misclassified examples. It is a dynamically scaled cross entropy loss, where the scaling factor decays to zero as confidence in the correct class increases. Intuitively, this scaling factor can automatically down-weight the contribution of easy examples during training and rapidly focus the model on hard examples.

The mathematical formulation of hinge loss is as follows:
![Figure 7](https://drive.google.com/uc?export=view&id=10WpNPR_kzix11vvdSfnaIvV9gt6YcD5T)
```
import numpy as np
import matplotlib.pyplot as plt

alpha = 1
gammas = [0, 0.5, 1, 2, 5]
p = np.linspace(0, 1, 200)[1:]

def _focal_loss(p, gamma, alpha = 1):
  loss = -(1-p)**gamma*np.log(p)
  return loss
```
* **IoU Loss**

Intuitively, IoU loss maximizes the coincidence between the predicted box and the ground truth box. From the formula point of view, when calculating the area of intersection and union of two boxes, four variables of measuring each box are used at the same time. Therefore, this loss function regards a box as a whole for training, and can get more accurate predicted box. 

In addition, regardless of the scale of the ground truth, IoU is normalized to [0, 1], which can prevent the model from focusing too much on large objects and ignoring small ones. The found that the use of IoU loss not only makes the location more accurate, but also speeds up the convergence rate.

The mathematical formulation of hinge loss is as follows:
![Figure 7](https://drive.google.com/uc?export=view&id=1j0gj9Lq26Pa1vavvjukIQ4lqEqI9KbrA)

```
import torch

SMOOTH = 1e-6

def iou_pytorch(outputs: torch.Tensor, labels: torch.Tensor):
    # You can comment out this line if you are passing tensors of equal shape
    # But if you are passing output from UNet or something it will most probably
      # be with the BATCH x 1 x H x W shape
      outputs = outputs.squeeze(1)  # BATCH x 1 x H x W => BATCH x H x W

      intersection = (outputs & labels).float().sum((1, 2))  # Will be zero if Truth=0 or Prediction=0
      union = (outputs | labels).float().sum((1, 2))         # Will be zzero if both are 0

      iou = (intersection + SMOOTH) / (union + SMOOTH)  # We smooth our devision to avoid 0/0

      thresholded = torch.clamp(20 * (iou - 0.5), 0, 10).ceil() / 10  # This is equal to comparing with thresolds

      return thresholded  # Or thresholded.mean() if you are interested in average across the batch
```

* **Contrastive Loss**

The goal of contrastive loss is to discriminate the features of the input vectors. Here an image pair is fed into the model, if they are similar the model infers it as 1 otherwise zero. We can intuitively compare it with the goals of cosine similarity as an objective function. Contrastive learning methods are also called distance metric learning methods where the distance between samples is calculated.
 
The mathematical formulation of hinge loss is as follows:

![Figure 8](https://drive.google.com/uc?export=view&id=1IlrRv1Fykjv4heCqPQwb8V8VzvtGOYrc)
```
import torch

def criterion(x1, x2, label, margin: float = 1.0):
    """
    Computes Contrastive Loss
    """

    dist = torch.nn.functional.pairwise_distance(x1, x2)

    loss = (1 - label) * torch.pow(dist, 2) \
        + (label) * torch.pow(torch.clamp(margin - dist, min=0.0), 2)
    loss = torch.mean(loss)

    return loss
```
* **Triplet Loss**

The goal of Triplet loss, in the context of Siamese Networks, is to maximize the joint probability among all score-pairs i.e. the product of all probabilities. By using its negative logarithm, we can get the loss formulation as follows:

![Figure 9](https://drive.google.com/uc?export=view&id=14Bs3jDATWl_TQKq7X5E6Uw0omhAT0cao)

where the balance weight 1/MN is used to keep the loss with the same scale for different number of instance sets.

**4. Loss Function for GAN**

* **Standard GAN loss function (min-max GAN loss)**

The standard GAN loss function, also known as the min-max loss, was first described in a 2014 paper by Ian Goodfellow in paper titled Generative Adversarial Networks.

![Figure 10](https://drive.google.com/uc?export=view&id=1m2-UxSA6eFjtBFQGz-qcsxT9aXwFXDok)

The generator tries to minimize this function while the discriminator tries to maximize it. Looking at it as a min-max game, this formulation of the loss seemed effective. 

In practice, it saturates for the generator, meaning that the generator quite frequently stops training if it does not catch up with the discriminator.

The Standard GAN loss function can further be categorized into two parts: **Discriminator loss** and **Generator loss**.

* **Discriminator loss**

While the discriminator is trained, it classifies both the real data and the fake data from the generator.

It penalizes itself for misclassifying a real instance as fake, or a fake instance (created by the  generator) as real, by maximizing the below function.

![Figure 11](https://drive.google.com/uc?export=view&id=1oAdFa4sXUUSW2z0l-eFpP4itJG0ugpH1)

log(D(x)) refers to the probability that the generator is rightly classifying the real image, maximizing log(1-D(G(z))) would help it to correctly label the fake image that comes from the generator.

* **Generator loss**

While the generator is trained, it samples random noise and produces an output from that noise. The output then goes through the discriminator and gets classified as either Real or Fake based on the ability of the discriminator to tell one from the other.

The generator loss is then calculated from the discriminator classification. It gets rewarded if it successfully fools the discriminator, and gets penalized otherwise. 

The following equation is minimized to training the generator:
![Figure 12](https://drive.google.com/uc?export=view&id=1YhVQkHVEz4iuHT8e3teP2PLPYeg0CtD_)

## Conclusion

In this post, you discovered the role of loss and loss functions in training deep learning neural networks and how to choose the right loss function for your predictive modeling problems. 

## References
1. [How to Choose Loss Functions When Training Deep Learning Neural Networks](https://machinelearningmastery.com/how-to-choose-loss-functions-when-training-deep-learning-neural-networks/)
2. [Regression losses](https://keras.io/api/losses/regression_losses/#meansquaredlogarithmicerror-class)
3. [Common Loss functions in machine learning](https://towardsdatascience.com/common-loss-functions-in-machine-learning-46af0ffc4d23)
4. [The 7 Most Common Machine Learning Loss Functions](https://builtin.com/machine-learning/common-loss-functions)