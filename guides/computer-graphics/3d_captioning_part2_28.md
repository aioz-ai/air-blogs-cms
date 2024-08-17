---
last_modified_on: "2024-16-08"
title: 3D Downstream Tasks for Foundation Model (Part 2).
description: Different 3D Dense Captioning Methods.
series_position: 27
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: 3D"]
---
import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

In the previous part, we have introduced 3D Captioning and task definition. In this part, we continue with implementations of recent methods and their key contributions.


## Methods

Based on the 3d dense captioning task, many methods first extract the object bounding boxes, then generate captions for detected bounding boxes. However, overall performance is heavily influenced by the accuracy of the detection results

**MORE**: MORE (**M**ulti-**O**rder **RE**lation Mining for Dense Captioning in 3D Scenes) incorporates multi-order relation mining based on graphs. Built upon Scan2Cap, MORE captures more complex inter-object relationships through progressive encoding so as to narrow the gap between point cloud and relational concepts.

![Fig-1](https://vision.aioz.io/f/4a067a683b174a118aea/?dl=1)
*<center>**Figure 1**: Overview architecture of MORE. MORE explore complex spatial relations by triplet attention graphs </center>*

**D3Net** D3Net (**d**etect-**d**escribe-**d**escriminate) introduced unified networks for both 3D dense captioning and 3D visual grounding tasks in the context of 3D scenes. A joint speaker-listener architecture is introduced, where the *speaker* corresponds to the captioning model and the *listener* corresponds to the grounding task. This architecture enables semi-supervised training and facilitates the generation of distinctive descriptions.

![Fig-2](https://vision.aioz.io/f/37e9e390afe54489b60a/?dl=1)
*<center>**Figure 2**: Overview architecture of D3Net</center>*


**Vote2Cap-DETR**: Unlike other methods that follow the pipeline *"detect-then-describe"*, Vote2Cap-DETR reverses the order of captioning and detection by directly feeding the decoder’s output into the localization head and caption head in parallel. The model directly decodes the features into object sets with their locations and corresponding captions by applying two parallel prediction heads

![Fig-3](https://vision.aioz.io/f/78ae05fffa6c43db819f/?dl=1)
*<center>**Figure 3**: Different between Vote2Cap-DETR to other cascade "detect-then-describe" methods.</center>*

![Fig-4](https://vision.aioz.io/f/8dcd87b8d282407a949b/?dl=1)
*<center>**Figure 4**: Overview of Vote2Cap-DETR architecture.</center>*

## Dataset 

The availability of large-scale datasets significantly helps improve the performance of image captioning and dense image captioning tasks, as well as other vision and language tasks. However, collecting and annotating 3D datasets remain challenges. results in small RGB-D datasets. The ScanNet dataset introduced by *Dai et al.* is a richly annotated 3D indoor scan dataset of real-world environments captured employing a self-created RGB-D capture system.
 
The ScanNet dataset consists of 2.5M views obtained from 1513 scans, making it large in scale. However, ScanNet is designed for 3D object classification and 3D object retrieval with instance-level category annotation in a 3D point cloud, and it may not be suitable for 3D dense captioning and other related language-integrated 3D tasks. To overcome this limitation, ScanRefer, Nr3D, and MultiRefer are developed as datasets specifically tailored for 3D dense captioning, building on top
of ScanNet. 

### ScanRefer 
ScanRefer is widely used for 3D dense captioning tasks. ScanRefer consists of 51,583 descriptions for objects in 800 ScanNet scenes. On average, there are 13.81 objects, 64.48 descriptions per scene, and 4.67 descriptions per object after filtering. The descriptions are complex
and diverse, covering over 250 types of common indoor objects, and exhibiting interesting linguistic phenomena. With each object of each scene, there are approximately 5 descriptions, providing a rich understanding of the 3D scene at the instance-level. 


![Fig-5](https://vision.aioz.io/f/d8a815cb03674c9f873f/?dl=1)
*<center>**Figure 5**: Overview of ScanRefer dataset.</center>*

### Nr3D

The 3D Natural Reference (Nr3D) dataset is one of the datasets from ReferIt3D, along with the synthetic language descriptions dataset Sr3D. Nr3D focuses on a challenging setup where the referred object belongs to a fine-grained object class and the underlying scene contains multiple object instances of that class (a scene that contains many chairs belonging to the same type). Nr3D consists of 41,503 natural, free-form utterances describing objects belonging to one of 76 fine-grained object classes within 5,878 communication contexts. 


## Metric

To assess the effectiveness of 3D dense captioning methods, both detection and captioning evaluations are conducted. The Intersection over Union (IoU) is used to compare the predicted local bounding boxes with ground-truth ones, and is formulated: 

$$
E@kIoU = \sum_{i=1}^{N}U_iE_i,\ \ U_i = \left\{\begin{matrix}
 1 , IoU \geq k\\
0, IoU <k
\end{matrix}\right. 
$$

where $N$ is the number of bounding boxes, $E_i$ is the metric used in $i$-th bounding box such as detection metric (mAP) or captioning metric (CIDEr, METEOR, BLEU or ROUGH). $k$ is the IoU threshold and is commonly set to 0.25 or 0.5. 
### Detection Metric
For detection metrics, Mean Average Precision (mAP) is used as an object detection metric. mAP is thresholded by IoU, and provides a comprehensive assessment of the detection performance in localizing objects.


### Captioning Metric 

Inspired by image-dense captioning evaluation, translation machine metrics such as BLEU, ROUGE, and METEOR are used for evaluation in 3D dense captioning. In addition, CIDEr is used to improve the human assessment:

- **BLEU**: Bilingual Evaluation Understudy (BLEU) is a metric for evaluating machine translation outputs based on precision. It involves matching different parts of a candidate text with reference texts and then calculating the percentage of successfully matched sequences. In most of the research, BLEU-4 is commonly used, which considers 4 consecutive words in the generated text as one matching unit.

- ROUGE: **R**ecall-**O**riented **U**nderstudy for **G**isting **E**valuation (ROUGE) is a generally employed metric for assessing the quality of text summarization generated by machine learning algorithms. It is based on recall and measures the similarity between the candidate and reference summaries with various n-gram-based and sequence-based statistics.

- METEOR:  **M**etric for **E**valuation of **T**ranslation with **E**xplicit **OR**dering (METEOR ) is an automatic metric for machine translation evaluation that is based on a generalized concept of unigram matching between the machine-produced translation and human-produced reference translations.



- CIDEr: **C**onsensus-based **I**mage **D**escription **E**valuation (CIDEr)  measures the similarity of a generated sentence against a set of ground truth sentences written by humans. shows high agreement with consensus as assessed by humans, which is lacking in BLEU, ROUGH, and METEOR. CIDEr incorporates TermFrequency Inverse Document Frequency (TF-IDF) weighting for each n-gram, resulting in more accurate human assessments. The n-grams that enable frequently in the reference sentence are assigned low weights since they usually don't contain useful information. 
