---
last_modified_on: "2023-12-23"
title: Medical Image Tasks.
description: Medical Image Downstream tasks using Foundation Model.
series_position: 2
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: normal", "guides: medical"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Foundation models have gained significant interest lately in various deep-learning problems. Being trained by large-scale data from multiple modalities, pretrained foundation model can be efficiently adapted to many downstream tasks in computer vision and the medical field. This blog surveys works that integrate the foundation model as the backbone in medical image segmentation tasks.

## Introduction

Foundation model, by definition of Stanford HAI, is mentioned as any model that is trained on broad data (generally using self-supervision at scale) that can be adapted (e.g., fine-tuned) to a wide range of downstream tasks. By training on large-scale data, the foundation model can narrow the gap between multi modalities, and support contextual reasoning, generalization, and prompt capabilities in real time. 

Thanks to the ability to learn from large-scale data from multiple modalities, the foundation model can easily applied to various downstream tasks by adapting task-specific hints (Linear layer for classification, mask decoder for segmentation, or general decoder for generation) and fine-tuning the knowledge weight. Moreover, the downstream task adaption does not require extensive labeled data. 

Foundation models play a crucial role in advancing the field of deep learning by providing a starting point for various applications, saving computational resources, and accelerating the development of models for specific tasks within a given domain.

In medical applications, foundation models also attract attention from researchers. They can adaptively interpret multiple medical modalities, including medical images, 2D and 3D scans, records, textual data, ... Hence, foundational models have the potential to provide an enhanced foundation for addressing clinical issues, advancing the field of medical imaging, and improving the efficiency and effectiveness of diagnosing and treating diseases, leading to the opportunity to develop a unified biomedical AI system that can interpret complex multimodal data. Their ability to learn from vast amounts of data and adapt to new information positions them as powerful tools in advancing the quality and accessibility of healthcare services.

The application of knowledge from foundation models to the medical field also helps release the challenges for researchers in this field: 
- Most medical models are limited to single-task, unimodal functions. For instance, a mammogram interpretation AI excels at breast cancer screening but can’t incorporate patient records, and additional data like MRI, or engage in meaningful dialogue, limiting its real-world applicability
- The absence of explainability in deep learning models can erode trust among clinicians accustomed to clear clinical insight. Foundation models address these issues by offering a unified framework for tasks like detection and classification, often trained on diverse datasets from various medical centers, enhancing their potential for clinical use by ensuring interpretability and broad applicability.
- The computer vision community has a history of open-sourcing datasets, but in medical imaging, privacy regulations limit data sharing. Foundation models offer a privacy-preserving alternative by allowing knowledge
transfer without direct access to sensitive data.
- Existing medical models struggle when faced with distribution shifts caused by changes in technology, procedures, settings, or populations. In contrast, the foundation model can effectively adapt to these shifts through in-context learning.

Medical image segmentation has emerged as a crucial area of research and development by partitioning an input medical image into meaningful and distinct regions, such as organs, tissues, or pathological lesions. Accurate segmentation is essential for extracting quantitative information, understanding anatomical structures, and facilitating precise localization of abnormalities.



## Image Segmentation Models

### UNet

UNet is the most well-known network architecture in the medical image segmentation models. It has been widely adopted in medical image segmentation tasks, such as segmenting organs, tumors, or other structures in medical images. Its ability to capture contextual information while preserving spatial details makes it well-suited for applications where precise localization is crucial, such as in medical imaging. 

UNet architecture consists of a contracting path (left side) and an expansive path (right side). The contracting path uses a deep convolutional neural network for extracting deep features, where the number of features is doubled. Every step in the expansive path consists of an upsampling of the feature map followed by an up convolution that halves the number of feature channels, a concatenation with the correspondingly cropped feature map from the contracting path. 

Since the success of the U-Net model, several improved versions based on UNet have emerged, including UNet++, Attention-UNet, TransUNet, and Swin-Unet, among others. The design of the UNet encoder can be used in another network for extracting deep features.

