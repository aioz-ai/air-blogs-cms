
---
last_modified_on: "2022-08-27"
title: Pytorch Mobile
description: Library to support deployment work on IOS and android mobile device
series_position: 3
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

Nowadays, ML systems are in high demand for implementation on mobile devices. Similar to tensorflow lite, Pytorch has also developed management capabilities that provide an end-to-end workflow that simplifies research to production environments for mobile devices. In addition, it paves the way for privacy-preserving features through federated learning techniques.

PyTorch Mobile is in beta stage right now, and is already in wide scale production use. It will soon be available as a stable release once the APIs are locked down.


## **DEPLOYMENT WORKFLOW**
A typical workflow from training to mobile deployment with the optional model optimization steps is outlined in the following figure.

![Figure 1](https://drive.google.com/uc?export=view&id=1K7TUlyiOiEEFq8DWfEbAxcbOjdHDDKDC)*<center>**Figure 1**. Workflow Pytorch Mobile.</center>*

## **iOS**

To avoid lengthy, let's go to the step of converting from pytorch to mobile with the format for ios.

Let's start with the model preparation. If you are familiar with PyTorch, you should probably know how to train and save your model. In case you don't, we will use a pre-trained image classification model - MobileNet v2, which is already packaged in TorchVision. To install it, run the command below.

```
import torch
import torchvision

# An instance of your model.
model = torchvision.models.mobilenet_v2(pretrained=True)

# An example input you would normally provide to your model's forward() method.
example = torch.rand(1, 3, 224, 224)

# Use torch.jit.trace to generate a torch.jit.ScriptModule via tracing.
traced_script_module = torch.jit.trace(model, example)

traced_script_module.save("model.pt")
```

## **Android**

Similar to ios, we will use the MobileNetV2 model for implementation.
```
import torch
import torchvision
from torch.utils.mobile_optimizer import optimize_for_mobile

model = torchvision.models.mobilenet_v2(pretrained=True)
model.eval()
example = torch.rand(1, 3, 224, 224)
traced_script_module = torch.jit.trace(model, example)
traced_script_module_optimized = optimize_for_mobile(traced_script_module)
traced_script_module_optimized._save_for_lite_interpreter("model.ptl")
```
Above is a basic example of pytorch mobile in android format where the DL model requires only one input. In some special cases when the DL model requires multiple inputs, we can change it as follows:

```
# multiple inputs
example_1 = torch.rand(1, 3, 224, 224)
example_2 = tensor.Size(1,1258,256)
example_3 = tensor.Size(10,2,2) 

# put them together
example = (example_1, example_2, example_3)
traced_script_module = torch.jit.trace(model, example)
```
What's worth noting here is specifying how the tensor form of the input should be fed to the model to generate similar "examples".


## **Demo App**
To do the work on mobile with ios format, pytorch mobile has demoed some basic ios apps for some problems, you can download github according to the solution:
```
https://github.com/pytorch/ios-demo-app.git
cd ios-demo-app
```
With the format of android, you can refer to some demo apps of pytorch mobile by:

```
git clone https://github.com/pytorch/android-demo-app.git
cd android-demo-app
```
**Image Segmentation**

Image Segmentation demonstrates a Python script that converts the PyTorch DeepLabV3 model and an [Android](https://github.com/pytorch/android-demo-app/tree/master/ImageSegmentation) or [IOS](https://github.com/pytorch/ios-demo-app/tree/master/ImageSegmentation) app that uses the model to segment images.

![Figure 2](https://drive.google.com/uc?export=view&id=18h8Wv4ypEzr81CvKyewSU4GLVYwsHGtp)*<center>**Figure 2**. App demo for Image Segmentation.</center>*

**Object Detection**

Object Detection demonstrates how to convert the popular YOLOv5 model and use it in an [Android](https://github.com/pytorch/android-demo-app/tree/master/ObjectDetection) or [IOS](https://github.com/pytorch/ios-demo-app/tree/master/ObjectDetection) app that detects objects from pictures in your photos, taken with camera, or with live camera.

![Figure 3](https://drive.google.com/uc?export=view&id=1EFamcJNooyLvYX4Rwq2oXYI8Dg8dep2i)*<center>**Figure 3**. App demo for Image Segmentation.</center>*

In addition, there are some other popular problems such as Neural Machine Translation, Question Answering, Vision Transformer, Speech recognition, Video Classification,...


## **Application**
Realizing the advantages of pytorch mobile, we tested their app with the [AnimeGANv2](https://github.com/bryandlee/animegan2-pytorch) project on Android devices.
The results we get are quite good when the model can run on low-profile mobile devices, but the execution speed is still quite poor, not able to meet the real-time problem.

![Figure 4](https://drive.google.com/uc?export=view&id=1YEjk_C1PNL_wxSKvMSd8FloyekXLjnzt)*<center>**Figure 4**. App demo android for AnimeGanv2.</center>*

Table 1: performance speed and image size chart.
| size image | 640x640 | 400x400 | 300x300 | 256x256 |
| --------   | --------| --------|---------|---------|
| times      |    9s   |    4s   |    2s   |    2s   |

Due to the nature of the AnimeGanv2 model, it will be good for large images, this inadvertently affects the execution speed of the model because currently pytorch mobile only supports running on the CPU.


## **References**
[1] PYTORCH MOBILE: End-to-end workflow from Training to Deployment for iOS and Android mobile devices.
[2] Jie Chen, Gang Liu, Xin Chen “AnimeGAN: A Novel Lightweight GAN for Photo Animation.” ISICA 2019: Artificial Intelligence Algorithms and Applications pp 242-256, 2019.