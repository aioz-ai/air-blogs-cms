---
last_modified_on: "2022-10-16"
title: Model Compression Part 8 (Lightweight variations of Squeeze U-Net).
description: Systematic and Efficient Hyperparameter Tuning.
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: medium"]
---
## Searching for a smaller model with a quick inference time
As the current Squeeze U-Net performs surprisingly well on the distillation task, we are searching for a smaller model that can maintain the performance with a quicker inference time. Given that our goal is to deploy the model on mobile devices, we target models with fast CPU runtimes. In this article, we will explore some experiments we have conducted to shorten further Squeeze U-Net runtime, our reasoning for each decision, whether it was successful, and other models designed for CPU runtime.  

Note: Experiments were done on CPU Intel Xeon(R) CPU E5-2640 v3 @ 2.60GHz.

## Lightweight variations of Squeeze U-Net  
  

### Depthwise Separable Convolutions   

In previous articles, we have discussed the potential of Depthwise Separable Convolutions (DepSepConv). We considered replacing all non-pointwise convolutions in the current Squeeze U-Net with DepSepConv due to its advantages in terms of parameter count and computation costs. As expected, the parameter count and computation cost were greatly reduced. However, our new runtime is 0.0119s compared to 0.0142s, not exactly a big improvement. 

One of the reasons for this modest reduction is that the runtime gain with DepSepConv decreases proportionally with spatial resolution. Given a convolution operation where the input shape is (64, S, S) and output shape is (128, S, S), where S is the spatial resolution variable of interest, and as we experimented with different values of S (range [4, 8, 16, 32, 64]), we found that as the spatial resolution decreased to 4, the runtime of regular conv and DepSepConv became very similar, unless C_in and C_out are large. Therefore, when integrate with an already optimised network such as Squeeze U-Net, DepSepConv does not contribute substantial reduction. 

