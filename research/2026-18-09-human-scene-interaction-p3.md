---
last_modified_on: "2026-09-09"
title: Efficient Human-Contact Representation for Human-Scene Interaction
description: Lightweight network, human-scene interaction
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';



In the previous part, we examined the detailed architecture of the ECO method. We now turn to the experimental analysis, covering implementation details, evaluation metrics, and the effectiveness of our approach compared to baseline methods. 


## Experiments
We validate our method on two tasks: contact prediction and scene synthesis. For contact prediction, we train a conditional Variational Autoencoder (cVAE) model as in POSA, replacing each traditional layer with our sparse layers. For scene synthesis, we use the predicted contact labels from our efficient contact model to generate objects that make contact with the human body at predicted points, ensuring alignment with human intent and avoiding body penetration.



### Contact Prediction Results
**Datasets.** We use PROXD, GIMO, and BEHAVE datasets for contact prediction. In all datasets, human bodies are modeled by SMPL-X format. In the PROXD dataset, the contact labels are from PROX-E dataset.

**Evaluation Metrics.** The Reconstruction Accuracy and Consistency Score are used for comparison. We also compare the inference time (second/sample) of all methods on the same NVIDIA Tesla V100 GPU.



**Baselines.** We compare our method with recent works, including POSA, ContactFormer, multi-layer perceptron predictor or bidirectional LSTM , MIME, PIAL-Net, TRUMANS, Ins-HOI and HOT. We train our ECO using $K=10$ masks and keep only $\kappa=3$ masks with the highest values of mask score $\bm\alpha$ during inference. 





**Results.** Table 1 shows the comparison between our method and other baselines. This table indicates that our model surpasses all other baselines by a large margin with a reconstruction accuracy of $93.69\%$, and a consistency score of $0.981$. Furthermore, our inference speed is $0.009$ second/sample, which is 12 times faster than the runner-up.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788952936/main_table_gs2rje.png)
*<center>**Table 1**: Quantitative comparison of ECO against different contact prediction methods.  </center>*


**Visualization.** Figure 4 shows the qualitative comparison of contact prediction results with different methods. We can see that our method stands out by achieving accurate contact predictions in both the contact labels and contact locations while other methods show a mismatch on contact points. 


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944368/contact_comparison_vd36ou.png)
*<center>**Figure 4**:  Contact prediction visualization. The first row includes the contact no-tation and inputs while the second row shows prediction results.  </center>*

### Scene Synthesis Results
**Datasets.**
We use the PROXD and GIMO datasets for conducting experiments as in recent works. Note that BEHAVE dataset cannot be used in the scene synthesis task since this dataset only has contacts with independent objects, not ones synchronized in a scene. 


**Baselines.**
We compare our method with recent baselines on the scene synthesis domain, including ContactICP, PosePrior, SUMMON, MIME, SceneDiffuser, HAISOR,  and INFERACT.





**Evaluation Metric.** We use **non-collision score** as a metric for the scene synthesis task. We also perform a user study to compare different methods.



**Results.** Table 2 and Figure 5 show comparisons between scene synthesis results. %ContactICP, although exhibiting relatively lower non-collision values, represents an initial approach in this task. Recent works such as SUMMON, MIME, and SceneDiffuser show high reconstruction accuracy, outperforming PosePriors on both datasets. However, our method surpasses all techniques with a clear margin. 

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788952936/scene_synthesis_results_xjvugw.png)
*<center>**Table 2**:  Scene synthesis results.</center>*


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944368/comparison_zhq3l7.png)
*<center>**Figure 5**: Scene synthesis visualization between different methods. Our method efficiently utilizes predicted contacts to produce more reasonable and comprehensive scenes.  </center>*


**User Study.**
We conduct a user study with 40 participants from various backgrounds.
Participants are presented with a choice between our method and other models, displayed side by side. Both sets of samples are generated using the PROXD test set. This process is repeated five times for each model and the user scores are from $1$ to $5$. There are two judgment criteria: *(i) Naturalness* identifies if the position and orientation of facilities are generated properly in the scene and matched with the human poses or not, and *(ii) Non-Collision* shows if the generated object collides with human motions. The results in Figure 6 show that our method is preferred over other models. 




![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944369/userstudy_jtyk5f.png)
*<center>**Figure 6**:  User evaluation between methods.</center>*


### Comparison with Sparse Coding Methods
**Baselines.** We compare our method with four representative sparse representation and sparse computation approaches for the contact prediction task, including Minkowski Engine, EsCoin, pSConv, and 1-D Blocking. For a fair comparison, all methods are integrated into the same contact prediction framework and evaluated under identical experimental settings.

