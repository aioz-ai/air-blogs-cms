---
last_modified_on: "2023-01-17"
title: Diffusion Model: A breakthrough compared to Auto-encoder, GAN and Flow-based models
description: Introducing the differences and advantages of diffusion compared to previous models.
series_position: 49
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---


In previous articles we have thoroughly reviewed the diffusion model, but in this article we want to review some of the advantages of the diffusion model compared to its "predecessors" such as Auto-Encoder, GAN and Flow-based model

## What did the seniors do? 

Generative models, including Auto-Encoder (AE), Generative Adversarial Networks (GAN), and Flow-based models, play a crucial role in the field of artificial intelligence. These models are designed to understand and replicate the underlying distribution of a given dataset, enabling them to generate new samples that resemble the original data.

The primary function of generative models is to learn the complex patterns and structures present in the data, allowing them to generate new instances that are similar in nature. They capture the underlying characteristics of the training data and use this knowledge to create new samples that possess similar attributes, whether it be images, texts, or any other form of data.

However, these current models also reveal many limitations:

- **Auto-Encoder (AE)**:

The idea of Auto-Encoder (AE) is a remarkable approach in generating new data by optimizing the loss function between the generated data and the original input. This model works by mapping data onto a hidden space, then using a decryption process to restore the original data.

![Figure 1](https://drive.google.com/uc?export=view&id=1nU72aJ4wbzwbhs2ieYXQ0G8ua35E1kcB)


However, a limitation of AE is its dependence on a surrogate loss function (e.g. hinge loss function). This surrogate loss function has a drawback, which is that it acts as a "fence" between data classes, similar to how it is used in the SVM algorithm. This means that the surrogate loss function is not guaranteed to predict whether the newly generated data has the same distribution as the original data.

In summary, although the idea of AE is very innovative in generating new data, this model depends on the surrogate loss function, which may affect the ability to accurately reproduce the distribution of the original data. Research continues to search for new methods to improve the data generation ability of AE and overcome this limitation.

- **Generative Adversarial Networks (GAN)**: 

![Figure 1](https://drive.google.com/uc?export=view&id=1nHtortax6tJjpV0UDC0zu79_JaJXJoiK)


The Generative Adversarial Networks (GAN) model made a significant breakthrough upon its launch. However, it also has limitations that need to be mentioned.

First, GAN depends on the Discriminator layer to "check" the data and evaluate its authenticity. Usually the Discriminator layer is previously trained and is considered a pre-trained model. This means that GANs lack "creativity" and diversity in generating new data. It is not possible to automatically discover and generate new features without support from the Discriminator layer.

Second, when the GAN network architecture is too large, training may have problems with vanishing gradients. This reduces the data generation performance of the GAN, and also makes the training process more difficult. Handling this phenomenon requires complex knowledge and techniques to ensure stability and enhance the data generation ability of GANs.

Finally, although a GAN uses a noisy input to generate data, it does not actually find a probability distribution of that data. Instead, the GAN learns a mapping from the noise space to the data space, and new data is generated from points in this noise space. This means that GANs do not provide an accurate way to determine the probability distribution of the generated data.

In summary, the GAN model has brought many important contributions, but also encountered some limitations such as limited "creativity", the problem of vanishing gradients and not finding a probability distribution of the data. Continued research in this area aims to improve and overcome these limitations.

- **Flow-based model**: 

Flow-based model is a model that uses the Normalizing Flows method to generate data. This method is based on learning the Probability Density Function (PDF) of the input through multiple transformation processes. The main idea of the model is to map data from a simple distribution (e.g. Gaussian distribution) through invertible (invertible) transformations to produce data according to the desired target distribution.

![Figure 1](https://drive.google.com/uc?export=view&id=18Z7zfjkBJjmVA5hW-zN6v6pL5aPJ7rmn)


However, the weakness of the Flow-based model is that the layers in it must be invertible to be able to generate data. This means that each layer in the transformation process must be able to calculate back (inverse) to restore the original data. For example, in the figure below illustrating a Flow-based model, the function used at the encoder layer must be invertible so that it can be applied during the decoding process at the decoder layer.

Ensuring invertibility in layers can be a challenge in building a Flow-based model. This requires special and skillful design of transformations to ensure the invertibility of each layer and at the same time maintain the probabilistic nature of the data.

## From there, a descendant was born...

It can be said that the Diffusion model is a combination of the advantages of its predecessors: This is a probabilistic model that is responsible for creating a distribution for input similar to the Flow-based model, capable of approximating the distribution of data. is generated with a distribution of the original data similar to VAE, and has a pre-determined layer similar to GAN. That is, whatever the older generations have, they have it all.

![Figure 1](https://drive.google.com/uc?export=view&id=1c1gpEJbdcU6a0PPflXjA5QsrXx3F_zLa)

The Diffusion model includes two processes: Forward Diffusion Process (I call it FDP for short) corresponding to the encoder layer and Reverse Diffusion Process (I call it RDP for short) corresponding to the decoder layer. If we take the example of the glitter painting above, FDP is the process of sprinkling glitter on the painting, and RDP is the process of "sieving" and dusting off glitter.

![Figure 1](https://drive.google.com/uc?export=view&id=1IycfUuRR_3L9TCt4x88s-hKrL-1puT3r)

However, the Diffusion model also has some limitations. The diffusion process can be time consuming and computationally resource intensive, especially with large and complex data. Additionally, the Diffusion model needs to be carefully trained to ensure the diffusion process is efficient and produces high-quality data.

In short, the Diffusion model is a powerful and potential method of data generation. It allows the generation of diverse and complex new data without having to rely on specific probabilistic models. However, it is important to note the time and computational resources required for diffusion and model training.

## References
1. [Lilian Weng, What are Diffusion models?](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/)
2. [How diffusion models work: the math from scratch](https://theaisummer.com/diffusion-models/)