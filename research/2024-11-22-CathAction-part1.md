---
last_modified_on: "2024-11-22"
id: CathAction1
title: CathAction - A Benchmark for Endovascular Intervention Understanding (Part 1)
description: An introduction about CathAction.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "CathAction", "Benchmark for Endovascular Intervention Understanding"]
---

Real-time visual feedback from catheterization analysis is crucial for enhancing surgical safety and efficiency during endovascular interventions. However, existing datasets are often limited to specific tasks, small scale, and lack the comprehensive annotations necessary for broader endovascular intervention understanding. To tackle these limitations, we introduce CathAction, a large-scale dataset for catheterization understanding. Our CathAction dataset encompasses approximately 500,000 annotated frames for catheterization action understanding and collision detection, and 25,000 ground truth masks for catheter and guidewire segmentation. For each task, we benchmark recent related works in the field. We further discuss the challenges of endovascular intentions compared to traditional computer vision tasks and point out open research questions. We hope that CathAction will facilitate the development of endovascular intervention understanding methods that can be applied to real-world applications.
![Intro](https://vision.aioz.io/f/cb314bdb43df43edbfcd/?dl=1)
## 1. Introduction

| **Dataset**                   | **Collection**     | **Type** | **#Frames** | **Source**       | **Annotation** | **Public** | **Task**                          |
|-------------------------------|--------------------|----------|-------------|------------------|----------------|------------|-----------------------------------|
| Barbu et al.                  | X-ray             | Video    | 535         | Real             | Manual         | No         | Segmentation                      |
| Wu et al.                     | 3D Echo           | Video    | 800         | Real             | Manual         | No         | Segmentation                      |
| Ambrosini et al.              | X-ray             | Image    | 948         | Real             | Manual         | No         | Segmentation                      |
| Mastmeyer et al.              | 3D MRI            | Image    | 101         | Real             | Manual         | No         | Segmentation                      |
| Yi et al.                     | X-ray             | Image    | 2,540       | Synthesis        | Automatic      | No         | Segmentation                      |
| Nguyen et al.                 | X-ray             | Image    | 25,271      | Phantom          | Semi-Auto      | No         | Segmentation                      |
| Danilov et al.                | 3D Ultrasound     | Video    | 225         | Synthetic        | Manual         | No         | Segmentation                      |
| Delmas et al.                 | X-ray             | Image    | 2,357       | Simulated        | Automatic      | No         | Reconstruction                    |
| Brost et al.                  | X-ray             | Image    | 938         | Clinical         | Semi-Auto      | No         | Tracking                          |
| Ma et al.                     | X-ray, CT         | Image    | 1,048       | Clinical         | Manual         | No         | Reconstruction                    |
| **CathAction (ours)**         | X-ray             | Video    | 500,000+    | Phantom & Animal | Manual         | Yes        | Segmentation, Action Understanding, Collision Detection |

**Table 1:** Endovascular intervention datasets comparison.

Cardiovascular diseases are one of the leading causes of death worldwide. Endovascular intervention has become the gold standard treatment for these diseases, preferred for its advantages over traditional open surgery, including smaller incisions, reduced trauma, and lower risks of comorbidities for patients. Endovascular interventions involve maneuvering small and long medical instruments, such as catheters and guidewires, within the vasculature through small incisions to reach targeted areas for treatment delivery, such as artery stenting, tissue ablation, and drug delivery. However, such tasks require high technical skill, with the primary challenge being to avoid collisions with the vessel wall, which could result in severe consequences, including perforation, hemorrhage, and organ failure. In practice, surgeons rely on 2D X-ray fluoroscopy images to perform these tasks within the 3D human body, which adds a significant challenge in safely controlling the catheter and guidewire.

Recently, learning-based methods for computer-assisted intervention systems have emerged for diverse tasks. Numerous methodologies have been developed to address the challenges of endovascular interventions, including catheter and guidewire segmentation, vision-based force sensing, learning from demonstration, and skill training assistance. Additionally, various deep learning approaches have been proposed for specific tasks in endovascular interventions, such as instrument motion recognition in X-ray sequences, interventionalist hand motion recognition, and collision detection. However, due to challenges in acquiring medical data, most of these methods rely on synthetic data or small, private datasets. Consequently, despite the critical nature of interventions, current methods have not fully capitalized on recent advancements in deep learning, which typically require large-scale training data.

Over the years, several datasets for endovascular intervention have been introduced. Table 1 shows a detailed comparison between current endovascular intervention datasets. However, these datasets share common limitations. First, they are relatively small in terms of the number of images, as collecting real-world medical data is costly. Second, due to privacy challenges in the medical domain, most existing datasets are kept private. Finally, these datasets are often created for a single task, such as segmentation, and do not support other important tasks in endovascular interventions, such as collision detection or action understanding. 

![Intro](https://vision.aioz.io/f/1c54c73c8b1c47878d3a/?dl=1)

To address these issues, we present **CathAction**, a large-scale dataset encompassing several endovascular intervention tasks, including segmentation, collision detection, and action understanding. To our knowledge, CathAction represents the largest and most realistic dataset specifically tailored for catheter and guidewire tasks.

In summary, we make the following contributions:
- We introduce CathAction, a large-scale dataset for endovascular interventions, providing manually labeled ground truth for segmentation, action understanding, and collision detection.
- We benchmark key tasks in endovascular interventions, including catheterization anticipation, recognition, segmentation, and collision detection.
- We discuss the challenges and open questions in endovascular intervention. Our code and dataset are publicly available.

## 2. Related Work

**Endovascular Intervention Dataset**  
Several endovascular intervention datasets have been introduced. Barbu et al. proposed a dataset that effectively localizes the entire guidewire and validated it using a traditional threshold-based method. Other datasets consider fluoroscopy videos at the image level, with mask annotations for each frame from the fluoroscopy videos. For instance, Ambrosini et al. developed a dataset with 948 annotated mask segmentation instances considering both catheter and guidewire as one class. Similarly, Mastmeyer et al. collected and annotated a dataset with 101 segmentation masks for the real catheter from 3D MRI data. More recently, Nguyen et al. proposed a dataset that considers both catheter and guidewire as one class. Overall, most of these datasets have limitations in terms of size, task categories, and focus. To overcome these limitations, we introduce CathAction, a large-scale dataset with various tasks, including catheter and guidewire segmentation, collision detection, and catheter action recognition and anticipation. The CathAction dataset enables the development of more accurate and reliable deep learning methods for endovascular interventions.

**Catheterization Action Understanding**  
Deep learning techniques have demonstrated notable achievements in endovascular intervention action understanding. Jochem et al. presented one of the first works utilizing deep learning for catheter and guidewire activity recognition in fluoroscopy sequences. Subsequently, deep learning-based approaches have gained prominence as the most widely utilized solution for interventionalist hand motion recognition. For instance, Akinyemi et al. introduced a deep learning model based on convolutional neural networks (CNNs) that incorporates convolutional layers for automatic feature extraction and identifies operators' actions. Additionally, Wang et al. proposed a multimodal fusion architecture for recognizing eight common operating behaviors of interventionists. Despite extensive research on deep learning methods for endovascular intervention, it comes with the limitation of medical data: most of these methods use synthetic data or small, private datasets. This leads to the fact that although intervention is a crucial procedure, it has not fully benefited from recent deep learning advancements, where large-scale training data are usually required.

**Catheter and Guidewire Segmentation**  
Catheter and guidewire segmentation is crucial for real-time endovascular interventions. Many methods have been proposed to address the challenges of catheter and guidewire segmentation. The outcomes of catheter and guidewire segmentation can be applied in vision-based force sensing, learning from demonstration, or skill training assistance applications. Traditional methods for catheterization segmentation adopt thresholding-based techniques and do not generalize well on X-ray data. Deep learning methods can learn meaningful features from input data, but they are challenging to apply to catheter segmentation due to the lack of real X-ray data and the tediousness of manual ground truth labeling. Many current learning-based techniques for catheter segmentation and tracking are limited to training on small-scale datasets or synthetic data due to the challenges of large-scale data collection. Our dataset provides manual ground truth labels for both the catheter and guidewire, offering substantial development for catheter and guidewire segmentation.

**Collision Detection**  
Collision detection is a crucial task in endovascular interventions to ensure patient safety. Several attempts have been made to incorporate deep learning models into collision detection, but these methods have focused on identifying risky actions in simulated datasets. While existing methods can be useful for identifying potential hazards, they cannot localize the position of collisions or provide visual feedback. Additionally, these methods have not been widely used in real-world settings due to the lack of annotated bounding boxes for collisions of guidewire tips with vessel walls. Our dataset addresses this limitation by providing annotated bounding boxes for collision events in both phantom and real-world data. This enables the development of deep learning models that can detect collisions in real-time and provide visual or haptic feedback to surgeons.

## Next
In the next post, we will describe our new dataset **CathAction**.
