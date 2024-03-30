---
last_modified_on: "2024-03-10"
title: Medical Image Classification Downstream tasks using Foundation Model (Part 1).
description: Medical Image Classification Downstream tasks using Foundation Model, starting with Image Classification tasks used for Foundation Model.
series_position: 3
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: normal", "guides: medical"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Foundation models have gained significant interest lately in various deep-learning problems. Being trained by large-scale data from multiple modalities, the pre-trained foundation model can be efficiently adapted to many downstream tasks in computer vision and the medical field. This blog surveys works that integrate the foundation model as the backbone of medical image classification as a downstream task.

## Introduction

Computer Vision, a field at the intersection of computer science and artificial intelligence, has witnessed remarkable advancements with the integration of deep learning techniques. Deep Learning, particularly Convolutional Neural Networks (CNNs), has emerged as a powerful paradigm in transforming the way computers perceive and interpret visual information. This synergy has revolutionized various industries, ranging from autonomous vehicles and robotics to healthcare and security.

In recent years, the integration of deep learning techniques into the medical field has revolutionized the way healthcare professionals approach diagnosis, treatment, and patient care. Deep learning, a subset of artificial intelligence (AI), involves the use of sophisticated neural networks to analyze vast amounts of data, recognize patterns, and make complex decisions. The application of deep learning in medicine has ushered in a new era of precision and efficiency, enhancing the capabilities of healthcare systems worldwide.

One of the primary strengths of deep learning lies in its ability to extract meaningful insights from large and diverse datasets. In the medical domain, this capability is particularly crucial given the exponential growth of medical data, including electronic health records, medical imaging, genomic information, and more. Deep learning algorithms excel at identifying intricate patterns within these datasets, leading to improved diagnostic accuracy, early disease detection, and personalized treatment plans.

Medical imaging, such as radiology and pathology, has witnessed significant advancements through the incorporation of deep learning. Deep neural networks can analyze medical images with a level of precision and speed that was previously unattainable. This has resulted in more accurate and timely diagnoses, reducing the margin of error and enhancing patient outcomes. Additionally, deep learning models can aid in the early detection of diseases, enabling proactive intervention and potentially saving lives.

## Medical Image Classification

Medical image classification is a pivotal task within the field of medical imaging that leverages advanced technologies, particularly machine learning, and deep learning, to categorize and interpret visual data acquired through various imaging modalities. The objective is to automatically assign predefined labels or classes to images, and identify which parts of the human body are infected by the disease, enabling healthcare professionals to streamline diagnostics, enhance treatment planning, and improve patient care.

The big challenge of medical image classification tasks, as well as other tasks in the medical field, is the lack of labeled and reliable data. This is a common challenge that researchers and practitioners face in the field of healthcare and artificial intelligence. Several factors contribute to this issue:

- **Data Privacy and Security**: Medical data, including images, are often sensitive and subject to strict privacy regulations. Access to such data is limited due to concerns about patient confidentiality. This restricts the amount of data that can be used for training deep learning models.

- **Data Fragmentation**: Medical data is often fragmented across different healthcare institutions, making it challenging to aggregate a large and diverse dataset. Different imaging devices, formats, and protocols also contribute to the fragmentation of data.

- **Annotation and Labeling**: Creating accurate and reliable annotations for medical images is a time-consuming and labor-intensive process. Expert radiologists are typically required to review and label images, and this process can be a bottleneck in generating large labeled datasets.

- **Imbalance and Rarity of Diseases**: Some medical conditions are rare, leading to imbalances in the dataset. Deep learning models may struggle to learn from underrepresented classes, potentially leading to biased or less accurate predictions for these cases.

- **Data Quality**: The quality of medical images can vary, and low-quality images may not provide sufficient information for training effective deep learning models. Noise, artifacts, and inconsistencies in data quality can impact the model's performance. 

Medical images are visual representations of the interior of the human body or other living organisms, obtained through various imaging modalities for diagnostic or research purposes. These images provide valuable information to healthcare professionals, aiding in the detection, diagnosis, and monitoring of diseases and conditions. Here are some common types (modalities) of medical images:

- **Computed Tomography (CT):** Computerized procedure used to generate cross-sectional X-ray images, which can be combined to create a three-dimensional image, providing a more detailed representation of the region compared to conventional X-rays. CT scans can be used in medical tasks such as organ and tissue visualization, detection of tumors, or trauma assessment.

- **X-Ray:** X-Ray is the most common medical imaging technique in which electromagnetic waves are used to create a two-dimensional representation of the body’s internal structure based on the varying rates of wave absorption in tissues with different densities. Research works using X-Ray in medical imaging applications include fracture detection, lung imaging, or dental examinations.

- **Magnetic resonance imaging (MRI):** MRI generates three-dimensional anatomical images based on the behavior of the atomic nuclei in response to powerful magnetic fields and radiofrequency pulses. MRI scans can be used in deep-learning medical tasks such as brain and spinal cord imaging, musculoskeletal studies, or cardiac imaging.

- **Ultrasound**: Ultrasound imaging uses high-frequency sound waves to generate two-dimensional images of organs and tissues based on the reflection of the sound waves from different body structures. The datasets that contain ultrasound images are used in tasks such as obstetrics (pregnancy monitoring), abdominal imaging, vascular studies, breast cancer detection (benign/malignant/normal)

- **Dermoscopic**: Dermoscopic imaging involves the use of a dermatoscope, a handheld device equipped with magnification and light sources. By illuminating the skin and eliminating surface reflections, dermatoscopy enhances the visualization of pigment and vascular patterns, structural elements, and other morphological features within skin lesions. 

## Next
In the next section, we will introduce how Foundation Model works with the integration of classification.

## References 
1. Zhu, Zhonghang, et al. "Cross-View Deformable Transformer for Non-displaced Hip Fracture Classification from Frontal-Lateral X-Ray Pair." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
2. Sun, AnLan, et al. "Boosting Breast Ultrasound Video Classification by the Guidance of Keyframe Feature Centers." arXiv preprint arXiv:2306.06877 (2023).
3. Shareef, Bryar, et al. "Breast Ultrasound Tumor Classification Using a Hybrid Multitask CNN-Transformer Network." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
4. Nie, Weizhi, et al. "Chest X-ray Image Classification: A Causal Perspective." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
5. Yang, Yijun, et al. "Diffmic: Dual-guidance diffusion network for medical image classification." International Conference on Medical Image Computing and Computer-Assisted Intervention. Cham: Springer Nature Switzerland, 2023.
6. Litjens, Geert, et al. "A survey on deep learning in medical image analysis." Medical image analysis 42 (2017): 60-88.
