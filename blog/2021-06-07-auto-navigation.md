---
last_modified_on: "2021-06-07"
id:  auto-navigation
title: Autonomous Navigation in Complex Environments with Deep Multimodal Fusion Network
description: "Deep Multimodal Fusion Network"
author_github: https://github.com/nqanh
tags: ["type: post", "Robotics", "Deep Learning", "Navigation", "Autonomous"]
---


# Introduction
Autonomous navigation is a long-standing field of robotics research, which provides an essential capability for mobile robot to execute a series of tasks on the same environments performed by human everyday. In general, the task of autonomous navigation is to control a robot navigate around the environment without colliding with obstacles. It can be seen that navigation is an elementary skill for intelligent agents, which requires decision-making across a diverse range of scales in time and space. In practice, autonomous navigation is not a trivial task since the robot needs to close the perception-control loop under the uncertainty in order to obtain the autonomy.

Furthermore, autonomous navigation with intelligent robots in complex environments is also a crucial task, especially in time-sensitive scenarios such as disaster response, search and rescue, etc. Recently, the DARPA Subterranean Challenge was organized to explore novel methods for quickly mapping, navigating, and searching in complex underground environments such as human-made tunnel systems, urban underground, and natural cave networks.

Inspired by the DARPA Subterranean Challenge, in this work, we propose a learning-based system for end-to-end mobile robot navigation in complex environments such as  collapsed cities, collapsed houses, and natural caves. To overcome the difficulty in data collection, we build a large scale simulation environment which allows us to collect the training data and deploy the learned control policy. We then propose a new Navigation Multimodal Fusion Network (NMFNet) that effectively learns the visual perception from sensor fusion and allows the robot to autonomously navigate in complex environments. To summarize, our main contributions are as follows:   
- We introduce new simulation models that can be used to record large-scale datasets for autonomous navigation in complex environments.
- We present a new deep learning method that fuses both laser data, 2D images, and 3D point cloud to improve the navigation ability of the robot in complex environments.
- We show that the use of multiple visual modalities is essential to learn a robust robot control policy for autonomous navigation in complex environments in order to deploy in real-world scenarios

