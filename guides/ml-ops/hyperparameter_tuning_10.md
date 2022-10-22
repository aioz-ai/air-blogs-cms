---
last_modified_on: "2022-10-16"
title: Model Compression Part 7.
description: Systematic and Efficient Hyperparameter Tuning.
series_position: 10
author_github: https://github.com/aioz-ai
tags: ["type: practical", "level: medium"]
---
## 1. Goal
Hyperparameter Tuning is an optimization task that can be seen as a regression task itself, where the variables are the hyperparameters to be tuned and the optimisation function is to minimise the validation loss. It is a time- and resource-demanding task, thus allowing engineers plenty of room to explore what can be improved. While many algorithms have been developed to search for the best configurations (e.g. Bayesian Optimization, Grid Search, Random Search, etc.), in this article we will focus more on a systematic and efficient workflow that can potentially save engineers lots of time and computations. We will also look at some practices to manage your workings and some intriguing tips that can potentially save training time. 

##  2. Workflow
The performance of a machine learning model, of course, largely depends on the combination of hyperparameters used. At the start of my training task, I knew about grid search, random search, and bayesian optimization, but I didn't have a clear workflow to follow. After many experiments and consulting with colleagues, my workflow slowly came about as follows: 

###  What was my workflow?
1. Decide the hyperparameters to tune. The ones  I went with were batch size, learning rate, learning rate scheduler, and optimizer. 
2. Start off with a random setup and train until convergence. After this point, we will have a general behaviour of the dataset and the training task we have defined, e.g. the minimum range at which train_loss and val_loss can reach and the significant phases of the training duration. More experiments here can help us eliminate some variables such as learning rate scheduler and optimizer. 
	
	For example, at this step, I figured that the AdamW optimizer can reach smaller val_loss compared to SGD and Adam and that a learning rate scheduler of [60, 90] works best. Note that this finding is incredibly subjective to the task, the dataset, and the range of hyperparameters we tested with. 
