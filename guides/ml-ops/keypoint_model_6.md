---
last_modified_on: "2022-09-11"
title: Model Compression Part 3.
description: Explore techniques used in a specific task, the Deep Keypoint Detection for Motion Driving task.
series_position: 6
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: advance"]
---
In this post, we will explore the deep keypoint detection task, its architecture and then analyse how to lightweigh it.

## Model goal and output 

Within the First-Order Motion model, this keypoint detector network is part of the motion estimation module, where keypoints and local affine transformations predicted by this network will be used to model motion from one frame to another. The model does not require any keypoint annotations during training and is trained as part of the whole system. The keypoints learned are not specific to any structure and are often associated with highly moving parts. 

In this report, we will explore the details of the keypoint detector network, its overall structure, each specific modules and layers.   


## Overall analysis

The model input is an RGB image of size (256,256). The image is then passed through four large modules:
1) **anti-alias interpolation** to reduce the spatial dimension down to (64,64), 
2) **hourglass module** to extract feature maps and reconstruct to,
3) **post-process module** where the reconstructed feature maps will be converted into normalised heatmaps and in turn keypoints, and  
4) **jacobian module** where the keypoints will be used to learn the local affine transformation, i.e. focus prediction near the keypoints.

Below is a brief summary of running time of each module. 

| video_id | anti_alias | hourglass | heatmap_keypoint | jacobian | total_infer_time |
|----------|------------|-----------|------------------|----------|------------------|
| id10282  | 1.3195     | 24.1060   | 0.9515           | 1.6774   | 28.2299          |
| id10286  | 2.3068     | 28.2253   | 1.7142           | 3.0940   | 35.5235          |
| id10291  | 2.3897     | 28.3323   | 1.7671           | 3.1878   | 35.8659          |
| id10281  | 2.5809     | 30.1129   | 1.9414           | 3.4794   | 38.3216          |
| id10283  | 2.7626     | 30.6548   | 2.0634           | 3.7266   | 39.4218          |
| id10280  | 2.8620     | 30.8420   | 2.1227           | 3.8315   | 39.8855          |


## Hourglass module 

A standard U-Net was used for extracting features from the original image and reconstruct before converting to heatmaps. 5 downblocks make up the Encoder and 5 upblocks make up the Decoder. With feature maps from the encoder being concatenated to the corresponding feature maps in the decoder using residual connections. 

|                 | input_shape | output_shape | # params  | 
|-----------------|-------------|--------------|-----------|
| down_sampling_1 | (3,64,64)   | (64,32,32)   | 1,920     |
| down_sampling_2 | (64,32,32)  | (128,16,16)  | 74,112    |
| down_sampling_3 | (128,16,16) | (256,8,8)    | 295,680   |
| down_sampling_4 | (256,8,8)   | (512,4,4)    | 1,181,184 |
| down_sampling_5 | (512,4,4)   | (1024,2,2)   | 4,721,664 |
| upsampling_1    | (1024,2,2)  | (512,4,4)    | 4,720,128 |
| upsampling_2    | (1024,4,4)  | (256,8,8)    | 2,360,064 | 
| upsampling_3    | (512,8,8)   | (128,16,16)  | 590,208   |
| upsampling_4    | (256,16,16) | (64,32,32)   | 147,648   |
| upsampling_5    | (128,32,32) | (32,64,64)   | 36,960    |
| hour_glass      | (3,64,64)   | (35,64,64)   | 14.130M   |


### Encoder 
Made up of 5 downblocks. Each downblock is a sequence of Conv2d -> BatchNorm -> Relu -> AvgPool2d. The number of channels doubles and the spatial dimensions reduce by a factor of two after every block.

> "the first convolution layers have 32 filters and each subsequent convolution doubles the number of filters".

| layer      | downblock_1 | downblock_2 | downblock_3 | downblock_4 | downblock_5 | average  |
|------------|-------------|-------------|-------------|-------------|-------------|----------|
| conv       | 0.306828    | 0.617031    | 0.900335    | 1.761473    | 6.210853    | 1.959304 |
| batchnorm  | 0.162862    | 0.097506    | 0.055971    | 0.063747    | 0.083998    | 0.092817 |
| relu       | 0.15854     | 0.075486    | 0.037425    | 0.03163     | 0.031038    | 0.066824 |
| avgpool    | 0.06177     | 0.037166    | 0.026693    | 0.02421     | 0.027748    | 0.035517 |


The majority of a forward pass in the encoder is spent on the convolutional layers, especially the ones in the final downblock, making the convolutional layers a candidate to be replaced for better efficiency. 

