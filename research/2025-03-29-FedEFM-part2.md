---
last_modified_on: "2025-03-29"
id: FedEFM2
title: FedEFM - Federated Endovascular Foundation Model with Unseen Data (Part 2)
description: Data collection process and how the dataset is managed in the training and fine-tuning phases.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Medical", "Robotic"]
---

In this part, we will outline the data collection process and how the dataset is managed in the training and fine-tuning phases.

## Robotic Setup
To collect large-scale X-ray images, we employ a robotic platform and a full-size silicon phantom. A surgeon uses a master device joystick to control a follower robot for cannulating three arteries: the left subclavian (LSA), left common carotid (LCCA), and right common carotid (RCCA). During each catheterization procedure, the surgeon activates the X-ray fluoroscopy using a pedal in the operating room. The experiments are conducted using the Epsilon X-ray Generator. We develop a real-time image grabber to transmit the video feed of the surgical scene to a workstation, a computer-based device equipped with an 8-Core ARM v8.2 64-bit CPU. Overall, we collect and label 4,700 new X-ray images to create our EIPhantom dataset. An overview of our robotic setup is demonstrated in Figure 1.

![](https://vision.aioz.io/f/43bdd90230af4e9da8dd/?dl=1)*<center>**Figure 1**. Data collection with endovascular robot.</center>*

## Data collection 

Apart from X-ray images collected from our real robot, we also collect an EISimulation dataset from the CathSim simulator for simulated X-ray images. We manually label both data from the robot and CathSim simulator to use them in downstream tasks. We note that the datasets used to train the foundation model are not being used in downstream endovascular understanding tasks. Figure 2 provides a detailed summary of the datasets used for training and fine-tuning. Additionally, we also visualize samples from each dataset in Figure 3.

![](https://vision.aioz.io/f/ee55c6ed6f8b40f68f73/?dl=1)*<center>**Figure 2**. X-ray dataset used in our work.</center>*

<!-- 
<div style="display: flex; justify-content: center; text-align: center;">
    <img src="https://vision.aioz.io/f/26a6324d5a584773b692/?dl=1" alt="Image 1" width="24%">
    <p>Vessel 12</p>
    <img src="https://vision.aioz.io/f/3ab99f8e24664b3d929e/?dl=1" alt="Image 2" width="24%">
    <img src="https://vision.aioz.io/f/98b3d6c1f7634ee28e5f/?dl=1" alt="Image 3" width="30%">
</div> -->


<div style="display: flex; justify-content: center;">
    <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/d04fa06a7c754812a996/?dl=1" alt="Vessel12" width="400"/>
    <p>CathACtion</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/26a6324d5a584773b692/?dl=1" alt="Vessel12" width="400"/>
    <p>Vessel 12</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/3ab99f8e24664b3d929e/?dl=1" alt="DRIVE" width="400"/>
    <p>DRIVE</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/98b3d6c1f7634ee28e5f/?dl=1" alt="Image 3" width="400"/>
    <p>SenNet</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/a43093ca96354d8b9043/?dl=1" alt="Image 3" width="400"/>
    <p>Medical Decathlon</p>
  </div>
</div>

<div style="display: flex; justify-content: center;">
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/2192e49c36de41f58724/?dl=1" alt="Vessel12" width="400"/>
    <p>EI Simulator</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/6deb2119b0d54f55b852/?dl=1" alt="DRIVE" width="400"/>
    <p>EI Phantom</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/4bb8d4ef5316486ca51d/?dl=1" alt="Image 3" width="400"/>
    <p>RANZCR</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/02c504080e444514ada7/?dl=1" alt="Image 3" width="400"/>
    <p>Cath Animal</p>
  </div>
</div>

<p align="center"><em><b>Figure 3.</b>Visualization of datasets used in our work</em></p>

## Motivation 

Our goal is to train a federated foundation model for endovascular intervention that incorporates all available types of X-ray data. However, in practice, each hospital (silo) possesses specific data sources that may not be accessible to others. This results in disparities in data corpora across institutions, meaning that certain datasets are present in one hospital but absent in another. Figure 4 illustrates this challenge, which leads to the unseen data issue—an obstacle that must be addressed to ensure effective federated training.

Federated learning preserves data privacy by preventing direct data sharing while allowing the exchange of model weights among hospital silos. To leverage this feature, we introduce the Federated Endovascular Foundation Model (FedEFM), a multishot federated distillation algorithm that utilizes Earth Mover’s Distance (EMD) to facilitate learning. Our approach enables local silo models to learn from neighboring silos and incorporate the acquired knowledge back into their own models through a distillation process. Unlike traditional methods that require consistent label sets across both local and global models trained within federated silos, devices, or servers, our method ensures seamless federated training without requiring hospitals to share their datasets—further enhancing data privacy. Additionally, once trained, the foundational model’s weights provide a valuable initialization for downstream tasks.


![](https://vision.aioz.io/f/e0b6dea096414350ad0a/?dl=1)*<center>**Figure 4**. Unseen data issue</center>*

## Next
In the next part, we will explore the technical method to train our FedEFM.
