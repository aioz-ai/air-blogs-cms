---
last_modified_on: "2022-04-09"
title: Video Compression (Part 2)
description: Experimental Results of AI-Learning video compression method.
series_position: 17
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advantage", "guides: compression"]
---
In previous section, we have investigate how Real-time AI-based video compression can work. In this section, we will explore the effectiveness of mentioned methods for video compression in comparison with centralized P2P streaming.

## Overall Experimental Setup.
**Datasets.** The Vimeo-90k dataset, which was recently created for testing diverse video processing tasks such as video denoising and video super-resolution, was used to train the video compression framework. It comprises of 89,800 distinct clips with content that differs from one another.
We assess the algorithm on the UVG dataset and the HEVC Standard Test Sequences to report on its performance (Class B, Class C, Class D and Class E). These datasets provide a wide range of content and resolution, and they are commonly used to evaluate video compression performance.

**Implementation Details** We train four models with varying $\lambda  (\lambda = 256, 512, 1024, 2048)$. We utilize the Adam optimizer for each model, setting the starting learning rate to $0.0001, 0.9$, and $0.999$, respectively. When the loss is steady, the learning rate is divided by ten. The mini-batch size has been set at four. The training pictures have a resolution of $256 \times 256$ pixels. The motion estimation module is set up with the weights that have been pre-trained. The entire system is built on Tensorflow, and training the network with two Titan X GPUs takes roughly seven days.
## Experimental Results.
Both H.264 and H.265 are included in this section for comparison. In addition, for comparison, a learning-based video compression method is offered. We utilize the Wu' setting and FFmpeg in extremely fast mode to build the compressed frames using H.264 and H.265. The UVG dataset and the HEVC dataset have 12 and 10 GOP sizes, respectively. 

The experimental findings on the UVG dataset, the HEVC Class B and Class E datasets are shown in Fig.1. In the supplemental material, you'll find the findings for HEVC Class C and Class D. Our solution clearly surpasses previous video compression research by a wide margin. The technique gained roughly 0.6dB at the same Bpp level on the UVG dataset. It's worth noting that our technique only requires one preceding reference frame, whereas Wu's work requires two neighboring frames and uses bidirectional frame prediction. As a result, by combining temporal information from various reference frames, we can increase the compression performance of our framework even further.

When tested by PSNR and MS-SSIM, the framework outperforms the H.264 standard on the vast majority of datasets. Furthermore, when compared to H.265 in terms of MS-SSIM, the technique achieves equivalent or superior compression performance. MSE is used to assess the distortion component in the introduced loss function, as previously stated. Nonetheless, in terms of MS-SSIM, the approach can still deliver acceptable visual quality.

