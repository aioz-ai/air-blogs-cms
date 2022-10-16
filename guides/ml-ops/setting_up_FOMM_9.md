---
last_modified_on: "2022-10-16"
title: Model Compression Part 6.
description: Getting FOMM repo up and ready.
series_position: 9
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: easy"]
---

## Getting FOMM repo up and ready

Goal: To get FOMM repo up and ready for VoxCeleb dataset. 
Requirements: Python >= 3.8.

### Setting up 
1. Clone repo at https://github.com/AliaksandrSiarohin/first-order-model
2. Create a folder  `data/vox-png`  with 2 subfolders  `train`  and  `test`, put training videos in the  `train`and testing in the  `test`.
3. Edit the config file `config/vox-256.yaml` according to your training/reconstruction, in dataset_params specify the root dir the  `root_dir: data/vox-png`. Also adjust the number of epoch in train_params.
4. Save model checkpoints `vox-cpk.pth.tar` from [Google Drive](https://drive.google.com/drive/folders/1PyQJmkdCsAkOYwUyaj_l-l0as-iLDgeH) to a folder called `checkpoints`.  
3. Install packages according to the following `requirements.txt`
	```
	imageio==2.22.0  
	matplotlib==3.6.0  
	numpy==1.23.3  
	pandas==1.5.0  
	python-dateutil==2.8.2  
	pytz==2022.2.1  
	PyYAML==6.0  
	scikit-image==0.19.3  
	scikit-learn==1.1.2  
	scipy==1.9.1  
	torch==1.7.1  
	torchvision==0.8.2  
	tqdm==4.64.1
	imageio--fmpeg==0.4.7
	```


### Changes in loading dataset
If VoxCeleb data filenames take the format of `id100001|xxxx|xxxx|xxxx.mp4`, in `frames_dataset.py` line 75, change 
```{python}
train_videos = {os.path.basename(video}.split('#')[0] for video in 
                os.listdir(os.path.join(root_dir, 'train'))}
```
to 
```{python}
train_videos = {os.path.basename(video}.split('|')[0] for video in 
                os.listdir(os.path.join(root_dir, 'train'))}
```

### Changes in code 
1. In `augmentation.py` line 13, change `from skimage import pad` to `from numpy import pad`. 
2. In `logger.py`:
	- Line 7: Change `from skimage.draw import circle` to `from skimage.draw import disk`
	- Line 111: Change 
	  `rr, cc = circle(kp[1], kp[0], self.kp_size, shape=image.shape[:2])` to 
	   `rr,cc = disk(center=(kp[1], kp[0]), radius=self.kp_size, shape=image.shape[:2])`.
3. In `modules.generator.py` line 93, change `F.sigmoid` to `torch.sigmoid`.

### Run code 
At this point, model should be ready to run reconstruction or further training (without any customisation). 
- To do reconstruction:
```{bash}
CUDA_VISIBLE_DEVICES=0 python run.py --config config/vox-256.yaml \ 
					   --mode reconstruction --checkpoint checkpoints/vox-cpk.pth.tar
```
- To train from scratch:
```{bash}
CUDA_VISIBLE_DEVICES=0 python run.py --config config/vox-256.yaml \ 
					   --device_ids 0 --mode train
```
Put the above commands in an `.sh` file to change easier. 

### Possible errors to come across during training 
Currently,  only 1 error is found: CUDA Out of memory ---> Solution: reduce batch size.

