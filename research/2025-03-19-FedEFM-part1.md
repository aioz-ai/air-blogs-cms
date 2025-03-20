---
last_modified_on: "2025-03-19"
id: FedEFM1
title: FedEFM - Federated Endovascular Foundation Model with Unseen Data (Part 1)
description: Introducing FedEFM: The Future of Federated Endovascular Foundation Models.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Medical", "Robotic"]
---


In endovascular surgery, the precise identification of catheters and guidewires in X-ray images is essential for reducing intervention risks. However, accurately segmenting catheter and guidewire structures is challenging due to the limited availability of labeled data. Foundation models offer a promising solution by enabling the collection of similar-domain data to train models whose weights can be fine-tuned for downstream tasks. Nonetheless, large-scale data collection for training is constrained by the necessity of maintaining patient privacy. This paper proposes a new method to train a foundation model in a decentralized federated learning setting for endovascular intervention. To ensure the feasibility of the training, we tackle the unseen data issue using differentiable Earth Mover's Distance within a knowledge distillation framework. Once trained, our foundation model's weights provide valuable initialization for downstream tasks, thereby enhancing task-specific performance. Intensive experiments show that our approach achieves new state-of-the-art results, contributing to advancements in endovascular intervention and robotic-assisted endovascular surgery, while addressing the critical issue of data sharing in the medical domain.

## Introduction
Endovascular surgery is now usually a minimally invasive procedure that diagnoses and treats vascular diseases with several advantages such as reduced trauma and quick recovery time. During endovascular surgery, surgeons use catheters and guidewires to access arteries. However, this procedure also entails risks such as potential vessel wall damage. Precise identification of catheters and guidewires within X-ray images is crucial for patient safety. The rise of deep learning has played a vital role in improving surgical precision and enhancing patient safety in endovascular intervention. However, accurately segmenting intricate catheters and guidewires in X-ray images remains challenging due to the limited quantity of data.

Recently, vision-language models have gained significant attention from researchers across various fields. Foundation models like CLIP and ALIGN have demonstrated strong capabilities in cross-modal alignment and zero-shot learning tasks. In the medical field, EndoFM and LVM-Med have been introduced as foundation models designed to handle medical data across multiple modalities. While these models perform well on downstream tasks, they typically assume that data can be centrally collected and trained, which is often difficult in medical applications.

Gathering large-scale medical data is particularly challenging due to privacy concerns. To address this issue, federated learning has emerged as a potential solution, allowing models to be trained collaboratively across multiple hospital silos without requiring direct access to patient data.

Despite its benefits, federated learning faces challenges such as ensuring stable convergence across different silos and handling heterogeneous data. In endovascular interventions, these challenges arise primarily from variations in data collected from different sources, leading to domain gaps in X-ray images. As illustrated in Figure 1, X-ray images from various endovascular datasets differ significantly. Additionally, due to privacy restrictions, datasets containing real human X-ray images tend to be smaller compared to those obtained from animal models, silicon phantoms, or simulated environments.

<div style="display: flex; justify-content: center;">
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/e2cea79f94df4d148eb6/?dl=1" alt="Vessel12" width="400"/>
    <p>Phantom X-ray</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/17f51c65171145ffb442/?dl=1" alt="DRIVE" width="400"/>
    <p>Animal X-ray</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/8d8cc2f6820f4b1486ff/?dl=1" alt="Image 3" width="400"/>
    <p>Human X-ray</p>
  </div>
  <div style="text-align: center;">
    <img src="https://vision.aioz.io/f/40d83350df914df78011/?dl=1" alt="Image 3" width="400"/>
    <p>Simulation X-ray</p>
  </div>
</div>

<p align="center"><em><b>Figure 1.</b> Different types of endovascular X-ray data. We aim to train a foundation model which can leverage diverse X-ray data from multiple hospitals (silos) </em></p>


In this work, our goal is to train a foundation model using diverse endovascular datasets with federated learning. Since we aim to use all possible endovascular data (i.e., from humans, animals, phantoms, etc.), there is an unseen data problem between silos. To tackle this problem, we propose the **Fed**erated **E**ndovascular **F**oundation **M**odel (**FedEFM**), a new distillation algorithm using differentiable Earth Mover's Distance (EMD). Once trained, FedEFM provides crucial initializations for downstream tasks, thereby enhancing task-specific performance. Our approach outperforms existing methods and holds significant potential for application in robotic-assisted endovascular surgery, while effectively maintaining data privacy. 


Our contribution can be summarized as below:
- We propose a new method to train a federated endovascular foundation model with unseen data using a multishot distillation technique.
- We propose the **M**ultishot Foundation **F**ederated **D**istillation algorithm **(MFD)**, powered by differentiable Earth Mover's Distance, to address the unseen label corpus issue and ensure the feasibility of learning for the foundation model.
- We collect new datasets for training endovascular foundation models. Our proposed model is verified under several downstream tasks. 
## Related Works

### Endovascular Intervention 

Endovascular intervention has greatly improved the treatment of vascular diseases such as aneurysms and embolisms using X-ray fluoroscopy. However, these procedures still encounter challenges, including low contrast, complex anatomical structures, and the scarcity of expert-annotated data. Recent studies have aimed to address these issues through advancements in imaging technology and machine learning techniques. For instance, researchers have proposed an enhanced U-Net-based approach for localizing guidewire endpoints in X-ray images. More recently, FW-Net has been introduced to improve catheter segmentation by utilizing frame-to-frame temporal consistency. While many studies focus on conventional tasks, few have explored the development of foundation models for endovascular intervention. A key obstacle is the strict requirement for patient data privacy, which significantly limits the ability to train such models.
![](https://vision.aioz.io/f/fa34d42a0b274b91a02f/?dl=1)*<center>**Figure 2**. Endovascular procedure.</center>*
### Federated Learning 

Federated learning has emerged as a promising solution for training machine learning models on decentralized data while preserving data privacy. This approach is especially valuable in the medical field, where data sensitivity and confidentiality are critical concerns. Although numerous studies have investigated the use of federated learning for training foundation models in healthcare, privacy challenges can be mitigated but not entirely eliminated. A major hurdle is the heterogeneity and non-IID nature of medical data across different institutions. Moreover, the issue of unseen data-where certain data types appear in some datasets but are missing in others—further complicates model training and generalization.

![](https://vision.aioz.io/f/91a55c4fa6f64981a0f6/?dl=1)*<center>**Figure 3**. Decentralized federated learning setup</center>*


### Knowledge Distillation with Earth Mover’s Distance.
Knowledge distillation involves transferring knowledge from a large, complex model (the teacher) to a smaller, more efficient model (the student). In the context of federated learning, distillation can be used to enable local models to learn from aggregated global models without sharing raw data. The Earth Mover's Distance (EMD), also known as the Wasserstein distance, measures the dissimilarity between two probability distributions and is particularly useful for comparing distributions that do not have overlapping support. By leveraging the differentiable EMD, it is possible to align distributions of labels across different models, facilitating better model convergence and knowledge transfer. In this paper, we leverage EMD within a distillation training process to address the unseen label data issue when training endovascular foundation models in federated scenarios.

## Next
In the next part, we will explore the process of collecting and managing our dataset for training the foundation model.
