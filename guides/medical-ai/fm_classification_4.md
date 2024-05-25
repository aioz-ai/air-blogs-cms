---
last_modified_on: "2024-03-20"
title: Medical Image Classification Downstream tasks using Foundation Model (Part 2).
description: Medical Image Classification Downstream tasks using Foundation Model, ending with Foundation Model.
series_position: 4
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: normal", "guides: medical"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In the previous part,  we have discussed about classification task. In this week, we will mention how foundation model is trained and different foundation models  that can be used for conducting initial weights for down-stream tasks.


## Foundation model for medical image classification task 

Medical images are a major component of patients’ electronic
health records, making medical image analysis an active field of research for machine learning and deep learning. Currently, the extraction of information from medical images is performed manually by radiologists and other professionals in the healthcare field, making the process dependent on the individual’s experience and knowledge, vulnerable to human error, and time-consuming. To address these issues, extensive research has been performed on the usage of machine learning and deep learning in medical image analysis to achieve more efficient and precise results.

Features are important measurements for image understanding. By capturing and extracting the features, the accuracy of the classification task can be greatly improved. The extracted features can be low-level such as color, gradient, edge, shape, intensity, texture, boundary, and position information, as well as high-level such as bag-of-words, scale-invariant feature transform, blobs, and fisher vector. We can select the most important features and remove the redundant ones (by using principal component analysis, single value decomposition, or linear discriminant analysis), then feed the remaining features to classification models such as Support Vector Machine, or Random Forest.

Deep learning algorithms allow the feature extraction and classification steps to be combined in a deep neural network. In addition, deep learning models apply end-to-end learning. Taking the dataset containing the image and label, the deep learning methods extract descriptive, hierarchical, and highly representative features concerning each label, and then feed these features to a classifying module. 

CNN or Deep CNN become the most popular choice in related tasks in computer vision, such as image classification, segmentation, and object detection, ... By stacking multiple convolutional layers to extract the local and global features (AlexNet or LeNet), CNNs have shown remarkable success in computer vision. More and more deeper CNN architectures (ResNet, GoogLeNet, VGG19, OxfordNet, Inception V3, MobileNet) are proposed to improve the performance and minimize the limitations, e.g parameters reduction , efficiently training, lower memory footprint during inference which can support deploying on low-memory devices. Later, Transformer is proposed, which uses attention mechanism instead of convolutional layers. Researchers has shown that deep learning models frequently outperform, traditional machine learning algorithms in image classification tasks. 