![Fig-1](https://drive.google.com/uc?export=view&id=1KJuhPFWjI1_pEHDK4bZQV7kVGxJImd4a)
*<center>**Figure 1**:  UNet architecture.</center>*


### UNet++ 

UNet++ is an extension of the original UNet architecture designed to further improve the performance of semantic segmentation tasks, including medical image segmentation. It was proposed by *Zhou et al.* in 2018. UNet++ builds upon the strengths of UNet while addressing some of its limitations.

UNet++ enhances the connectivity between encoder and decoder paths by introducing nested skip pathways. In the original UNet, skip connections were established between corresponding layers in the encoder and decoder. UNet++ goes further by incorporating additional connections at different resolutions within each encoder level, creating a nested and more interconnected structure.

![Fig-2](https://drive.google.com/uc?export=view&id=13daaxGH-WoEhmhE_JslG9mdAX5q5wLbL)
*<center>**Figure 2**:  UNet++ Architecture.</center>*
- Transformer (TransUNet, SwinUNet)

### TransUNet
 
TransUNet is a neural network architecture designed for medical image segmentation, combining the strengths of transformer models and convolutional neural networks (CNNs). It was introduced by *Chen et al.* in the paper **"TransUNet: Transformers Make Strong Encoders for Medical Image Segmentation"** in 2021. 

TransUNet follows the architecture of UNet but uses a Transformer combined with CNN instead of deep CNN in the encoder. In particular, TransUNet first uses CNN to extract features and generate feature maps of the input image. These feature maps are then divided into patches of size 1x1 and fed into an additional stack of 12 Transformer modules. The decoder in TransUNet performs upsampling on the encoded features and combines them with high-resolution CNN feature maps to enrich the semantic information, achieving more precise localization. 

TransUNet has shown promising results in various medical image segmentation tasks, demonstrating its ability to capture long-range dependencies and spatial information effectively. The hybrid architecture addresses some of the limitations of using transformers alone in tasks where spatial relationships are crucial, such as in medical imaging.

![Fig-3](https://drive.google.com/uc?export=view&id=1JvuYnXIsL45x40ic2YLsqoshHbIh8ePh)
*<center>**Figure 3**:  TransUNet Architecture.</center>*

### Swin-UNet 
Swin-Unet network architecture was proposed by *Cao et al.* in 2023. Swin-Unet utilizes Swin Transformer blocks to extract hierarchical features from the input image. The idea behind combining Swin Transformer and UNet architectures is to leverage the efficiency in capturing long-range dependencies offered by transformers along with the precise localization capabilities of UNet, making it potentially well-suited for medical image segmentation tasks.

![Fig-4](https://drive.google.com/uc?export=view&id=1thU7TLR8HtUdka6swpiaGWWmLiMjMlHE)
*<center>**Figure 4**: SwinUNet Architecture.</center>*

### SegViT
SegViT is a plain vision transformer for semantic segmentation developed by *Zhang et al.*. Different from the traditional pixel-level classification, they aim to pay attention to the spatial feature maps for the segmentation mask given the class tokens as queries through the simple decoder module named Attention-to-mask (ATM). Moreover, they employ the shrink structure consisting of query-based down-sampling (QD) and query-based up-sampling (QU). While QD can reduce the resolution, QU can be used parallel to recover the resolution. By applying the ATM module, SegViT outperforms plain ViT in ADE20k, COCO-Stuff-10k, and PASCAL-Context datasets. By implementing a shrink structure, SegViT can save up to 40% on computation costs in the structure with ViT backbone. 

A new version of SegViT (SegViTv2) has been released in a conference work at *IJCV 2023*. SegViTv2 employed more powerful backbones (BEiT-V2) and achieved better mIoU.  Moreover, SegViT can further reduce computation costs with Shrink++, in which they extend QD to edge-aware QU (EQU) to preserve tokens situated at object edges. 

![Fig-5](https://drive.google.com/uc?export=view&id=1FHcEsOlLZFYYymHbyNnOBKV11ru0Mn5p)
*<center>**Figure 5**: SegViT architecture with Attention-to-mask (ATM) module.</center>*


### SAM
SAM (Segment Anything model) is the model for image segmentation developed by Meta. SAM produces high-quality object masks from input prompts such as points or boxes, and it can be used to generate masks for all objects in an image. It has been trained on a dataset of 11 million images and 1.1 billion masks and has strong zero-shot performance on a variety of segmentation tasks. Therefore, SAM is considered as the first promptable foundation model for segmentation. The architecture of SAM has three components: 
- Image encoder: Vision transformer is used for extracting features, and it is trained using *Mask Autoencoder* (MAE) for efficiently processing high-resolution images.
- Prompt Encoder: The authors consider two types of prompts. For the sparse (including point, box, and text) prompt, point, and box are represented by summing the position encoding and learned embedding, while text is free-formed and encoded by *pre-training text encoder of CLIP*.  For the dense prompts (masks), are embedded using convolutions and summed element-wise with the image embedding. 
- Mask encoder:  The mask decoder consists of two transformer layers with a dynamic mask prediction head and an Intersection-over-Union (IoU) score regression head.

![Fig-6](https://drive.google.com/uc?export=view&id=1LAwTP4AZI6Gvu4OBz0FRS7Kffhk5Gi_p)
*<center>**Figure 6**:  Segment Anything Model (SAM).</center>*

## Medical Image Segmentation Works


**Brain Tumor Segmentation**. *Wang et al.* propose A2FSeg, a simple multi-model fusion network for brain tumor segmentation. A2FSeg first extracts the 3D input from different modalities using UNet. Then, the averaged fusion module aggregates the image features from different modalities by averaging them and also handles the possibility of missing any modality. Finally, the Adaptive Fusion Module takes the input feature from each modality and the mean feature from the averaged fusion module, and measures the voxel-level contributions of that modality to the ﬁnal segmentation using the attention mechanism. They perform experiments on 3D MRI scans with 4 different modalities: T1, T1c, T2, and Flair, and achieve promising results for the incomplete multi-modal brain tumor segmentation.

![Fig-7](https://drive.google.com/uc?export=view&id=1t7Ym474EtCbj4EvQQE6PzoDdOh_Fe8AQ)
*<center>**Figure 7**:  A2FSeg architecture.</center>*




**Pelvic Fracture Segmentation**. *Liu et al.* propose a deep-learning method for automatic pelvic fracture segmentation. They aim to segment the major and minor fragments of target bones (left and right ilia and sacrum). First, an anatomical segmentation network which uses a pretrained cascaded 3D UNet architecture, extracts the pelvic bones from the CT scan. Second, fracture segmentation separates the bone fragments from each iliac and sacral region extracted from the ﬁrst step. Finally, isolated components are further separated and labeled, and small isolated bone fragments are removed to form the ﬁnal output. 

![Fig-8](https://drive.google.com/uc?export=view&id=1EYS3h1e26N6iAplJxTHAlGD9rOQX0psz)
*<center>**Figure 8**:  Pelvic Fracture Segmentation network architecture. It uses 3D UNet for extracting features.</center>*



**Pathology Image Segmentation**. *Deng et al.* evaluate SAM on tumor segmentation, non-tumor tissue segmentation, and cell nuclei segmentation on whole slide imaging (WSI). By conducting several different scenarios including SAM with a single positive point prompt, with 20 point prompts with 10 positive and 10 negative points, and with all points/boxes on every single instance object, the results suggest that SAM achieves remarkable segmentation performance for large connected objects.

![Fig-9](https://drive.google.com/uc?export=view&id=1JGxlJttO-HIRBQ3xI6MzbY76t5XXGqC3)
*<center>**Figure 9**:  Qualitative pathology segmentation visualizations.</center>*

**Polyps Segmentation from Colonoscopy Images**. *Zhou et al.* evaluate the performance of SAM with unprompted settings in segmenting polyps from colonoscopy images on five benchmark datasets.

![Fig-10](https://drive.google.com/uc?export=view&id=1hftrkIi2WX8vf7w6SzXNt_OQsTwOqI96)
*<center>**Figure 10**: Segmentation results comparison between SAM and other methods.</center>*

**Endoscopic Surgical Instrument Segmentation**. *Wang et al.* evaluate the performance of SAM on two classical datasets in endoscopic surgical instrument segmentation. Experimental results suggest that SAM is deficient in segmenting the entire instrument with point-based prompts and unprompted settings

![Fig-11](https://drive.google.com/uc?export=view&id=191BtCMnrcwr-993YrKZifZh-4ACeVQo4)
*<center>**Figure 11**: Qualitative results of SAM on some challenging frames. Red rectangles highlight the typical challenging regions which cause unsatisfactory predictions </center>*

## References 
1. R. Deng, C. Cui, Q. Liu, T. Yao, L. W. Remedios, S. Bao, B. A. Landman, L. E. Wheless, L. A. Coburn, K. T. Wilson et al., “Segment anything model (sam) for digital pathology: Assess zero-shot segmentation on whole slide imaging,” arXiv preprint arXiv:2304.04155, 2023.
2. T. Zhou, Y. Zhang, Y. Zhou, Y. Wu, and C. Gong, “Can sam segment polyps?” arXiv preprint arXiv:2304.07583, 2023.
3. A.-C. Wang, M. Islam, M. Xu, Y. Zhang, and H. Ren, “Sam meets robotic surgery: An empirical study in robustness perspective,” arXiv preprint arXiv:2304.14674, 2023.
4. Wang, Zirui, and Yi Hong. "A2FSeg: Adaptive Multi-modal Fusion Network for Medical Image Segmentation." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
5. Liu, Yanzhen, et al. "Pelvic Fracture Segmentation Using a Multi-scale Distance-Weighted Neural Network." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
6. Ronneberger, Olaf, Philipp Fischer, and Thomas Brox. "U-net: Convolutional networks for biomedical image segmentation." Medical Image Computing and Computer-Assisted Intervention–MICCAI 2015: 18th International Conference, Munich, Germany, October 5-9, 2015, Proceedings, Part III 18. Springer International Publishing, 2015.
7. Zhou, Zongwei, et al. "Unet++: A nested u-net architecture for medical image segmentation." Deep Learning in Medical Image Analysis and Multimodal Learning for Clinical Decision Support: 4th International Workshop, DLMIA 2018, and 8th International Workshop, ML-CDS 2018, Held in Conjunction with MICCAI 2018, Granada, Spain, September 20, 2018, Proceedings 4. Springer International Publishing, 2018.
8. Chen, Jieneng, et al. "Transunet: Transformers make strong encoders for medical image segmentation." arXiv preprint arXiv:2102.04306 (2021).
9. Cao, Hu, et al. "Swin-unet: Unet-like pure transformer for medical image segmentation." European conference on computer vision. Cham: Springer Nature Switzerland, 2022.
10. Zhang, Bowen, et al. "Segvit: Semantic segmentation with plain vision transformers." Advances in Neural Information Processing Systems 35 (2022): 4971-4982.
11. Zhang, Bowen, et al. "SegViTv2: Exploring Efficient and Continual Semantic Segmentation with Plain Vision Transformers." arXiv preprint arXiv:2306.06289 (2023).
