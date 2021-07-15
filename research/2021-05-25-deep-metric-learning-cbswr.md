---
last_modified_on: "2021-05-25"
id: deep-metric-learning-cbswr
title: Deep Metric Learning Meets Deep Clustering - An Novel Unsupervised Approach for Feature Embedding
description: A new Unsupervised Deep Metric Learning method.
author_github: https://github.com/xuanbinh-nguyen96
tags: ["type: post", "AI", "Computer Vision", "Metric-learning", "Clustering", "Retrieval", "Similarity-measure"]
---

## Motivation

The objective of deep distance metric learning (DML) is to train a deep learning model that maps training samples into feature embeddings that are close together for samples that belong to the same category and far apart for samples from different categories. Traditional DML approaches require supervised information, i.e., class labels, to supervise the training. Although the supervised DML achieves impressive results on different tasks, it requires large amount of annotated training samples to train the model. Unfortunately, such large datasets are not always available and they are costly to annotate for specific domains. That disadvantage also limits the transferability of supervised DML to new domain/applications which do not have labeled data. These reasons have motivated recent studies aiming at learning feature embeddings without annotated datasets --- unsupervised deep distance metric learning (UDML). Our study is in that same direction, i.e., learning embeddings from unlabeled data.

There are two main challenges for UDML:

- Firstly, how to define positive and negative samples for a given anchor data point, such that we can apply distance-based losses, e.g., pairwise loss or triplet loss, in the embedding space.
- Secondly, how to make the training efficient, given a large number of pairs or triplets of samples, in the order of $\mathcal{O}(N^2)$ or $\mathcal{O}(N^3)$, respectively, in which $N$ is the number of training samples.

In this paper, we propose a new method that utilizes deep clustering for deep metric learning to address the two challenges mentioned above. In particular,

- We propose to use a deep clustering loss to learn centroids, i.e., pseudo labels, that represent semantic classes.
- During learning, these centroids are also used to reconstruct the input samples. It hence ensures the representativeness of centroids — each centroid represents visually similar samples. Therefore, the centroids give information about positive (visually similar) and negative (visually dissimilar) samples.
- Based on pseudo labels, we propose a novel unsupervised metric loss which enforces the positive concentration and negative separation of samples in the embedding space.

## Methodology

