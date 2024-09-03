---
last_modified_on: "2024-09-01"
title: 3D Downstream Tasks for Foundation Model (Part2).
description: 3D Visual Question Answering.
series_position: 30
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


In the previous part, we introduced the 3D VQA task and the model architecture designed to address it. In this part, we will explore the implementation of recent state-of-the-art methods, datasets, and metrics used to measure the effectiveness of these methods"


## Method

Some works may not design an end-to-end framework for 3D VQA. Instead, they try to align or match the 3D point cloud and text description by pre-training on 3D-text data. The pre-trained backbone can be adapted to various 3D tasks, including 3D VQA.


**3D VisTA.**   **3D Vis**ion and **T**ext **A**lignment (3D VisTA - ICCV 2023) is a trained Transformer that aligns 3D data and textual description. It takes a 3D scene point cloud and a sentence as input is pre-trained using self-supervised learning and can be easily fine-tuned to various downstream tasks. 

- **Encoder.** They use the first four layers of pre-trained BERT to encode the text. For the scene, they sample a subset of points and apply PointNet++ to obtain its point features and semantic class.

- **Multi-modal fusion.** They concatenate the text and the 3D object tokens and send them to a $L$-layer Transformer. The output of the multi-modal transformer contains CLS token $w_{CLS}$, text token $w$, and 3D object token $o$.