![Fig-1](https://vision.aioz.io/thumbnail/3db33b2413fc4301b323/1024/fi1.png)
*<center>**Figure 1**: A visualization of a collapsed city, a complex environment from our simulation. Top: The collapsed city from a top view; Bottom: A mobile robot’s view..</center>*

# Methodology
## 2D Features
In this work, we use ResNet8 to extract deep features from the input RGB image and laser distance map. The ResNet8 has 3 residual blocks, each block consists of a convolutional layer, ReLU, skip links and batch normalization operations. A detailed visualization of ResNet8 architecture can be found in Fig. 2. We choose ResNet8 to extract deep features from the 2D images since it is a light weight architecture and can achieve competitive performance while being robust again the vanishing/exploding gradient problems during training.

![Fig-2](https://vision.aioz.io/thumbnail/395c171b0e2b420383e4/1024/fig2.png)
*<center>**Figure 2**: A detailed visualization of ResNet8.</center>*

## 3D Point Cloud
To extract the point cloud feature vector, our network has to learn a model that is invariant to input permutation. We extract the features from the unordered point cloud by learning a symmetric function on transformed elements. Given an unordered point set $\{x_1, x_2,..., x_n\}$ with $x_i \in \mathbb{R}^3$ , we can define a symmetric function $f : X \rightarrow \mathbb{R}$ that maps a set of unordered points to a feature vector as follow:

$$
f(x_1, x_2,...,x_n) = \delta(\underset{i=1..n}{MAX}\{\gamma(x_i)\})
$$

where $MAX$ is a vector max operator that takes $n$ input vectors and returns a new vector of the element-wise maximum; $\delta$ and $\gamma$ are usually presented by neural networks. In practice, $\delta$ and $\gamma$ function are approximated by an affine transformation matrix with a mini multi-layer preception network (i.e., T-net) and a matrix multiplication operation.

## Multimodal Fusion
Given the features from the point cloud branch and the RGB image branch, we first do an early fusion by concatenating the features extracted from the input cloud with the deep features extracted from the RGB image. This concatenated feature is then fed through two $1 × 1$ convolutional layers. Finally, we combine the features from 2D-3D fusion with the extracted features from the distance map branch. The steering angle is predicted from a final fully connected layer keeping all the features from the multimodal fusion network.

## Training
To train an end-to-end navigation network, two popular approaches are used: classification loss and regression loss. Our training NMFNet with three branches to handle three visual modalities is illustrated in Fig. 3.

![Fig-3](https://vision.aioz.io/thumbnail/8520941d0e7342e08322/1024/fig3.png)
*<center>**Figure 3**: An overview of our NMFNet architecture. The network consists of three branches: The first branch learns the depth features from the distance map. The second branch extracts the features from RGB images, and the third branch handles the point cloud data. We then employ the 2D-3D fusion to
predict the steering angle.</center>*

# Experiments
## Data Collection
We create the simulation models of these environments in Gazebo and collect the visual data from simulation. In particular, we
collect the data from three types of complex environment:
- Collapsed house: The house suffered from an accident or a disaster (e.g. an earthquake) with random objects on the ground.
- Collapsed city: Similar to the collapsed house, but for the outdoor environment. In this scenario, the road has many debris from the collapsed house/wall.
- Natural cave: A long natural tunnel in a poor brightness condition with irregular geological structures.

For each environment, we use a mobile robot model equipped with a laser sensor and a depth camera mounted on top of the robot to collect the visual data. The robot is controlled manually to navigate around each environment. We collect the visual data when the robot is moving. All the visual data are synchronized with a current steering signal of the robot at each timestamp. Examples of different robot’s views in our simulation of complex environments (collapsed city, natural cave, and collapsed house) are described in Fig. 4.

![Fig-4](https://vision.aioz.io/thumbnail/b58dd1f6561847a99dcb/1024/fig4.png)
*<center>**Figure 4**: Top row: RGB images from robot viewpoint in normal setting. Other rows: The same viewpoint when applying domain randomization to the simulated environments.</center>*

## Results
Table I summarizes the regression results using Root Mean Square Error (RMSE) of our NMFNet and other state-of-theart methods. From the table, we can see that our NMFNet outperforms other methods by a significant margin. In particular, our NMFNet trained with domain randomisation data achieves 0.389 RMSE which is a clear improvement over other methods using only RGB images such as DroNet [29]. This also confirms that using multi visual modalities input as in our fusion network is the key to successfully navigate in complex environments.

![Fig-5](https://vision.aioz.io/thumbnail/e1e5acf24e984880bb14/1024/fig5.png)
*<center>**Table I**: RMSE SCORES ON THE TEST SET.</center>*

Table II shows the RMSE scores when different modalities are used to train the system. We first notice that the network that uses only point cloud data as the input does not converge. This confirms that learning meaningful features from point cloud data is challenging, especially in complex environments. On the other hand, we achieve a surprisingly good result when the distance map modality is used as the input. The other combinations between two modalities show reasonable accuracy, however, we achieve the best result when the network is trained end-to-end using the fusion from all three modalities: rgb, distance map from laser camera, and point cloud from the depth camera.

![Fig-6](https://vision.aioz.io/thumbnail/3e396b2cbd11475f9f85/1024/fig6.png)
*<center>**Table II**: RMSE SCORES OF NETWORKS USING DIFFERENT MODALITY.</center>*

# Conclusion
We propose NMFNet, an end-to-end and real-time deep learning framework for autonomous navigation in complex environments. Our network has three branches and effectively learns the visual fusion input data. Furthermore, we show that the use of mutilmodal sensor data is essential for autonomous navigation in complex environments. The extensively experimental results show that our NMFNet outperforms recent state-of-the-art methods by a fair margin while achieving real-time performance and generalizing well on unseen environments.
