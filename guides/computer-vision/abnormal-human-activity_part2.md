---
last_modified_on: "2022-04-18"
title: Abnormal Human Activity Recogition (Part 2 - Two-Dimensional AbHAR)
description: A review about Abnormal Human Activity Recognition.
series_position: 16
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: abnormal_activity_recognition"]
---

# Two-Dimensional AbHAR
## Single person AbHAR

Ambient Assistance Living (AAL), home monitoring when a single person is under observation for abnormal activity identification, is one of the principal applications of Single Person AbHAR. As a result, we've included single-person AbHAR techniques that assist aged health-care difficulties and AAL in this section.


![Fig-1](https://vision.aioz.io/f/7da620e2b07b4c0b8054/?dl=1) *<center>**Table 1**:  Scene representation & feature abstraction techniques for two-dimensional single person abnormal human activity recognition (AbHAR) approaches [Source](https://www.sciencedirect.com/science/article/abs/pii/S0952197618301775).</center>*

Table 1 summarizes multiple ways for single person activity identification based on two-dimensional basics. For a better understanding of the process, it shows scene representation and feature extraction methodologies with individual important parts and constraints.

Table 1 shows that the LOTAR framework provides a more robust feature representation platform for AAL applications, which collect data from many sensors such as temperature, pressure, and RFID sensors, as well as vision sensors, to investigate both short and long term abnormalities. The framework is used in a genuine patient home for testing, but it has to be extended to numerous patients for actual outcomes. In the work (Huang et al., 2014), the person's habit is researched and analyzed for the first time by combining ISUS (Intelligent space for understanding and service) with a multi-camera placement algorithm. Single Person, AbHAR methods are addressed in length below, with silhouette based, spatio-temporal based, and some miscellaneous techniques being classified as silhouette based, spatio-temporal based, and some miscellaneous approaches.

### Silhouette-based approaches:

Any AbHAR method's robustness is determined by how well action evidence in an image is extracted as a feature. R-Transform has been utilized to tackle geriatric health care challenges and provide a very practical solution (Khan and Sohn, 2013, 2011). There are six atypical activities: (i) fainting; (ii) backward; (iii) forward falling; (iv) vomiting; (v) chest discomfort and (vi) headache. It used R-Transform in conjunction with Kernel Discriminant Analysis (KDA) to reduce the interclass similarity of different activity postures as binary silhouettes while ensuring individual privacy. They devised a two-level hierarchical anomalous human activity recognition system to boost recognition rates for intra-class actions, notably for forward-vomiting and backward-fainting postures. The first level applies KDA on binary silhouettes' R-Transform. A k-mean clustering approach is used at the second level to create groups of comparable postures, and a four-state HMM classifier is used in the training and recognition of the activities.

In a limited setting, this hierarchical strategy boosted the average detection rate of comparable behaviors to 97.1 percent, but it still has to be expanded with a real-time anomalous activity dataset with noise. When a binary silhouette fails to depict a posture due to self-occlusion, depth silhouettes can be used to overcome this constraint. Early identification of a person's short- or long-term aberrant behavior while using AAL (Ambient Assistive Living) is developing as a potential field of abnormal activity recognition. A FABER hybrid technique (Riboni et al., 2015) is presented with the help of medical models provided by cognitive neuroscience researchers, which focuses on abnormal activity routines to identify symptoms of mild cognitive impairment (MCI) at a fine-grained level with the assimilation of supervised learning and symbolic reasoning, i.e. problems with memory, language, thinking, and judgment, which is important information for early detection of MCI.

Individual privacy is generally limited in closed-environment camera or sensor-based surveillance systems (Crispim Junior et al., 2012). Low-level behavioral indicators such as walking pace and steps taken are monitored, but fine-grained depictions of action during the execution of Instrumental Activities of Daily Living are not obtained (IADL). Some methods (Sacco et al., 2012) have a large implementation cost and cannot be used in the long run. Riboni et al. (2016) created the LOTAR framework (see Fig. 6) as a hybrid behavior analysis system that integrates knowledge-based and data mining based machine learning algorithms for long-term aberrant behavior identification. It collects data from non-intrusive sensors installed in the patient's house, maintaining the patient's privacy and providing fine-grained process statistics necessary for detecting long-term aberrant behavior.

### Spatio-temporal based approaches.

The usage of spatio-temporal representation to characterize the activities in a video series for normal/abnormal human action recognition has been seen. For anomaly identification and localization from collected video volumes,

To locate and detect the anomalous region effectively, an unsupervised statistical learning framework is constructed that excavates global activity patterns and local conspicuous behavior patterns using clustering and sparse coding, respectively, coupled with multiple scale analysis. However, in order to provide a viable solution in real-time applications, the codebook and dictionary learning algorithm must be updated for fresh input movies during this process. The Phase Spectrum of Quaternion Fourier Transform (PQFT) (Guo et al., 2008) has been found as a promising approach for detecting spatiotemporal salience in video frames. PQFT outperforms previous approaches such as spectral Residue (SR), Phase Spectrum of Fourier Transform (PFT), Neuromorphic Vision Toolbox (NVT), and Saliency Toolbox (ST) in terms of computing complexity, noise resistance, and speed (SBT). 

In the presence of whitecolored noise, the addition of a motion cue to PQFT improves the depiction of spatiotemporal saliency in movies. The technique, however, fails to detect saliency when the noise is identical to a salient item. PQFT is unable to detect the image's closing patterns, which people can recognize in a matter of seconds. Vishwakarma et al. (2016b) established an activity recognition framework based on baseline information such as object shape and motion direction. To extract binary silhouette, a texture-based segmentation approach and spatial edge distribution of gradients are utilized. However, to give a solution for real-life applications, the work must be tested in a more sophisticated and realistic setting.

### Miscellaneous approaches.

The present approaches for detecting anomalous human behavior (classification-based, clustering-based, and statistical methods) are currently confronting the difficulty of making the system independent of human arbitrators. Candás et al. (2014) used the Jerk based Inactivity Magnitude (JIM) feature to try to detect human behavior abnormality autonomously, without making any assumptions. The experimental work, on the other hand, has to be expanded to include additional people and situations. For the first time, Huang et al. (2014) used an intelligent space for understanding and user service (ISUS) for abnormal habit recognition, combining a multicamera positioning algorithm with the concept of time spent at different duration histograms of key points in the intelligent system under surveillance for the entire day to locate the person, making the habit recognition task easier. However, this model's use is confined to the ISUS environment to monitor the habits of the elderly, which may be beyond an individual's financial means. There is a need to build low-cost fall detectors for smart homes based on artificial vision algorithms that are inexpensive to the average person (Miguel et al., 2017).

In the next blogs, we will discuss about "Multiple persons AbHAR" in details. 
