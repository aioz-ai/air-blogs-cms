---
last_modified_on: "2021-06-14"
id:  gps-reid
title: Graph-based Person Signature for Person Re-Identifications
description: "We propose a new method for Person Re-Identifications."
author_github: https://github.com/xuanbinh-nguyen96
tags: ["type: post", "AI", "Computer Vision", "Person ReID", "Re-identification", "Graph-based"]
---

## Motivation

Person re-identification (ReID) aims to retrieve a particular person image in a collection of images captured by multiple cameras from various viewpoints across time.

The challenges of the person ReID task come from significant variations of human attributes such as poses, gaits, clothes, as well as challenging environmental settings like illumination, complex background, and occlusions. With the rise of deep learning, most of the recent studies utilize Convolutional Neural Network (CNN) to tackle the person ReID problem.

Recently, attribute-based methods have shown great success in providing semantic features for the deep network. Unlike the person identity label, which offers only coarse information to identify one identity among all other person identities, the attributes are the detailed descriptions that are highly intuitive and mostly unchanged between images captured from different cameras. Therefore, they can be used to explicitly guide the model to learn a robust person representation by defining human characteristics.

In this work, we propose to utilize the person attribute information with its associated body part to encode the visual person signature in one unified framework.

- We hypothesize that the detailed person descriptions (attributes labels) can be integrated with visual features (body parts and global features) to create a unique signature for a particular person.
- Since both body parts and attributes provide local representations, by linking them together, the network can have a better understanding of the relationship between visual features and attribute descriptions.
- Although previous works have investigated how person identity, body parts, and attributes benefit the task of person ReID, our key difference is that we utilize Graph Convolutional Networks (GCN) to effectively construct and model the correlation between attributes and body parts with global features. In particular, we treat body part regions and attributes as nodes in a graph and utilize a GCN to learn the topological structure of a person's signatures. The GCN propagates messages on a graph structure. After message traversal on the graph, the node's final representations are obtained from its data and from other node's information. Figure 1 shows the effectiveness of our approach.