![Fig-1](https://vision.aioz.io/f/53a7cd6e64b9409d8381/?dl=1)
*<center>**Figure 1**: Extraction of low-level features from an image of a human brain CT scan. </center>*

![Fig-2](https://vision.aioz.io/f/ee30105d1db34fc4b483/?dl=1)
*<center>**Figure 2**: Extraction of high-level features from an image of a human brain CT scan. VGG-19 is used to extract features from different layers.</center>*


The foundation model refers to a large and powerful pre-trained deep learning model that serves as a starting point for classification downstream tasks. By training on the large-scale dataset, the foundation model can learn from various data modalities and can be easily adapted to many tasks by fine-tuning the weight of the backbone using techniques like transfer learning or meta-learning. Moreover, using foundation models can reduce a huge amount of training time and extensive labeled data. These keys show the potential of foundation models making them valuable tools across a wide range of applications and industries in the field of deep learning.

Here are some researches that can integrate the knowledge of foundation models to the medical image classification tasks:

- **Breast Ultrasound:** *Sun et al.* apply the 2D CNN backbone in the breast ultrasound video classification network and breast ultrasound image classification network. The 2D CNN first extracts the features of frames in the video, then an attention module predicts the attention weight for each frame, and finally, the features are aggregated into a feature vector. The model weights are shared for the two backbones of two models (video and image)

![Fig-3](https://vision.aioz.io/f/6fc5e1c52c7e4342bd2f/?dl=1)
*<center>**Figure 3**: Qualitative results of SAM on some challenging frames. Red rectangles highlight the typical challenging regions which cause unsatisfactory predictions </center>*

- **Breast Ultrasound Tumor Classification:** *Bryar et al.* propose the Hybrid-MT-ESTAN approach, which encompasses tumor classiﬁcation and tumor segmentation. The approach combines the advantages of CNNs and Transformers in a framework incorporating anatomical tissue information in breast ultrasound images. Focus on the classification task, the Hybrid-MT-ESTAN consists of three modules: the ESTAN encoder, the Swin Transformer-based encoder, and a branch with fully connected layers for the classiﬁcation task. The ESTAN encoder consists of the blocks that employ row-column-wise kernels to learn and fuse context information in BUS images at diﬀerent context scales, while the Swin Transformer-based encoder is redesigned with the Anatomy-Aware Attention (AAA) Block by adding the attention block based on the breast anatomy to enhance the ability to model both global and local features.

![Fig-4](https://vision.aioz.io/f/4324608237aa4be1857f/?dl=1)
*<center>**Figure 4**: Hybrid-MT-ESTAN consists of MT-ESTAN and AAA encoders, a segmentation branch, and a classiﬁcation branch. The AAA block improves the learning of contextual information based on the anatomy of the breast </center>*


- **Chest X-ray:** *Nie et al.* propose a causal approach to address the CXR classiﬁcation problem, which involves constructing a structural causal model (SCM) and utilizing backdoor adjustment to select relevant visual information for CXR classiﬁcation. They adopt a ResNet backbone to extract the feature of input images, then capture the causal feature and confounding feature by cross-attention mechanism. The causal part aims to estimate the really useful feature, The confounding part is undesirable for classiﬁcation.


![Fig-5](https://vision.aioz.io/f/f747b3bbb8274b118f4a/?dl=1)
*<center>**Figure 5**: Overview of the Chest X-Ray classification Network. They adopt a CNN-based model to extract the feature of input images, then capture the causal feature and confounding feature by cross-attention mechanism in the transformer module.</center>*


**Hip fracture diagnosis:** *Zhu et al.* aim to integrate features from cross-view X-ray images (Frontal/Lateral-view) for an accurate diagnosis. They present a novel cross-view deformable transformer framework to model relations of critical representations between diﬀerent views for non-displaced hip fracture identiﬁcation. To model the relations among features across diﬀerent views, the framework is designed as a joint of two view-speciﬁc deformable transformer branches. For each view-speciﬁc branch, the input image is processed with shifted window attention modules to aggregate discriminative local features, while deformable self-attention modules are utilized to model the relations among tokens. Moreover, frontal queries are passed to model relations among Lateral features for cross-view deformable attention.


![Fig-6](https://vision.aioz.io/f/78e22b80110b4afb9acb/?dl=1)
*<center>**Figure 6**: Illustration of the proposed framework depicted in two view-speciﬁc branches (frontal, lateral).</center>*


![Fig-7](https://vision.aioz.io/f/781a5ff6ec3d4346bbc8/?dl=1)
*<center>**Figure 7**:  Visualization classification results. The interest area is annotated with green arrows and rectangles</center>*


- **Skin lesion:** *Yang et al.* leverage the diffusion strategy to eliminate unexpected noise and perturbations in medical images and capture semantic representation. Given the image $x$, it was fed to the image encoder to obtain the image embedding $\rho(x)$. For the diffusion step, the image $x$ is first fed into the encoder and a $1\times 1$ convolution layer to generate a saliency map of the whole image, and the global prior $\hat{y}_p$. The local prior $\hat{y}_l$ is computed with the significant region of interests (ROIs) in the saliency map. The ground truth $y_0$,$\hat{y}_p$ and $\hat{y}_l$  are then passed to the diffusion process to obtain three variables $y^g_t$, $y^l_t$, and $y_t$. These variables are then combined, projected, and then fed to the denoising UNet together with the image embedding $\rho(x)$ for further classification. The global and local conditional priors can help the model learn the ﬁne-grained information, and diﬀerentiate the regions from the background in low contrast image. 

![Fig-8](https://vision.aioz.io/f/98b60f423ba745a0a515/?dl=1)
*<center>**Figure 8**: DiﬀMIC framework. The Image Encoder can be treated as the foundation model.</center>*



## References 
1. Zhu, Zhonghang, et al. "Cross-View Deformable Transformer for Non-displaced Hip Fracture Classification from Frontal-Lateral X-Ray Pair." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
2. Sun, AnLan, et al. "Boosting Breast Ultrasound Video Classification by the Guidance of Keyframe Feature Centers." arXiv preprint arXiv:2306.06877 (2023).
3. Shareef, Bryar, et al. "Breast Ultrasound Tumor Classification Using a Hybrid Multitask CNN-Transformer Network." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
4. Nie, Weizhi, et al. "Chest X-ray Image Classification: A Causal Perspective." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
5. Yang, Yijun, et al. "Diffmic: Dual-guidance diffusion network for medical image classification." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
6. Litjens, Geert, et al. "A survey on deep learning in medical image analysis." Medical image analysis 42 (2017): 60-88.
