---
last_modified_on: "2023-04-22"
id: GDANCE3
title: Music-Driven Group Choreography (Part 3)
description: Experimental results part of group dance choreography
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---

This  is the final part of the series group dance choreography, In this part, we will provide detailed analyses of our proposed group dance generation method.

## Experiments

### AIOZ-GDANCE Statistics


![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOSz2wY04GpZUf2CnNEuTEe8yFI8dDplM9OBO6j31iqFBYtdQpytA1tmdte-Pmke25f2Lroboz669TWHCwAKmi48_wK0Wg=s2560)*<center>**Figure 1**.  Distribution (%) of music genres (a) and dance styles (b) in our dataset. </center>* 

In Figure 1, we show the distribution of music genres and dance styles in our dataset. As illustrated in Figure 1 (Left), *Pop* and *Electronic* are popular music genres while other music genres nearly share the same distribution. Meanwhile, on the right of Figure 1, *Zumba*, *Aerobic*, and *Commercial* are the dominant dance styles. 



![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOTu-Bm4xmt2Wdq4heuoG7PFsimYOGJAA53krq1_qvhDVJ4IEtgwnmZYNTUkTchdLck7HbdOL9HmvEON-JlmQwDH4tZQ=s2560)*<center>**Figure 2**. The correlation between dance styles and number of dancers (a); and between dance styles and music genres (b). </center>* 

Figure 2 (Left)  shows the number of dancers in each dance style. Naturally, we see that *Zumba*, *Aerobic*, and *Commercial* have more dancers. On the right of this figure, we illustrate the correlation between music genres and dance styles. 






### Evaluation Metrics
Similar to prior works on single-dance generation, we evaluate the generated motion quality by calculating the distribution distance between the generated and the ground-truth motions using Frechet Inception Distance (FID)[1, 2]. To evaluate how well the generated 3D motion correlates to the input music, we use the Motion-Music Consistency metric (MMC) [2,3]. We also evaluate our model's ability to generate diverse dance motions when given various input music by measuring Generation Diversity (GenDiv) [2,3]. 

To evaluate the group dancing quality, we propose three new metrics: Group Motion Realism (GMR), Group Motion Correlation (GMC), and Trajectory Intersection Frequency (TIF). Detailed calculations of these metrics are described as follows:

**Group Motion Realism (GMR).** To calculate the realism between generated and ground-truth group motion, we need to find a single unified representation for all dancers' motions in the scene. Based on the kinetic features of a single motion sequence [4], we propose to calculate Group Motion Realism (GMR), smaller is better. Specifically, for each entity, we compute the velocity of each element $j$ of the pose vector: $v^n_t = \frac{y^n_{t+1} - y^n_t}{\Delta t}$ where $\Delta t$ is the time period between two consecutive frames. Note that the pose vector of each entity at each frame consists of the root orientation,  root position and joint angles. The group kinetic features of a sequence is approximated by taking the logarithm of the total kinetic energy of all group entities as:
$$
 e_j = \log \left(1 + \frac{1}{T}\frac{1}{N} \sum_{t=1}^T \sum_{n=1}^N m_j (v^n_{t,j})^2\right)
$$
where $m_j$ is the moment of inertia or mass of each joint. We assume that $m_j$ is constant with respect to time and entity. Then, we split the sequence into smaller chunks and calculate the features of these chunks. This process is identical for both the generated and ground-truth sequences. Finally, we utilize these sets of features (from generated and ground-truth group dance) to calculate the GMR using the standard FID formulation as in [1]. 

**Group Motion Correlation (GMC).** We also evaluate the synchrony and the correlation between dancers within the generated group. We assume that the correlation of movements between individuals is likely to reflect their interaction in the choreography. For every pair of motions within a group, we first align the two motion sequences using Dynamic Time Warping algorithm based on the Euclidean distance in the joint position space (obtained by SMPL joint regressor). We then calculate the mean cross-correlations between the time-aligned motion pairs using the kinetic features [4]. The generated group motion correlation degree is then calculated as the average of all motion pairs.

**Trajectory Intersection Frequency (TIF).** For the generated group sequences, the intersection rate is calculated over all $F$ frames as: 
$$
\text{TIR} = \frac{\sum_{F}\sum_{i,j : i\neq j} \mathbb{I}[\text{intersect}(M(y^i),M(y^j))]}{F},
$$
where $M$ is the SMPL skinning function [5] which can output a 6890-vertices human mesh from the input pose parameters $y$. $\text{intersect}(x,y)$ is a function that returns 1 if the two meshes are intersect with each other and 0 otherwise.  For TIF, smaller value is better and indicates less intersection of the generated group.


### Cross-entity Attention Analysis
We compare our method with FACT [2]. FACT is a recent state-of-the-art method designed for single dance generation, thus giving our method an advantage. However, it is still the closest competing method as we propose a new group dance dataset that is not available for benchmarking before. We also analyse our method with and without using Cross-entity Attention. We train all methods with mini-batch containing all dancers within the group instead of sampling each dancer independently as in FACT’s original implementation.