**Implementation.** We adopt POSA as the backbone architecture for all contact prediction experiments. Following prior work, the input consists of human-scene interaction representations derived from the corresponding datasets. To evaluate both effectiveness and efficiency, we report Reconstruction Accuracy (\%), Consistency Score, and inference speed (seconds/sample). All experiments are conducted using the same hardware configuration and evaluation protocol to ensure fair comparison across methods.

**Results.** Table 3 presents the performance of different sparse representation methods. We can see that our method achieves the highest accuracy compared to all the other sparse coding baselines. For inference speed, our method is only slower than ME ($0.009$ second/sample vs. $0.008$ second/sample) while our accuracy is $10.08\%$ higher.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788952936/sparse_method_v3qx2r.png)
*<center>**Table 3**:  Comparison between sparse representation methods on PROXD.</center>*

### Component Analysis 
Table 4 below shows some results regarding the contribution of sparse masks and a sparse network. POSA serves as our baseline, and if setups involve masks, three masks are used. It is evident that when we use sparse masks without the decomposition, the number of data points remains unchanged, leading to no improvement in speed and, in fact, a decrease in accuracy due to missing information. Applying a sparse network to original inputs improves speed, but the trade-off for accuracy is noticeable, as discussed in many previous papers. When we apply the decomposition to the original inputs, the differences in speed and accuracy are not significant compared to the original baseline since the based-decomposition method cannot work properly with full dense tensors.

With our introduced refinement process that works on integrated decomposition, we can preserve the performance of the model but the speed improvement is not guaranteed. Ultimately, when we use decomposed inputs obtained from sparse masks and a sparse network together, the input shape problem can be addressed, achieving optimization in both speed and accuracy.


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788952936/redundant_info_xgbqyz.png)
*<center>**Table 4**:  Redundant information analysis by selecting masks based on mask score $\bm{\alpha}$</center>*


### Sparse Mask Analysis
**Sparsity ratio and the number of sparse masks.**
Figure 7 illustrates the correlation between reconstruction accuracy and inference speed of our method under different values of sparsity ratio and the number of sparse masks $K$. We note that $K=50$ masks are used during training. During inference, we consequently only select $\kappa$ masks based on the value of mask score $\bm \alpha$. We can see that using $\kappa=1$ mask leads to faster model performance, however, this also significantly reduces accuracy due to the loss of input information. In contrast, employing multiple sparse masks helps retain essential information and improves the overall model performance. Overall, Figure 1 shows that using $\kappa=3$ masks with $90\%$ sparsity ratio during the inference brings the balance of the accuracy and inference speed.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944369/sparse_ratio_l2qurs.png)
*<center>**Figure 7**:  Model Effectiveness with different sparsity ratios and numbers of masks. Three sparse masks provide balancing between reconstruction accuracy and inference speed across varying sparsity ratios.</center>*

**How sparse masks help reduce redundancy and improve the results?** Our sparse masks act as filters to reduce non-useful information in human-scene input. Specifically, they reduce vertices in representations, influencing both inference speed and accuracy. Table 5 shows how sparse masks help eliminate redundant input. The *"Original Input"* uses all vertices, while *"Keep only 01 mask"* uses only $\kappa=1$ mask at inference. We also evaluate setups with $3$, $10$, and all $50$ masks. Our method is trained with $K=50$ sparse masks, each with $90\%$ sparsity. Masks are kept based on mask score $\bm{\alpha}$ in Section 1. As shown in Table 1 , the model using just 1 sparse mask reduces vertex processing requirements by $90\%$, significantly enhancing inference speed but causing a $7.51\%$ accuracy drop compared to the *"Original Input"* setup. With $50$ masks, our ECO maintains accuracy but increases inference time since too many masks are used. Using mask score $\bm{\alpha}$, we can remove non-useful masks and retain only $10$ or even $3$ informative masks during inference. %We see that using only $3$ masks during inference helps reduce the verticle input while increasing the accuracy and reducing the inference speed.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788952936/ablation_result_y5bllm.png)
*<center>**Table 5**:  The effectiveness of each component in our method. Results are benchmarked on the PROXD dataset.</center>*

Figure 8 illustrates the ground truth, the contact heatmaps from the baseline POSA using dense tensors, and our ECO approach with sparse masks. Vertices within the highlighted red ellipse are redundant and ignored by ECO, while other methods still consider them, leading to incorrect contact predictions. These results imply that ECO effectively removes redundant input information, improving model performance.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944369/heatmap_vis_zc6sio.png)
*<center>**Figure 8**:   Contact heatmaps visualization. Red circles indicate that our outputs contain less noisy information and exhibit more concentrated contact regions compared to POSA, resulting in cleaner contact predictions.</center>*

