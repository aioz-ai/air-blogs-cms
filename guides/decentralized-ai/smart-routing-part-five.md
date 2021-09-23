---
last_modified_on: "2021-09-13"
title: AI-driven routing (part 5) Challenges and Open Issues.
description: A series of Smart Routing based on AI.
series_position: 9
author_github: https://github.com/aioz-ai
tags: ["type: tutorial", "level: advance"]
---

In previous parts, we have mentioned about hybrid smart routing and its effectiveness. Although AI&ML-driven networking control is a potential paradigm for future networks, numerous problems remain, and much more effort is required. In this part, I'll go through the key problems and unresolved concerns surrounding AI&ML-powered networking. The Challenges and Open Issues, including:
- New Hardware Architectures.
- Promoting Machine Learning Algorithms
- Advanced Software Systems.

![Fig-0](https://vision.aioz.io/f/ce0c34940f144eda900c/?dl=1)

## New Hardware Architectures

As a result, every advancement in upper-level services is predicated on considerable improvements in the underlying hardware, such as a CPU for general-purpose calculations, a digital signal processor (DSP) for a communication system, and a GPU for image processing. In the same way, a dedicated AI networking processor is urgently needed to satisfy the demands of the AI-driven networking age.

Millisecond after millisecond, the current networks create a myriad of various sorts of fluxes. Such large data sets make it difficult to run AI systems. As of now, routers lack the computational power necessary for AI&ML implementation. GPU and Tensor Processing Unit (TPU) chips have recently become a cornerstone of the AI age as highly parallel, multicore, multithreaded computers. According to certain research, the use of a GPU can enhance packet processing capabilities. However, due to the necessity for high-speed processing of huge quantities of data (more than 10 Gb/s) and rigorous reaction delay requirements (less than 1 ms) for future networks, there is still a significant gap between universal AI processor chips and their real networking implementation possibilities. See Figure 1 to 3 for more details about CPU, GPU, TPU, and DPS.

![Fig-1](https://assets-global.website-files.com/5debb9b4f88fbc3f702d579e/5e08f35d7436081481e15d61_e7b08ad97410491586d63028740b90c1.png)
*<center>**Figure 1**: Differences between CPU and GPU ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.omnisci.com%2Ftechnical-glossary%2Fcpu-vs-gpu&psig=AOvVaw0K04qVJ7Gj01MRjwjCn58E&ust=1631062359202000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCJizyvXS6_ICFQAAAAAdAAAAABAJ)).</center>*

![Fig-2](https://devopedia.org/images/article/12/5335.1531331342.png)
*<center>**Figure 2**: TPU Architecture ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fdevopedia.org%2Ftensor-processing-unit&psig=AOvVaw1mc1lgF1FKL5s2HJZtw62f&ust=1631062525021000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCMCO3-nT6_ICFQAAAAAdAAAAABAD)).</center>*

![Fig-3](https://www.researchgate.net/profile/J-Simoes-2/publication/3846855/figure/fig2/AS:668965827272714@1536505290541/Overall-architecture-of-the-Digital-Signal-Processor.png)
*<center>**Figure 3**: DPS Architecture ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.researchgate.net%2Ffigure%2FOverall-architecture-of-the-Digital-Signal-Processor_fig2_3846855&psig=AOvVaw3Iq64SsS2FanRZ3wQZ5tud&ust=1631062469026000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCMDY45fU6_ICFQAAAAAdAAAAABAD)).</center>*

## Promoting Machine Learning Algorithms

Despite the fact that several ML algorithms have been created, contemporary ML methods are generally motivated by the demands of certain existing applications, such as Computer Vision (CV) and Natural Language Processing (NLP). Convolutional neural networks, for example, are intriguing and powerful tools for image and audio recognition that can reach superhuman performance on a variety of tasks. However, the networking domain requires very distinct theoretical mathematical models than those found in the disciplines of computer vision and natural language processing. Convolutional or recurrent layers may not perform well in the networking area. Furthermore, networks include considerably more data and severe reaction time requirements, posing significant hurdles for ML implementation. As a result of the rigorous needs and particular characteristics of the networking domain, both modification of current algorithms and invention of new ones will be required. As a result, attempts to fulfill the demands of the networking domain as a new application area for ML will propel advancements in both the ML and networking domains to new heights.

## Advanced Software Systems
Currently, network data handling is facing typical big data difficulties; recent years have witnessed a threefold rise in total IP traffic and a >60% increase in the number of devices installed and the amount of telemetry data transmitted in near real time. Meanwhile, the geo-distributed nature of networking makes widespread adoption of systems for network data analytics much more challenging. For example, determining how to combine data such as log data, metric data, and network telemetry data; scaling up to the consumption of millions of flows every millisecond; and efficiently sharing information across distant network nodes are all problems. Current end-to-end solutions are highly complicated and time-consuming, combining various technologies like as Apache Spark and Hadoop MapReduce.
As a result, a strong, scalable big data analytics platform for networks and network services is required.

Furthermore, software libraries for ML networking activities are a critical enabler for AI-based networking. High-level programming interfaces for developing, training, and evaluating ML algorithms are provided by ML frameworks. Current machine learning frameworks, such as TensorFlow, Caffe, and Theano, are built for general-purpose applications and put an undue load on the networking domain. They must be further tuned to meet the demands of networking applications, such as fast processing speed, low complexity, and light weight. See Figure 4 for comparision between different ML frameworks with different criteria.

![Fig-4](https://i.morioh.com/7685e2cbe3.png)
*<center>**Figure 4**: DPS Architecture ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fmorioh.com%2Fp%2Fa80813c4a01c&psig=AOvVaw2WZOYovgxo1jpSSnPIohw-&ust=1631063392454000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCOCgjeHW6_ICFQAAAAAdAAAAABAp)).</center>*

## Conclusion
In the whole series, we first looked at two deployment models for an intelligent control plane in a network, as well as the benefits and drawbacks of the centralized and distributed paradigms. Then, to satisfy the demands of various network services, we visited a hybrid ML paradigm for packet routing that combines distributed AI routers with a centralized network mind. We use a centralized AI control plane for tunneling-based routing and a hybrid AI architecture for hop-by-hop routing in the paradigm. Finally, we examined the effectiveness of smart routing and open challenges in this field. We hope that the series provides meaningful information for readers who are interested in AI-based routing.
