---
last_modified_on: "2022-06-20"
title: Abnormal Human Activity Recognition (Part 4 - Three-Dimensional AbHAR)
description: A review about Abnormal Human Activity Recognition (3D AbHAR) (cont.)
series_position: 19
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: abnormal_activity_recognition"]
---
# Three-Dimensional AbHAR

When compared to one-dimensional data processing, multidimensional data processing always produces more accurate and exact outcomes. This fact has always drawn researchers to 3D and 4D data. Because of the enormous popularity of Microsoft depth sensors in recent years, research on aberrant Human Activity employing depth information has blossomed. A slew of new datasets have given researchers unprecedented possibilities to develop fresh designs and test them against a larger variety of sequences. We may create picture depth information as well as a skeleton view with each joint location in a three-dimensional coordinate system. Some intriguing applications have been created by utilizing image depth and skeleton modeling.

In this blog, we will dicuss more about the "Depth based AbHAR" aspect.

# Depth based AbHAR

Depth images not only simplify and accelerate low-level image processing, but they also produce better processing results for background removal, object motion detection, and localisation. The next series discusses and summarizes numerous common depth-based techniques in Table 1, highlighting the important contributions and limitations of these methodology with year of publication. It is discovered that depth-silhouette based statistics such as height to width ratio, centroid, and silhouette shape deformations are widely utilized to extract aspects of people in motion. Human motion and form variation characteristics can deal with realistic obstacles including occlusion and many views.

![Table-1](https://vision.aioz.io/thumbnail/ae9f4caa0962446db65f/1024/part4-table1.png) *<center>**Table 1**:  Techniques for feature extraction and representation for three-dimensional depth-based fall detection.</center>*

Vision-based AAL systems are excellent instances of depth-based anomaly detection, which assists the elderly in conducting daily living tasks by utilizing depth information. To create depth-based AAL solutions, a large amount of work has been done ([Tran et al., 2014](https://ieeexplore.ieee.org/document/6916752); [Rashidi and Mihailidis, 2013](https://ieeexplore.ieee.org/document/6399501)). [Jyothilakshmi et al, 2016](https://ieeexplore.ieee.org/document/7754775) expanded the Ambient notion of patient assistance through the use of a Smart Ward System (SWS) and a Kinect sensor camera. In existing hospital patient support systems, the patient is given a remote control to fine-tune the bed and summon a nurse for help. Because of the costs needed, many hospitals do not have this automation system, and traditional gesture recognition systems rely on wearable sensors. The Smart Ward System idea enables the hospital workforce to focus on patients by collecting data at the bedside and eliminating the possibility of duplication and double-handling, as well as security concerns regarding data storage in the cloud. HMM model is utilized with depth features in [Uddina et al, 2011](https://journals.sagepub.com/doi/10.1177/1420326X10391140); [Triantafyllou et al, 2016](https://www.sciencedirect.com/science/article/pii/S2405896316324752) to produce a real-time solution that generates a warning alert for the person walking in a room, which can be expanded to other household activities in the drawing room, living room, and kitchen. Rather to PCA, ICA aids in the extraction of local features of the human gait depth silhouette. Local Directional Patterns (LDPs) are utilized to extract local characteristics from depth silhouettes in [Uddin et al, 2014](https://journals.sagepub.com/doi/abs/10.1177/1420326X14522670), generating better gait recognition rate using HMM model. LDP edge responses are computed in eight directions using Kirsch edge masks

Various approaches for fall detection have been described, highlighting this serious health risk. Few studies have shown that falls are the leading cause of hospitalization for elderly, disabled, or overweight/obese adults. [Yu, 2008](https://ieeexplore.ieee.org/document/4600107) conducts a thorough review on fall detection, focusing on finding concepts and methods to current work and literature. As illustrated in Fig. 1, it grouped fall detection systems into three categories:
* wearable devices
* ambient devices
* vision-based techniques.

![Fig-1](https://vision.aioz.io/thumbnail/8483eed0c6a64154a480/1024/part4-figure1.PNG) *<center>**Figure 1**:  An ordered structure of fall detection methods for elderly and patients.</center>*

Most wearable devices are cost-effective and simple to use, but they may not be appealing to wearers. Furthermore, this wearable device-based strategy is prone to a high rate of false alert. Pressure, vibration, acoustic sensors, and stabile-meters are often used in ambient device-based techniques to capture information about the person in their operating range. As a result of environmental variables, these sensors have a restricted range of use and are noise sensitive. In compared to wearable devices and ambience devices, the vision-based method has an advantage over others in that it is not obtrusive to the individual being monitored. They are classified into three sub-categories:
* Shape Change Analysis (SCA)
* Inactivity Analysis (IA) (SCA)
* Head Motion Analysis (HMA).

Table 2 provides an overview of wearable devices and 2D and 3D vision-based fall detection systems. It also demonstrates the conditions under which wearable and 2D & 3D vision-based system performances are good and unsatisfactory. The 3D vision system can identify a fall between two people with maximum accuracy using any combination of fall features such as absence of major movement, laying posture, lying on the ground, vertical speed, impact shock, and body shape change.

![Table-2](https://vision.aioz.io/thumbnail/dcb5e83ba6f54552bde1/1024/part4-table2.PNG) *<center>**Table 2**:  Effectiveness of wearable devices and visual systems for fall detection.</center>*