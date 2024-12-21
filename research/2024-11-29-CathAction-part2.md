---
last_modified_on: "2024-11-29"
id: CathAction2
title: CathAction - A Benchmark for Endovascular Intervention Understanding (Part 2)
description: Describe and analyze our CathAction dataset.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "CathAction", "Benchmark for Endovascular Intervention Understanding"]
---

## 1. The CathAction Dataset

This section introduces the CathAction dataset. Specifically, we describe the data collection process and annotation pipeline. We then present statistics regarding different aspects of our large-scale dataset.

### Data Collection

Given that endovascular intervention constitutes a medical procedure, acquiring extensive human data is often impractical and time-consuming due to privacy constraints. To address this challenge, we suggest an alternative approach involving the collection of data from two distinct sources: 

1. Utilizing vascular soft silicone phantoms modeled after the human body.
2. Employing animal subjects, specifically pigs. The selection of pigs is justified by their vascular anatomy, which is widely acknowledged as highly analogous to that of humans.

**Ethics**  
Since our data collection involves experiments with radiation sources (X-ray radiology fluoroscopic systems) and live animals, all relevant ethical approvals were obtained in advance of the collection process. The human subjects who collect the data are well-trained and professional endovascular surgeons, wearing protective suits as part of daily practice in the hospital.

(a) Silicon phantom        |  (b) Data collection setup
:-------------------------:|:-------------------------:
![Figure-1a](https://vision.aioz.io/thumbnail/99ff8cef229b4f679bf3/1024/phantom.jpg)|![Figure-1b](https://vision.aioz.io/thumbnail/ec08b6b4c56543c7bf5c/1024/scanner.jpg)

**Figure 1:**: The human silicon phantom model (a), and data collection setup in the operating room (b).


**Phantom Setup**  
To ensure that data is collected from various models, we use five adult human aortic arch phantoms made of soft silicone, manufactured by Elastrat Ltd., Switzerland. To enhance realism in the interaction between surgical tools and tissues, the phantoms are connected to a pulsatile pump to simulate the flow of normal human blood. All phantoms are placed beneath an X-ray imaging system to mimic a patient lying on an angiography table, preparing for an endovascular procedure.

**Animal Setup**  
We use five live pigs as subjects for data collection. The animal setup is identical to that of a human procedure. During the endovascular intervention, professional surgeons use an iodine-based contrast agent to enhance visibility of specific structures or fluids within the body. Iodine contrast agents are radiopaque, meaning they absorb X-rays, resulting in improved visibility of blood vessels, organs, and other structures such as the catheter and guidewire during imaging.

![Figure 2](https://vision.aioz.io/thumbnail/8d94c421aafa46dbab01/1024/vls_collect_data.v2-1.png)

**Figure 2:** Example data collected with phantom models (top row) and animals (bottom row). Animal data are more challenging with less visible catheters or guidewires.

**Data Collection**  
Ten skilled professional surgeons are tasked with cannulating three arteries, namely the left subclavian (LSA), left common carotid (LCCA), and right common carotid (RCCA), using a commercial catheter and guidewire. Throughout each catheterization process, the surgeon operator activates the X-ray fluoroscopy using a pedal in the operating room. We developed a real-time image grabber to transmit the video feed of the surgical scene to a workstation. The experiments are conducted under two interventional radiology fluoroscopic systems: Innova 4100 IQ GE Healthcare and EMD Technologies Epsilon X-ray Generator. Fig 1 shows the data collection setup with human silicon phantoms and Fig 2 visualizes the collected data with phantom models and real animals. From 3, we can see that there is a huge *domain gap* between data collected using phantom models and live animals.

### Data Annotation

**Actions**  
Based on advice from expert endovascular surgeons, we define five classes to annotate catheterization actions. These classes fall into three groups: catheter (\texttt{advance catheter} and \texttt{retract catheter}), guidewire (\texttt{advance guidewire} and \texttt{retract guidewire}), and one action involving both the catheter and guidewire (\texttt{rotate}). Surgeons typically rotate both the catheter and guidewire simultaneously, so we use one rotation class. We utilize a free, open-source video editor to annotate the start and end times of each narrated action. All fluoroscopy videos are processed at a 500 x 500 resolution and 24 frames per second (FPS). To ensure annotation quality, all ground-truth actions are manually checked and modified by an experienced endovascular surgeon.

**Collision Annotation**  
In practice, the collision between the catheter (or guidewire) and the blood vessel wall mainly occurs at the instrument's tip. Therefore, for each frame of the fluoroscopy video, we annotate the catheter (or guidewire) tip with a bounding box. There are two classes for the bounding boxes: \texttt{collision} (when the instrument collides with the blood vessel) and \texttt{normal} (when there is no collision). We used an open-source labeling tool to annotate bounding boxes in each video, with all videos encoded at 24 FPS to ensure dataset coherence.

**Segmentation**  
The combination of guidewire and catheter is common in endovascular interventions, where precise navigation through blood vessels is essential for procedure success. Unlike most previous datasets that consider both catheter and guidewire as one class, we manually label catheter and guidewire classes separately in our dataset. Our segmentation ground truth thus provides a more detailed understanding of endovascular interventions.

### Dataset Statistics

**Overview**  
As summarized in Table 1 in the previous part 1, CathAction is a large-scale benchmark for endovascular interventions. Our dataset consists of approximately 500,000 annotated frames for action understanding and collision detection, and around 25,000 ground-truth masks for catheter and guidewire segmentation. There are a total of 569 videos in our dataset. Some collected video samples are illustrated in Fig 2. We believe CathAction is currently the largest, most challenging, and most comprehensive dataset of endovascular interventions.

**Statistics**  
The CathAction dataset is annotated with a primary focus on catheters and guidewires. Fig. 3 provides an overview of the distribution of action classes in both animal and phantom data, while Fig. 4 portrays the distribution of action segment lengths, illustrating the substantial variability in segment duration. Additionally, Fig. 5 visually compares the number of bounding boxes between phantom and animal data, revealing a significant disparity between counts of normal and collision boxes, as expected due to the infrequency of collisions in real-world scenarios.

![Figure 3](https://vision.aioz.io/thumbnail/1ef51fef0ba640e6b066/1024/statitics_fig1.v22-1.png)

**Figure 3:** Distribution of the number of action classes in the CathAction dataset. Left-side: Distribution on real animal data. Right-side: Distribution on phantom data.

![Figure 4](https://vision.aioz.io/thumbnail/d749b0326ade408c885d/1024/statitic_fig.2_v24-1.png)

**Figure 4:** Duration distribution of segments' actions in the CathAction dataset, on real animal data and phantom data.

![Figure 5](https://vision.aioz.io/thumbnail/756e0dbf87c344479569/1024/statitic_fig4_v31-1.png)

**Figure 5:** Comparison of the number of bounding box objects in real animal data and phantom data.

**Adaptation Property**  
Since data is collected from two sources—phantoms and real animals—a domain gap exists between the two data types. Fig. 2 and Fig. 5 also demonstrate the adaptation property shared between phantom and animal data. This distinctive domain gap renders CathAction a formidable benchmark for evaluating domain adaptation, a critical problem in medical domains where collecting real human data is often infeasible. Using CathAction, we can develop domain adaptation techniques, learning from synthetic or phantom data and effectively applying that knowledge to genuine animal or human data, bridging the gap between controlled simulation and real-world scenarios.


## Next
In the next post, we will benchmark our new dataset **CathAction** on various tasks.