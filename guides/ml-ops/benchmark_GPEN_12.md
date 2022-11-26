---
last_modified_on: "2022-11-13"
title: Benchmarking of GPEN, an image restoration method.
description: Benchmarking of GPEN, an image restoration method.
series_position: 12
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: medium"]
---

## Runtime

Image size: (512, 512, 3)  
Pytorch==1.12.1, Torchvision==0.13.1

### CPU

CPU config: Intel Xeon(R) CPU E5-2640 v3 @ 2.60GHz × 16

| Workflow                    | GPEN (original repo) | Re-arranged version |
|-----------------------------|----------------------|---------------------|
| Real-ESRGAN                 | 1.1215               | 1.0924              |
| Pre-processing              | 0.2390               | 0.0388              |
| Restoration                 | 1.9060               | 1.3512              |
| Post-processing             | 1.6701               | 0.0254              |
| Total (without Real-ESRGAN) | 3.8151               | <b> 1.4153 </b>     |  
| Total                       | 4.9369               | 2.5078              | 


CPU config: 2.8 GHz Quad-Core Intel Core i7

| Workflow                    | GPEN (original repo) | Re-arranged version |
|-----------------------------|----------------------|---------------------|
| Real-ESRGAN                 | 4.2515               | 4.0423              |
| Pre-processing              | 0.8013               | 0.0645              |
| Restoration                 | 6.8322               | 5.3539              | 
| Post-processing             | 6.3969               | 0.0186              |
| Total (without Real-ESRGAN) | 14.0304              | <b> 5.4370 </b>     |
| Total                       | 18.2923              | 9.4841              |  

### GPU
GPU config: NVIDIA TITAN V/PCIe/SSE2

| Workflow                    | GPEN (original repo) | Re-arranged version |
|-----------------------------|----------------------|---------------------|
| Real-ESRGAN                 | 0.0421               | 0.0430              |
| Pre-processing              | 0.0321               | 0.0281              |
| Restoration                 | 0.0589               | 0.0558              | 
| Post-processing             | 0.1389               | 0.0270              |
| Total (without Real-ESRGAN) | 0.2299               | <b> 0.1109 </b>     |
| Total                       | 0.2722               | 0.1539              |  

## CPU Memory usage 
Memory usage was tested by iterating through all 3000 samples of the [CelebA-LQ dataset](https://xinntao.github.io/projects/gfpgan). All images are (512,512,3). PID was used to measure the memory usage of the python script.

CPU config: Intel Xeon(R) CPU E5-2640 v3 @ 2.60GHz × 16

| Model                             | CPU Memory usage                                       | Visualisation                                                                        | 
|-----------------------------------|--------------------------------------------------------|--------------------------------------------------------------------------------------| 
| GPEN (with Real-ESRGAN)           | starts at 5.43GB, slowly increases, can go upto 5.70GB | [link](https://drive.google.com/file/d/1midTZw-8elGO2LNEd1W8TgX1JC-HdKPE/view?usp=share_link) | 
| GPEN (without Real-ESRGAN)        | starts at 5.43GB, slowly increases, can go upto 5.75GB | [link](https://drive.google.com/file/d/1mAZriWfzk_vG5wyhA92xQrLwbOkzDk1C/view?usp=share_link)  |   
| Re-arranged (with Real-ESRGAN)    | ~5.32GB, no sign of memory leakage                     | [link](https://drive.google.com/file/d/1TxS6HUPJfrFMUP9wmq-qFpJHQLaWcnx1/view?usp=share_link) | 
| Re-arranged (without Real-ESRGAN) | ~5.13GB, no sign of memory leakage                     | [link](https://drive.google.com/file/d/1Uv2FFpnDFZZGKdP5OX9j8T6U7LdF4F7R/view?usp=share_link) | 

## Performance 

| Input | GPEN (w/o Real-ESRGAN) | GPEN (w/ Real-ESRGAN) | Re-arranged version |
| --- | --- | --- | --- |
| ![](https://drive.google.com/uc?export=view&id=1zFYgWJJhX5jkTeSVvf7zfoYgqdv7hswf) | ![](https://drive.google.com/uc?export=view&id=1pOepnGzz8FLyBCf61NqROeLz4kMUCi0F) | ![](https://drive.google.com/uc?export=view&id=1XP9kjEYqhAbEj2j9MJe4OzZXHgd4Wdl_) | ![](https://drive.google.com/uc?export=view&id=1J7mb9fGhaBHyrcAwiFysLinVasX_3s0Y) | 
| ![](https://drive.google.com/uc?export=view&id=1fkv1lNtYBi05RjAk77eTI6htxGc16AGx) | ![](https://drive.google.com/uc?export=view&id=1JWABxjTbvO-1gSWBbBuQxbdEmMwwhYBp) | ![](https://drive.google.com/uc?export=view&id=1IWotCfv9QCifk7TOswU4swlyPaKQzjDq) | ![](https://drive.google.com/uc?export=view&id=1fNOox925iypEi2enM4UubPLQ8FbjDVBN)



