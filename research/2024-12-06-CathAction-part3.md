---
last_modified_on: "2024-12-06"
id: CathAction3
title: CathAction - A Benchmark for Endovascular Intervention Understanding (Part 3)
description: Benchmarks on our CathAction dataset.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "CathAction", "Benchmark for Endovascular Intervention Understanding"]
---

## 1. Tasks and Benchmarks

In this section, we benchmark five tasks, including anticipation, recognition, segmentation, collision detection, and domain adaptation, to demonstrate the usefulness of the CathAction dataset. We then discuss the challenges and opportunities for improvement in each task.

### A. Catheterization Anticipation

The anticipation task aims to predict the next catheterization action based on a sequence of frames. We adapt the conventional anticipation task framework in computer vision, introducing two timing parameters: anticipation time ($\tau_a$) and observation time ($\tau_o$). The anticipation time denotes the required duration to recognize an action, while the observation time indicates the length of the video footage to analyze before making a prediction. The objective is to predict the action class \( c_a \) for the frames within the anticipation time $\tau_a$, given the frames during the observation time $\tau_o$.

**Network and Training**. We leverage state-of-the-art action anticipation methods as baselines: CNN&RNN, RU-LSTM, TempAggRe-Fusion, AFFT, and Trans-SVNet. The future action predictions are supervised using cross-entropy loss with labeled future actions. Following prior works, we set $\tau_a = 1s$ and $\tau_o = 1s$. Training was performed on a single Nvidia A100 GPU with a batch size of 64 for 80 epochs, starting with a learning rate of 0.001, reduced by a factor of 10 after epochs 30 to 60. We split approximately 80% of the dataset for training and 20% for testing. Performance metrics include top-1 accuracy, precision, and recall.

| Baseline                  | Venues       | Accuracy | Precision | Recall  |
|---------------------------|--------------|----------|-----------|---------|
| CNN                       | CVPR 2018   | 28.98    | 30.14     | 29.76   |
| RNN                       | CVPR 2018   | 29.64    | 30.38     | 30.44   |
| RU-LSTM                   | CVPR 2019   | 35.08    | 34.29     | 34.77   |
| TempAggRe                 | ECCV 2020   | 34.64    | 35.56     | 34.71   |
| Trans-SVNet               | IJCARS 2022  | 29.06    | 19.67     | 20.28   |
| AFFT                      | WACV 2023   | **37.91**| **36.87** | **37.63**|

**Table 1:**  Catheterization anticipation results on the CathAction dataset. All values are reported in percentages (\%).

