---
last_modified_on: "2022-11-13"
title: Techniques for evaluating deep learning models.
description: The article briefly introduces methods to evaluate the accuracy of a DL model.
series_position: 26
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

# Introduce
  When building or surveying a Machine Learning or Deep Learning model, we must pay attention to the work of evaluating the performance of the model. To be able to make the most suitable choice for the job we are aiming for. In this article, we will introduce to you some evaluation methods for object classification and object detection problems.

![Figure 0](https://drive.google.com/uc?export=view&id=1J5p3j1tGUdoqk8R6cn8hH1UGEnt-Uz9L)
  
  Today, many different assessment methods have been developed for different types of problems. The important thing that you need to pay attention to is to choose the evaluation method that is suitable for the current problem. Some of the methods that we will cover are ccuracy score, confusion matrix, ROC curve, and Area Under the Curve.
  
---
# Reasons to evaluate the model
Once you've built a deep learning model and trained it on a dataset or surveyed a deep learning model, the next thing you should do is evaluate the model's performance on the new dataset.

Model evaluation helps us to solve the following problems:
   * Has the model been trained successfully?
   * How good is the success of the model?
   * When should the training be stopped?
   * When should the model be updated?

Once you answer these 4 questions, we will be able to decide if this model is really suitable for the problem.

Evaluation of a good model is usually performed on data for which the model has not been trained. The typical ratio of a training dataset to a test dataset is 70% and 30%.

We use the new data when evaluating the model to minimize the possibility of overfitting the training set. Sometimes it is useful to evaluate the model and at the same time train it to find the best indicators of a model. However, we cannot use the test suite to make this assessment. Or we will have to choose the parameters that work best on the test data, but may not be the most extensive ones.

# Training dataset division technique
Before talking about the data evaluation methods, we should also consider the problem of training datasets. Because this is an important factor to form a good model.

A typical dataset will have a ratio of 60% for the training set, 20% for the evaluation set, and 20% for the test set. It is very important that the disorder of the data set can be greatly increased when you shuffle the data before dividing the data set into many small parts. This makes it possible for each sub-data set to more clearly represent the properties of the large data set.

**1. The Hold-out method**
The hold-out method for model evaluation represents the mechanism of splitting the dataset into training and test datasets. The model is trained on the training set and then tested on the testing set to get the most optimal model.

![Figure 1](https://drive.google.com/uc?export=view&id=1sSbfO1Gqrt5GkuKZ3b01JoRjPyrEjKc4)*<center>**Figure 1**. illustration of hold-out method.</center>*

In the above diagram, you may note that the data set is split into two parts. One split is set aside or held out for training the model. Another set is set aside or held out for testing or evaluating the model. The split percentage is decided based on the volume of the data available for training purposes. 

However, there is always a possibility that trying to use this technique can result in the model fitting well to the test dataset. In other words, the models are trained to improve model accuracy on the test dataset assuming that the test dataset represents the population. 

**2. Stratified sampling**

Stratified Sampling is a sampling method that reduces the sampling error in cases where the population can be partitioned into subgroups. 

![Figure 2](https://drive.google.com/uc?export=view&id=1wryiL9rCuk-kVLSrJL3WLqKt0QQ4R3rR)*<center>**Figure 2**. illustration of stratified sampling method.</center>*

Stratified Sampling ensures each group within the population receives the proper representation within the sample. When the population can be partitioned into homogeneous subgroups, this technique gives a more accurate estimate of model parameters than random sampling.

However, simple random sampling is more advantageous when the population can’t be divided into subgroups, since there are too many differences within the population.

**3. Cross validation**

Cross-validation is a statistical method used to estimate the skill of machine learning models.

It is commonly used in applied machine learning to compare and select a model for a given predictive modeling problem because it is easy to understand, easy to implement, and results in skill estimates that generally have a lower bias than other methods.

Many cross-validation techniques define different ways to divide the dataset at hand. We’ll focus on the two most frequently used: the k-fold and the leave-one-out methods.

**3.1 K-Fold Cross-Validation**
![Figure 3](https://drive.google.com/uc?export=view&id=1PfIYyg7LVHS_LGyGaPJUZFas-dcTk1WP)*<center>**Figure 3**. illustration of K-Fold Cross-Validation method.</center>*
In k-fold cross-validation, we first divide our dataset into k equally sized subsets. Then, we repeat the train-test method k times such that each time one of the k subsets is used as a test set and the rest k-1 subsets are used together as a training set. Finally, we compute the estimate of the model’s performance estimate by averaging the scores over the k trials.

**3.2 Leave-One-Out Cross-Validation**
![Figure 4](https://drive.google.com/uc?export=view&id=1FZR4eW-GKE52aUsQEWfAF8EpvjqMpBnJ)*<center>**Figure 4**. illustration of leave-one-out Cross-Validation method.</center>*

In the leave-one-out (LOO) cross-validation, we train our machine-learning model n times where n is to our dataset’s size. Each time, only one sample is used as a test set while the rest are used to train our model.

**4. Bootstrap sampling**
The bootstrap method is a resampling technique used to estimate statistics on a population by sampling a dataset with replacement.

It can be used to estimate summary statistics such as the mean or standard deviation. It is used in applied machine learning to estimate the skill of machine learning models when making predictions on data not included in the training data.

![Figure 5](https://drive.google.com/uc?export=view&id=1rYN2aZPhWWQKiI8tR79KOvnjkFiuSCFo)*<center>**Figure 5**. illustration of leave-one-out Cross-Validation method.</center>*

A desirable property of the results from estimating machine learning model skill is that the estimated skill can be presented with confidence intervals, a feature not readily available with other methods such as cross-validation.

# Evaluation Methods

**1. Accuracy**
The simplest evaluation method today can be visualized by comparing the predicted value with previously labeled values to show the performance of the model.

```
    import numpy as np 

    def acc(y_true, y_pred):
        correct = np.sum(y_true == y_pred)
        return float(correct)/y_true.shape[0]

    y_true = np.array([0, 0, 0, 0, 1, 1, 1, 2, 2, 2])
    y_pred = np.array([0, 1, 0, 2, 1, 1, 0, 2, 1, 2])
    print('accuracy = ', acc(y_true, y_pred))
```
The calculation using accuracy as above only tells us what percentage of the data is correctly classified without specifying how each type is classified, which class is classified most correctly, and data belonging to one class is often misclassified into another class. To be able to evaluate these values, we use a matrix called confusion matrix.

**2. Confusion matrix**
Basically, the confusion matrix represents how many data points actually belong to a class, and are expected to fall into a class. For better understanding, see the table below:

```
    def my_confusion_matrix(y_true, y_pred):
        N = np.unique(y_true).shape[0] # number of classes 
        cm = np.zeros((N, N))
        for n in range(y_true.shape[0]):
            cm[y_true[n], y_pred[n]] += 1
        return cm 

    cnf_matrix = my_confusion_matrix(y_true, y_pred)
    print('Confusion matrix:')
    print(cnf_matrix)
    print('\nAccuracy:', np.diagonal(cnf_matrix).sum()/cnf_matrix.sum())
```
```
    Confusion matrix:
    [[ 2.  1.  1.]
     [ 1.  2.  0.]
     [ 0.  1.  2.]]

    Accuracy: 0.6
```
Confusion matrices are often illustrated in color for a clearer view. The code below helps to display the confusion matrix in both forms.

![Figure 6](https://drive.google.com/uc?export=view&id=1iZzU3KIxlUS2-Gz1s9bf75CIc1coTLEh)*<center>**Figure 6**. illustration of confusion matrices method.</center>*

**3. Precision and Recall**
For classification problems where the data sets of the classes differ greatly (unbalanced), the Precision-Recall evaluation method is often used.

![Figure 7](https://drive.google.com/uc?export=view&id=15kKGRxjSyDGBuupXLdvOC4XZYAKDE6fl)*<center>**Figure 7**. illustration of Precision and Recall.</center>*

Precision is defined as the ratio of true positive samples among those classified as positive (TP + FP). With precision = 0.9 it means that the model correctly predicts 90 out of 100 samples the model predicts positive.

Recall is defined as the ratio of true positive samples among the points that are actually positive (TP+FN). With recall = 0.9, it means that the model that correctly predicts 90 samples out of 100 is actually positive.
```
    from __future__ import print_function
    import numpy as np 
    # confusion matrix to precision + recall
    def cm2pr_binary(cm):
        p = cm[0,0]/np.sum(cm[:,0])
        r = cm[0,0]/np.sum(cm[0])
        return (p, r)

    # example of a confusion matrix for binary classification problem 
    cm = np.array([[100., 10], [20, 70]])
    p,r = cm2pr_binary(cm)
    print("precition = {0:.2f}, recall = {1:.2f}".format(p, r))
```
```
    precition = 0.83, recall = 0.91
```
Note: High precision means high accuracy of true samples. High recall means that the miss of really positive samples is low. A good classification model is one that has both Precision and Recall high, i.e. as close to one as possible.

In addition to balancing Precision and Recall, F-1 score is also used, which is the harmonic mean of precision and recall, with the same importance as with FNs and FPs. Specifically:

![Figure 8](https://drive.google.com/uc?export=view&id=14j-cpNftgZos7ncOBhuubd9z9ASha98k)*<center>**Figure 8**. illustration of leave-one-out F1 Score method.</center>*


The higher the F-1 score, respectively, the higher the precision and recall, the better the classification model.

**4. AUC và ROC**
ROC (Receiver operating characteristic) is a commonly used graph in the validation of binary classification models. This curve is generated by plotting the forecast true positive rate (TPR) based on the predicted failure positive rate (FPR) at different Threshold thresholds.
* TPR (True Positive Rate): Also known as recall or sensitivity. Is the ratio of cases that are correctly classified as positive to the total number of cases that are actually positive. This index will evaluate the accuracy of the model's prediction on positive. The higher its value, the better the model predicts on the positive group. If , TPR = 0.9, we believe that 90% of samples in the positive group have been correctly classified by the model.
![Figure](https://drive.google.com/uc?export=view&id=1yjujwMgw9w-r4McR8AUvBW8QCNh6TYgP)
* FPR (false positive rate): The ratio of falsely predicting cases that are actually negative to positive out of the total number of cases that are actually negative. If the value of FPR = 0.1 , the model incorrectly predicted 10% of the total cases to be negative. The lower the FPR of a model, the more accurate the model is because its error on the negative group is lower. The FPR's complement is the specificity that measures the ratio of correctly predicting negative cases to the total number of actual negative cases.
![Figure](https://drive.google.com/uc?export=view&id=1Bq3GiSsAVEiq8RYS0G-6DkOXPVD2Rxlu)

The ROC graph is a convex curve based on TPR and FPR with the shape as below:
![Figure 9](https://drive.google.com/uc?export=view&id=1GbXLrs-2KrX6jpSxa8YHHb0jd5BYSFfv)*<center>**Figure 9**. illustration of ROC graph.</center>*

AUC is an index calculated based on the recieving operating curve (ROC) to assess the classification ability of the model. The diagonal area under the ROC curve and on the horizontal axis is AUC (area under curve) with a value in the range [0, 1]. When this area is larger, the ROC curve tends to be asymptotic to the straight line and the better the model's classification ability. When the ROC curve is close to the diagonal passing through the two points (0, 0) and (1, 1), the model is equivalent to a random classifier.

We can calculate the AUC as follows:
```
    from sklearn.metrics import auc, roc_curve
    fpr, tpr, thres = metrics.roc_curve(y_label, y_pred)
    
    # Calculate auc
    auc(fpr, tpr)
```
Represent them on the thalamus by:
```
    def _plot_roc_curve(fpr, tpr, thres):
        roc = plt.figure(figsize = (10, 8))
        plt.plot(fpr, tpr, 'b-', label = 'ROC')
        plt.plot([0, 1], [0, 1], '--')
        plt.axis([0, 1, 0, 1])
        plt.xlabel('False Positive Rate')
        plt.ylabel('True Positive Rate')
        plt.title('ROC Curve')

    _plot_roc_curve(fpr, tpr, thres)
```
# References
[1] [Accuracy, Precision, Recall or F1?](https://towardsdatascience.com/accuracy-precision-recall-or-f1-331fb37c5cb9)

[2] [Sklearn: Model Selection](https://scikit-learn.org/stable/auto_examples/model_selection/)

[3] [Understanding AUC - ROC Curve](https://towardsdatascience.com/understanding-auc-roc-curve-68b2303cc9c5)


































































