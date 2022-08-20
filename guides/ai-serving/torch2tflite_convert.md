
---
last_modified_on: "2022-08-20"
title: Convert weight TF lite from Pytorch for Android app
description: Support how to convert weight from pytorch to tf lite for android app.
series_position: 2
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: hard"]
---

# **Introduction**
Today, the majority of scientific research in the field of AI uses the Pytorch framework for research purposes. But to put them into practice is still not much. Although now [Pytorch](https://pytorch.org/mobile/home/) has developed a library to support this work. However, there are still many limitations such as not being able to use it on GPUs. So TF Lite is still the first choice of users when they are interested in mobile themes.

In this article, we will show you how to convert weights from pytorch to tensorflow lite from our own experience with several related projects.

# **Work To Do**
To make the work easier to visualize, we will use the MobileNetv2 model as an example.
This conversion will include the following steps:
Pytorch - ONNX - Tensorflow TFLite

**Converting Pytorch to ONNX**
This maybe is a simple job for the most of onnx support all the problem when convert from torch sang onnx.

Requirements:
* ONNX ==1.7.0
* PyTorch ==1.7.0

Code: 
```
import onnx
import torch

example_input = torch.rand(1, 3, 256, 256)
model = torchvision.models.mobilenet_v2(pretrained=True)

ONNX_PATH="./my_model.onnx"

torch.onnx.export(
    model= model,
    args= example_input, 
    f= ONNX_PATH, # where should it be saved
    verbose=False,
    export_params=True,
    do_constant_folding=False,  # fold constant values for optimization
    # do_constant_folding=True,   # fold constant values for optimization
    input_names=['input'],
    output_names=['output'],
    opset_version=11
) 

onnx_model = onnx.load(ONNX_PATH)
onnx.checker.check_model(onnx_model)
```
You will probably come across many other tutorials that add additional parameters to the onnx export function. Here we only list some basic parameters. You can use more if it is absolutely necessary for your model.

**Converting ONNX to TensorFlow**

Next, we will convert the model from the newly acquired onnx to Tensorflow. You can find many different tutorials for this job. In my experience, each model will require a suitable version of Tensorflow so take the time to install the Tensorflow version that best suits the model you are working on.

Requirements:
* Onnx_tf == 1.10.0
* Tensorflow == 2.8.0

Code:
```
from onnx_tf.backend import prepare
import onnx

onnx_model_path = './my_model.onnx'
tf_model_path = './my_model_tf'

onnx_model = onnx.load(onnx_model_path)
tf_rep = prepare(onnx_model)
tf_rep.export_graph(tf_model_path)
```
Test: 
```
import tensorflow as tf
tf_model_path = './my_model_tf'

model = tf.saved_model.load(tf_model_path)
model.trainable = False

input_tensor = tf.random.uniform([1, 3, 256, 256])
out = model(**{'input': input_tensor})
print(out)
```
**Converting TensorFlow to TensorFlow Lite**

Finally, the step to convert to TF lite also has a lot of different instructions, but in this article I will introduce the simplest way to convert weights from TF (.pb) to TF Lite (.tflite). 

Code: 
```
import tensorflow as tf

saved_model_dir = './my_model_tf'
tflite_model_path = './my_model.tflite'

# Convert the model
converter = tf.lite.TFLiteConverter.from_saved_model(saved_model_dir)
tflite_model = converter.convert()

# Save the model
with open(tflite_model_path, 'wb') as f:
    f.write(tflite_model)
```

Test: 
```
import numpy as np
import tensorflow as tf

tflite_model_path = './my_model.tflite'

# Load the TFLite model and allocate tensors
interpreter = tf.lite.Interpreter(model_path=tflite_model_path)
interpreter.allocate_tensors()

# Get input and output tensors
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# Test the model on random input data
input_shape = input_details[0]['shape']
input_data = np.array(np.random.random_sample(input_shape), dtype=np.float32)

interpreter.set_tensor(input_details[0]['index'], input_data)
interpreter.invoke()

# get_tensor() returns a copy of the tensor data 
# use tensor() in order to get a pointer to the tensor 
output_data = interpreter.get_tensor(output_details[0]['index']) 
print(output_data.shape)
```

The conversion work here can be used on android devices, In addition to being able to speed up the performance of the TF model, it also supports the quantization library. This library makes the model faster and can run on the GPU.

**Converting TensorFlow (quantization) to TensorFlow Lite**
There are three common forms of quantization today.

**Dynamic range quantization**

At inference, weights are converted from 8-bits of precision to floating point and computed using floating-point kernels. This conversion is done once and cached to reduce latency.

To further improve latency, "dynamic-range" operators dynamically quantize activations based on their range to 8-bits and perform computations with 8-bit weights and activations. This optimization provides latencies close to fully fixed-point inference. However, the outputs are still stored using floating point so that the speedup with dynamic-range ops is less than a full fixed-point computation.

Code: 
```
import tensorflow as tf

saved_model_dir = './my_model_tf'
tflite_model_path = './my_model_dr3.tflite'

# Convert the model
converter = tf.lite.TFLiteConverter.from_saved_model(saved_model_dir)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_model = converter.convert()

# Save the model
with open(tflite_model_path, 'wb') as f:
    f.write(tflite_model)
```
**Full integer quantization**
You can get further latency improvements, reductions in peak memory usage, and compatibility with integer only hardware devices or accelerators by making sure all model math is integer quantized.

For full integer quantization, you need to calibrate or estimate the range, i.e, (min, max) of all floating-point tensors in the model. Unlike constant tensors such as weights and biases, variable tensors such as model input, activations (outputs of intermediate layers) and model output cannot be calibrated unless we run a few inference cycles. As a result, the converter requires a representative dataset to calibrate them. This dataset can be a small subset (around ~100-500 samples) of the training or validation data. Refer to the representative_dataset() function below.

Code:
```
def representative_dataset_gen():
    for image_path in images_list:
        image = cv2.imread('image1.jpg')
        image = cv2.resize(image, (256, 256))
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        image = image.astype(np.float32)/127.5 - 1.0
        image = np.expand_dims(image, 0)
        yield [image]
```
Additionally, to ensure compatibility with integer only devices (such as 8-bit microcontrollers) and accelerators (such as the Coral Edge TPU), you can enforce full integer quantization for all ops including the input and output, by using the following steps:

Code:
```
import tensorflow as tf

saved_model_dir = './my_model_tf'
tflite_model_path = './my_model_int8.tflite'

# Convert the model
model = tf.saved_model.load(saved_model_dir)
concrete_func = model.signatures[
    tf.saved_model.DEFAULT_SERVING_SIGNATURE_DEF_KEY]
    
converter = tf.lite.TFLiteConverter.from_concrete_functions([concrete_func])
converter.optimizations = [tf.lite.Optimize.DEFAULT]

converter.representative_dataset = representative_dataset_gen

converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.int8  # or tf.uint8
converter.inference_output_type = tf.int8  # or tf.uint8
converter.target_spec.supported_ops = [
  tf.lite.OpsSet.SELECT_TF_OPS # enable TensorFlow ops.
]
tflite_model = converter.convert()

# Save the model
with open(tflite_model_path, 'wb') as f:
    f.write(tflite_model)
```
**Float16 quantization**
You can reduce the size of a floating point model by quantizing the weights to float16, the IEEE standard for 16-bit floating point numbers. To enable float16 quantization of weights, use the following steps:

Code: 

```
import tensorflow as tf

saved_model_dir = './my_model_tf'
tflite_model_path = './my_model_fp16.tflite'

model = tf.saved_model.load(saved_model_dir)
concrete_func = model.signatures[
    tf.saved_model.DEFAULT_SERVING_SIGNATURE_DEF_KEY]
concrete_func.inputs[0].set_shape([1, 3, 256, 256])
    
converter = tf.lite.TFLiteConverter.from_concrete_functions([concrete_func])
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.target_spec.supported_ops = [
  tf.lite.OpsSet.TFLITE_BUILTINS, # enable TensorFlow Lite ops.
  tf.lite.OpsSet.SELECT_TF_OPS # enable TensorFlow ops.
  ]
converter.target_spec.supported_types = [tf.float16]
tflite_model = converter.convert()

# Save the model
with open(tflite_model_path, 'wb') as f:
    f.write(tflite_model)
```
The advantages of float16 quantization are as follows:
* It reduces model size by up to half (since all weights become half of their original size).
* It causes minimal loss in accuracy.
* It supports some delegates (e.g. the GPU delegate) which can operate directly on float16 data, resulting in faster execution than float32 computations.

The disadvantages of float16 quantization are as follows:
* It does not reduce latency as much as a quantization to fixed point math.
* By default, a float16 quantized model will "dequantize" the weights values to float32 when run on the CPU. (Note that the GPU delegate will not perform this dequantization, since it can operate on float16 data.)

I hope that you found my experience useful, good luck!


# **Related Links**
Official Documentation:

* https://pytorch.org/docs/stable/onnx.html
* https://pytorch.org/tutorials/advanced/super_resolution_with_onnxruntime.html
* https://github.com/microsoft/onnxruntime
* https://github.com/onnx/tutorials
* https://www.tensorflow.org/lite/guide/ops_compatibility
* https://www.tensorflow.org/lite/guide/ops_select
* https://www.tensorflow.org/lite/guide/inference#load_and_run_a_model_in_python
* https://www.tensorflow.org/lite/performance/post_training_quantization