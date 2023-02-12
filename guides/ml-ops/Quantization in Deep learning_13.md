---
last_modified_on: "2023-02-10"
title: Quantization in Deep learning
description: One of the techniques to lighten the deep learning model.
series_position: 13
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

Deep learning models get more and more accurate over time. But most of the high-precision models are not capable of deploying on IoT devices and mobile with very weak hardware because it is too big to deploy.

Quantization is a technique that helps you reduce the size of deep learning models many times, while reducing latency and increasing inference speed. Let's learn about this amazing technique through today's article.

## Introduction
Quantization is an effective method to speed up the processing time of deep learning models without significantly reducing the accuracy by calculating and storing tensors in data types with a lower number of bits than data types. float.
![Figure 1](https://drive.google.com/uc?export=view&id=1C186dXxoamSsoq2JKS5ZjjB_aPsW614S)*<center>**Figure 1:** Quantization process illustration. </center>*
As everyone knows, deep learning models are matrix calculations. A matrix is represented by an infinite number of single numbers in the form of columns and rows. Therefore, the accuracy and computation time of deep learning models are closely related to how we store and represent these single numbers. In the process of storing single numbers, there are two problems that we often encounter:

* Precision (Number of numbers after the comma that we can store).
* Number of bits needed to represent that number.

The larger the number of bits, the more values can be represented, but the computation and storage are complex, so it takes time to process, especially with operations such as matrix multiplication.


## Quantization format classification
In practice, there are two main ways to go about quantization: 

**Post-training quantization** 

As the name implies, post-training quantization is a technique in which the neural network is entirely trained using floating-point computation and then gets quantized afterward.

To do this, once the training is over, the neural network is frozen, meaning its parameters can no longer be updated, and then parameters get quantized. The quantized model is what ultimately gets deployed and used to perform inference without any changes to the post-training parameters.

Though this method is simple, it can lead to higher accuracy loss because all of the quantization-related errors occur after training is completed, and thus cannot be compensated for.

![Figure 2](https://drive.google.com/uc?export=view&id=1fSyFWWbdAR8Rg3Efj8UtfdjXQWYE8po5)*<center>**Figure 2:** Size reduction table when performing quantization on different data types.. </center>*

In the table above are 3 basic types of quantization with different properties.

* **Dynamic range quantization**: This is tflite's default quantization type. The main idea is to use the range of data on a batch to estimate quantization values. This type of quantization reduces the memory size by 4 times, increasing the speed 2-3 times on the CPU.
* **Full Integer quantization**: All coefficients will be converted to integer type. Memory size reduced by 4 times, increased by 3 times on CPU, TPU, Microcontrollers.
* **Float16 quantization**: Of course, it will be converted to float type and reduced in size 2 times (from 32 bits to 16 bits). Helps speed up computation on GPUs.

**Quantization-aware training** 

Quantization-aware training, as shown in Figure 3, works to compensate for the quantization-related errors by training the neural network using the quantized version in the forward pass during training.

![Figure 3](https://drive.google.com/uc?export=view&id=1av7Damdjtl6OxeHmQ5wgCTVuybSme5IA)*<center>**Figure 3:** Flow chart of quantization-aware training. Image used courtesy of Novac et al. </center>*

The idea is that the quantization-related errors will accumulate in the total loss of the model during training, and the training optimizer will work to adjust parameters accordingly and reduce error overall.

Quantization-aware training has the benefit of much lower loss than post-training quantization.

For a much more in-depth, technical discussion about the mathematics behind quantization, it is recommended to [read this paper](https://arxiv.org/pdf/2103.13630.pdf) from Gholami, et al.

In this rather long article, we also introduced the basic concepts of quantization in Deep Learning. In the next article, we will go deeper into how to implement Quantization in Pytorch and Tensorflow


## References
1. [A Survey of Quantization Methods for Efficient
Neural Network Inference](https://arxiv.org/pdf/2103.13630.pdf)
2. [Neural Network Quantization: What Is It and How Does It Relate to TinyML?](https://www.allaboutcircuits.com/technical-articles/neural-network-quantization-what-is-it-and-how-does-it-relate-to-tiny-machine-learning/)