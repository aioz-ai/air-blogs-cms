---
last_modified_on: "2025-04-14"
id: FedEFM4
title: FedEFM - Federated Endovascular Foundation Model with Unseen Data (Part 4)
description: Evaluation of the FedEFM using diverse experiments on collected EI Dataset.
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Medical", "Robotic"]
---



We will evaluate the FedEFM using diverse experiments on our collected EI Dataset.

## Federated Endovascular Foundation Model Validation

We first validate our proposed method (FedEFM) and compare it with different foundation models in different learning scenarios. In particular, we consider three scenarios, including Centralized Local Learning (CLL), Client-server Federated Learning (CFL), and Decentralized Federated Learning (DFL). We note that CLL is the traditional training scenario (i.e., no federated learning) where data are merged for local training. Multiple algorithms have been conducted for the comparison purpose, including CLIP, SAM, LVM-Med, FedAvg, MOON, STAR, MATCHA, RING, and CDL. We use ViT backbone in all benchmarking algorithms and train on datasets for the training phase in Table~\ref{tab:data}. Note that our default setup is maintained at 100\% unseen label corpus.


Figure 1 shows the comparison with different algorithms on multiple learning scenarios. When we train ViT in CFL and DFL setup using FedAvg and MATCHA, the accuracy is only 80.9\% and 42.4\%, respectively, reflecting the inherent challenges in federated learning. Applying our proposed FedEFM method resulted in a substantial accuracy improvement to 98.2\% and 97.5\%. These results show that our proposed method can obtain competitive results even compared with the centralized training that can gather all data and only has a minor cycle time trade-off compared with most of the federated learning methods. 

![](https://vision.aioz.io/f/80811a18c9594f28a500/?dl=1)*<center>**Figure 1**. Foundation model performance comparison</center>*

## Downstream Task Fine-tuning 

We use ViT backbone and fine-tune it using our FedEFM and different foundation models, including, CLIP, SAM, and LVM-Med. Note that, all models are evaluated under segmentation and classification tasks in endovascular intervention. 

**Metric**. We use Accuracy (\%) for the classification task; 2D Dice score, mIoU, and Jaccard metric are used for the segmentation task. For the segmentation task, we compare on our collected EIPhantom, EISimulation dataset, and CathAnimal. In the classification task, we benchmark using the RANZCR dataset.


Figure 2 shows the comparison between our method and other foundation models. This table shows that the ViT backbone under our proposed algorithm outperforms other models with a clear margin. Furthermore, models trained on medical data such as LVM-Med and our FedEFM archive better results compared with models trained on non-medical data such as CLIP and SAM. This shows that developing a domain-specific foundation model is important in the medical domain.


![](https://vision.aioz.io/f/cbf09c4822ef44b8ae80/?dl=1)*<center>**Figure 2**. Fine-tuning results on different foundation models on endovascular classification & segmentation task.</center>*

## Ablation Study 

### Unseen Data Proportion Analysis 

Figure 3 presents an analysis of our method under different percentages of unseen data. In this experiment, we assume that each silo (hospital) only keeps an amount of data (e.g., human / animal / simulated X-ray) where their data corpus only shares the similarity in a given percentage. A 100\% unseen data corpus means that the data of each hospital silo have no similarity in their data types compared to others. As the percentage of unseen data types increases, we observe a notable decline in the accuracy of the baseline on CFL and DFL scenarios. However, our proposed approach demonstrates remarkable resilience to unseen data, maintaining high accuracy even when confronted with a higher percentage of unfamiliar semantic data. In specific instances, when all data labels are unseen (100\%), ViT under CFL and DFL scenarios exhibit significantly lower accuracies at 32.1\% and 23.8\%, respectively. In contrast, our approach achieves an accuracy of 84.9\%, showcasing its effectiveness in handling unseen data.
![](https://vision.aioz.io/f/356c357eade44720a5a3/?dl=1)*<center>**Figure 3**. Result with different unseen data proportions.</center>*
### Backbone Analysis 

We verify the stability of our method on different networks, including UNet, TransUNet, and SwinUnet and ViT under federated learning scenario. Figure 4 shows the performance of the different backbones when we fine-tune them using our FedEFM. We can see that using our foundation model to initialize the weights of those backbones significantly improves the results. These results validate the effectiveness of our training process in addressing the unseen data problem, and our FedEFM is useful for different backbones in endovascular downstream tasks.

![](https://vision.aioz.io/f/1943e4d95e4843999484/?dl=1)*<center>**Figure 4**. Performance on different network when fine-tuning using our foundation model.</center>*


Figure 5 illustrates the catheter and guidewire segmentation results of fine-tuning ViT on our method and different foundation models. The visualization portrays that our method excels in accurately delineating the catheter and guidewire structures, showcasing superior segmentation performance compared to other approaches. This figure further confirms that we can successfully train a federated endovascular foundation model without collecting users' data and the trained foundation model is useful for the downstream segmentation task.

![](https://vision.aioz.io/f/08314a9cb04842e7af96/?dl=1)*<center>**Figure 5**. Unseen data issue</center>*


## Limitations

While our proposed approach demonstrates significant potential, it is subject to certain limitations that warrant further investigation. Firstly, the requirement for additional weight exchange among silos extends the overall training time. However, this limitation is mitigated to some extent by the higher convergence speed of our method compared to other approaches. Additionally, our method is designed for deployment in silos with strong GPU computing resources, but the varying hardware capabilities present in many real-world federated learning networks necessitate further examination. Overcoming these limitations will open new research in federated foundation learning for endovascular interventions and other medical applications. Furthermore, addressing the challenges of managing heterogeneous data distributions and ensuring robust data privacy remains a critical focus. Moving forward, we plan to extend our approach to robotic-assisted endovascular surgery and other areas, such as pathology, to further investigate the application of federated foundation models in medical imaging and robotic systems.


## Conclusion

We present a new approach to train an endovascular foundation model in a federated learning setting, leveraging differentiable Earth Mover's Distance and knowledge distillation to handle the unseen data issue. Our method ensures that once the foundational model is trained, its weights can be effectively fine-tuned for downstream tasks, thereby enhancing performance. Our approach achieves state-of-the-art results and contributes to the field of endovascular intervention, particularly by addressing the critical issue of data sharing in the medical domain. By enabling weight exchange among local silos and fostering knowledge transfer, our method improves model generalization while preserving data privacy.
Experimental results across various endovascular imaging tasks validate the efficacy of our approach, demonstrating its potential for application in privacy-sensitive medical domains. We will release our implementation and trained models to facilitate reproducibility and further research.