![Fig-1](https://vision.aioz.io/thumbnail/798a958b1c6a4687ac50/1024/cbswr-Fig_1.png)
*<center>**Figure 1**: Illustration of the proposed framework which consists of an encoder (G), an embedding module (F), a decoder (D) and three losses, i.e., clustering loss $L_{rim}$, reconstruction loss $L_{rec}$ and metric loss $L_m$. The details are presented in the text [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

The proposed framework is presented in Figure 1.

- For every original image in a batch, we make an augmented version by using a random geometric transformation.
- The input images are fed into the backbone network which is also considered as the encoder (G) to get image representations.
- The image representations are passed through the embedding module which consists of fully connected and L2 normalization layers (F), which results in unit norm image embeddings.
- The clustering module takes image embeddings as inputs, performs the clustering with a clustering loss, and outputs the cluster assignments.
- Given the cluster assignments, centroid representations are computed from image representations, which are then passed through the decoder (D) with a reconstruction loss to reconstruct images that belong to the corresponding clusters.
- The centroid representations are also passed through the embedding module (F) to get centroid embeddings. The centroid embeddings and image embeddings are used as inputs for the metric loss.

### Discriminative Clustering

We formulate the clustering of embedding features as a classification problem. Given a set of embedding features $X=\{x_i\}_{i=1}^m \in \mathbb{R}^{128 \times m}$ in a batch and the number of clusters $K\le m$ (i.e., the number of clusters $K$ is limited by the batch size $m$). The cluster assignment for $x$ then is estimated by $c^* = argmax_c y$. Let $Y=\{y_i\}_{i=1}^m$ be the set of softmax outputs for $X$.

Inspired by Regularized Information Maximization (RIM), we use the following objective function (1) for the clustering.

$$
L_{rim} = \mathcal{R}(\theta) - \lambda \left[ H(Y) - H(Y|X) \right] (1)
$$

where $H(.)$ and $H(.|.)$ are entropy and conditional entropy, respectively; $\mathcal{R}(\theta)$ regularizes the classifier parameters (in this work we use $l_2$ regularization); $\lambda$ is a weighting factor to control the importance of two terms.

Minimizing (1) is equivalent to maximizing $H(Y)$ and minimizing $H(Y|X)$. Increasing the marginal entropy $H(Y)$ encourages cluster balancing, while decreasing the conditional entropy $H(Y|X)$ encourages cluster separation. 

### Reconstruction

In order to enhance the representativeness of centroids, we introduce a reconstruction loss (2) that penalizes high reconstruction errors from centroids to corresponding samples. Specifically, the decoder takes a centroid representation of a cluster and minimizes the difference between input images that belong to the cluster and the reconstructed image from the centroid representation.

$$
L_{rec} = \frac{1}{m} \sum_{j=1}^K\sum\limits_{I_i\in X_j}||I_i - D(r_j)||^2, (2)
$$

where $D(.)$ is the decoder which reconstructs samples in the batch using their corresponding centroid representations and $m$ is the number of images in the batch.

### Metric loss

Let $f_i \in \mathbb{R}^{128}$ and $\hat{f_i} \in \mathbb{R}^{128}$ be the image embeddings of $I_i$ and $\hat{I_i}$, respectively. The proposed metric loss (3) aims to minimize the distance between $f_i$ and $\hat{f_i}$ while pushing $f_i$ far away from negative clusters.

$$
L_{m} = -\sum_i log \left( l(I_i,\hat{I_i}) \right) - \sum_i\sum\limits_{j=1, j \neq q}^K log \left(l(I_i,c_j) \right) (3)
$$

### Final Loss

The network in Figure 1 is trained in an end-to-end manner with the following multi-task loss.

$$
L = \alpha L_{m} + \beta L_{rim} + \gamma L_{rec}, (4)
$$

where $L_m$ is the center-based softmax loss (3) for deep metric learning, $L_{rim}$ is the clustering loss (1), and $L_{rec}$ is the reconstruction loss (2).

## Experimental Results

### Ablation Study

We denote our model with:

- only clustering loss (1) as only $L_{rim}$.
- both clustering and the metric losses (2) and (3) as Center-based Softmax (CBS).
- Center-based Softmax with Reconstruction (CBSwR).

<img src="https://vision.aioz.io/thumbnail/7b3d14404ae2485897b0/1024/cbswr-Tab_1.png" alt="Tab-1" width="400"/>

*<center>**Table 1**: The impact of each loss component on the performance on CUB200-2011 dataset and the comparison to the baseline [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

<img src="https://vision.aioz.io/thumbnail/718a8a3160cb43f79c99/1024/cbswr-Tab_2.png" alt="Tab-2" width="400"/>

*<center>**Table 2**: The impact of each loss component on the performance on Car196 dataset and the comparison to the baseline [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

Tables 1 and 2 present the comparative results between methods. The results show that using only the clustering loss, the accuracy is significantly lower than the baseline SME. However, when using the centroids from the clustering for calculating the metric loss (i.e., CBS), it gives the performance boost over the baseline (i.e., SME). Furthermore, the reconstruction loss enhances the representativeness of centroids, as confirmed by the improvements of CBSwR over CBS on both datasets.

<img src="https://vision.aioz.io/thumbnail/693bb798860a42fba08b/1024/cbswr-Tab_3.png" alt="Tab-3" width="400"/>

*<center>**Table 3**: The training time (seconds) of different methods on CUB200-2011 and Car196 datasets with 20 epochs. The models are trained on a NVIDIA GeForce GTX 1080-Ti GPU [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

<img src="https://vision.aioz.io/thumbnail/534cd828896a40219bfe/1024/cbswr-Tab_4.png" alt="Tab-4" width="400"/>

*<center>**Table 4**: The impact of the number of clusters of the final model CBSwR on the performance on CUB200-2011 dataset [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

Table 3 presents the training time of different methods on the CUB200-2011 and Car196 datasets. Although the asymptotic complexity of CBSwR for training one batch is $\mathcal{O}(Km)$, it also consists of a decoder part which affects the real training. It is worth noting that the decoder is only involved during training. During testing, our method has similar computational complexity as SME.

Table 4 presents the impact of the number of clusters $K$ in the clustering loss on the CUB200-2011 dataset with our proposed model CBSwR (recall that the number of clusters $K$ is limited by the batch size $m$). During training, the number of samples per clusters vary depending on batches and the number of clusters. At $K=32$ which is our final setting, the number of samples per cluster varies from 2 to 11, on the average. The retrieval performance is just slightly different for the different number of clusters. This confirms the robustness of the proposed method w.r.t. the number of clusters.

### Comparison to the state of the art

<img src="https://vision.aioz.io/thumbnail/f4f34a6fef134ed98dc7/1024/cbswr-Tab_5.png" alt="Tab-5" width="700"/>

*<center>**Table 5**: Clustering and Recall performance on the CUB200-2011 dataset [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

<img src="https://vision.aioz.io/thumbnail/bd367ee3bb2141b3a41e/1024/cbswr-Tab_6.png" alt="Tab-6" width="700"/>

*<center>**Table 6**: Clustering and Recall performance on the Car196 dataset [(Source)](https://www.bmvc2020-conference.com/assets/papers/0328.pdf).</center>*

Table 5 presents the comparative results on CUB200-2011 dataset. In terms of clustering quality (NMI metric), the proposed method and the state-of-the-art UDML methods MOM and SME achieve comparable accuracy. However, in terms of retrieval accuracy R@K, our method outperforms other approaches. Our proposed method is also competitive to most of the supervised DML methods.

Table 6 presents comparative results on Car196 dataset. Compared to unsupervised methods, the proposed method outperforms other approaches in terms of retrieval accuracy at all ranks of K. Our method is comparable to other unsupervised methods in terms of clustering quality.

### Quantity Visualization

![Fig-5](https://vision.aioz.io/thumbnail/0f112cca39324fb6b824/1024/cbswr-Fig_2.png)
*<center>**Figure 2**: Barnes-Hut t-SNE visualization of our embedding on the CUB200-2011 dataset.</center>*

Figure 2 shows the t-SNE plots on our learned embedding features on CUB200-2011. We can see that our embedding produces reasonable results in grouping similar visual objects despite the significant variations in view-point, pose, and configuration.


## Conclusion

We propose a new method that utilizes deep clustering for deep metric learning to address the two challenges in UDML, i.e., positive/negative mining and efficient training. The method is based on a novel loss that consists of  a learnable clustering function, a reconstruction function, and a center-based metric loss function. Our experiments on CUB200-2011 and Car196 datasets show state-of-the-art performance on the retrieval task, compared to other unsupervised learning methods.

## Open Source
:cat: Github: https://github.com/aioz-ai/BMVC20_CBSwR
