---
last_modified_on: "2021-04-20"
id:  polyp-detection
title: "Data Augmentation for Colon Polyp Detection: A systematic Study"
description: "A systematic study on the performance of different data augmentation methods for colon polyp detection."
author_github: https://github.com/quangduytran
tags: ["type: post", "AI", "Medical", "Image Processing", "Polyp detection", "Deep Learning"]
---

import Steps from '@site/src/components/Steps';

*Colorectal cancer (CRC):cancer:, also known as bowel cancer or colon cancer, is a cancer development from the colon or rectum called a polyp. Detecting polyps is a common approach in screening colonoscopies to prevent CRC at an early stage. Early colon polyp detection from medical images is still an unsolved problem due to the considerable variation of polyps in shape, texture, size, color, illumination, and the lack of publicly annotated datasets. At AIOZ, we adopt a recently proposed auto-augmentation method for polyp detection. We also conduct a systematic study on the performance of different data augmentation methods for colon polyp detection. The experimental results show that the auto-augmentation achieves the best performance comparing to other augmentation strategies.*

<Steps headingDepth={3}>
</Steps>

## Introduction
Colorectal cancer (CRC) is the third-largest cause of worldwide cancer deaths in men and the second cause in women, with the number of patients, died each year up to 700,000 [1]. Detection and removal of colon polyps at an early stage will reduce the mortality from CRC. There are several methods for colon screening, such as CT colonography or wireless capsule endoscopy, but the gold standard is colonoscopy [2].

The colonoscopy is performed by an experienced doctor who uses a colonoscope to screen and scan for abnormalities such as intestinal signs, symptoms, colon cancer, and polyps. Abnormal polyps can be removed, and small amounts of tissue can be detached for analysis during the colonoscopy. However, the most crucial drawback of colonoscopy is polyp miss rate, especially with polyp more diminutive than10mm. Several factors cause the miss rate. They are both subjective factors such as bowel preparation, the specific choice of an endoscope, video processor, clinician skill, and objective factors such as polyp appearance and camera movement condition. For these reasons, automatic polyp detection is a potential approach to assist clinicians in improving the sensitivity of the diagnosis.

Previous research shows that automatic polyp detection using deep learning-based methods outperforms hand-craft-based methods demonstrated by both top two results in the MICCAI 2015 challenge [3]. For deep learning-based approaches and model architectures, data augmentation is also a critical factor in making significant improvements due to the lack of annotated data. The recent work [4]shows that learning an optimal policy from data for auto augmentation instead of hand-crafted defining data augmentation strategies can generalize objects better. Thus, studying auto augmentation for polyp detection problems is necessary. In this research, we adapt Faster R-CNN [5] together withAutoAugment [6] to detect polyp from colonoscopy video frames. Besides, we also evaluate traditional data augmentation [7] to see the effectiveness of different augmentation strategies.

## Methodology
### 1. Polyp Detector
Thanks to the power of deep learning, recent works [5, 12,11] show that deep-based detection methods give impressive detection performance. In this work, we use the Faster RCNN object detector [5] with Resnet101 [13] backbone pre-trained on COCO dataset.  Our experiments show that this architecture gives a competitive performance on the Polyp detection problem. The experimental setting for the detector is set as follows. The network is trained using stochastic gradient descent  (SGD)with 0.9 momentum; learning rate with initial value is set to3e-4 and will be decreased to 3e-5 from the iteration 900k.The number of anchor boxes per location used in our model is 12 (4 scales, i.e.,64×64, 128×128, 256×256, 512×512 and 3 aspect ratios, i.e.,1 : 2,1 : 1,2 : 1).

