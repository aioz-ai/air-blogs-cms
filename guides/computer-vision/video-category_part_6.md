---
last_modified_on: "2022-01-10"
title: Video recognition and categorization (Part 6 - A basic tutorial)
description: A basic implementation for video classification task.
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: basic", "guides: video_category"]
---
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Ymdab42dyvsjpomB0VADbQuNlBTcaLEM?usp=sharing)

A video is a sequence of frames or images that are played one after the other. The most of videos we see in our daily life have a frame rate of greater than 30 frames per second. As a result, even for short movies, we must deal with a considerable amount of data in comparison to image classification. Because the images are closely connected, skipping intermediate frames and processing fewer frames per second is usual.

In this post, we'll show you how to use PyTorchVideo models, datasets, and transformations to create a basic video classification training pipeline. We will train video classification task on Kinetics dataset using 3D ResNet and a standard video transform augmentation. Because PyTorchVideo lacks training code, we'll use on PyTorch Lightning, a lightweight PyTorch training framework.

## Install Necessary Libraries and Data Preparation 

Please install necessary libraries by using following comands:

```bash
!pip install torch pytorch_lightning pytorchvideo
!pip install --upgrade youtube-dl
```

The dataset must first be prepared. To train our model, we'll need a training dataset, as well as a test or validation dataset to evaluate it. For this purpose, we will use [Kinetics](https://deepmind.com/research/open-source/kinetics) dataset - A collection of large-scale, high-quality datasets of URL links of up to 650,000 video clips that cover 400/600/700 human action classes, depending on the dataset version. The videos include human-object interactions such as playing instruments, as well as human-human interactions such as shaking hands and hugging. Each action class has at least 400/600/700 video clips. Each clip is human annotated with a single action class and lasts around 10 seconds.