![](https://lh3.googleusercontent.com/drive-viewer/AAOQEORj7YkRgKsbXa3JZK40qBYI72X0208dYfLtE5GdyN_6xg2aZObHROTj5-XoAQZ9p72TcYX0r8EXx5lWjizaS5K--8F7Mw=s2560)*<center>**Figure 3**. Comparison between FACT and our GDanceR. Our method handles better the consistency and cross-body intersection problem between dancers. </center>* 


Table 1 shows the method comparison between the baseline FACT[2] and our proposed GDanceR with and without Cross-entity Attention. The results show that GDanceR, especially with the Cross-entity Attention, outperforms the baseline by a large margin in all metrics. In Figure 3, we also visualize the example outputs of FACT and GDanceR. It is clear that FACT does not handle well the intersection problem. This is understandable as FACT is not designed for group dance generation, while our method with the Cross-entity Attention can deal with this problem better.

![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOR57cCAq6_1hXYIcR5WNvIHTn0gchGT-TGcgIXIQ2rqkAvpJhVJiO46cfAslwyqxM0EEQ4Jjz3xeQvUkyXO6tkc1-TrYw=s2560)*<center>**Table 1**. Generation results comparison on AIOZ-GDANCE dataset. w/o CA denotes without using Cross-entity Attention. </center>* 


### Number of Dancers Analysis.


Table 2 demonstrates the generation results of our method when we want to generate different numbers of dancers. In general, the FID, GMR, and GMC metrics do not show much correlation with the numbers of generated dancers since the results are varied. On the other hand, MMC shows its stability among all setups ($\sim 0.248$), which indicates that our network is robust in generating motion from given music regardless of the changing of initial positions. The generation diversity (GenDiv) decreases while the intersection frequency (TIF) increases when more dancers are generated. These results show that dealing with the collision during the group generation process is worth further investigation.


![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOTLDBo-encLsP3vtfNCg0-cVoRuUIvd4OBzafY3aBfARgMyQdgiuYEMdcuEDFLuEDPtFLRfmSBx2r_E2Ong-MiXHAxZ=s2560)*<center>**Table 2**. Performance of our proposed method when increasing the number of generated dancers.</center>* 

### Dance Style Analysis

![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOSYIpB7RV2enIN75jpGo61_iDRfWRRP6-hYimsOu1OEypQAmoxUreMAgf9AH1Yh27fgH7mCItTbVSHvS8zBO_ZL7fix8A=s2560)*<center>**Figure 4**. Examples of generated group motions from our method. </center>* 

Different dance styles exhibit different challenges in group dance generation. As shown in Table 3, *Aerobic* and *Zumba* are quite similar for generating choreography as they usually focus on workout and sporty movements. Besides, while *Commercial* and *Irish* are easier for the model to reproduce the motions, *Bollywood* and *Samba* contain highly skilled movements that are challenging to capture and represent accurately. In Figure 4, we show the generated results of GDanceR with different dance styles. Our Supplementary Material and Demonstration Video also provide more examples.

![](https://lh3.googleusercontent.com/drive-viewer/AAOQEOQg9xefLINLqYXJIy68wCcOyqI7hvjiIhBE72S1qIL9zmqNfD03kJCgunBSbkdAPE8PjqKacKywIrj5y19YYLzrt28Suw=s2560)*<center>**Table 3**. The results of different dance styles. These results are obtained by training the model on each dance style.</center>* 


### Ablation on Latent Motion Fusion.
We investigate different fusion strategies between the local motion $h^i$ and global-aware motion $g^i$ to obtain the final motion representation $z^i$. Specifically, we experiment with three settings: *(i)* No Fusion: the final motion is the global-aware motion obtained from our Cross-entity Attention ($z^i = g^i$); (ii) Concatenate: the final motion is the concatenation of the local and global-aware motion ($z^i = [h^i; g^i]$); *(iii)* Add: the final motion is the addition between local and global ($z^i = h^i + g^i$). Table 4 summarizes the results. We find that fusing the motion by adding both the local and global motion features achieves the best results. In this strategy, the global information between entities is encoded to the local motion in an effective way so that the final motion retain the comprehensive information of their own past motion as well as the motion of every other entity. While in the concatenation, the model is prone to overfitting due to the redundant information of both the local and global representation. On the other hand, No Fusion can degrade the amount of information of the past motion, leading to insufficient input information and the Decoder may fail to generate the temporally-coherent motion aligned with the music. 


![](https://lh3.googleusercontent.com/drive-viewer/AAOQEORBdzuotmGUAOir7hjY1k0AK6CJy5xPSNOd0IEvV0ZpQHalNcfPMAyHC97r1qGN5dD1Z2pP9ZkWsDJIMMLHl_wn9IWk=s2560)*<center>**Table 4**. Ablation study on different fusion strategies for the latent motion representation. </center>* 

## Conclusion

In summary, we have introduced AIOZ-GDANCE, the largest dataset for audio-driven group dance generation. Our dataset contains in-the-wild videos and covers different dance styles and music genres. We then propose a strong baseline along with new evaluation metrics for group dance generation task. We also perform extensive experiments to validate our method on this interesting yet unexplored problem, using our new dataset and evaluation protocols. We hope that the release of our dataset will foster more research on audio-driven group choreography.

## References


[1] Martin Heusel, Hubert Ramsauer, Thomas Unterthiner, Bernhard Nessler, and Sepp Hochreiter. Gans trained by a two time-scale update rule converge to a local nash equilibrium. NIPS 2017.


[2] Ruilong Li, Shan Yang, David A Ross, and Angjoo Kanazawa. Ai choreographer: Music-conditioned 3d dance generation with aist++. ICCV 2021

[3] Hsin-Ying Lee, Xiaodong Yang, Ming-Yu Liu, Ting-Chun Wang, Yu-Ding Lu, Ming-Hsuan Yang, and Jan Kautz. Dancing to music. NIPS 2019.

[4]  Kensuke Onuma, Christos Faloutsos, and Jessica K Hodgins. Fmdistance: A fast and effective distance function for motion capture data. Eurographics 2008.

[5] Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, and Michael J. Black. SMPL: A skinned multi-person linear model. ACM Trans. Graphics, 2015