- **Self-supervised training strategy.** They perform Masked Language Modeling (MLM) Masked Object Modeling (MOM) and Scene-Text Matching (STM) to pre-trained 3D VisTA. In MLM, they randomly choose a proportion of text tokens and the model is trained to predict the masked text tokens given the unmasked text tokens and object tokens. In MOM, the strategy is similar to MLM by masking out some object tokens. In STM, they aim to enhance the global fusion of scene and text, which seems very beneficial for downstream question-answering tasks. They extract the output that corresponds to CLS as the global representation of the input scene-text pair and predict if the scene and the text are matched. 
![Fig-2](https://vision.aioz.io/f/ad44291dc0f84fbc9428/?dl=1)
*<center>**Figure 2**: 3D VisTA architecture. Similar to ScanQA, encoders are used to extract features from input, but they mask the features before feeding them to the multi-modal fusion module to predict the masked tokens by training using self-supervised learning.</center>*

**LL3DA.** LL3DA (**L**arge **L**anguage **3D A**ssistant - CPVR 2024) is a framework that could respond to both textual and visual interactions from humans, and understand, reason, and plan in complex 3D environments. The required input contains a 3D scene, a textual instruction, and additional potential visual interactions $I_v$. In the proposal, they consider $I_v$ as one of two types: user click $p_{click} \in \mathbb{R}^3$ and bounding box which is presented by the ROI feature $f_{box}$ extracted by a pre-trained 3d object detector.

- **Visual Prompt Encoder.** The two types of visual prompts are projected with a Feed Forward Network. 

$$
f_{click} = FFN(p_{click}), \ \ f_{box} = FFN(f_{box})
$$

- **Multi-model Transformer.** a multi-modal transformer that aggregates information from textual instructions, visual prompts, and 3D scenes into a fixed number of 32 querying tokens, denoted as $Q \in \mathbb{R}^{32 \times 768}$ via the attention mechanism. The querying tokens are projected and used as the prefix of the textual instructions with a linear projector. 

![Fig-3](https://vision.aioz.io/f/995bbe2d57d64004a827/?dl=1)
*<center>**Figure 3**: LL3DA architecture. It considers the visual interactions as additional input. LL3DA uses Interactor3D inspired by Q-former to aggregate the visual prompts, textual instructions, and 3D scene embeddings into a fixed-length querying tokens.</center>*

**3D VLP.** 3D VLP (**3D V**ision-**L**anguage **P**re-training with object
contrastive learning - AAAI 2024) is a vision-language pre-training framework. It takes a scene point cloud and a sentence (not a question) as input and takes visual grounding as a proxy task for training. After training, they can flexibly transfer to other 3D downstream tasks, including 3D VQA. During training, they design object contrastive learning strategies to better enhance scene understanding.

- **Object-level IoU-guided Detection Loss.**
They propose Object-level IoU-guided Detection Loss to enhance the object detector, therefore obtaining a high-quality proposal. Given the ground-truth box $b_{gt}$ and predicted box $b_{p}$, they calculate the IoU and regressing loss

$$
\mathcal{L}_{DIoU}(b_{gt}, b_{p}) = 1- IoU + \dfrac{\rho^2(b_p, b_{gt})}{c^2}
$$
where $c$ is the diagonal length of the smallest enclosing box
covering the two boxes and $\rho^2$ is the Euclidean distance


- **Object-level Cross-contrastive Alignment.**
The Object-level Cross-Contrastive alignment (OCC) task is introduced to align the distribution of cross-modal data (between bounding box proposals and text description) when contrastive learning can handle embedding alignment across different distribution (from point cloud and text) better than cross-model attention. First, for the given threshold $\delta$, they define positive and negative proposal samples.

$$
P_{pos} = \left\{ p | IoU(b_p, b_{gt}) \geq \delta \right\}
$$
$$
P_{neg} = \left\{ p | IoU(b_p, b_{gt}) < \delta \right\}
$$
Then they design contrastive loss to align the features of positive samples with the language embedding and push the features of negative samples away:

$$
\mathcal{L}_{OCC} = -\dfrac{1}{2}\mathbb{E}_{(b_{gt},T)\sim D}\left [  \log \dfrac{\sum_{p \in P_{pos}}\exp(s(H_p,T))}{\sum_{\hat{p} \in P_{pos}\cup P_{neg}}\exp(s(H_{\hat{p}}, T))}  + \log \dfrac{\sum_{p \in P_{pos}}\exp(s(T, H_p))}{\sum_{\hat{p} \in P_{pos}\cup P_{neg}}\exp(s(T, H_{\hat{p}}))}  \right ] 
$$

where $H_p$ represents the embedding of proposal $p$, $T$ represents the embedding of text, and $s()$ represents the similarity score function, such as cosine or dot product. 
- **Object-level Self-contrastive Learning.**
They design an object-level self-contrastive loss to effectively differentiate between objects and improve the model’s semantic understanding of the scene.
$$
\mathcal{L}_{OSC} = -\dfrac{1}{2}\mathbb{E}_{b_{gt}\sim D}\left [ \log \dfrac{\sum_{p, \hat{p} \in P_{pos}}\exp(s(H_p,H_{\hat{p}}))}{\sum_{p, \hat{p} \in P_{pos}\cup P_{neg}}\exp(s(H_p, H_{\hat{p}}))} \right ] 
$$
![Fig-4](https://vision.aioz.io/f/5fbc6138f84c481ebb79/?dl=1)
*<center>**Figure 4**: 3D VLP pipeline. 3DVLP takes visual grounding as the proxy task and utilizes multiple contrastive learning losses to enable the model to better differentiate the objects.</center>*


![Fig-5](https://vision.aioz.io/f/e4e0ca8b47364fe2baf1/?dl=1)
*<center>**Figure 5**: Illustrations of losses used in 3DVLP. Proposals are filtered by comparing IoU to threshold $\delta$.</center>*

## Dataset 

The availability of large-scale datasets significantly helps improve the performance of Visual Question Answering tasks, as well as other vision and language tasks. However, collecting and annotating 3D datasets remain challenges. results in small RGB-D datasets. The ScanNet dataset introduced by *Dai et al.* is a richly annotated 3D indoor scan dataset of real-world environments captured employing a self-created RGB-D capture system.
 
The ScanNet dataset consists of 2.5M views obtained from 1513 scans, making it large in scale. However, ScanNet is designed for 3D object classification and 3D object retrieval with instance-level category annotation in a 3D point cloud, and it may not be suitable for 3D VQA and other related language-integrated 3D tasks. To overcome this limitation, researchers have annotated and developed some datasets tailored for 3DV VQA, building on top of ScanNet.

**ScanQA.**
ScanQA is a large-scale dataset for 3D object-grounded question answering on point cloud. It consists of 41,363 questions and 58,191 answers, including 32,337 unique questions and 16,999 unique answers. Various types of questions in the dataset are collected through auto-generation and editing by humans. Alongside question-answer pairs, the dataset also includes 3D object localization annotations for 800 indoor 3D scenes of theScanNet dataset. There are two test sets with and without object annotations in the ScanQA dataset.


**Scan2Inst.** Scan2Inst is a 3D instruction tuning dataset constructed using ScanNet. It contains 83261 data pairs comprising instructions and responses generated by GPT-API. The instructions are classified into 3 categories: 3D description, 3D classification, and 3D VQA, in which 9971 instructions are from the 3D VQA task. The VQA samples contain conversations about the scene (scene level) and the relationship among local objects in the scene (object level).


**FE-3DGQA.** FE-3DGQA is a dataset with diverse and relatively free-form question-answer pairs, as well as dense and completely grounded bounding box annotation. Each object in the scene is annotated with at least 2 questions to enhance the diversity of 3D scene details. In total, there are 20, 215 question-answer pairs collected for 703 ScanNet scenes with 42, 456 annotated grounded objects. Unlike ScanQA and ScanRefer, the questions in FE-3DGQA do not follow any given templates, and each question covers these aspects: 
- Attributes and local relations of a specific object. 
- Global context (scene details).
- Complex relationship among local objects.
- Direction or location of objects (for navigation).

Moreover, the objects are fully annotated and related to both questions and answers in three types

- The answer-grounded objects appeared in the question (e.g. the *table* in the question "What shape is the **table** in front of the window?" )
- The answer-grounded objects not appear in the question (e.g. the *table* in the question "Which object is next to the bed?" and the *table* in the answer "the **table**").
- The contextual objects related to the answer-grounded objects appeared in the question (e.g. the *window* in the question "What shape is the table in front of the **window**?").


## Metric

To assess the effectiveness of 3D VQA methods, detection, classification, and captioning evaluations are conducted.


**Detection metric.**
For detection metrics, Mean Average Precision (mAP) is used as an object detection metric. mAP is thresholded by IoU, and provides a comprehensive assessment of the detection performance in localizing objects. In most of the research, a threshold of 0.25 or 0.5 is used.

**Classification metric.**
Some works use the EM@1 and EM@10 metrics for evaluation in 3d VQA, where EM@$k$ represents the percentage of prediction in which the ground truth appears in the top $k$ predicted answers.



**Captioning metric.**
Inspired by image-dense captioning evaluation, translation machine metrics such as BLEU, ROUGE, and METEOR are used for evaluation in 3D VQA, in case there are many possible answers to a question. In addition, CIDEr is used to improve the human assessment. 
