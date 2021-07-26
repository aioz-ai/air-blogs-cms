---
last_modified_on: "2021-07-26"
id: medicalvqa-chall
title: Medical Visual Question Answering Challenges
description: Discuss about the challenges in Medical VQA
author_github: https://github.com/xuanbinh-nguyen96
tags: ["type: post", "AI", "Medical VQA", "Visual Question Answering", "Challenges"]
---
## Visual Question Answering in Medical Domain

**Visual Question Answering (VQA)** aims to provide a correct answer for a given question consistent with the visual content of a given image. The overarching goal of this issue is to create systems that can comprehend the contents of an image in the same way that humans do and communicate effectively about that image in natural language. It is indeed a challenging task as it necessitates the interaction and complementation of both image feature extractor and natural language processor.

In medical domain, VQA could benefit both doctors and patients. VQA systems capable of understanding medical images and answering questions related to their content could support clinical education, clinical decision, and patient education. For example, doctors could use answers provided by VQA system as support materials in decision making. It can also help doctors to get a second opinion in diagnosis and reduce the high cost of training medical professionals. While patients could ask VQA questions related to their medical images for better understanding their health.

Compared with VQA in the general domain, Med-VQA is a much more challenging problem. On one hand, clinical questions are more difficult but need to be answered with higher accuracy since they relate to health and safety

![Fig-1](https://vision.aioz.io/thumbnail/9b116d2c03f44db89b79/1024/vqa-mevf-Fig_1.png)
*<center>**Figure 1**: An example of Medical VQA ([Source](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6244189/pdf/sdata2018251.pdf)).</center>*

There are challenges that need to be addressed when using VQA in Medical domain:

* Lack of large scale labeled datasets.
* Using Transfer learning in medical domain.
In this blog, we will analyze this challenges.

## Challenges in Medical VQA

### 1. Lack of large scale labeled datasets.

One major problem with medical VQA is the lack of large scale labeled training data which usually requires huge efforts to build.
Difficulties in building a medical VQA dataset
- (i) designing goal-oriented VQA systems and datasets
- (ii) categorizing the clinical questions
- (iii) selecting (clinically) relevant images
- (iv) capturing the context and the medical knowledge.

The first attempt for building the dataset for medical VQA is by ImageCLEF-Med. In this, images were automatically captured from PubMed Central articles. The questions and answers were automatically generated from corresponding captions of images. By that construction, the data has high noisy level, i.e., the dataset includes many images that are not useful for direct patient care and it also contains questions that do not make any sense.

Well-annotated datasets for training Medical VQA systems are extremely lacking, but it is very laborious and expensive to obtain high-quality annotations by medical experts. The first manually constructed VQA-RAD dataset for medical VQA task is released. Unfortunately, it contains only 315 images, which prevents to directly apply the powerful deep learning models for the VQA problem.

### 2. Using Transfer learning

Transfer learning is an important step to extract meaningful features and overcome the data limitation in the medical Visual Question Answering (VQA) task. Transfer learning, in which the pretrained deep learning models that are trained on the large scale labeled dataset such as ImageNet, is a popular way to initialize the feature extraction process. However, due to the difference in visual concepts between ImageNet images and medical images, finetuning process is not sufficient.
Recently, Model Agnostic Meta-Learning (MAML) has been introduced to overcome the aforementioned problem by learning meta-weights that quickly adapt to visual concepts. However, MAML is heavily impacted by the meta-annotation phase for all images in the medical dataset. Different from normal images, transfer learning in medical images is more challenging due to:
* (i) noisy labels may occur when labeling images in an unsupervised manner;
* (ii) high-level semantic labels cause uncertainty during learning;
* (iii) difficulty in scaling up the process to all unlabeled images in medical datasets.