![Figure 1](https://vision.aioz.io/thumbnail/83d9fc0cf64b4166ab71/1024/vls_ant_v38-1.png)

**Figure 1:** Qualitative catheterization prediction results. The predicted and ground truth of the next action are displayed on the right of each sample. The green color shows the correct prediction, and the red color shows the incorrect prediction.

**Results**. Table 1 shows the catheterization anticipation results of different baselines. This table shows that transformer-based methods show superior performance advantages over CNN or LSTM-based models. Qualitative results are illustrated in Fig 1. We can see that transformer-based models can make more accurate predictions in challenging scenarios, especially when the catheter is moving quickly or when the occlusion is partially obscured.

**Discussion**. Despite the advancements, existing methods for catheterization anticipation still struggle to achieve high accuracy, revealing areas for future research. The rapid motion of the catheter and guidewire poses significant challenges for this task, and real-time performance is crucial as surgeons require immediate feedback during procedures.

### B. Catheterization Recognition

Following the traditional action recognition task in computer vision, in catheterization recognition, given an input video segment, our goal is to predict the action class for that video segment.

**Network and Training**. We explore state-of-the-art methods in action recognition to benchmark the catheterization recognition task, including TDN, Video Swin Transformer, and BERT Pretraining of Video Transformers (BEVT). Each model is trained using two Nvidia A100 GPUs for 80 epochs, with a mini-batch size of 512. The initial learning rates are set to 0.01 for the spatial stream and 0.001 for the temporal stream, reduced by a factor of 10 at the 20 and 40 epochs. All other parameters are re-used from the baseline methods.

**Results**. Table 2 show the catheterization recognition results of three baseline methods: TDN, Video Swin Transformer, and BEVT, on the CathAction dataset are summarized in the table below. TDN with ResNet101 achieves the best top-1 accuracy of 62.5% on five classes. Action recognition in endovascular intervention remains challenging due to the similarity in the appearance of catheters and guidewires across different environments, while actions depend on the visual characteristics of the catheters and guidewires.

| Baseline                        | Venues     | Accuracy | Precision | Recall  |
|---------------------------------|------------|----------|-----------|---------|
| TDN-ResNet50                    | CVPR 2021  | 58.34    | 59.12     | 57.22   |
| TDN-ResNet101                   | CVPR 2021  | **62.50**| **61.89** | **62.77**|
| Video Swin Transformer          | CVPR 2022  | 51.67    | 52.14     | 51.24   |
| BEVT                            | CVPR 2022  | 49.28    | 50.27     | 49.92   |

**Table 2:** Catheterization recognition results on the CathAction dataset. All values are reported in percentages (\%).

**Discussion**. Compared to the anticipation task (Table 1), catheterization recognition methods (Table 2) show higher accuracy. However, the overall performance is not yet significant enough for real-world applications. Further research can utilize advanced techniques such as multi-modality learning, combining pre-operative data or synthetic data with transfer learning to improve the results. Additionally, exploring the capabilities of large-scale medical foundation models is an interesting research direction.

### C. Catheter and Guidewire Segmentation

Catheter and guidewire segmentation is a well-known task in endovascular interventions. In this task, we aim to segment the catheter and guidewire from the background. Unlike catheterization recognition or anticipation, which take a video as input, this segmentation task only uses the X-ray image as input.

| Baseline                         | Dice Score | Jaccard Index | mIoU  | Accuracy |
|----------------------------------|------------|---------------|-------|----------|
| UNet                            | 51.69      | 57.51         | 31.17 | 63.26    |
| TransUNet                       | 56.52      | 55.93         | 34.13 | 55.61    |
| SwinUNet                        | 61.26      | **59.54**     | 39.53 | **76.60**|
| SSL                             | 56.95      | 56.87         | 40.87 | 72.24    |
| SegViT                          | **63.47**  | 54.12         | **42.48** | 68.73  |

**Table 3:** Segmentation results on the CathAction dataset.

**Network and Training**. We benchmark U-Net, Trans-UNet, SwinUNet, and SegViT. We follow the default training and testing configurations provided in the published papers. We use the Dice Score, Jaccard Index, mIoU, and Accuracy as the evaluation metrics in the segmentation task.

**Results**. Table 3 shows the catheter and guidewire segmentation results. This table shows that the transformer-based networks such as TransUNet or SegViT achieve higher accuracy than the traditional UNet. The SegViT that utilizes the vision transformer backbone shows the best performance, however, the increase compared with other methods is not a large margin.

**Discussion**. In contrast to traditional segmentation tasks in computer vision, which typically involve objects occupying substantial portions of an image, the segmentation of catheters and guidewires presents a considerably greater challenge. These elongated instruments have extremely slender bodies, making their spatial presence in the image less pronounced. Additionally, the unique characteristics of X-ray images can lead to misidentification of catheters or guidewires as blood vessels. Addressing these challenges in future research is imperative to enhance the accuracy of segmentation outcomes.

### D. Collision Detection

Detecting the collision of the tip of the catheter or guidewire to the blood vessel wall is an important task in endovascular intervention. We define the collision detection task as an object detection problem. In particular, the tip of the catheter or guidewire of all frames in our dataset is annotated with a bounding box. Each bounding box shares the class of either `collision` when the tip collides with the blood vessel, or `normal` when there is no collision with the blood vessel. 

**Network and Training.** We use YOWO, YOWO-Plus, STEP, and HIT. Since the bounding boxes in our ground truth have relatively small sizes, we also explore tiny object detection methods such as Yolov and EFF. The training process starts with a learning rate of 0.0003, which is then decreased by a factor of 10 after 20 epochs, concluding at 80 epochs. We train all methods with a mini-batch size of 4 on an Nvidia A100 GPU. The average precision (AP) and mean average precision (mAP) are used to evaluate the detection results.

| Baseline       | Collision | Normal | Mean | Collision | Normal | Mean |
|----------------|-----------|--------|------|-----------|--------|------|
|                | **AP**    |        |      | **mAP**   |        |      |
|                |           |        |      |           |        |      |
| STEP           | 7.79      | 11.21  | 10.98| 6.92      | 11.29  | 9.08 |
| YOWO           | 8.32      | 12.18  | 11.73| 7.46      | 12.28  | 9.92 |
| YOWO-Plus      | 8.92      | 12.23  | 11.77| 7.86      | 12.48  | 10.28|
| HIT            | 9.37      | 12.74  | 12.14| 8.18      | 12.72  | 10.81|
| Yolov*         | 12.30     | 21.08  | 15.89| 11.88     | 20.04  | 14.11|
| EFF*           | **13.70** | **22.10**| **16.91**| **12.14**| **20.78**| **14.88**|

**Table 4:** Collision detection results on the CathAction dataset. The symbol (*) denotes tiny object detectors.

![Figre 2](https://vision.aioz.io/thumbnail/776498afcf5f40909ab6/1024/vls_detection.v12-1.png)

**Figure 2:** Qualitative results for the collision detection task. The first two columns visualize the collision results, the third column visualizes no collision cases, and the last column visualizes a failure case where the tip was not detected.

**Results.** Table 4 shows the collision detection results. This table indicates that tiny object detectors such as Yolov and EFF achieve higher accuracy compared to other normal object detectors. Furthermore, we observe that the performance of all methods remains relatively low. This highlights the challenges that lie ahead for collision detection in endovascular intervention. Figure 2 shows detection examples where EFF has difficulty detecting collisions between the catheter and the blood vessel.

**Discussion.** Compared to traditional object detection results on vision datasets, the collision detection results on our dataset are significantly lower, with the top mean AP being only 16.91. The challenges of this task come from two factors. First, the tip of the catheter or guidewire is relatively small in X-ray images. Second, the imbalance between the `collision` and `normal` class makes the problem more difficult. Therefore, there is a need to develop special methods to address these difficulties. Future works may rely on attention mechanisms, transformers, or foundation models to develop more sufficient endovascular collision detectors.

### E. Domain Adaptation

Our dataset is sourced from two distinct environments: vascular *phantom data* and *animal data*. To assess the capacity for learning from phantom data and applying it to real data, we benchmark endovascular interventions under domain adaptation setups. For each task, we train the model on the phantom data and then test it on real animal data. In practice, animal data is similar to human data we capture from humans, and it is much more challenging to perform tasks on real animal or human data.

| Baseline | Venues | Accuracy | Precision | Recall |
|----------|--------|----------|-----------|--------|
| RU-LSTM | CVPR 2019 | 22.93 | 23.91 | 22.57 |
| TempAggRe | ECCV 2020 | 17.16 | 18.41 | 18.23 |
| Trans-SVNet | IJCARS 2022 | 19.06 | 17.67 | 19.58 |
| AFFT | WACV 2023 | **25.67** | **26.29** | **26.33** |

**Table 5:** Catheterization anticipation results under domain adaptation setup. All methods are trained on phantom data and tested on animal data.

**Anticipation Adaptation**. We use the same methods RU-LSTM, TempAggRe, Trans-SVNet, and AFFT for anticipation adaptation experiments. Table 5 shows the results. Compared with the setup in Table 1, we can see that there is a significant accuracy drop. This highlights the challenges of applying baseline methods in practical real-world scenarios, particularly when dealing with unforeseen situations in catheterization procedures.

| Baseline | Venues | Accuracy | Precision | Recall |
|----------|--------|----------|-----------|--------|
| TDN-ResNet50 | CVPR 2021 | 24.19 | 23.17 | 24.56 |
| TDN-ResNet101 | CVPR 2021 | 25.62 | 24.52 | 25.68 |
| Video Swin Transformer | CVPR 2022 | 28.79 | 27.98 | 28.12 |
| BEVT | CVPR 2022 | **31.22** | **30.48** | **31.79** |

**Table 6:** Catheterization recognition results under domain adaptation setup.

**Recognition Adaptation**. We repeat the catheterization recognition task under the domain adaptation setup. Table 6 shows the results when all baselines are trained on phantom data and tested on animal data. This table also demonstrates that training under domain adaption setup is very challenging, as compared to Table 2 under normal setting, the accuracy drops approximately $30\%$.


|                   |           |  **AP** |      |            | **mAP** |      |
|-------------------|-----------|---------|------|------------|---------|------|
| Baseline          | Collision | Normal  | Mean | Collision  | Normal  | Mean |
| STEP              | 1.53      | 2.12    | 1.87 | 1.09       | 1.98    | 1.62 |
| YOWO              | 2.12      | 4.11    | 3.09 | 1.97       | 3.68    | 2.92 |
| YOWO-Plus         | 1.18      | 1.43    | 1.21 | 1.07       | 1.26    | 1.09 |
| HIT               | 1.31      | 1.19    | 1.24 | 1.06       | 1.18    | 1.11 |
| **Yolov**         | **7.31**  | **8.92**| **8.09** | **6.28** | **7.49** | **7.21** |
| **EFF**           | **8.27**  | **9.16**| **8.19** | **7.61** | **8.29** | **7.88** |



**Table 7:** Collision detection results under domain adaptation setup. All methods are trained on phantom data and tested on animal data. The symbol (*) denotes tiny object detectors.

**Collision Detection Adaptation**. Table 7 shows the results collision detection results under domain adaptation. We can see that under domain adaptation setup, most object detection methods achieve very low accuracy. Therefore, there is an immediate need to improve or design new methods that can detect the collision in real-time for endovascular catheterization procedures.

| Baseline | Dice Score | Jaccard Index | mIoU | Accuracy |
|----------|------------|---------------|------|----------|
| UNet | 26.58 | 31.38 | 12.13 | 46.07 |
| TransUNet | 16.16 | 24.19 | 17.23 | 33.61 |
| SwinUNet | 17.41 | **38.14** | 7.52 | 40.79 |
| SSL | 26.91 | 32.04 | **18.72** | 42.44 |
| SegViT | **30.74** | 32.22 | 11.46 | **50.00** |

**Table 8:** Domain adaptation segmentation results.

**Segmentation Adaptation**. Table 8 shows the catheter and guidewire segmentation results when the networks are trained on phantom data and tested on animal data. Similar to other tasks under the domain adaptation setting, we observe a significant accuracy drop in all methods. Overall, SegViT still outperforms other segmentation methods. This shows that the vision transformer backbone may be potentially a good solution for this task. 

## 2. Discussion

We introduce CathAction, a large-scale dataset for endovascular intervention tasks, encompassing annotated ground truth for segmentation, action understanding, and collision detection. While CathAction marks a significant advancement in endovascular interventions, it is important to acknowledge certain limitations. First, despite its comprehensiveness, the dataset may not encompass every possible clinical scenario and could potentially lack representation for rare or outlier cases. Second, our work currently benchmarks vision-based methods, which exhibit insufficient accuracy, and persisting challenges in generalizability and adaptability to real-world scenarios are evident. This is highlighted by the results presented in Section 1 for all catheterization anticipation, recognition, segmentation, and collision detection tasks. Thirdly, we mostly utilize metrics from the vision community to evaluate the results. These metrics may not fully reflect the clinical needs, and the continuous refinement of evaluation metrics and exploration of potential interdependencies among tasks would demand further research.

From our intensive experiments, we see several research directions that benefit from our large-scale datasets: 
1. There is an immediate need to develop more advanced methods for catheterization anticipation, recognition, collision detection, and action understanding, especially under domain adaptation setup. Future work can explore the potential of graph neural networks, temporal information, and multimodal or transfer learning to improve the accuracy and reliability of the methods.
2. Currently, we address endovascular intervention tasks independently; future work can combine those tasks and tackle them simultaneously (e.g., the anticipation and collision detection tasks can be jointly trained). This would make the research outputs more useful in clinical practice.
3. Given the fact that CathAction is a large-scale dataset, it can be used to train a foundation model for endovascular interventions or related medical tasks.

## 3. Conclusion
We introduce CathAction as a large-scale dataset for endovascular intervention research, offering the largest and most comprehensive benchmark to date. With intensive annotated data, CathAction addresses crucial limitations in existing datasets and helps connect computer vision with healthcare tasks. By providing a standardized dataset with public code and public metrics, CathAction promotes transparency, reproducibility, and the collective exploration of different tasks in the field. Our code and dataset are publicly available to encourage further study.