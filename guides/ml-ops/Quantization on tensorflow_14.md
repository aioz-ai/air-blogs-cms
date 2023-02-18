---
last_modified_on: "2023-02-16"
title: Quantization on tensorflow
description: Quantization method with tensorflow framework.
series_position: 14
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

Regarding Quantization on tensorflow, there are quite detailed instructions for the 3 types above, we will summarize the main ideas including:

* How to convert model on tflite.
* Quantization model.
* Check the accuracy of the model after quantization.

On tensorflow we usually quantize the model on tensorflow lite before deploying on mobile devices. If you do not know about tensorflow lite, this is a format of type FlatBuffer, a cross platform that supports many different languages such as C++, C#, C, Go, Java, Kotlin, JavaScript, Lobster, Lua, TypeScript, PHP, Python, Rust, Swift. FlatBuffer allows data serialization faster than other data types like Protocol Buffer because it skips data parsing. Therefore, the process of loading the model will be significantly faster.

Next we will practice Quantization model on tflite format.


## Convert model on tflite

Tensorflow provides a module called **tf.lite.TFLiteConverter** to convert data from formats like **.pb**, **.h5** to **.tflite**.

For the sake of simplicity, we will not train the model from scratch, but load a pretrain model from the tensorflow hub. Tensorflow hub is a great place to share opensource and pretrain models on tensorflow. From images, text to voice. You can find a lot of pretrain models here. Also if you want to share weights from your models, you can also become a Tensorhub publisher.

To load a pretrain layer on tensorflow hub do the following:
```
import tensorflow as tf
import tensorflow_hub as hub

mobilenet_v2 = tf.keras.Sequential([
  tf.keras.layers.InputLayer(input_shape=(28, 28, 1)),
  hub.KerasLayer("https://tfhub.dev/tensorflow/tfgan/eval/mnist/logits/1"),
  tf.keras.layers.Dense(10, activation='softmax')
])
```

Retrain the model on the **mnist dataset**
```
# Load MNIST dataset
mnist = tf.keras.datasets.mnist
(train_images, train_labels), (test_images, test_labels) = mnist.load_data()

# Normalize the input image so that each pixel value is between 0 to 1.
train_images = train_images.astype(np.float32) / 255.0
test_images = test_images.astype(np.float32) / 255.0


mobilenet_v2.compile(optimizer='adam',
              loss=tf.keras.losses.SparseCategoricalCrossentropy(
                  from_logits=True),
              metrics=['accuracy'])
mobilenet_v2.fit(
  train_images,
  train_labels,
  epochs=5,
  validation_data=(test_images, test_labels)
)
```

Our model is initialized from keras so it has **.h5** format. We can convert the model to **.tflite**.
```
import pathlib

converter = tf.lite.TFLiteConverter.from_keras_model(mobilenet_v2)
mobilenetv2_tflite = pathlib.Path("/tmp/mobilenet_v2.tflite")
mobilenetv2_tflite.write_bytes(converter.convert())
!ls -lh /tmp
```
So in tflite format we have the **mobilenet_v2.tflite** model which is **13 Mb** in size.

## Quantization model
To quantize the input, output and intermediate layers, we have to go through RepresentativeDataset, RepresentativeDataset is a generator function with a large enough size, which estimates the range of variation for all the model's parameters.

```
def _quantization_int8(model_path):
  def representative_data_gen():
    # prepare batch with batch_size = 100
    for input_value in tf.data.Dataset.from_tensor_slices(train_images).batch(1).take(100):
      input_value = tf.reshape(input_value, (1, 28, 28, 1))
      yield [input_value]
  # Declare optimizations model
  converter.optimizations = [tf.lite.Optimize.DEFAULT]
  # representative dataset
  converter.representative_dataset = representative_data_gen
  # Declare target specification data type
  converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
  # Define input and output data type
  converter.inference_input_type = tf.uint8
  converter.inference_output_type = tf.uint8
  tflite_model_quant = converter.convert()
  mobilenetv2_quant_tflite = pathlib.Path(model_path)
  mobilenetv2_quant_tflite.write_bytes(tflite_model_quant)
  return tflite_model_quant

tflite_model_quant = _quantization_int8("/tmp/mobilenet_v2_quant.tflite")
```

In the _quantization_int8 function we have to declare the parameters for the converter. It includes the following:

* **optimizations**: The optimization method used to calculate quantization so that the error is minimal compared to the original format.
* **representative_dataset**: Generator returns a batch input for the model. Have a size large enough to calculate the dynamic range for the model's parameters.
* **target_spec**: Computational format of operations in deep learning networks.
* **inference_input_type**: Input format.
* **inference_output_type**: Output format.

The input, output, and target_spec formats must be of the same type as the quantization data format.

Before quantization, the format of input and output is float32, after quantization, they are converted to int8.
```
interpreter = tf.lite.Interpreter(model_content=tflite_model_quant)
input_type = interpreter.get_input_details()[0]['dtype']
print('input: ', input_type)
output_type = interpreter.get_output_details()[0]['dtype']
print('output: ', output_type)
!ls -lh /tmp
```

We can see that quantization has reduced the original tflite file size by **4 times**. Readers can also check the model's inference rate for themselves after quantization.

##  The accuracy of the quantization model
After performing quantization we always have to check the accuracy on an independent validation set. This process ensures that the model after quantization has a quality that is not too reduced compared to the original model.

Check model accuracy on validation set:
```
def _run_tflite_model(model_file, test_images, test_image_indices):
  # Initialize model tf.lite
  interpreter = tf.lite.Interpreter(model_path=str(model_file))
  interpreter.allocate_tensors()

  input_details = interpreter.get_input_details()[0]
  output_details = interpreter.get_output_details()[0]

  # predictions
  predictions = np.zeros((len(test_image_indices),), dtype=int)
  for i, test_image_index in enumerate(test_image_indices):
    test_image = tf.reshape(test_images[test_image_index], (28, 28, 1))
    test_label = test_labels[test_image_index]

    # Check if input is quantized, then rescale input data to uint8
    if input_details['dtype'] == np.uint8:
      input_scale, input_zero_point = input_details["quantization"]
      test_image = test_image / input_scale + input_zero_point

    test_image = np.expand_dims(test_image, axis=0).astype(input_details["dtype"])
    interpreter.set_tensor(input_details["index"], test_image)
    interpreter.invoke()
    output = interpreter.get_tensor(output_details["index"])[0]
    predictions[i] = output.argmax()
  return predictions

# Helper function to evaluate a TFLite model on all images
def _evaluate_model(model_file, model_type):
  global test_images
  global test_labels

  test_image_indices = range(test_images.shape[0])
  predictions = _run_tflite_model(model_file, test_images, test_image_indices)

  accuracy = (np.sum(test_labels== predictions) * 100) / len(test_images)

  print('%s model accuracy is %.4f%% (Number of test samples=%d)' % (
      model_type, accuracy, len(test_images)))
  
_evaluate_model("/tmp/mobilenet_v2_quant.tflite", model_type="unit8"")
```

We see that after quantization, the accuracy of the model decreases by about 0.05%. This is a negligible reduction compared to the benefits gained from reducing model size and inference speed. However, for complex deep learning architectures, the degree of accuracy loss after quantization will vary significantly. Users need to consider carefully before using.


## References
1. [Quantization Tutorial in TensorFlow for ML model](https://medium.com/codex/quantization-tutorial-in-tensorflow-to-optimize-a-ml-model-like-a-pro-cadf811482d9)
2. [Model Quantization Methods In TensorFlow Lite](https://studymachinelearning.com/model-quantization-methods-in-tensorflow-lite/)