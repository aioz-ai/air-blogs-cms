---
last_modified_on: "2024-09-01"
id: Style4
title: Style Transfer for 2D Talking Head Generation (Part 4)
description: Effectiveness and Efficiency of Style Transfer 2D Talking Head Generation
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

In the previous series, we have presented a comprehensive algorithmic overview of our method, which is key to achieve personalized talking head animation creation. Now, we would like to demonstrate our experimental procedure to validate the efficacy of the proposed system.


![image](https://vision.aioz.io/f/c1a222c66dec4a26b3dc/?dl=1)



## Experimental setups

### Datasets
**Face.** We use the VoxCeleb2 dataset to learn facial expressions. All videos are extracted at 60 FPS. We first trim the video to retain the face in the center, then resize it to $512\times 512$. Our internal face tracker is leveraged to obtain 68 key points on the face. Face segmentation model is used to obtain the skin mask. The head and torso motion is manually identified for the first frame of each series and tracked for the remaining frames using optical flow. 

**Audio.** We use the Common Voice dataset to train the Audio Encoder. There are around 26 hours of unlabeled statements throughout all samples. $80$-dimensional log Mel spectrograms are employed as surface representation and are computed with $\frac{1}{120}$(s) frame-shift, $\frac{1}{60}$(s) frame length, and $512$-point STFT representation. 

We evaluate and benchmark our results in the RAVDESS dataset. RAVDESS is a validated multimodal database of emotional speech and song, which is suitable and challenging to validate our method and different baselines. Note that, we  use this dataset for benchmarking only to avoid training bias.

### Implementation

We implement our framework using PyTorch. We train the network on the NVIDIA Titan V100 GPU with Adam optimizer. The learning rate is set to $10^{-4}$, $10^{-4}$, $10^{-5}$, $10^{-4}$ to train the Audio Encoding, the Motion Generator, the Style-Aware Generator, and the Style Mapping, respectively. The batch size is set to $8$ for the Style-Aware Generator and $64$ for other modules.

## Qualitative Evaluation


![fig3](https://vision.aioz.io/f/f8e8621638384a0fbd7e/?dl=1)*<center>**Figure 1**. Our 2D photo-realistic talking head results with different styles. (a), (c), (e) are ballad, rap, and opera style references, respectively; (b), (d), (f) are the corresponding style transfer results.</center>*


Figure 1 shows that our method successfully transfers different styles such as *ballad*, *rap*, or *opera* to a new target character. Figure 2(a) shows the comparison between our method and recent works on 2D photorealistic talking head animation (MakeItTalk, LSP, ...) when the character sings an opera song. Focusing on the mouth, we notice that our method produces better results in mouth motion variance and eye expression compared to the results from MakeItTalk and LSP. In Figure 2(b), we show the comparison between different styles when they are encoded in one input audio to generate talking heads. Note that, in this case, different input images are used to verify the synthesis effectiveness of our method. Although different styles are encoded into different images to generate different talking heads, the animation is realistic and the performance of lip-synchronization is well-reserved.

![fig3](https://vision.aioz.io/f/6fe5e8fdeb05497184b1/?dl=1)*<center>**Figure 2**. Comparison between different 2D talking head galleries on multiple styles. Our method generates more natural and realistic motion, especially around the mouth and the eye of the character.</center>*


## Quantitative Evaluation
### Evaluation metrics
We use six different metrics to evaluate how good and natural the animation of the generated talking head is: Cumulative Probability Blur Detection (CPBD), Landmark Distance (D-L), Landmarks Distance around the Mouth (LMD), Landmark Velocity difference (D-V), Difference in the open mouth area (D-A).

**Style transfer metric.** To evaluate style transfer results efficiently, we introduce three new following metrics. 

*Style-Aware Landmarks Distance* (SLD): To evaluate the style information encoded in a generated talking head, we design a metric called Style-Aware Landmarks Distance (SLD). This metric calculates the accuracy of mouth, eyes, head pose shapes between a chunked window of style reference and a chunked window of corresponding talking head animation. Lower is better. 

Let's assumed that a style reference video with $N_s$ frames is split into multiple temporal periods of $\rm F$ frames (window size), i.e., style reference windows $\rm W_s = \left(w_s^{(0:F)}, w_s^{(v:F+v)}, w_s^{(2v:F+2v)},\cdots, w_s^{(\kappa v:F +\kappa v)}\right)$, with $\rm w_s^{(i:F+i)}$ being the frames from $\rm i$*-th* to $\rm (F+i)$*-th* of the reference video, $\rm v$ is the stride, and $\rm \kappa = \lfloor(N_s - F) / v\rfloor$.

Similar to the reference video, we chunk the generated animation video into smaller chunked windows $\rm W_a = \Bigl(w_a^{(0:F)}, w_a^{(v:F+v)}, w_a^{(2v:F+2v)}, \cdots , w_a^{(\kappa v:F+ \kappa v)}\Bigr)$. The SLD is then calculated with the core is the D-L metric as:
$$
    \rm{SLD} = \frac{1}{\rm \vert W_s\vert}\sum_{\rm w_s \in \rm W_s} \left(\underset{\rm w_a \in \rm W_a}{\rm{min}} \left(\rm{D\rm{-}L}\left(\rm w_s, \rm w_a\right)\right)\right) 
$$
where $\rm{D\rm{-}L}$ is the Landmark Distance metric. 

Similarly, we calculate the *Style-Aware Landmarks Velocity Difference* (SLV) for landmark velocity, and *Style-Aware Mouth Area Difference* (SMD) for open mouth area accuracy. 


### Talking Head Generation Results

![fig3](https://vision.aioz.io/f/ca6d9bdbbfac4ead95a0/?dl=1)*<center>**Table 1**. Result of different 2D talking head generation methods.</center>*


Table 1 shows the 2D talking head result comparison between our method and recent baselines. From Table 1, we can see that our method outperforms recent state-of-the-art approaches by a large margin. In particular, our method achieves the highest accuracy in CPBD, LMD, D-L, D-V, and D-A metrics. These results show that our method successfully renders the 2D talking head and increases the quality of the rendered results. Overall, our method can increase the sharpness of the head (identified by CPBD) metric, while generating natural facial motion (identified by LMD, D-L, D-V, and D-A metric). 


### Style Comparison


![fig3](https://vision.aioz.io/f/a310108b815c4d6f89cd/?dl=1)*<center>**Table 2**. Result comparison in terms of style transfer between different 2D talking head generation methods.</center>*

Table 2 shows the comparison between our method and other baselines in terms of style transfer. Three designed metrics (SLD, SLV, and SMD) are used for evaluation and benchmarking. The results show that our method outperforms others by a large margin in all three metrics. This substantial performance gap strongly suggests the efficacy of our method in accurately capturing style characteristics from the reference image and seamlessly transferring them onto the target image. This not only highlights the robustness of our method but also underscores its potential for practical applications in various domains requiring high-quality style transfer. Furthermore, these results underscore the importance of our approach in advancing the state-of-the-art in style transfer techniques, promising richer and more faithful artistic transformations.


## Conclusion

In this work, we have introduced a novel method designed to generate lifelike 2D talking heads from input audio signals, revolutionizing the realm of character animation. In addition to the primary audio stream and an accompanying image, our framework harnesses a meticulously curated set of reference frames to effectively learn the character style characteristics. Notably, our approach excels even with the most demanding and challenging vocal styles, including *ballad*, *opera*, and *rap*, where complex movements are necessary to produce animations that are faithful and natural. Extensive experiments demonstrate the superior performance of our talking head synthesis, showing qualitative and quantitative advantages over recent state-of-the-art methods. The versatility of our framework can be potential for diverse applications, ranging from dubbing, video conferencing experiences, to the creation of dynamic virtual avatars. With the ability to accurately capture and animate diverse head movement styles, we hope to further advance the field of character animation, allowing more expressive and vivid human-like facial talking animation.