![Fig-1](https://vision.aioz.io/thumbnail/eeded91d03ba4925a63d/1024/GPS-Fig_1.PNG)
*<center>**Figure 1**: The effectiveness of our GPS in improving retrieval results on Market-1501 dataset. The details are presented in the text [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

## Methodology

![Fig-2](https://vision.aioz.io/thumbnail/8491670d67d347b29513/1024/GPS-Fig_2.PNG)
*<center>**Figure 2**: Illustration of our proposed framework including two branches: (1) global branch which extracts person global features; (2) GPS branch which performs reasoning the person attributes and body parts using GCN. The details are presented in the text [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

The proposed framework is presented in Figure 2.

- We denote $I$ is a probe person image. This probe image $I$ is first passed through a backbone CNN to get the feature map $\mathbf{F}$.
- By utilizing a human parsing pretrained model, we extract the body part masks to obtain the visual features of each part.
- The person attributes are then represented by a lookup word embedding.
- Given body part features and attribute features, we construct the Graph-based Person Signature which includes attribute nodes and body part nodes conditioned on the correlation matrix. We employ the GCN for reasoning on the person signature graph and encoding the graph into more representativeness features.

Our proposed method is a multi-branch multi-task framework for person ReID, where the main branch performs the verification task by optimizing two well-known loss functions: Triplet loss and Center loss. The auxiliary branch performs reasoning on the proposed person signature graph and solves the attribute recognition as well as the person identity classification tasks.

## Experimental Results

### Ablation Study

**Loss Contribution.**  In Table 2, we show the contribution of each loss to the final performance on the Market1501 dataset. The person ID classification loss, triplet loss, center loss, and attribute recognition loss are denoted as $\mathbf{\mathcal{L}_{id}}$, $\mathbf{\mathcal{L}_{triplet}}$, $\mathbf{\mathcal{L}_{center}}$, and $\mathbf{\mathcal{L}_{a}}$, respectively. The performance is improved when we incorporate all losses to the framework, which justifies the effectiveness of our proposed method. By using only $\mathbf{\mathcal{L}_{id}}$, we still achieve comparative results with other mask-guided and attribute-based methods. While the triplet loss $\mathbf{\mathcal{L}_{triplet}}$ demonstrates its capability on improving the performance, the center loss $\mathbf{\mathcal{L}_{center}}$ shows a slight impact on the performance. Notably, the attribute loss $\mathbf{\mathcal{L}_{a}}$ shows stability when being incorporated with other loss functions.

<img src="https://vision.aioz.io/thumbnail/f68e9cde75624ca7a7ef/1024/GPS-Tab_2.PNG" alt="Tab-2" width="400"/>

*<center>**Table 2**: The contribution of losses to the performance of person ReID task on Market1501 dataset. Note that the experiments are conducted with ResNet-50 as backbone CNN network [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

**Model Interpretability.** In this section, we conduct cross-dataset experiments to evaluate the effectiveness of GPS. The model is trained on the source dataset and test directly on the target dataset without finetuning. As shown in Table 3, our GPS archives a significant improvement over the Bag-of-Tricks baseline (BoT). This demonstrates the interpretability of our proposed method as well as confirms the effectiveness of learning the attributes for the person ReID task.

<img src="https://vision.aioz.io/thumbnail/0b83f055393c42aeb1da/1024/GPS-Tab_3.PNG" alt="Tab-3" width="400"/>

*<center>**Table 3**: The transferable ability of our GPS evaluated on cross-dataset [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

**Training Parameters.** We also provide the number of training parameters of our GPS and the baseline BoT in Table 4 to show the complexity of each method. Overall, our GPS slightly increases about 3M parameters in comparison with the baseline BoT while achieving much better performance.

<img src="https://vision.aioz.io/thumbnail/0776d4ba028242ebae65/1024/GPS-Tab_4.PNG" alt="Tab-4" width="400"/>

*<center>**Table 4**: The number of parameters of our GPS in comparision with the baseline BoT on Market1501 and DukeMTMC-ReID datasets using ResNet-50 as the backbone network. #nParam indicates the number of parameters and 1K=1000 [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

### Comparison to the state of the art

**GPS vs. Baseline.** The last two rows of Table 5 show the result of our GPS when being integrated into the baseline BoT. The results clearly show that our GPS significantly improves the performance of BoT in both Market-1501 and DukeMTMC-ReID dataset. This demonstrates the effectiveness of our GPS and confirms the usefulness of learning the attributes in the ReID task.

<img src="https://vision.aioz.io/thumbnail/dcf6842fe65b47f89208/1024/GPS-Tab_5.PNG" alt="Tab-5" width="600"/>

*<center>**Table 5**: Comparison with state-of-the-art methods on Market-1501 and DukeMTMC-ReID datasets. The cyan and yellow boxes are the best results corresponding to mask-guided/attribute-based and other approaches, respectively. Note that no post-processing is applied to our method [(Source)](https://arxiv.org/pdf/2104.06770.pdf).</center>*

**Evaluation on Market-1501.** We evaluate our GPS with other methods on Market-1501 dataset in Table 5. The results show that our method outperforms the state-of-the-art attribute-based methods AANet that use attribute and body part information in all evaluation metrics. Specifically, we outperforms AANet by 5.3% and 1.3% at mAP and R-1, respectively. Our GPS also outperforms the state-of-the-art mask-guided methods, and especially, we outperform P$^2$-Net by 2.2% at mAP. At the same time, we also get comparative results when comparing with other recent ReID approaches.

**Evaluation on DukeMTMC-ReID.** Table 5 also summaries the results of our GPS and other methods on DukeMTMC-ReID dataset. Our GPS significantly outperforms other attribute-based methods in all metrics. Specifically, our method outperforms the recent state-of-the-art attribute-based method AANet by 6.1% at mAP and 1.8% at R-1. In addition, we also outperforms ADPR by 9.0%, 3.9%, 2.8%, 2.0% at mAP, R-1, R-5, R-10, respectively. Moreover, our GPS outperforms the state-of-the-art mask-guided method P$^2$-Net by 4.9%, 1.7%, 2.1%, 1.7% at mAP, R-1, R-5, R-10, respectively. Besides, we also achieve comparative results with other ReID approaches.

**Attributed-based and Mask-guided vs. Other approaches.** From Table 5, we notice that although our GPS shows a definite improvement over mask-guided and attributed-based methods, it achieves competitive results with methods from other approaches and particularly being outperformed by st-ReID method. Note that the results of st-ReID also completely dominate all methods from all other approaches. The effectiveness of st-ReID comes from the fact that it also uses the spatial-temporal information (i.e., the spatial map of camera setting and temporal information from video timestamp) into the network. This extra information allows the network to encode the person identity from multiple viewpoints, which significantly reduces the effect of different poses, viewpoints, or ambiguity challenges. From experiments, we have observed that our GPS, as well as other attribute-based and mask-guided methods, suffers from the fact that the pretrained body part network cannot provide adequate segmentation masks, so the retrieval results are also affected.

### Quantity Visualization

![Fig-5](https://vision.aioz.io/thumbnail/f1f97214d7b048379c08/1024/GPS-Fig_3.PNG)
*<center>**Figure 3**: Top 5 retrieval results of some queries on Market-1501 dataset. Note that the green/red boxes denote true/false retrieval results, respectively.</center>*

We present some retrieval examples with five retrieved images for each query in Figure 3. As in the visualization, our GPS obtained better retrieval results than the baseline. In the first row of Figure 3, the baseline gets the false retrieval result at Rank-5 due to the similarity of gender, wearing a hat, etc., except the color of the clothes. By leveraging our GPS, the extracted features are more robust to attribute and body part information, then, lead to better retrieval results for ReID model. In the second row, the model with our GPS gives better results by extracting more information about the relationship between `backpack' attribute and this person identity, thereby eliminating false cases. We also show an example that our GPS does not yet produce entirely correct retrieval results in the third line of the Figure 3. In this case, the lower body of the probe image is partly covered by the bicycle. Thus, the extracted features (i.e., the color of the pants) are not fully captured, which results in the feature misalignment between the probe image and retrieval results.

## Conclusion

This paper proposes Graph-based Person Signature (GPS) that effectively captures the dependencies of person attributes and body parts information. We utilize the GCN on the GPS to propagate the information among nodes in the graph and integrate the graph features into a novel multi-branch multi-task network. The experimental results on benchmark datasets confirm the effectiveness of our GPS and demonstrate that our GPS performs better than recent state-of-the-art attribute-based and mask-guided ReID methods.

## Open Source
:cat: Github: https://github.com/aioz-ai/CVPRW21_GPS
