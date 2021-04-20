---
last_modified_on: "2021-04-20"
id:  polyb-detection
title: "Data Augmentation for Colon Polyp Detection: A systematic Study"
description: "A systematic study on the performance of different data augmentation methods for colon polyp detection."
author_github: https://github.com/quangduytran
tags: ["type: post", "AI", "Medical", "Image Processing"]
---

*Colorectal cancer (CRC), also known as bowel cancer or colon cancer, is a cancer development from the colon or rectum called a polyp. Detecting polyps is a common approach in screening colonoscopies to prevent CRC at an early stage. Early colon polyp detection from medical images is still an unsolved problem due to the considerable variation of polyps in shape, texture, size, color, illumination, and the lack of publicly annotated datasets. At AIOZ, we adopt a recently proposed auto-augmentation method for polyp detection. We also conduct a systematic study on the performance of different data augmentation methods for colon polyp detection. The experimental results show that the auto-augmentation achieves the best performance comparing to other augmentation strategies.*

## Introduction
Colorectal cancer (CRC) is the third-largest cause of worldwide cancer deaths in men and the second cause in women, with the number of patients, died each year up to 700,000 [1]. Detection and removal of colon polyps at an early stage will reduce the mortality from CRC. There are several methods for colon screening, such as CT colonography or wireless capsule endoscopy, but the gold standard is colonoscopy [2].

The colonoscopy is performed by an experienced doctor who uses a colonoscope to screen and scan for abnormalities such as intestinal signs, symptoms, colon cancer, and polyps. Abnormal polyps can be removed, and small amounts of tissue can be detached for analysis during the colonoscopy. However, the most crucial drawback of colonoscopy is polyp miss rate, especially with polyp more diminutive than10mm. Several factors cause the miss rate. They are both subjective factors such as bowel preparation, the specific choice of an endoscope, video processor, clinician skill, and objective factors such as polyp appearance and camera movement condition. For these reasons, automatic polyp detection is a potential approach to assist clinicians in improving the sensitivity of the diagnosis.

Previous research shows that automatic polyp detection using deep learning-based methods outperforms hand-craft-based methods demonstrated by both top two results in the MICCAI 2015 challenge [3]. For deep learning-based approaches and model architectures, data augmentation is also a critical factor in making significant improvements due to the lack of annotated data. The recent work [4]shows that learning an optimal policy from data for auto augmentation instead of hand-crafted defining data augmentation strategies can generalize objects better. Thus, studying auto augmentation for polyp detection problems is necessary. In this research, we adapt Faster R-CNN [5] together withAutoAugment [6] to detect polyp from colonoscopy video frames. Besides, we also evaluate traditional data augmentation [7] to see the effectiveness of different augmentation strategies.

## Open Source
Github: https://github.com/aioz-ai

## Reference
[1] Hamidreza Sadeghi Gandomani, Mohammad Aghajani, et al., “Colorectal cancer in the world: incidence, mortality and risk factors,”Biomedical Research and Therapy, 2017.

[2] Florence  B ́enard,  Alan  N  Barkun,  et  al., “Systematic  review  of  colorectal  cancer  screening  guidelines for average-risk adults: Summarizing the current global recommendations,”World journal of gastroenterology, 2018.

[3] Jorge  Bernal,   Nima  Tajkbaksh,   et  al., “Comparative  validation  of  polyp  detection  methods  in  video colonoscopy:  results from the miccai 2015 endoscopic vision challenge,”IEEE Transactions on Medical Imaging, pp. 1231–1249, 2017.

[4] Barret Zoph, Ekin D Cubuk, et al.,  “Learning data augmentation strategies for object detection,”arXiv, 2019.

[5] Shaoqing  Ren,  Kaiming  He,  Ross  Girshick,  and  Jian Sun,   “Faster R-CNN: Towards real-time object detection with region proposal networks,” inNIPS, 2015.

[6] Ekin  D  Cubuk,  Barret  Zoph,  et  al.,“Autoaugment: Learning augmentation strategies from data,”  in CVPR, 2019.

[7] Younghak  Shin,  Hemin  Ali  Qadir,  et  al.,   “Automatic colon polyp detection using region based deep cnn and post learning approaches,”IEEE Access, 2018

[8] Yangqing Jia, Evan Shelhamer, et al.,  “Caffe: Convolutional architecture for fast feature embedding,”  in ACMMM, 2014.