### 2. Data Augmentation
![autoaugment_figure_small](https://vision.aioz.io/thumbnail/5a2c6412b7144569a9d2/1024/autoaugment_figure_small.jpg)
*<center>**Fig. 1.** Example of applying learned augmentation policies tocolonoscopy image.</center>*

Data augmentation can be split into two types: self-defined data augmentation (a.k.a traditional augmentation) and auto augmentation [6]. In this study, we adopt an automated data augmentation approach for object detection, i.e., Auto-augment [6], which finds optimal data augmentation policies during training. In Auto-augment, an augmentation policy consists of several sub-policies; each sub-policy consists of two operations. Each operation is an image transformation containing two parameters: probability and the magnitude of the shift. There are three types of transformations used in Auto-augment for object detection [4], which are
- **Color operations:** distort color channels without impacting the locations of the bounding boxes
- **Geometric operations:** geometrically distort the image, which correspondingly alters the location and size of the bounding box annotations
- **Bounding box operations:** only distort the pixel content contained within the bounding box annotations.
One of the essential conclusions in [4] is that the learned policy found on COCO can be directly applied to other detection datasets and models to improve predictive accuracy. Hence, in this study, we apply the learned policies from [4]to augment data when training the detector in Sec. 3.1. The learned policed we use for training our detector is summarized in Table 1. In Table 1, each operation is a triple which describes the transformation, the probability, and the magnitude of the transformations. Due to the space limitation, we refer the reader to [4] for detail on the descriptions of trans-formation. Fig. 1 showed augmented examples when applying the learned augmentation policy on a polyp image from the training dataset.

![tab-1](https://vision.aioz.io/thumbnail/40c5d897f39d4093b4f4/1024/polyp-tab1.png)
*<center>**Table 1.** Sub-policies and operations used in our experiment.</center>*

In addition to auto-augmentation, we also investigate the effect of traditional augmentation and the combination of traditional and automatic augmentation. We randomly apply several transformations to the image for traditional data augmentation, such as rotation, mirroring, sheering, translation, and zoom. We propose different strategies to combine those data augmentation types, i.e., (1) firstly, the detector is trained with Auto-augment; after that, it is trained with the traditional data augmentation; (2) training with the traditional augmentation, then with auto augmentation; (3) training with AutoAugmenton the original data and the data generated by traditional augmentation. All augmentation strategies are evaluated with the same model architecture and training configuration. This allows us to explore which data augmentation method is suitable for the polyp detection problem.

## Experiments
We  use  CVC-ClinicDB  [14]  for  training  and  ETIS-Larib[15] for testing.   This allows us to make a fair comparison with  MICCAI2015  challenge  results  which  are  reported  on the same dataset.  The CVC-CLINIC database contains 612polyp  image  frames  of  31  unique  polyps  from  31  different colonoscopy videos.  The ETIS-LARIB dataset contains 196high resolution image frames of 44 different polyps.

![fp_figure](https://vision.aioz.io/thumbnail/4dcf8c111cc9442cb080/1024/fp_figure.jpg)
*<center>**Fig. 2.** Examples of false positive detection on testing dataset. Green boxes and blues boxes are ground truths and predictions, respectively.</center>*

![fn_figure](https://vision.aioz.io/thumbnail/105ef91b507140c78d0f/1024/fn_figure.jpg)
*<center>**Fig. 3.** Examples of false-negative detection on the testing dataset. Green boxes and blues boxes are ground truths and predictions, respectively.</center>*

Fig. 2 and Fig. 3 visualize several failed results from our model in the testing dataset in which the blue boxes are the predicted locations, and green boxes are ground truths. These false-positive samples (Fig. 2) caused by a shortcoming in bowel preparation (i.e., leftovers of food and fluid in the colon), while false negative (Fig. 3) samples are caused by the variations of polyp type and appearance (i.e., small polyp, flat polyp, similarities of polyp and colon vein)

![tab-3](https://vision.aioz.io/thumbnail/350d82d6210d4934a384/1024/polyp-tab3.png)
*<center>**Table 2.** Comparison among traditional data augmentation (TDA), auto augmentation (AA) and their combinations.</center>*
Table 2 shows the comparative results between different augmentation strategies. The results show that the third combination method (AA-TDA-3) achieves higher performance in Precision than AutoAugment, i.e., 75.90% and 74.51%, respectively. However, overall, Auto-augment (AA) achieves the best results because of its performance in covering polyp miss rate (i.e., 152) with an acceptable false-positive rate (i.e., 52). The competitive performance of auto augmentation (AA) confirms the transferable learned data augmentation policies on the COCO dataset [4].

![tab-4](https://vision.aioz.io/thumbnail/7ab3afe2f19345febcfe/1024/polyp-tab4.png)
*<center>**Table 3.** Comparative results between our model and the state of the art.</center>*
Table 3 presents the comparative results between the auto augmentation in our model and other state-of-the-art results. Among compared methods, CUMED, OUR, and UNS- UCLAN are end-to-end deep learning-based approaches. The results show that compared to methods from MICCAI challenge [3], auto augmentation achieves better performance on all metrics. Comparing to the recent method [10], auto augmentation also achieves better performance on all metrics but FP. These results confirm the effectiveness of auto augmentation for polyp detection problems.

## Conclusion
This study adopts a deep learning-based object detection method with auto data augmentation for polyp detection problems. Different augmentation strategies are evaluated. The experimental results show that the learned auto augmentation policies learned from the general object detection dataset are well transferred to the polyp detection problem. Although auto augmentation achieves competitive results, it still has a high FP compared to the state of the art. This weakness can be improved by several post-processing, such as false-positive learning.

## Open Source
:tomato: Github: https://github.com/aioz-ai/polyp-detection

:strawberry: Blog post: https://ai.aioz.io/blog/polyp-detection

## Acknowledgements
*This research was conducted by Phong Nguyen, Quang Tran, Erman Tjiputra, and Toan Do. We’d like to give special thanks to the other AIOZ AI team members for their supports and feedbacks.*

:tada: All the above contributions were incredibly enabling for this research. :tada:


## Reference
[1] Hamidreza Sadeghi Gandomani, Mohammad Aghajani, et al., “Colorectal cancer in the world: incidence, mortality and risk factors,”Biomedical Research and Therapy, 2017.

[2] Florence  B ́enard,  Alan  N  Barkun,  et  al., “Systematic  review  of  colorectal  cancer  screening  guidelines for average-risk adults: Summarizing the current global recommendations,”World journal of gastroenterology, 2018.

[3] Jorge  Bernal,   Nima  Tajkbaksh,   et  al., “Comparative  validation  of  polyp  detection  methods  in  video colonoscopy:  results from the miccai 2015 endoscopic vision challenge,”IEEE Transactions on Medical Imaging, pp. 1231–1249, 2017.

[4] Barret Zoph, Ekin D Cubuk, et al.,  “Learning data augmentation strategies for object detection,”arXiv, 2019.

[5] Shaoqing  Ren,  Kaiming  He,  Ross  Girshick,  and  Jian Sun,   “Faster R-CNN: Towards real-time object detection with region proposal networks,” inNIPS, 2015.

[6] Ekin  D  Cubuk,  Barret  Zoph,  et  al.,“Autoaugment: Learning augmentation strategies from data,”  in CVPR, 2019.

[7] Younghak  Shin,  Hemin  Ali  Qadir,  et  al.,   “Automatic colon polyp detection using region based deep cnn and post learning approaches,”IEEE Access, 2018

[8] Yangqing Jia, Evan Shelhamer, et al.,  “Caffe: Convolutional architecture for fast feature embedding,”  in ACMMM, 2014.
