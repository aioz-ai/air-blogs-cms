---
last_modified_on: "2021-04-15"
id:  intro-federated-learning
title: Introduction to Federated Learning
description: "Building better product with on-device data and privacy by default."
author_github: https://github.com/quangduytran
tags: ["type: post", "AI", "Federated Learning"]
---

**Federated Learning:** machine learning over a distributed dataset, where user devices (e.g., desktop, mobile phones, etc.) are utilized to collaboratively learn a shared prediction model while keeping all training data locally on the device. This approach decouples the ability to do machine learning from storing the data in the cloud.

Conceptually, federated learning proposes a mechanism to train a high-quality centralized model. Simultaneously, training data remains distributed over many clients, each with unreliable and relatively slow network connections.

The idea behind federated learning is as conceptually simple as its technologically complex. Traditional machine learning programs relied on a centralized model for training in which a group of servers runs a specific model against training and validation datasets. That centralized training approach can work very efficiently in many scenarios. Still, it has also proven to be challenging in use cases involving a large number of endpoints using and improving the model. The prototypical example of the limitation of the centralized training model can be found in mobile or internet of things(IoT) scenarios. The quality of a model depends on the information processed across hundreds of thousands or millions of devices. Each endpoint can contribute to a machine learning model's training in its own autonomous way in those scenarios. In other words, knowledge is federated.