3. For the remaining hyperparameters, use Bayesian Optimization with a large range to find a smaller range for later grid search.

	 For example, I set the range for learning rate at this step to be from 1e-5 to 1e-2 and the range for batch size to be from 4 to 512. Using [Ax from Facebook Open Sources](https://ax.dev/docs/bayesopt.html), I found a smaller and more suitable range for learning rate to be 5e-4 to 1e-3, and batch size to be from 8 to 64. Given the limited resources and time, I used half the amount of data and trained for only 30 epochs for each experiment. 
5. Grid search with Early Stopping with half amount of data (split to train & val) to find the best 3 setups. Train until convergence. 
6. For the best 3 setups, train the full amount of data (spit to train & val) and get the epoch at which the best val_loss was reached. 
    
    For example, I have found that learning rate = 5e-4 and batch size = 8 works best for my task and that the smallest val_loss was reached at epoch 91.   
7. With the best setup in step 6, train the full amount of data (this time no longer split to train & val, just one training set so that we can take advantage of the all available data) until the best epoch as found in step 6. Save the weights at the best epoch for future inference/training.

### What would I do differently?
In all honesty, my workflow was not as one-way as I had written out. To reduce this back-and-forth nature of hyperparameter tuning works, it is important to **have a roadmap beforehand and follow it closely and chronologically**. As I was observing a training procedure (perhaps too closely), it was tempting to stop training prematurely and test a new set of hyperparameters once I saw training has slowed down or val_loss has spiked. However, this practice is subjective and does not allow one combination to reach its best performance. For a more systematic approach, I have learned to **set up every experiment one by one in a `.sh` file according to the roadmap and let it run**, while I can do other tasks.

After building the roadmap, it is also important to **have a proper metadata management system ready**. This promotes systematic testing, reduces error or missing data, and allows objective observation of data. More about this is in section 4.

One thing I would change about my workflow is to remove step 3 and add more hyperparameter combinations in step 4. While this approach may take more time,  it allows more parameter combinations to reach their best performance, hence we can compare them more objectively. 

## 4. Good practices to have

One thing I learn from this experience is that organisation are 100x more important in hyperparameter tuning than anything else. Here are some of the good practices I have learned:

1. Training code with **little to none hard-coded filepath or variables**  
It is utmost important to have a consistent code file that allows for customisation of hyperparameters without direct change in training code. This requires the code to have little to none hard-coded filepath or variables (e.g. batch size, learning rate, etc.). Changes in parameters should be declared using `argparse` only. Manual changes in code are heavily prone to error and time-consuming. 


2. **Saving checkpoints frequently**   
For every experiment done, we need to save checkpoints frequently throughout the training schedule. This will save time down the run. 


3. **Metadata management**: consistent and detailed tracking of experiments.   
The information to log here includes training settings, data used in train and validation set, evaluation metrics, and hardware metrics. Logging is incredibly important to compare different training settings, and these differences can guide us which setting to study next, thus improve productivity and make your work reproducible. Having a dedicated Logger class is a good idea. 


## 5. Some points of attention when training data 
### Efficient DataLoader
When we have large amount of data and that it is saved into multiple pickle files, we essentially load data into memory every epoch and this poses a large amount of data transfer between CPU and GPU. [[1]](https://medium.com/towards-data-science/gpus-are-fast-datasets-are-your-bottleneck-e5ac9bf2ad27) suggests setting the `num_workers` argument to be some multiplier over the number of GPUs and [[2]](https://medium.com/towards-data-science/7-tips-for-squeezing-maximum-performance-from-pytorch-ca4a40951259) suggests to set the argument `pin_memory=True`. 


### Batch size/learning rate schedule  
Smith et al. [[3]](https://arxiv.org/pdf/1711.00489.pdf) observed that a combination of decreasing the learning rate by $\alpha$ and increasing the batch size by $\alpha$ achieves a very similar result on validation loss with a significantly smaller number of parameter update, comparing to that of just decreasing the learning rate. Thus, will save a lot of training time. To test this approach, we perform two experiments:
   1. **Decaying learning rate**: Training with a [40,60] scheduler where at epoch 40, 60, and 90, the learning rate with be reduced by half. Batch size is kept constant throughout the whole schedule. 
   2. **Hybrid schedule**: Training with a [40,60] scheduler where at epoch 40, batch size will be increased by two while learning rate is kept constant, and at epoch 60 and 90, the learning rate is reduced by half while batch size stays the same. 
  
![increase_batch_size](https://drive.google.com/uc?id=1qZh_5q403v20xorQkPMlv3sKO6THBwGO)
*<center>**Figure 1**: Train loss (left) and validation loss (right) of the knowledge distillation task on VoxCeleb data. </center>*

Figure 1 shows that in both train_loss and val_loss, the two schedules match each other closely. For the hybrid schedule, as the batch size doubles at epoch 40, we know that the number of parameter updates decreases by two from epoch 40 onwards. This result is indeed promising and should be considered for future training tasks. 

### Argparse
 When using `argparse`, consider not using boolean arguments as it will lead to unexpected behaviours where [both True and False will be interpreted as True](https://gist.github.com/nanaze/db63e3f63e318408e3223bf1245d9752). 
	
**Solution**: Use `int` instead, with `type=int` clearly defined when adding arguments. For example, instead of

```{python}
parser.add_argument("--example", default=False, type=bool)
```
use
```{python}
parser.add_argument("--example", default=0, type=int)
```

### Training time increases after each epoch
This might be due to low GPU utilisation or bottleneck due to CPU/GPU transfer. Aside from looking for the bottleneck, we can also use `.detach()` when storing loss values to avoid storing computation graphs throughout the epoch. 


## Reference 

[1] W. Falcon, "GPUs Are Fast! Datasets Are Your Bottleneck", _Towards Data Science_, 2021. [Online]. Available: https://towardsdatascience.  com/gpus-are-fast-datasets-are-your-bottleneck-e5ac9bf2ad27.   
[2] W. Falcon, "7 Tips To Maximize Performance From PyTorch", _Towards Data Science_, 2020. [Online]. Available: https://medium.com/towards-data-science/7-tips-for-squeezing-maximum-performance-from-pytorch-ca4a40951259.  
[3] S. Smith, P. Kindermans, C. Ying and Q. V. Le, "Don't Decay the Learning Rate, Increase the Batch Size", _Sixth International Conference on Learning Representations_, 2018. Available: https://arxiv.org/abs/1711.00489.