![Fig-1](https://production-media.paperswithcode.com/datasets/kinetics.jpg)  
*<center>**Figure 1**:A few sample videos in Kinetics dataset.</center>*

To prepare the Kinetics dataset, please following below steps:
*   First, clone this repository and make sure that all the submodules are also cloned properly.
    ```bash
    !git clone https://github.com/activitynet/ActivityNet.git
    cd ActivityNet/Crawler/Kinetics
    ```
*   Second, download a dataset split by calling:
    ```bash
    mkdir "data_dir"
    python download.py "dataset_split".csv "data_dir"
    ```
    In here, we choose Kinetics 400 version.
    ```bash
    mkdir kinetics-400
    !python download.py "data/kinetics-400_train.csv" "kinetics-400"
    !python download.py "data/kinetics-400_val.csv" "kinetics-400"
    !python download.py "data/kinetics-400_test.csv" "kinetics-400"
    ```

## Define Dataloader

In this step, we define the Kinetic dataloader which contains train and validate set and corresponding transforms.

```python
import os
import pytorch_lightning
import pytorchvideo.data
import torch.utils.data
```

```python
from pytorchvideo.transforms import (
    ApplyTransformToKey,
    Normalize,
    RandomShortSideScale,
    RemoveKey,
    ShortSideScale,
    UniformTemporalSubsample
)

from torchvision.transforms import (
    Compose,
    Lambda,
    RandomCrop,
    RandomHorizontalFlip
)
```

```python
class KineticsDataModule(pytorch_lightning.LightningDataModule):

  # Dataset configuration
  # Insert the data path in here
  _DATA_PATH = "ActivityNet/Crawler/Kinetics/kinectics-400"
  _CLIP_DURATION = 2  # Duration of sampled clip for each video
  _BATCH_SIZE = 8
  _NUM_WORKERS = 8  # Number of parallel processes fetching data

  def train_dataloader(self):
    """
    Create the Kinetics train partition from the list of video labels
    in {self._DATA_PATH}/train
    """
    train_transform = Compose(
            [
            ApplyTransformToKey(
              key="video",
              transform=Compose(
                  [
                    UniformTemporalSubsample(8),
                    Lambda(lambda x: x / 255.0),
                    Normalize((0.45, 0.45, 0.45), (0.225, 0.225, 0.225)),
                    RandomShortSideScale(min_size=256, max_size=320),
                    RandomCrop(244),
                    RandomHorizontalFlip(p=0.5),
                  ]
                ),
              ),
            ]
        )
    train_dataset = pytorchvideo.data.Kinetics(
        data_path=os.path.join(self._DATA_PATH, "train"),
        clip_sampler=pytorchvideo.data.make_clip_sampler("random", self._CLIP_DURATION),
        transform=train_transform
    )
    return torch.utils.data.DataLoader(
        train_dataset,
        batch_size=self._BATCH_SIZE,
        num_workers=self._NUM_WORKERS,
    )

  def val_dataloader(self):
    """
    Create the Kinetics validation partition from the list of video labels
    in {self._DATA_PATH}/val
    """
    val_transform = Compose(
            [
            ApplyTransformToKey(
              key="video",
              transform=Compose(
                  [
                    UniformTemporalSubsample(8),
                    Lambda(lambda x: x / 255.0),
                    Normalize((0.45, 0.45, 0.45), (0.225, 0.225, 0.225)),
                  ]
                ),
              ),
            ]
        )
    val_dataset = pytorchvideo.data.Kinetics(
        data_path=os.path.join(self._DATA_PATH, "val"),
        clip_sampler=pytorchvideo.data.make_clip_sampler("uniform", self._CLIP_DURATION),
        transform=val_transform
    )
    return torch.utils.data.DataLoader(
        val_dataset,
        batch_size=self._BATCH_SIZE,
        num_workers=self._NUM_WORKERS,
    )
```

## Model Implementation

Simple, repeatable factory functions may be used to create all PyTorchVideo models and layers. Because the args don't require hierarchical configs, we call this the "flat" model interface. Below is an example of how to create a default ResNet.

```python
import pytorchvideo.models.resnet

def make_kinetics_resnet():
  return pytorchvideo.models.resnet.create_resnet(
      input_channel=3, # RGB input from Kinetics
      model_depth=50, # For the tutorial let's just use a 50 layer network
      model_num_class=400, # Kinetics has 400 classes so we need out final head to align
      norm=nn.BatchNorm3d,
      activation=nn.ReLU,
  )
```

Next step, we define the train and validation code, and the optimizer.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class VideoClassificationLightningModule(pytorch_lightning.LightningModule):
  def __init__(self):
    super().__init__()
    self.model = make_kinetics_resnet()
    
  def forward(self, x):
    return self.model(x)

  def training_step(self, batch, batch_idx):
    # The model expects a video tensor of shape (B, C, T, H, W), which is the
    # format provided by the dataset
    y_hat = self.model(batch["video"])

    # Compute cross entropy loss, loss.backwards will be called behind the scenes
    # by PyTorchLightning after being returned from this method.
    loss = F.cross_entropy(y_hat, batch["label"])

    # Log the train loss to Tensorboard
    self.log("train_loss", loss.item())

    return loss

  def validation_step(self, batch, batch_idx):
    y_hat = self.model(batch["video"])
    loss = F.cross_entropy(y_hat, batch["label"])
    self.log("val_loss", loss)
    return loss

  def configure_optimizers(self):
    """
    Setup the Adam optimizer. Note, that this function also can return a lr scheduler, which is
    usually useful for training video models.
    """
    return torch.optim.Adam(self.parameters(), lr=1e-1)
```

## Model Training

We put all everything in a train() function which incluces a model, a dataloader and a trainer.

```python
def train():
  classification_module = VideoClassificationLightningModule()
  data_module = KineticsDataModule()
  trainer = pytorch_lightning.Trainer(gpus=1)
  trainer.fit(classification_module, data_module)
```

Finally,

```python
train()
```

Hopefully with the video classification series, you can understand and have the most overview about this task so that you can develop with bolder ideas.
