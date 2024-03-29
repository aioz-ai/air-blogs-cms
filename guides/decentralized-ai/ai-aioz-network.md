---
last_modified_on: "2021-05-23"
title: Intelligent Agents in AIOZ Network
description: Transforming the AIOZ Network with AI & Robotics
series_position: 1
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium", "guides: decai_aioznetwork"]
---

Decentralized AI and robotics have the potential to transform a wide range of applications, from conventional use cases like sensing & perception, data (image/video/text) analytics; to more sophisticated tasks like targeted material delivery, assisted living, and disaster relief.

With the help of technological advances such as cloud computing, innovative hardware design, and manufacturing techniques, AI (virtual) and robotics (physical) are becoming more prevalent in areas such as human-machine interaction, logistics, autonomous transportation, healthcare, or complex manufacturing sectors like aerospace. However, essential issues in the fields of data privacy, security, safety, and transparency, on the other hand, could pose a barrier to the future use of AI and robotics in high-sensitive scenarios. As a result, innovative solutions to these problems could be necessary steps on the way to widespread adoption.

Blockchain, an evolving technology that emerged in the time-stamping field in the 1990s and has recently been applied to digital currency, shows how agents can conclude agreements in a transparent and decentralized manner without the need for a controlling authority by integrating peer-to-peer networks with cryptographic algorithms.

**At AIOZ, we want to learn more about the possibilities of combining physical and virtual autonomous agents with blockchain-based technology beyond the conventional view of distributed and decentralized AI and robotics. The following are some of our thoughts and insights on the topic.**

## I. Decentralized AI
### 1. Decentralized AI System  
A decentralized AI system opens a new era for remote learning. It removes the barrier between companies and research scientists and allows all users, who know nothing about AI, to join the AI community.  
**Scenario:** In general, there is a big gap between 3 groups below:  
- **Companies**: who have a lot of data and the practical problems that need to be solved but are not allowed to share this information.  
- **AI Scientists and Researchers**: willing to investigate and solve the problems but may not get enough data or strong computation hardware to design learning methods.  
- **Non-AI Users**: who know nothing about AI and Deep learning but have hardware resources to support two other groups of users. We call them **hardware supporters**.  


**Solution from the introduced scenario**: We need:  
- A learning method that can leverage multiple hardware from Non-AI users.  
- A communication platform that connects companies and AI Scientists. The platform must have an option to (i) preserve the data for companies; (ii) preserve the techniques for AI Scientists; (iii) connect both mentioned users to the appropriate hardware supporters.
- An app that supports both companies and AI Scientists to release their works as API solutions. Other users can use these APIs that have required inputs to test or apply other industrial processes. A system that takes comments and feedback from users should be established to help companies and AI Scientists improve their solutions.   

**Highlight Characteristic**: Compared to other systems:  
- Companies and AI Scientists can work with many partners simultaneously (but may be limited by working sessions identified by managers to prevent overloading).  
- Companies and AI Scientists are connected by a task trading floor. In the task trading floor, inputs and outputs are clarified by following designed template formats. This can help the dealing process more accurately and faster for both mentioned groups of users.  
- Work under a Decentralized Infrastructure; there is no limit in resource scaling from hardware supporters.  

**Current unsolved problem**:  
- A part of data and a fragment of training codes will be held at each hardware to support the training process. This characteristic goes against private policy. However, we currently do not have a solution to deal with this.  


### 2. Decentralized AI System: Components  
These components are general sketches. The details for them will be in other parts:   
#### 2.1. Decentralized Federated Learning  
A policy where you can train your model using resources of multiple nodes.  

#### 2.2. Decentralized Cloud Platform  
A connection platform where anybody can host their own server.  

AIOZ Network can be the bridge between **researchers** who have ideas and proposals but do not have enough resources and **companies** who have resources and data but do not have solutions for their problems.  

#### 2.3. AI and Data Exchange
A place where AI solutions can be found, which may help the practical processes better. The market may contain these features:  
- APIs of different AI tasks. It can be used directly when the inputs required are met.  
- Pretrained models and deep networks: For supporting the online fine-tuning process; have different optional modifications for the fine-tuning process.  
- Deep networks: support pure networks for training from scratch, which different optional modifications.  
- Buying/Selling data, models, etc.  
- Challenges can also be integrated here.