### Decoder 
Made up of 5 upblocks. Each upblock is a sequence of Interpolate -> Conv2d -> BatchNorm -> ReLU. The first upblock's input is the output of the encoder. Every other upblock's input is a concatenation of the previous upblock's output and the corresponding downblock output. 

Note: all measurements reported are in ms.


| layer         | upblock_1 | upblock_2 | upblock_3 | upblock_4 | upblock_5 | average  |
|---------------|-----------|-----------|-----------|-----------|-----------|----------|
| interpolation | 0.034889  | 0.048182  | 0.069889  | 0.112527  | 0.192412  | 0.091580 |
| conv          | 5.479779  | 5.13461   | 1.754365  | 1.080764  | 0.917713  | 2.873446 |
| batchnorm     | 0.083391  | 0.086822  | 0.057727  | 0.065382  | 0.07786   | 0.074236 |
| relu          | 0.028462  | 0.034385  | 0.032103  | 0.043066  | 0.061287  | 0.039861 |


## Post-procesing (Heatmaps & Keypoints)
After the Hourglass module, there are two more steps to output keypoints:
1. generating K normalised heatmaps for K keypoints, and 
2. converting those heatmaps to corresponding (x,y) coordinates. 

We wish to get the coordinates of the keypoints from the heatmaps. In multiple landmark localisation works, the coordinates were usually extracted by applying argmax, i.e. taking the brightest point on the heatmap to be the keypoint. However, this process is non-differentiable and may lead to [quantization error](https://arxiv.org/pdf/1711.08229.pdf). Therefore, this paper employed the differentiable variant of argmax, or `soft-argmax`.

### Heatmaps
The goal of this sub-module is to create normalised heatmaps from the feature maps generated by the Hourglass module, i.e. every element ranges from 0 to 1 and sum of all elements add up to 1. First, the output of size (35,64,64) from the hourglass module goes through another convolutional layer to finally generate 10 heatmaps of size (58,58), for 10 keypoints. As the output of the decoder is a concatenation of the input image and earlier decoder output, this convolution layer not only reduces the channel down to the number of keypoints, but it also acts as the last learning bit, with a larger kernel size for larger receptive field.

```{python}
# a convolution layer to bring 35 channels down to 10
self.kp = nn.Conv2d(in_channels=self.predictor.out_filters, out_channels=num_kp, kernel_size=(7, 7), padding=pad)

prediction = self.kp(feature_map)

# normalised the heatmaps by applying softmax
final_shape = prediction.shape

# reshape to (10, 3364) to perform softmax 
heatmap = prediction.view(final_shape[0], final_shape[1], -1)
heatmap = F.softmax(heatmap / self.temperature, dim=2)

# bring back to (10, 58, 58)
heatmap = heatmap.view(*final_shape)
```

Then, the heatmaps will be normalised using `softmax`. This part does not have any trainable parameters. But there is one hyperparameter, `temperature` for determining the smoothness of the distribution after applying softmax. If set `temperature > 1`, will make the softmax distribution smoother. In this model, temperature was set to 0.1. Quoted by the author 
> Thanks to the use of a low temperature for softmax, we
obtain sharper heatmaps and avoid uniform heatmaps that
would lead to keypoints constantly located in the image center.

![heatmaps_bnw](https://drive.google.com/uc?id=1BGsIHmEa1adcxXK9aXooo7lmLV3kj5-K)
*<center>**Figure 1**: 10 normalised heatmaps for 10 keypoints.</center>*


### Gaussian2KP

This part is to implement soft-argmax, a variant of argmax that is differentiable. Its purpose is to convert a heatmap to a (x,y) coordinate. The process is purely computatonal and there is no learning here.

One clear advantage of soft-argmax is that it is differentiable, thus allows end-to-end training. It also alleviates the problems of [quantization error](https://arxiv.org/abs/1710.02322) due to the mismatch in size between the input image and the heatmaps. Efficiency-wise, it does not add extra parameters and computationally negligible. 


### Jacobian

To focus on the movements surrounding keypoints, Jacobian matrices are computed to represent the scale and rotation of each keypoint. 

## Keypoint Results 

![keypoints](https://drive.google.com/uc?id=1-8d5Mn3e3uRDvblzXHjE1vfTcv0diJCs)

![keypoints-vox](https://drive.google.com/uc?id=1Q59liQysvCrcGJdUfOvVwVAxPWhhIP2e)
*<center>**Figure 2**: Visualised keypoints.</center>*


It appears that the keypoints are semantically consistent across the images, such as the bright green keypoint is at between the eyes, the blue one is at the neck, and the dark blue point is at the left eyebrow, etc. 