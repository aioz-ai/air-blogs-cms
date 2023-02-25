---
last_modified_on: "2023-02-23"
title: Quantization in Pytorch.
description: Quantization method with Pytorch framework.
series_position: 15
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
Similar to Tensorflow, in recent years Pytorch has tried to develop a lot of things to catch up with Tensorflow. In particular, the Quantization method is also quite complete and can be used comfortably.

## Introduction 
Pytorch provides us with two different quantization modes:

* **Eager Mode Quantization**: In this mode, we need to merge layers like convolution, batchnorm, relu and define the start and end position of quantization manually. And we can only use modules that are supported by torch.
* **FX Grapg Mode Quantization**: A framework that supports automatic quantization of pytorch. This is an upgraded version of Eager Mode Quantization, which supports more functions instead of just torch.nn modules like Eager Mode Quantization. However, we need to modify the original model to match X Graph Mode Quantization.

![Figure 1](https://drive.google.com/uc?export=view&id=1lrzfxB5zRjia8VmwxyviQhWenvG7lKFn)*<center>**Figure 1:** Comparison table between the two methods. </center>*

## Quantization algorithm
Both Eager Mode Quantization and FX Graph Mode Quantization support the following three quantization algorithms:

* Dynamic Quantization
* Static Quantization
* Quantization-aware training

**Dynamic Quantization**
Dynamic here means that the optimization of the quantization algorithm takes place during the inference process. Where model weights are quantized immediately and activation functions will be quantized at inference. Dynamic quantization does the conversion by multiplying the input value by a value called the scaling factor then rounds this result to the nearest value and stores them.

Dynamic quantization is the least efficient of the three methods due to its simplicity, however it is often used in situations where execution time is more affected by the time it takes to load the weight from memory than it is. due to matrix multiplication. Therefore, this method is often used for models such as LTSM, Transformer, ....

```
import torch

quantized_model = torch.quantization.quantize_dynamic(model, {torch.nn.Linear}, dtype=torch.qint8)
```

Inside:

* **model** is the model that needs to be optimized.
* **{torch.nn.Linear}** is the set of classes in the model that need quantize.
* **dtype** is the quantize data type you want to convert.

**Static Quantization**
Static Quantization is also known as Post Training Quantization. Compared to the first method, static quantization has 4 differences:

* **The first point** is to do the quantize weights and activations of the model before inference.
* **The second point** is that there is an extra step to refine the model after quantize, this ensures that the model after quantize has a higher accuracy than just doing it at inference.
* **The third point** is that accuracy is hardware dependent. Because Pytorch uses two libraries to support the conversion: FBGEMM on x86 chips and QNNPACK on ARM chips. Therefore, we need to ensure that the machine we use for training and deployment needs to be architecturally similar.
* **The fourth point** is that we need to perform fuse layer or merge the convolution, batchnorm, relu layers into one thanks to the nn.Sequential layer. By consolidating multiple classes into one like this allows libraries to compute in a single pass thereby improving model performance.

```
import torch 

model = torch.quantization.fuse_modules(model, [['conv', 'bn', 'relu']])
```

## Quantize Aware Training.
Quantize Aware Training models the effects of quantization during training and corrects it, making it more efficient than other quantization methods.

Quantize Aware Training includes 4 steps as follows:

1. Train the network with floating point calculations.
2. Insert the Fake quantization Q classes into the newly trained network.
3. Make model finetune. Note in this process the gradient is still used as a floating point.
4. Perform inference by removing Q and quantized load weight.


![Figure 2](https://drive.google.com/uc?export=view&id=1oCDSnG57ai7BwhpvGKZEsZzMl_w7PqJt)*<center>**Figure 2:** Typical convolution layer followed by an activation(left). Quantized version of this network( right) </center>*

## Some notes when using
* **Quantification is only the method used when inference**: As we know floating point numbers have much more precise representation than integers (int8). Therefore, int8 cannot be used in backpropagation because this process is very sensitive to incorrect representation of weight and leads to model divergence.
* **Accuracy will decrease after quantization**: Quantization often reduces the accuracy of the model. This is a matter of tradeoff between accuracy and processing time. However, how much accuracy we trade off to reduce processing time depends on many factors such as the initial model size, the quantization technique or how many layers we quantize in the model and the layers. how does that affect the whole model,....
* **There is no need to perform quantization for the entire model**:
We can fully quantize part of the model and determine which classes are quantized or not. To do this, Pytorch provides us with two ways to do it as follows:
 1. Disable/enable the quantization mode of each layer by assigning the .qconfig values of the classes with a specific qconfig_dict value. For example conv1.qconfig = None means conv1 is not quantized or conv1.qconfig = custom_qconfig means use custom_qconfig instead of config that we have specified.
2. Use QuantStub and DeQuantSub.

```
import torch

# define a floating point model where some layers could be statically quantized
class M(nn.Module):
    def __init__(self, model_fp32):
        super(M, self).__init__()
        
        # QuantStub converts tensors from floating point to quantized.
        # This will only be used for inputs.
        self.quant = torch.quantization.QuantStub()
        
        # DeQuantStub converts tensors from quantized to floating point.
        # This will only be used for outputs.
        self.dequant = torch.quantization.DeQuantStub()
        
        # FP32 model
        self.model_fp32 = model_fp32

    def forward(self, x):
        # manually specify where tensors will be converted from floating
        # point to quantized in the quantized model
        x = self.quant(x)
        x = self.model_fp32(x)
        
        # manually specify where tensors will be converted from quantized
        # to floating point in the quantized model
        x = self.dequant(x)
        return x
```
* **Pytorch only supports quantization with CPU**: You can freely perform training with Quantize Aware Training on GPU devices, but when performing inference using quantization, you must use cpu or on mobile.

## References

1. [Pytorch Quantization](https://pytorch.org/docs/stable/quantization.html)