## II. Application in AIOZ Network
### 1. Video recognition and categorization
![Fig-1](https://pyimagesearch.com/wp-content/uploads/2019/07/keras_video_classification_sports_dataset.jpg)  
*<center>**Figure 1**:  Examples of video categorization [(Source)](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.pyimagesearch.com%2F2019%2F07%2F15%2Fvideo-classification-with-keras-and-deep-learning%2F&psig=AOvVaw307QAI-e6zS1FpwJjuzM1C&ust=1621478739032000&source=images&cd=vfe&ved=0CAIQjRxqFwoTCNCMnJTd1PACFQAAAAAdAAAAABAa).</center>*

**Main target**:  extract the categories of videos automatically, which may support different purposes or work as the input of other tasks.  
A video file contains audio, text, and visual content in frames, as it is a series of images with adequate motion. It is necessary to use and analyze all the available resources to result in video categorization efficiently. For this reason, to achieve a good result, a video categorization method needs to examine all the essential elements of video in the form of text, audio, and frames. These elements are called modalities. The process should be providing the analysis results of the text, audio, and visual contents,  which are then combined to get the final video category output.  

**Input of each modality** should be preprocessed before applying in a video categorization task.  
- Visual content should be parsed into frames or keyframes. Considered preprocessing method: keyframe extraction.  
- Textual content should be extracted from the speech. Considered method: speech to text.  
- Signal content should be extracted from video sound. Considered method: video sound extractor.  

**Feature learning**: joint learning or attention method is considered.  
**Classification**: multi-task supervised method may achieve better performance.  
**Output**: the multiple labels of each input video.

### 2. Correlated tasks to Video recognition and categorization  

#### 2.1. Video content filtering  
**Main target**: Identify and avoid the unacceptable videos be added in the networks.  
**Input**: Videos, their categories, and the output got from sub-task detections.  
**Output**: {0,1} where 1 identify the acceptance of input video and  0 identify the rejection of input video.  

**Sub-task detections for video content filtering**:  
- Negative content detection on video (porn video): skin segmentation, human navel area detection, sentiment analysis.  
- Abnormal scenes: bleeding detection, abnormal action detection.  
#### 2.2. Video Recommendation  
It turns out that there are (generally) three approaches to construct a recommendation engine:   
- **Engine for suggestion based on popularity**: This is the most basic type of recommendation engine you will encounter. This algorithm is used to generate the trending list that you see on YouTube or Netflix. It maintains track of the number of views for each movie/video and then lists movies in descending order depending on views (highest view count to lowest view count).  
- **Engine for content-based recommendations**: This form of recommendation system uses a movie that a user is currently interested in as input. Then it examines the movie's content (storyline, genre, actor, director, etc.) to locate other films with similar material. The content can be extracted by the output of the **Video recognition and categorization** module. Then, based on their similarity ratings, it ranks comparable movies and recommends the most relevant movies to the viewer.   
- **Recommendation engine based on collaborative filtering**: this algorithm initially attempts to locate people similar to them based on their behaviors and interests (for example, both the users watch the same type of movies or movies directed by the same director). If one of these users (say, A) has seen a movie that user B has not yet watched, that movie is recommended to user B, and vice versa. To put it another way, the recommendations are filtered based on the cooperation of comparable user preferences (thus the name "Collaborative Filtering"). This algorithm is often used on the Amazon e-commerce platform, where you can see the “Customers who saw this item also viewed” and “Customers who bought this item also purchased” lists.  

#### 2.3. Smart Ads  
**Definition** Smart Ads contain outputs that answer four questions: **what, when, how many times, and how long** ads are added in the video.  
**Criteria**:  
- Add Ads which contents do not conflict with the current video context.  
- Add Ads which a suitable duration and frequency.  
- Add Ads that are suitable with the user (a recommendation). Example: The user who frequently visits clips of their idols may be interested in ads that sell items sponsored by those correlated ones.  

**Input**: videos and their content-based information (categories, tabular data) can be used as inputs, ads databases, and correlated additional information.  
**Output**: period of each ads(regression), duration of each ads (regression), choosen ads (retrieval), and location of each ads(bin classification/regression).  


### 3. Near-Duplicate Video Detection  
**Introduction**  
We have seen an enormous rise of video data on a range of video-sharing websites. With billions of videos available on the internet, performing near-duplicate video retrieval (NDVR) from a large-scale video database becomes a considerable difficulty. NDVR intends to extract near-duplicate films from a vast video archive, where near-duplicate films are defined as visually similar to the original movies.  

To garner attention, users have a solid incentive to replicate a popular short video and publish an enhanced version. As the popularity of short videos grows, so do the problems and problems of finding near-duplicate short films.

![https://github.com/4ML-platform/ndvr/blob/master/images/ndvr-2.png?raw=true](https://github.com/4ML-platform/ndvr/blob/master/images/ndvr-2.png?raw=true)  

*<center>**Figure 2**:  An example of near-duplicated videos [(Source)](https://github.com/4ML-platform/ndvr).</center>*

**Solution**:  
**Paper ICCV-2017**: Near-Duplicate Video Retrieval  with Deep Metric Learning  
**Link research repo** github (not well-organized yet but allow to train model): [https://github.com/MKLab-ITI/ndvr-dml](https://github.com/MKLab-ITI/ndvr-dml)  
**Link wrapped-up code** in docker (can be used directly but can not train/re-train model): [https://github.com/4ML-platform/ndvr](https://github.com/4ML-platform/ndvr)  

### 4. Bot detection  
![BotDetection](https://datadome.co/wp-content/uploads/bot-detection-engine-1200x492.jpg)  
*<center>**Figure 3**:  An overview of bot detection schematic [(Source)](https://datadome.co/bot-management-protection/bot-detection-how-to-identify-bot-traffic-to-your-website/#chapter3).</center>*

**Main target**: Identify the abnormal action of users when they interact with the AIOZ network. For dealing with bots...  
More detail about definitions and solutions can be found in: [DataDome](https://datadome.co/bot-management-protection/bot-detection-how-to-identify-bot-traffic-to-your-website/#chapter3)  

### 5. Smart Routing for AIOZ Nodes
Where each task is picked up which some best nodes.  
This task is flexible to problems of different networks.  
