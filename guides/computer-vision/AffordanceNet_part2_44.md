---
last_modified_on: "2023-26-08"
title: Object Affordance Detection
description: Object Affordance Detection and its effectiveness
series_position: 44
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
![](https://vision.aioz.io/f/1e87d34d0d5e49e19004/?dl=1)
In this segment, we will proceed to validate and assess the efficiency and efficacy of the Object Affordance Detection Network. This pivotal phase will help us ascertain the capabilities and performance of our proposed network in accurately detecting and recognizing object affordances.

## Training and Inference
The authors employed stochastic gradient descent with a momentum of 0.9 and weight decay of 0.0005. The network underwent training for 200k iterations on a Titan X GPU. Initially, a learning rate of 0.001 was set for the first 150k iterations and subsequently reduced by a factor of 10 for the remaining 50k iterations. Input images were resized so that the shorter edge was 600 pixels, with the longer edge not exceeding 1000 pixels. In cases where the longer edge surpassed 1000 pixels, it was set to 1000 pixels, and image resizing was performed accordingly. The RPN utilized 15 anchors (5 scales and 3 aspect ratios). The top 2000 RoIs from the RPN were employed for computing the multi-task loss, maintaining a positive-to-negative ratio of 1:3. An RoI was deemed positive if its IoU with a groundtruth box was at least 0.5, and negative otherwise. During inference, the top 1000 RoIs produced by the RPN were selected, followed by the object detection branch and non-maximum suppression. Objects with classification scores greater than 0.9 were considered as detected objects. If no boxes fulfilled this criterion, the one with the highest classification score was chosen. Detected objects were then used as inputs for the affordance detection branch. For each pixel in the detected object, the affordance branch predicted C + 1 affordance classes, with the output affordance label determined by selecting the maximum across these classes. The predicted 244x244 affordance mask of each object was resized to the object size.

## Dataset and Baseline

**IIT-AFF Dataset.**
The IIT-AFF dataset encompasses 8,835 real-world images. This dataset is well-suited for deep learning methods and robotic applications, with approximately 60% of the images sourced from the ImageNet dataset, while the remaining images were captured by the authors in cluttered scenes. Specifically, the dataset includes 10 object categories, 9 affordance classes, 14,642 object bounding boxes, and 24,677 pixel-level affordance regions. The authors adopted the standard split for network training, allocating 70% for training and 30% for testing.

**UMD Dataset**
The UMD dataset encompasses roughly 30,000 RGB-D images showcasing daily objects from kitchens, workshops, and gardens. These RGB-D images were acquired using a Kinect camera on a rotating table within a clutter-free setup. The dataset comprises 7 affordance classes and 17 object categories. Due to the absence of ground truth for object bounding boxes, the authors deduced the rectangle coordinates of object bounding boxes based on the affordance masks. Solely the RGB images of this dataset were utilized.

**Baseline**
As the customary practice, the $F_B$ metric was employed for assessing the affordance detection outcomes. The authors compared their AffordanceNet against several state-of-the-art methods, which include DeepLab both with and without post-processing via CRF (designated as DeepLab and DeepLab-CRF, respectively), CNN with encoder-decoder architecture employed on RGB and RGB-D images (referred to as ED-RGB and ED-RGBD), CNN incorporating an object detector (BB-CNN) alongside CRF (BB-CNN-CRF). Additionally, results from geometric features-based approaches (HMD and SRF) and a deep learning-based approach that utilized both RGB and depth images (ED-RGBHHA) were reported for the UMD dataset. It's noteworthy that all deep learning-based methods employed VGG16 as the primary backbone, ensuring a fair comparison.

## Results
**IIT-AFF Dataset**
Table.1 presents a comprehensive summary of results achieved on the IIT-AFF dataset. The outcomes distinctly underscore the substantial enhancement that AffordanceNet brings over the existing state of the art. Particularly noteworthy is AffordanceNet's elevation of the $F_B$ score to 73.35, signifying a noteworthy 3.7% advancement compared to the second-ranking BB-CNN-CRF. It's pertinent to mention that AffordanceNet achieves this feat employing an end-to-end architecture, obviating the need for supplementary post-processing steps like CRF. Moreover, AffordanceNet secures the topmost position across all 9 affordance classes. Notably, in scenarios with cluttered scenes as exemplified by the IIT-AFF dataset, the combined approaches integrating object detectors with deep networks for affordance prediction (AffordanceNet, BB-CNN) substantially outperform methods solely reliant on deep networks (DeepLab, ED-RGB). See Figure.1 for visualization results.

![](https://vision.aioz.io/f/070616db3e9e40f0aec7/?dl=1)*<center>**Table 1**. PERFORMANCE ON IIT-AFF DATASET. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

**UMD Dataset**
Table.2 provides an inclusive overview of the outcomes registered on the UMD dataset. On average, AffordanceNet once again attains the highest performance on this dataset, outperforming the second-best competitor (ED-RGBD) by 2.9%. It's pertinent to acknowledge that the UMD dataset solely features clutter-free scenes, hence the improvement by AffordanceNet over its counterparts is somewhat moderated compared to the real-world IIT-AFF dataset. The authors reiterate that AffordanceNet is trained solely using RGB images, whereas the runner-up (ED-RGBD) employs both RGB and depth images. Table.2 notably establishes the dominance of deep learning-based methods, including AffordanceNet, DeepLab, and ED-RGB, as they markedly outstrip the geometric feature-based approaches (HMP, SRF).

![](https://vision.aioz.io/f/71685f1e980d4a04b824/?dl=1)*<center>**Table 2**. PERFORMANCE ON UMD DATASET. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

In conclusion, AffordanceNet yields remarkable advancements over the existing benchmarks, all while circumventing the necessity for additional post-processing or data augmentation steps. The robotic applications standpoint underscores the multifaceted utility of AffordanceNet, serving diverse tasks by furnishing object locations, categories, and affordances in an integrated manner. The operational time of AffordanceNet hovers around 150ms per image on a Titan X GPU, rendering it eminently suitable for robotic applications. 

![](https://vision.aioz.io/f/dc5496e01596461c902b/?dl=1)*<center>**Figure 1**.  Examples of affordance detection results by AffordanceNet on the IIT-AFF dataset. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

## Affordance Detection in The Wild
The experimental outcomes on both the relatively constrained UMD dataset and the real-world IIT-AFF dataset, as detailed by the authors, underscore the robust performance of AffordanceNet across publicly available research datasets. However, the authors do acknowledge the potential for greater challenges posed by real-life images. To substantiate the versatility of AffordanceNet across varied testing environments, qualitative results are presented. These qualitative assessments showcase AffordanceNet's adeptness in generalizing to different scenarios. An illustrative case is presented in Figure.2, where AffordanceNet efficiently detects objects and their respective affordances in diverse contexts, encompassing artwork images and images procured from a simulated camera within the Gazebo simulation. While this qualitative demonstration serves as a preliminary illustration, it resonates with the authors' assertion that AffordanceNet holds applicability across an extensive array of contexts, notably including simulation environments pivotal for advancing robotic applications.

![](https://vision.aioz.io/f/a14bfd6aba164af18d9e/?dl=1)*<center>**Figure 2**.   Affordance detection in the wild. (a) and (b):AffordanceNet is used to detect the objects in Gazebo simulation. c) AffordanceNet also performs well when the input is an artwork. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 


## Robotic Applications
Given its rapid processing speed of 150ms per image, AffordanceNet emerges as a well-suited candidate for integration into robotic applications. To substantiate this assertion, the authors employ the humanoid robot WALK-MAN for a series of diverse manipulation experiments. The robot's real-time control is orchestrated through the XBotCore framework, while the OpenSoT library generates whole-body motion planning. In this context, AffordanceNet serves as the visual information source for the robot. Notably, the authors elaborate that the 2D output from AffordanceNet is translated into 3D space using the corresponding depth image, a prerequisite for the real robot's operational dynamics. This setup empowers the robot to engage in multifarious tasks such as grasping, pick-place, and pick-pouring. Significantly, all the insights gleaned from AffordanceNet, encompassing object locations, object labels, and object affordances, prove integral for these tasks. For instance, the robot leverages the grasp affordance to pinpoint where to grasp a bottle, and it relies on the pan's contain affordance to accurately pour water into a pan, as visually depicted in Figure 3.

![](https://vision.aioz.io/f/cbe8b90f94f14d748d1f/?dl=1)*<center>**Figure 3**.   WALK-MAN is performing a pouring task. ([source](https://arxiv.org/pdf/1709.07326.pdf))</center>* 

## Summary
The authors have introduced AffordanceNet, a comprehensive end-to-end deep learning framework designed to concurrently detect objects and their associated affordances. This framework departs from conventional network architectures used for instance segmentation. Notably, the authors have devised a triad of crucial components to effectively tackle the challenge posed by multiple affordance classes within the affordance detection task. These components include a series of deconvolutional layers, a resilient resizing strategy, and an innovative loss function. It is demonstrated that these components collectively play a pivotal role in achieving heightened accuracy in affordance detection. Rigorous experimental evaluations affirm that AffordanceNet surpasses existing benchmarks on public datasets, thereby establishing itself as a cutting-edge solution. Furthermore, its applicability extends to a spectrum of robotic applications, underscoring its adaptability and versatility.

##  Reference
Do, Thanh-Toan, Anh Nguyen, and Ian Reid. "Affordancenet: An end-to-end deep learning approach for object affordance detection." 2018 IEEE international conference on robotics and automation (ICRA). IEEE, 2018.















