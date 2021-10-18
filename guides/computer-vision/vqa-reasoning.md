---
last_modified_on: "2021-07-30"
title: Visual Question Reasoning
description: Reasoning task in Visual Question Answering
author_github: https://github.com/Gazeal
series_position: 1
tags: ["type: survey", "level: advance"]
---

The Visual Question Answering (VQA) task aims to predict the correct answer of a given question such that the answer is consistent with the visual image content. Visual Question Reasoning task  is a VQA task that focus on learning the relations between visual regions and words in questions implicitly or explicitly (See Figure 1 for examples of visual question reasoning). In this part, we explore four different methods:

- Message passing.
- Pairwise relationship modeling.
- Sparser graph defined by inter/intra-class edges.
- External information and explicit scene graph.


![Fig-1](https://vision.aioz.io/f/7b6a40a741694bfe82fb/?dl=1)
*<center>**Figure 1**:  Example of visual question reasoning.</center>*

## Message passing.
In general, the technique can be summarized as below, including a description of the scene (a list of items with their visual attributes) and a parsed query are supplied as input (words with their syntactic relations). Each item has a node with a feature vector and edge features that describe their spatial connections in the scene-graph. The question-graph represents the question's parse tree, with a word embedding for each node and a vector embedding of syntactic dependency types for edges. Each node in both graphs is connected with a recurrent unit (GRU). The GRU updates a representation of each node that incorporates context from its neighbors inside the graph across several rounds. All objects and words have their features merged (concatenated) pairwise and weighted with a type of attention. The aspects of the question and the scenario are effectively matched in this way. The weighted total of characteristics is sent into a final classifier, which predicts scores based on a predetermined set of possible responses.
![Fig-2](https://vision.aioz.io/f/1a3c53d5c0394bcca7a6/?dl=1)
*<center>**Figure 2**:  Message Passing Technique [(Source)](https://arxiv.org/pdf/1609.05600.pdf).</center>*

## Pairwise relationship modeling.
For Visual Question Answering (VQA) tasks using real pictures, multimodal attentional networks are presently the most advanced models. Although attention helps you to focus on the visual material that is important to the inquiry, it is likely insufficient to represent the sophisticated reasoning characteristics needed for VQA or other high-level activities. MuRel is a multimodal relational network that can reason over actual pictures and is learnt end-to-end. The initial contribution is the development of the MuRel cell, an atomic reasoning primitive that models region relations using pairwise combinations and represents interactions between question and picture areas using a rich vectorial representation. Second, the cell is embedded in a complete MuRel network, which refines visual and question interactions over time and may be used to build visualization schemes that are more refined than simple attention maps.
![Fig-3](https://vision.aioz.io/f/242e28f9e908443b9ee2/?dl=1)
*<center>**Figure 3**:  MuRel: Pairwise relationshop modeling [(Source)](https://arxiv.org/pdf/1902.09487.pdf).</center>*

## Sparser graph defined by inter/intra-class edges.
Visual question answering is all about learning how to combine multi-modality features effectively. DFAF present a new technique for dynamically integrating multi-modal characteristics with intra- and inter-modality information flow that alternately passes dynamic information across and across visual and verbal modalities. It can reliably capture high-level interactions between language and vision domains, resulting in considerable improvements in visual question answering performance. DFAF further shows that the proposed dynamic intra-modality attention flow may dynamically regulate the target modality's intra-modality attention when conditioned on the other modality.
![Fig-4](https://vision.aioz.io/f/2799b7511d574779ba71/?dl=1)
*<center>**Figure 4**:  DFAF: Sparser graph defined by inter/intra-class edges [(Sourcce)](https://arxiv.org/pdf/1812.05252.pdf).</center>*

## External information and explicit scene graph.
With advances in picture understanding tasks like as object identification, attribute and connection prediction, and so on, scene graph creation has gotten a lot of attention. Existing datasets, on the other hand, are skewed in terms of object and relationship labels, and frequently contain noisy and missing annotations, making the creation of a trustworthy scene graph prediction model extremely difficult. To address these dataset difficulties, a unique scene graph creation technique with external knowledge and picture reconstruction loss is presented in this work. In particular, commonsense information is collected from the external knowledge base in particular to enhance object and phrase characteristics for scene graph creation generalizability. An additional image reconstruction path is introduced to regularize the scene graph generation network to overcome the bias of noisy object annotations.

![Fig-5](https://vision.aioz.io/f/6efb6529b10b4f788631/?dl=1)
*<center>**Figure 5**:   External information and explicit scene graph [(Sourcce)](https://arxiv.org/pdf/1812.00560.pdf).</center>*