![Figure 1](https://vision.aioz.io/f/3f0a135b62cd48afbb69/?dl=1)
*<center>**Figure 1**: Comparsion between the mentioned method with different learning based video codec: H.264 and H.265 [(Source)](https://openaccess.thecvf.com/content_CVPR_2019/papers/Lu_DVC_An_End-To-End_Deep_Video_Compression_Framework_CVPR_2019_paper.pdf).</center>*

## Ablation Analysis
**Motion Estimation.** In our provided method, we exploit the advantage of the end-to-end training strategy and optimize the motion estimation module within the whole network. As a result of rate-distortion optimization, our system's optical flow should be more compressible, resulting in more accurate warped frames. To show the efficacy, we do an experiment in which the initialization motion estimation module's parameters are fixed during the whole training stage. The motion estimation module is merely pre-trained in this case for more precise optical flow estimation, not for optimum rate-distortion. The experimental result in Fig. 2 reveals that when compared to the strategy of fixed motion estimation, which is designated by W/O Joint Training in Fig. 2, our approach with joint training for motion estimation may greatly increase performance (see the blue curve).

![Figure 2](https://vision.aioz.io/f/e56e8fbc6b03459c92a5/?dl=1)
*<center>**Figure 2**: Ablation study. We report the compression performance in the following settings. 1. The strategy of buffering previous frame is not adopted(W/O update). 2. Motion compensation network is removed (W/O MC). 3. Motion estimation module is not jointly optimized (W/O Joint Training). 4. Motion compression network is removed (W/O MVC). 5. Without relying on motion information (W/O Motion Information) [(Source)](https://openaccess.thecvf.com/content_CVPR_2019/papers/Lu_DVC_An_End-To-End_Deep_Video_Compression_Framework_CVPR_2019_paper.pdf).</center>*

Tab.1 shows the average bit costs for encoding the optical flow as well as the warped frame's related PSNR. When the motion estimation module is fixed during the training stage, it requires 0.044bpp to encode the produced optical flow, resulting in a PSNR of 27.33db for the warped frame. In comparison, our suggested technique requires 0.029bpp to encode the optical flow, and the PSNR of the warped frame is greater (28.17dB). As a result, the joint learning technique not only reduces the amount of bits needed to encode motion, but also improves the quality of the warped image. These results suggest that including motion estimation with rate-distortion optimization enhances compression performance.
![Tab 1](https://vision.aioz.io/f/db3fddfe636b45aa9155/?dl=1)
*<center>**Table 1**: The bit cost for encoding optical flow and the corresponding PSNR of warped frame from optical flow for different setting are provided [(Source)](https://openaccess.thecvf.com/content_CVPR_2019/papers/Lu_DVC_An_End-To-End_Deep_Video_Compression_Framework_CVPR_2019_paper.pdf).</center>*

**Motion Compensation.** Based on the predicted optical flow, the motion compensation network is used to improve the warped frame. We do another experiment to see how successful this module is by deleting the motion correction network from the proposed system. The PSNR without the motion compensation network drops by around 1.0 dB at the same bpp level, according to experimental results of the alternative strategy represented by W/O MC (see the green curve in Fig. 2).

**MV Encoder and Decoder Network.** The developed CNN model compresses the optical flow and encodes the motion representations that go with it. It is also possible to directly quantize and encode the raw optical flow measurements without the use of a CNN. By deleting the MV encoder and decoder network, we conduct a new experiment. The PSNR of the alternative technique marked by W/O MVC (see the magenta curve) drops by more than 2 dB after deleting the motion compression network, as seen in Fig. 2. Table 1 also includes the bit cost for encoding the optical flow in this condition, as well as the associated PSNR of the warped frame (denoted by W/O MVC). Directly encoding raw optical flow measurements needs significantly more bits (0.20Bpp), and the resulting PSNR (24.43dB) is significantly worse than our proposed solution (28.17dB). When optical flow is utilized to estimate motion, compression of motion is critical.

**Bit Rate Analysis.** To estimate the bit rate for encoding motion information and residual information, we employ a probability estimation network. In Fig. 3, we use arithmetic coding to compare the estimated bit rate with the actual bit rate to ensure dependability (a). The projected bit rate is obviously close to the real bit rate. In addition, we look into the many aspects of bit rate. We show the $\lambda$ value and the proportion of motion information at each position in Fig.3(b). When $\lambda$ in the goal function $\lambda \times D+R$ increases, the whole Bpp increases as well, while the proportion of motion information decreases.

![Figure 3](https://vision.aioz.io/f/41b771c21baf4add9710/?dl=1)
*<center>**Figure 3**: BItrate Analysis. (a) Actual and estimated bit ratte. (b) Motion information percentages [(Source)](https://openaccess.thecvf.com/content_CVPR_2019/papers/Lu_DVC_An_End-To-End_Deep_Video_Compression_Framework_CVPR_2019_paper.pdf).</center>*

## Conclusion Analysis
The system inherits the benefits of both existing video compression standards' classic predictive coding technique and DNNs' superior non-linear representation capabilities. The technique surpasses both the widely used H.264 video compression standard and a newer learning-based video compression system in tests. The research establishes a potential paradigm for using deep neural networks to compress video.