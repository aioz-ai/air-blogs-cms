---
last_modified_on: "2021-06-29"
title: Get familiar with medical imaging datasets
description: A guide on dealing with medical images
author_github: https://github.com/trqminh
tags: ["type: tutorial", "level: easy"]
---
At AIOZ, we have been involved in several projects applying computer science to assist the medical field. Those kinds of projects are very demanding recently, due to the impact of deep learning. Besides the robotics project to help doctors transport medical equipment in isolation areas safely, we also participate in academic research which utilizes machine learning in general and deep learning in particular. With some articles published in this field, we believe we can share some helpful tutorial to assist the community. Thus, in this post, we will introduce a sample of a medical image and how to load, visualize and preprocess it.


## Understand the image
Recently, we have worked on 2 types of medical imaging, namely Liver CT scan and Brain MRI, for the deformable registration problem. However, in this tutorial, we will introduce a sample of the Liver CT, belonged to the MSD dataset. MRI data might be introduced in another tutorial.

The MSD dataset is a labeled dataset for the medical image segmentation task (tumor segmentation for liver CT scan). Let's explore this CT image data. We prepare a sample that includes an image and its label (the segmentation of the tumors).


```python
!ls liverCT
```
```
liver_0_label.nii.gz  liver_0.nii.gz
```



This is NIfTI type of data, commonly used to store medical images. To load this data, we use the nibabel library for python.


```python
!pip install nibabel
```

Let import some necessary libraries.


```python
import nibabel as nib
import os
import numpy as np
```

After loading the image, we use the `get_data()` method to get the numeric data.


```python
root_dir = 'liverCT'
img = nib.load(os.path.join(root_dir, 'liver_0.nii.gz'))
label = nib.load(os.path.join(root_dir, 'liver_0_label.nii.gz'))
img_data = img.get_data()
label_data = label.get_data()
```

Let view some information of the data


```python
img_data.shape
```




    (512, 512, 75)




```python
label_data.shape
```




    (512, 512, 75)




```python
np.mean(img_data), np.max(img_data), np.min(img_data)
```




    (-523.4203, 1410.0, -1024.0)




```python
np.mean(label_data), np.max(label_data), np.min(label_data)
```




    (0.028239847819010417, 2, 0)



As can be seen, there are two kinds of tumor in this liver. Let see where a tumor is:


```python
np.where(label_data == 1)
```




    (array([133, 133, 133, ..., 441, 441, 441]),
     array([334, 335, 335, ..., 231, 232, 233]),
     array([65, 64, 65, ..., 63, 63, 63]))



Now, let show a slice of the liver that contains tumors.


```python
import skimage.io as io
def show_volume_slice(data):
    io.imshow(data, cmap = 'gray')
    io.show()
```


```python
show_volume_slice(img_data[133])
```



![png](https://vision.aioz.io/thumbnail/a4e2ddb712c94906b9e9/1024/output_18_0.png)




```python
show_volume_slice(label_data[133])
```

    /home/quangminh/.virtualenvs/rlenv/lib/python3.6/site-packages/skimage/io/_plugins/matplotlib_plugin.py:150: UserWarning: Low image data range; displaying image with stretched contrast.
      lo, hi, cmap = _get_display_range(image)




![png](https://vision.aioz.io/thumbnail/68090bb801de4f2986ea/1024/output_19_1.png)



## A handy tool for processing medical images
Medical images are special and how to preprocess them is also complex. Moreover, applying training techniques of deep learning requires the processing to be synchronous, fast, and convenient. One of the tools to support medical imaging is **monai**, an open-source framework developing healthcare imaging training workflows in a native PyTorch paradigm.

Indeed, monai supports DataSet and DataLoader objects very similar to Pytorch. The feature of the DataLoader is to allow transformations to be performed while sampling the data. With special transformations for medical images, monai allows the entire process of loading and processing data to go smoothly without too many implementations. However, the input data needs to be defined as a list of dictionaries as follows:


```python
data_dicts = [
    {'image': 'liverCT/liver_0.nii.gz',
     'label': 'liverCT/liver_0_label.nii.gz'}
]
```


```python
!pip install monai
```

Let import `Dataset, DataLoader` and some essential transformations. In fact, with the `CropForegroundd` transformation, we are able to get rid of uninformative data.


```python
from monai.data import Dataset, DataLoader
from monai.transforms import CropForegroundd, LoadImaged, AddChanneld, Compose
```


```python
# define the transforms
transforms = Compose([
                LoadImaged(keys=["image", "label"]),
                AddChanneld(keys=["image", "label"]),
                CropForegroundd(keys=["image", "label"], source_key="image")
            ])
```


```python
liver_dataset = Dataset(data=data_dicts,
                        transform=transforms)
```


```python
liver_loader = DataLoader(liver_dataset, num_workers=0, batch_size=1)
```

Let get a sample and visualize it.


```python
sample = next(iter(liver_loader))
```


```python
sample.keys()
```




    dict_keys(['image', 'label', 'image_meta_dict', 'label_meta_dict', 'foreground_start_coord', 'foreground_end_coord', 'image_transforms', 'label_transforms'])




```python
sample['image'].shape
```




    torch.Size([1, 1, 500, 355, 75])




```python
show_volume_slice(sample['image'][0][0].numpy()[133])
```



![png](https://vision.aioz.io/thumbnail/443ce8b2b9294ad2aef2/1024/output_32_0.png)



As we can see, the image is cropped according to the foreground.

## Conclusion
To sum up, in this tutorial, we introduce a sample of LiverCT scan images and a brief guiding to preprocess them. Although this tutorial is not sufficient enough for the whole medical image pipeline of deep learning, we hope that it can be a good start for you to explore more in the field.
