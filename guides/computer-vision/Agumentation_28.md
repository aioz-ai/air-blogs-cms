---
last_modified_on: "2022-12-27"
title: Augmentation
description: Techniques to support training model deep learning.
series_position: 28
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
There is a techniques to solve the problem of having less data for training models, which is data augmentation. Augmentation is a technique of generating training data from the data we already have. Let's take a look at some popular augmentation techniques with photos.


## Flip
You can flip images horizontally and vertically. Some frameworks do not provide function for vertical flips. But, a vertical flip is equivalent to rotating an image by 180 degrees and then performing a horizontal flip. Below are examples for images that are flipped.

![Figure 10](https://drive.google.com/uc?export=view&id=1Njt6AxfZMU3KA7y1fWy8vgBdAR4cd98A)*<center>**Figure 10**. The method flip image.</center>*

You can perform flips by using any of the following commands, from your favorite packages. Data Augmentation Factor = 2 to 4x.

```
# NumPy.'img' = A single image.
flip_1 = np.fliplr(img)

# TensorFlow. 'x' = A placeholder for an image.
shape = [height, width, channels]
x = tf.placeholder(dtype = tf.float32, shape = shape)

flip_2 = tf.image.flip_up_down(x)
flip_3 = tf.image.flip_left_right(x)
flip_4 = tf.image.random_flip_up_down(x)
flip_5 = tf.image.random_flip_left_right(x)
```

## Rotation
One key thing to note about this operation is that image dimensions may not be preserved after rotation. If your image is a square, rotating it at right angles will preserve the image size. If it’s a rectangle, rotating it by 180 degrees would preserve the size. Rotating the image by finer angles will also change the final image size. We’ll see how we can deal with this issue in the next section. Below are examples of square images rotated at right angles.

![Figure 11](https://drive.google.com/uc?export=view&id=1NV70GUHW2eqqXph0ye1wRck9zaIUREmR)*<center>**Figure 11**. The method flip image.</center>*

You can perform rotations by using any of the following commands, from your favorite packages. Data Augmentation Factor = 2 to 4x.

```
shape = [height, width, channels]
x = tf.placeholder(dtype = tf.float32, shape = shape)
rot_90 = tf.image.rot90(img, k=1)
rot_180 = tf.image.rot90(img, k=2)

# To rotate in any angle. In the example below, 'angles' is in radians
shape = [batch, height, width, 3]
y = tf.placeholder(dtype = tf.float32, shape = shape)
rot_tf_180 = tf.contrib.image.rotate(y, angles=3.1415)

# Scikit-Image. 'angle' = Degrees. 'img' = Input Image
# For details about 'mode', checkout the interpolation section below.
rot = skimage.transform.rotate(img, angle=45, mode='reflect')
```

## Gaussian Noise
Over-fitting usually happens when your neural network tries to learn high frequency features (patterns that occur a lot) that may not be useful. Gaussian noise, which has zero mean, essentially has data points in all frequencies, effectively distorting the high frequency features. This also means that lower frequency components (usually, your intended data) are also distorted, but your neural network can learn to look past that. Adding just the right amount of noise can enhance the learning capability.

A toned down version of this is the salt and pepper noise, which presents itself as random black and white pixels spread through the image. This is similar to the effect produced by adding Gaussian noise to an image, but may have a lower information distortion level.

![Figure 13](https://drive.google.com/uc?export=view&id=1CBB9SaLpznMlcwlrNt-UTZLMrG2yRQHI)*<center>**Figure 13**. The method Gaussian Noise image.</center>*

You can add Gaussian noise to your image by using the following command, on TensorFlow. Data Augmentation Factor = 2x.

```
#TensorFlow. 'x' = A placeholder for an image.
shape = [height, width, channels]
x = tf.placeholder(dtype = tf.float32, shape = shape)

# Adding Gaussian noise
noise = tf.random_normal(shape=tf.shape(x),
                        mean=0.0, 
                        stddev=1.0, 
                        dtype=tf.float32)
output = tf.add(x, noise)
```

## Crop
Unlike scaling, we just randomly sample a section from the original image. We then resize this section to the original image size. This method is popularly known as random cropping. Below are examples of random cropping. If you look closely, you can notice the difference between this method and scaling.

![Figure 14](https://drive.google.com/uc?export=view&id=1zc1NI4FlmVrRTb7NMeQpXWjejQskVS0F)*<center>**Figure 14**. The method Crop image.</center>*

You can perform random crops by using any the following command for TensorFlow. Data Augmentation Factor = Arbitrary.

```
# TensorFlow. 'x' = A placeholder for an image.
original_size = [height, width, channels]
x = tf.placeholder(dtype = tf.float32, shape = original_size)

# Use the following commands to perform random crops
crop_size = [new_height, new_width, channels]
seed = np.random.randint(1234)

x = tf.random_crop(x, size = crop_size, seed = seed)
output = tf.images.resize_images(x, size = original_size)
```
## Translation
Translation just involves moving the image along the X or Y direction (or both). In the following example, we assume that the image has a black background beyond its boundary, and are translated appropriately. This method of augmentation is very useful as most objects can be located at almost anywhere in the image. This forces your convolutional neural network to look everywhere.

You can perform translations in TensorFlow by using the following commands. Data Augmentation Factor = Arbitrary.

![Figure 12](https://drive.google.com/uc?export=view&id=13QQgWcM9qe6EFRAOcgL-Sc1il_ogRsRs)*<center>**Figure 12**. The method translation image.</center>*


```
# pad_left, pad_right, pad_top, pad_bottom denote the pixel 
# displacement. Set one of them to the desired value and rest to 0
shape = [batch, height, width, channels]
x = tf.placeholder(dtype = tf.float32, shape = shape)

# We use two functions to get our desired augmentation
x = tf.image.pad_to_bounding_box(x, 
                                pad_top, 
                                pad_left, 
                                height + pad_bottom + pad_top, 
                                width + pad_right + pad_left)
output = tf.image.crop_to_bounding_box(x, pad_bottom, pad_right, height, width)
```
## Summary
 Transfer learning and data aumentation is one of the effective methods in case of small size data. Application of transfer learning can help improve model accuracy and reduce training time at the same time. Data agumentation partially solves the problem of data imbalance, improving the diversity of data.

To be able to apply transfer learning and data aumentation effectively, we need experience. Through this article, I have guided you through the criteria for choosing a transfer learning model. Hopefully, you will apply it effectively to the process of building and training your model.

## References

[1] [Transfer learning for deep learning - Machine learning mastery
](https://machinelearningmastery.com/transfer-learning-for-deep-learning/)

[2] [Transfer Learning - C3W2L07 - Andrew Ng](https://www.youtube.com/watch?v=yofjFQddwHE)

[3] [Data Augmentation in Python: Everything You Need to Know](https://neptune.ai/blog/data-augmentation-in-python)