![fire_module](https://drive.google.com/uc?export=view&id=1XkWh8enlZcLUyagv9SYX8IhCy7bhvftH)
*<center>**Figure 1**:  Standard 3x3 conv and DepSepConv runtime by spatial resolution.</center>*

Furthermore, the authors of SqueezeNext has pointed out that DepSepConv is inefficient in terms of hardware (low arithmetic intensity) and may not have a good performance on some embedded device [8]. 

Therefore, we did not integrate DepSepConv into the keypoint detection model.   

### Remove residual connections   
Another attempt to reduce Squeeze U-Net runtime is to remove the residual connections between encoder and decoder. This brought the runtime down to 0.0122s (reported in Table 3), which is not satisfactory. For further reduction, we replaced the Transpose Fire module to another variation proposed in SqueezeSeg [3]. The current Transposed Fire module in Squeeze U-Net takes an input and performs `ConvTransposed2d` before sending it to a Fire Module. In the new variation, we perform ConvTransposed between the `squeeze` and `expand` operations. This reduces the input's channels before putting it to ConvTransposed, which reduces the number of parameters considerably.   
 
As can be seen in Table 3, the number of parameters, computations and pass size all greatly reduced with this new Transposed Fire module variant. However, the encoder-decoder architecture predictably falls short of the original Squeeze U-Net in terms of distillation performance. As shown in Table 4, Squeeze Encoder Decoder clearly underfits, which emphasizes the necessity of residual connections.  

*<center> **Table 3.** Models summary and runtime, tested on input of size (3, 64, 64).  </center>*

| Model                            | # parameters | Mult-adds (M) | Pass size (MB) | CPU Runtime (s) |  
|----------------------------------|--------------|---------------|----------------|-----------------|  
| Squeeze U-Net                    | 2,507,035    | 191.55        | 5.66           | 0.0142          |   
| Squeeze U-Net with dep_sep_conv  | 540,499      | 40.24         | 4.13           | 0.0119          | 
| Squeeze Encoder Decoder          | 781,375      | 26.01         | 2.78           | 0.0102          | 
| L3U-Net                          | 171,065      | 62.92         | 6.75           | 0.0079          |  
| EdgeSegNet                       | 651,617      | 125.37        | 3.46           | 0.0066          |  


*<center> **Table 4.**   Distillation results. </center>*
 
| Model                   | Train Loss | Val Loss | Val Distance |  
|-------------------------|------------|----------|--------------|  
| Squeeze U-Net           | 0.0025     | 0.0054   | 0.0094       |
| Squeeze Encoder Decoder | 0.0309     | 0.0343   | 0.0575       | 
| L3U-Net                 | 0.0080     | 0.0084   | 0.0131       |
| EdgeSegNet              | 0.0016     | 0.0039   | 0.0077       |  


## L^3U-Net  
We decided to look for other specifically built low-latency lightweight models that 
can operate in real-time on edge devices with little resources. One of them is L3U-Net [4]. The main contribution of L3U-Net is the data folding technique - at its core, a reshaping operation - that is when followed by a convolution operation, it is equivalent to a strided convolution on the original tensor.   
  
Their argument, which is supported by [5], is that the early conv layers take up the majority of runtime due to their low number of channels and high spatial resolution. Data folding allows us to downsample the image and at the same time increases the number of channels, reducing the imbalanced memory distribution and making it suitable for parallel processing.
  
> These initial layers induce a significant proportion of the entire network’s latency since only a small number of cores can be used to process the small number of input channels that are big in spatial size.  [4]
  
![data-folding](https://drive.google.com/uc?export=view&id=14elbmRU0QX3Fbf49vZPK4LHlOiDJrYD9)  
  *<center>**Figure 2**:  Data folding demonstration. [4]</center>*
  
By utilising this method with the L3U-Net architecture as the keypoint model, we were able to cut the number of parameters by a factor of 14 and the runtime virtually in half.
  
In terms of accuracy, L3U-Net displayed encouraging results (a validation loss of 0.0084 compared to 0.0054 achieved by Squeeze U-Net), and indications of improved performance with additional tunings. As a result, we came to the following conclusions: 
1. The keypoint detection task at hand might not require as complex a model as we had initially believed. 
2. Residual connections are very important.
  
  
## EdgeSegNet  
  
In our search for lightweight models catered for TinyML, we also came across EdgeSegNet, a result of "a human-machine collaborative design strategy, where human-driven principled network design prototyping is coupled with machine-driven design exploration" [6]. In other words, with defined design principles and runtime & hardware constraints, an architecture search algorithm was set out to find the optimal model design tailored for semantic segmentation. 

![EdgeSegNet](https://drive.google.com/uc?export=view&id=1QMd3-iAf73nrX8RdZlLFb5W3wM2yjKCa)
*<center>**Figure 3**:  EdgeSegNet architecture. [6]</center>*

EdgeSegNet is unique in a way that it was built under runtime and performance restrictions. It also abandons the U-Net architecture. What we have now is one main path, and a mix of long and short-range connections added to that path. The Refine Module was adapted from RefineNet [7], where it uses feature maps with lower resolutions to further refine the details of intermediate feature maps. Additionally, EdgeSegNet heavily uses the Bottleneck Modules to reduce the number of input channels before passing it to regular 3x3 conv. 

With EdgeSegNet, while the reduction in parameter count and computation cost is not as much as L3U-Net, the forward/backward pass size is half that of L3UNet, and we were able to reach the shortest runtime of 0.0077s. What's more astonishing is that with only 1/4th as many parameters, EdgeSegNet was able to obtain a superior distillation result than the original Squeeze U-Net (lower train_loss, val_loss, and val_distance). This shows that indeed, the keypoint detection problem is not as complex as expected and that we have not explored the full capacity of Squeeze U-Net. 
  
## Reference   
[1] N. Beheshti and L. Johnsson, "Squeeze U-Net: A Memory and Energy Efficient Image Segmentation Network", _2020 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW)_, 2020. Available: 10.1109/cvprw50498.2020.00190.    
[2] M. Hollemans, "How fast is my model?", _Machinethink.net_, 2018. [Online]. Available: https://machinethink.net/blog/how-fast-is-my-model/.     
[3] B. Wu, A. Wan, X. Yue and K. Keutzer, "SqueezeSeg: Convolutional Neural Nets with Recurrent CRF for Real-Time Road-Object Segmentation from 3D LiDAR Point Cloud", _2018 IEEE International Conference on Robotics and Automation (ICRA)_, 2018. Available: 10.1109/icra.2018.8462926.    
[4] O. Erman, M. Ulkar and G. Uyanik, "L^3U-net: Low-Latency Lightweight U-net Based Image Segmentation Model for Parallel CNN Processors", _ArVix_, vol. 220316528, 2022. Available: https://arxiv.org/pdf/2203.16528.pdf.     
[5] J. Lin, W. Chen, H. Cai, C. Gan and S. Han, "MCUNetV2: Memory-Efficient Patch-based Inference for Tiny Deep Learning", _NeurIPS_, 2021.     
[6] Z. Lin, B. Chwyl and A. Wong, "EdgeSegNet: A Compact Network for Semantic Segmentation", _ArVix_, 2019. Available: https://arxiv.org/abs/1905.04222.
[7] G. Lin, F. Liu, A. Milan, C. Shen and I. Reid, "RefineNet: Multi-Path Refinement Networks for Dense Prediction", _Computer Vision and Pattern Recognition Conference (CVPR)_, 2017. Available: 10.1109/tpami.2019.2893630.
[8] A. Gholami et al., "SqueezeNext: Hardware-Aware Neural Network Design", _2018 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW)_, 2018. Available: https://arxiv.org/abs/1803.10615. 