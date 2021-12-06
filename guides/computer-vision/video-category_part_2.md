---
last_modified_on: "2021-12-06"
title: Video recognition and categorization (Part 2 - Datasets and Evaluate Metrics)
description: Analysis of video classification datasets and evaluate metrics
series_position: 7
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: basic", "guides: video_category"]
---
## Video Classification Datasets

In media-related fields of research, several scientific and research organizations have spent a lot of work obtaining and labeling video data sets. There are many popular datasets such as:
- YouTube-8M
- HCF-50
- HCF-101
- HMDB51
- ...

The tiny datasets with smaller but well-labeled total quantities and video types.
- Weizmann
- KTH
- Hollywood.

In which, UCF101, Thumbos'14, and HMDB51 are medium datasets which have over 50 photos. Big data gathering, such as YouTube 8 million (which Google gathers), Sports1 million, ActivityNet, Kinetics, and others. Table 1 summarizes more extensive facts. All but the Weizmann and KTH datasets are static, whereas the others are dynamic.

![Fig-1](https://vision.aioz.io/thumbnail/8590c829f9a14686b5ce/1024/part2-table1.PNG)  
*<center>**Table 1**: Summary of video classification datasets [(Source)](https://www.techrxiv.org/articles/preprint/Deep_Learning_for_Video_Classification_A_Review/15172920).</center>*

## Evualate Metrics

We'll go through the most common video categorization efficiency metrics in this section. Performance metrics may be used to show how well a dataset method works. Precession (precision measurements conduct positive meaning determination), Recall, and other performance analysis processes are included in the scope of video categorization (Precision tests are percentage tests for productive detection of positive result of the classifier). Some of the performance metrics used for evaluating video classification research from the most recent work on video classification:

- Accuracy
- Recall
- F1 Score
- Micro F1
- K-Fold: 3,5,10-fold

## What differentiates video classification from image classification?

When classifying images, we must consider the following steps:
1. Input a photo to CNN.
2. Obtain CNN's predictions.
3. Choose the label that has the highest probability associated with it.

Because a video is nothing more than a collection of frames, a simple video categorization strategy would be to:

1. Loop through the whole video file.
2. Each frame should be sent via the CNN.
3. Classify each frame separately and without regard to the others.
4. Choose the label that has the highest probability associated with it.
5. Write the output frame to disk after labeling it.

But there's a problem: if you've ever tried to apply simple image classification to video classification, you've probably noticed "prediction flickering". How can we avoid CNN "flickering" between these many labels?

Using a rolling prediction average is a basic yet elegant method.
1. Loop through the whole video file.
2. Each frame should be sent via the CNN.
3. Obtain CNN's predictions.
4. Keep track of the last K predictions.
5. Calculate the average of the previous K predictions and select the label with the highest probability.
6. Write the output frame to disk after labeling it.

In the next blogs, we will discuss about about **deep learning approaches** of the video classification task.