For further clarification, we conduct an extended analysis to examine our proposal. Figure 9 illustrates the results of ECO when we change the number of sparse masks and the sparsity ratio. In particular, Figure 1a shows the Reconstruction Accuracy, and Figure 1b demonstrates the corresponding GPU inference time. The results indicate that as the number of masks and the sparsity ratio increase, the inference speed decreases. Additionally, more redundant masks can be established. However, with an appropriate trade-off, state-of-the-art results with efficient inference time can be achieved. We can observe that, with a $90\%$ sparsity ratio and 3 masks, we achieve a state-of-the-art $93.6\%$ accuracy while still maintaining efficient processing during inference ($0.01$ second/sample).

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944367/ablation_aqleu4.png)
*<center>**Figure 9**: Reconstruction Accuracy (%) and Inference Speed (s/sample) between setups. The visualization highlights the trade-off between achieving high sparsity in the model, maintaining reconstruction quality, and preserving computational efficiency.</center>*

**Feature Similarity Analysis.** Figure 10 presents the similarity between features of POSA baseline and features of our ECO model when we keep $1$, $3$, $5$, and all $10$ sparse masks during the inference. We train the ECO model with $K=10$ sparse masks, each mask has a sparsity ratio of $90\%$ in this experiment. The mask score $\bm{\alpha}$ is used to rank and choose useful masks during inference. To compare feature similarity maps, we pass test samples of PROXD datase to both POSA and our ECO model with the corresponding number of masks. Then, we extract the features from each layer and use the Euclidean distance to compute similarity. While features extracted from the POSA Network remain unchanged in all setups, features of our ECO change when the number of sparse masks is changed.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944367/attention_tfmfct.png)
*<center>**Figure 10**: Similarity between outputs of intermediate layers. </center>*

We can see that in Figure 10a, using only $1$ mask with the highest mask score $\bm \alpha$ only maintains feature similarity at abstract layers and the dissimilarity significantly increases in later layers (lightens in early layers and darkens in latter ones). Using $3$ masks (Figure 10b) or $5$ masks (Figure 10c) shows good feature similarity within corresponding masks (most features show high similarity in their corresponding layers).

This behavior shows that the representations extracted from each layer in our model are distinctive, highlighting how our proposed method handles redundant information compared with all features from the setup that does not use the mask score $\bm{\alpha}$ to select the useful masks. (Figure 10d).


### Failure Cases Analysis 
Figure 11 presents representative failure cases of scene synthesis. Most failures involve objects generated with suboptimal positions, orientations, or incomplete scene layouts, which may reduce the overall plausibility of the synthesized environments. These cases typically arise in highly ambiguous interaction scenarios where multiple scene configurations can satisfy similar contact patterns. Since our method selects a fixed number of sparse masks $\kappa$ during inference, certain interaction cues may receive less emphasis in particularly complex scenes. In addition, some interactions can be supported by multiple plausible object arrangements, making it difficult to recover the exact scene configuration from contact information alone. As a result, the generated scenes may contain redundant objects or local geometric inconsistencies despite preserving the overall interaction structure. Nevertheless, even in these challenging cases, the synthesized scenes generally maintain the major human-scene contact relationships and remain consistent with the intended human activity. These observations suggest that ECO successfully captures interaction-critical information.
![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1788944369/fail_cases_li1akd.png)
*<center>**Figure 11**: Visualization of a good case scene synthesis compared with failure cases.</center>*
## Discussion
**Limitations.**
While our method achieves strong performance across multiple human-scene interaction tasks, several limitations remain. First, training with multiple sparse contact masks introduces additional computational overhead compared with dense baselines. Second, the effectiveness of our framework depends on the choice of sparsity ratio and the number of active masks, which may vary across datasets and interaction scenarios. Finally, our current formulation uses a fixed number of selected masks $\kappa$ during inference, which may not fully capture the varying complexity of different interactions.

**Future Work.**
Several promising directions remain for future exploration. Learning adaptive mask selection and sparsity ratios directly from data could further improve both efficiency and representation quality. Extending contact-aware sparse representations to temporal interaction modeling may benefit related tasks such as action recognition, pose estimation, and object manipulation. In addition, integrating contact-aware sparsification with emerging generative frameworks may further improve the scalability of human-scene interaction systems.



**Conclusion.**
We present ECO, a contact-aware sparse representation framework for efficient human-scene interaction modeling. Our key insight is that physical interactions are inherently sparse, as only a small subset of human vertices actively participates in contact with the surrounding environment. By explicitly identifying informative contact regions through sparse contact masks and combining them with sparse operators, ECO effectively reduces representation redundancy while preserving interaction-critical information. Extensive experiments on three benchmark datasets demonstrate that our approach consistently improves reconstruction accuracy while substantially reducing inference cost compared with existing methods. These results suggest that contact-aware sparsity provides an effective and general representation paradigm for efficient human-scene interaction reasoning.


Across all three parts, we hope this blog provides valuable insights into contact prediction and scene synthesis, while also highlighting the ECO method, which leverages sparse tensors to achieve an effective balance between accuracy and computational efficiency.

</CodeExplanation>