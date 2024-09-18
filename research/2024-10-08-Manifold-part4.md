---
last_modified_on: "2024-10-08"
id: Manifold4
title: Scalable Group Choreography via Variational Phase Manifold Learning (Part 4)
description: Effectiveness of Scalable Group Choreography
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---
In the previous part,  we introduce training process and experimental setup. In this part, we  validate the effectiveness and efficiency of the proposed method.

![](https://vision.aioz.io/f/4248eadb89e143479193/?dl=1)*<center>**Figure 1**. We present a new group dance generation method that can generate a large number of dancers within a fixed resource consumption. The illustration shows a generated group dance sample with $100$ dancers. </center>*

## Experimental Results
### Quality Comparison
Table 1 presents a comparison among the baselines FACT, Transflower, EDGE, GDanceR, GCD, and our proposed manifold-based method. The results clearly demonstrate that our model outperforms the baselines across all evaluations on two datasets, AIOZ-GDANCE and AIST-M. We observe that recent diffusion-based dance generation models, such as EDGE or GCD, achieve competitive performance on both single-dance metrics (FID, MMC, GenDiv, and PFC) and group dance metrics (GMR, GMC, and TIF). However, due to limitations in their training procedures, they still struggle with generating multiple dancing motions when faced with a large number of dancers, as indicated by their lower performance compared to our method. This suggests that our approach successfully maintains the quality of dance motions as the number of dancers increases. 
![](https://vision.aioz.io/f/b9422a24364c49a3bc76/?dl=1)*<center>**Table 1**. Performance comparison. </center>*
Additionally, Figure 2 illustrates that our proposed method outperforms other state-of-the-art models like GDanceR and GCD in addressing issues such as monotonous, repetitive, sinking, and overlapping dance motions.
![](https://vision.aioz.io/f/2ca8d63c7af844c28ac2/?dl=1)*<center>**Figure 2**. Visualization of a dancing sample between different methods. GDanceR displays monotonous, repetitive, or sinking dance motions. GCD exhibits more divergence in dance motions, yet dancers may intersect since their optimization does not address this issue explicitly. Blue boxes mark these issues. In contrast, our manifold-based solution ensures the divergence of dancing motions, while the phase motion path demonstrates its effectiveness in addressing floating and crossing issues in group dances.</center>*
### Scalable Generation Analysis
Table 2 illustrates the performance of different group dance generation methods (GCD, GDancer, and Ours) when generating dance movements with an increasing number of dancers. When the number of dancers is increased to 10, GCD appears to run out of memory, which is also observed in GDanceR when the number of dancers increases to 100. Regardless of the number of dancers, our method consistently achieves stable and competitive results. This implies that our proposed method successfully addresses the scalability issue in group dance generation without compromising the overall performance of each individual dance motion.

![](https://vision.aioz.io/f/18fa3ed3a50241dea3a2/?dl=1)*<center>**Table 2**. Performance of group dance generation methods when we increase the number of generated dancers. The experiments are done with common consumer GPUs with 4GB memory. (N/A means models could not run due to inadequate memory footprint). </center>*

Figure 3 illustrates the memory consumption to generate dance motions in groups for each method. Noticeably, our proposal still achieves the highest performance while consuming immensely fewer resources required for generating group dance motions (See Figure 4 for illustrations). This, again, indicates that our method successfully breaks the barrier of limited generated dancers by using the manifold.

![](https://vision.aioz.io/f/91fee8ed6b0340528f9a/?dl=1)*<center>**Figure 3**. Memory usage vs. number of dancers in different dance generators.</center>*

![](https://vision.aioz.io/f/059bd16a2543453c9a43/?dl=1)*<center>**Figure 4**. Visualization of a dancing sample between different methods. GDanceR displays monotonous, repetitive, or sinking dance motions. GCD exhibits more divergence in dance motions, yet dancers may intersect since their optimization does not address this issue explicitly. Blue boxes mark these issues. In contrast, our manifold-based solution ensures the divergence of dancing motions, while the phase motion path demonstrates its effectiveness in addressing floating and crossing issues in group dances.</center>*

### Ablation Study
Table 3 presents the performance improvements achieved through the integration of consistency loss and phase manifold. Additionally, we showcase the effectiveness of our proposed approach across three different backbones: Transformer, LSTM, and CNN. Evaluation metrics including FID, GMR, and GMC are utilized. The results indicate that the absence of consistency loss leads to an increase in GMR and a decrease in GMC, suggesting a significant enhancement in the realism and correlation of group dance motions facilitated by the inclusion of the proposed objective. Meanwhile, with out the phase manifold, the model exhibits remarkably higher scores in both the FID and GMR metrics, suggesting that phase manifold can effectively improve the distinction in dance motions while maintaining the realism of group dances, even when the number of dancers in a group is large. In comparing three backbones—Transformer, LSTM, and CNN—we have observed that the chosen Transformer backbone achieved the best results compared to LSTM or CNN. 

![](https://vision.aioz.io/f/52b6bb00d66a4a588674/?dl=1)*<center>**Table 3**. Module contribution and loss analysis. </center>*

### User Study
User studies are vital for evaluating generative models, as user perception is pivotal for downstream applications; thus, we conducted two studies with around 70 participants each, diverse in background, with experience in music and dance, aged between 20 to 40, consisting of approximately 47\% females and 53\% males, to assess the effectiveness of our approach in group choreography generation.

In the user study, we aim to assess the realism of sample outputs with more and more dancers. Specifically, participants assign scores ranging from 0 to 10 to evaluate the realism of each dance clip with 2 to 10 dancers. Figure 5 shows that, across all methods, the more the number of dancers is increased, the lower the realism is found. However, the drop in realism of our proposed method is the least compared to GCD and GDanceR. The results indicate our method's good performance compared to other baselines when the number of dancers increases. 
![](https://vision.aioz.io/f/fbcff0c46d864294a36c/?dl=1)*<center>**Figure 5**. Realism between different methods when number of dancers is varied.</center>*

## Discussion

While our approach leverages the VAE as a primary solution for generating a manifold, it is important to acknowledge certain inherent limitations associated with this choice. One notable challenge is the susceptibility to issues such as posterior collapse and unstable sampling within the VAE framework. These challenges can result in generated group dance motions that may not consistently meet performance expectations.

One specific manifestation of this limitation is the potential for false decoding when sampling points that lie too far from the learned distribution. This scenario can lead to unexpected rotations or disruptions in the physics of the generated content. The impact of this problem becomes evident in instances where the generated samples deviate significantly from the anticipated distribution, introducing inaccuracies and distortions.

To address these challenges, we recognize the need for ongoing efforts to mitigate the effects of posterior collapse and unstable sampling. While the problem is acknowledged, our approach incorporates measures to limit its impact. Future research directions could explore alternative generative models or additional techniques to enhance the robustness and reliability of the generated results in the face of these identified limitations. 