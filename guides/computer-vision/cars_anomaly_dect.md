---
last_modified_on: "2022-09-17"
title: A simple system for traffic anomaly detection
description: Explore traffic anomaly detection
series_position: 22
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';  
import Highlight from '@site/src/components/Highlight';

:hourglass: There are many traffic accidents every year, and this leads to the increasing demand for an automatic system that could help the experts find and analyze the moment before the accident happened. AI City Challenge 2020 decided to organize a track that fully focuses on traffic anomaly detection, especially detecting crashing and stalling. In this short tutorial, I will present a method proposed by Tran et. al. [1] that utilizes a generative network and some car detection networks to create a simple 3 phases system.


## Overview 
Our task is to detect when an anomaly happens. Formally, given a sequence of frames $F = (f_1, \dots, f_N)$, where N is the number of frames of the video, the system will return all the tuples $(f_i, f_j)$, where $f_i$ is the starting frame, and $f_j$ is the ending frame, that represents the segment of the video that contains an anomaly event. In [1], the authors define two scenarios: 
- In the case of a stalled vehicle, the start time is the time when it comes to a complete stop. 
- In the case of multiple crashes, the start time is the instant when the first crash occurs.
Depending on the type of anomaly, the output from the detection phase is processed by two different approaches that will be discussed below.

![](https://vision.aioz.io/f/6e861d52094546aeb31e/?dl=1)
**Figure 1: The overview of [1]&#39;s three phases.**

![](https://vision.aioz.io/f/ba63fb1948654c9da5bd/?dl=1)
**Figure 2: The overview of the full process.**

## Preprocessing
There are three steps for preprocessing we need to do before using the neural network: removing frozen frames, motion encoding, and background extraction.
Removing frozen frames is important because the raw videos are full of noise and frozen frames, and this could lead to wrong stalled vehicle detection as a vehicle may appear for a long period, and may not provide sufficient motion information for the input of GAN-based future frame prediction. Therefore, it is important to filter out all the broken frames. We could use OpenCV to simply check with a color histogram to process this part. 
For motion encoding and background extraction, we can average the images of the frames. Formally, it is simply $\text{avg}_i = \alpha \text{avg}_{i-1} + (1-\alpha)f_i$. If we compute the full video, the moving vehicle will be blurred out and disappeared, only the road remains. This equation will help us in two things: producing the region of interest and motion encoding. Like I said earlier, if we compute the full video, the motion will disappear and only the road remains, but if we compute only a segment of the video, we will see a blurry motion of the vehicles of that segment, and we call this information motion information. The pixel values changing heavily also tell us the importance of the region containing those pixels. Therefore, we can also find the region of interest and remove all irrelevant regions. 

## Vehicle detection
![](https://vision.aioz.io/f/33a857b111854203ac43/?dl=1)
**Figure 3: The example output of the detection model in each context.**
To detect a stalled vehicle, we only need to find which vehicle does not move anymore in the video. Naively, we would want to train a neural network to detect vehicles in this phase. However, the range of values of the pixel varies widely. There are night videos, day videos, and big and small vehicles mixed into each other videos. Using the knowledge from the training dataset, the authors of [1] proposed that we should train at least 3 (exactly 3 in the paper) neural networks for each of the aforementioned scenarios. This phase requires heavily finetuning labor. We will need to train multiple state-of-the-art models, such as Faster-R CNN [2], to find the best one for each scenario. After that, the output, which contains bounding boxes for multiple vehicles, is used to construct a moving path. At this point, we can just write a simple heuristic to check the stalling condition: if a bounding box is stopping for M frames, then its vehicle is stalled. 

## Forward prediction 
![](https://vision.aioz.io/f/0e0e98923ef044df810f/?dl=1)
**Figure 4: The example output of the detection model in each context.**
At the same time we run the vehicle detection process, we also use a generative network to produce the expected next frame. For example, normally a vehicle moves in a straight line on a straight road, if the vehicle is moving weirdly, such as turning around multiple times, then we know that an anomaly is happening. To know what is normal and what is not, [1] proposes using a generative adversarial network (GAN) [3] trained on a normal scenario so that when it predicts the next frame, we can compare the normal knowledge it has and the real behavior of the vehicle. If two frames strongly disagree with each other, we know an anomaly may happen in this frame. The agreement degree is defined using Peak Signal-to-Noise Ratio (PSNR) [4] to calculate the likelihood of two frames. A higher value of PSNR means the pair of images are similar. If PSNR falls below a certain threshold, an anomaly is likely to happen.
As you can see, this phase heavily depends on the ability of the GAN that we choose, so careful engineering is needed. 

## Refinement
Unlike stalling detection, crashing detection may start earlier than when PSNR decides it to be a such event. Therefore, the authors [1] propose another heuristic based on the observation that if an anomaly happens, it is often accurate to use moving paths from the Vehicle detection phase to find the exact moment that an anomaly happens. To be specific, if the bounding box from frame $f_i$ and $f_j$ has a distance larger than a threshold $k$, then we can decide it starts from that point onward. And to reduce the false positive rate, we will move backward from the starting point of the output of the Forward prediction phase. 

## Conclusion 
I have introduced a simple system for traffic anomaly detection proposed by [1]. While the system is simple conceptually, it is clear that this requires a lot of effort in fine-tuning and engineering, and in the worst-case scenario, such as the context is too hard for the vehicle detection network to work, or the data is too unfit for GAN training, this system will immediately fall apart. However, it is worth noting that this is the first step for anomaly detection, and with the improvement of computer vision recently, we can positively hope that this system will work in real scenarios soon. 

## References
[1] Tran et al., &#34;iTASK - Intelligent Traffic Analysis Software Kit,&#34; 2020 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW), 2020, pp. 2607-2616, doi: 10.1109/CVPRW50498.2020.00314.

[2] Shaoqing Ren, Kaiming He, Ross Girshick, and Jian Sun. 2015. Faster R-CNN: towards real-time object detection with region proposal networks. In Proceedings of the 28th International Conference on Neural Information Processing Systems - Volume 1 (NIPS&#39;15). MIT Press, Cambridge, MA, USA, 91�99.


[3] Goodfellow, I., Pouget-Abadie, J., Mirza, M., Xu, B., Warde-Farley, D., Ozair, S., ... &amp; Bengio, Y. (2014). Generative adversarial nets. Advances in neural information processing systems, 27.

[4] Mathieu, M., Couprie, C., &amp; LeCun, Y. (2015). Deep multi-scale video prediction beyond mean square error. arXiv preprint arXiv:1511.05440