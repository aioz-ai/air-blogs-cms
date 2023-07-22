---
last_modified_on: "2023-06-30"
id: FADNet2
title: Deep Federated Learning for Autonomous Driving (Part 2)
description: Deep Federated Learning for Autonomous Driving (experimental results)
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Federated Learning"]
---
In previous part, we have discussed about Autonomous driving FADNetwork. In this post, we will verify the effectiveness and efficiency of it.

![](https://blog.openmined.org/content/images/2021/08/AV-procedure-artboard-1-3-04-1.png)

Our source code can be found at: https://github.com/aioz-ai/FADNet

## 1. Experimental Setup

**Udacity.** We use the popular Udacity dataset to evaluate our results. We only use front-forwarded images in this dataset in our experiment. We use $5$ sequences for training and $1$ for testing. The training sequences are assigned randomly to different silos depending on the federated topology (i.e., Gaia or NWS).

**Carla.** Since the Udacity dataset is collected in the real-world environment, changing the weather or lighting conditions is not easy. To this end, we collect more simulated data in the Carla simulator. We have applied different lighting (morning, noon, night, sunrise, sunset) and weather conditions (cloudy, rain, heavy rain, wet streets, windy, snowy) when collecting the data. We have generated $73,235$ samples distributed over $11$ sequences of scenes. 

**Gazebo.** Since both the Udacity and Carla datasets are collected in outdoor environments, we also employ Gazebo to collect data for autonomous navigation in indoor scenes. We use a simulated mobile robot and the built-in scenes to collect data. Table.1 shows the statistics of three datasets. We use $80\%$ of the collected data in Gazebo and Carla data for training, and the rest $20\%$ for testing.

![](https://drive.google.com/uc?id=1wmq9jY4TYZhg2k9lBDwrEFNE8Wcm0PHq)*<center>**Figure 1**. Visualization of sample images in three datasets: Udacity (first row), Gazebo (second row), and Carla (third row). </center>*

| Dataset  | Total samples | Average samples in each silo (Gaia) | Average samples in each silo (NWS) |
|----------|---------------|------------------------------------|------------------------------------|
| Udacity  | 39,087        | 3,553                              | 1,777                              |
| Gazebo   | 66,806        | 6,073                              | 3,037                              |
| Carla    | 73,235        | 6,658                              | 3,329                              |
*<center>**Table 1**. The Statistic of Datasets in Our Experiments. </center>*

**Network Topology.** We conduct experiments on two topologies: the Internet Topology Zoo (Gaia), and the North America data centers (NWS). We use Gaia topology in our main experiment and provide the comparison of two topologies in our ablation study. 

**Training.** The model in a silo is trained with a batch size of $32$ and a learning rate of $0.001$ using Adam optimizer. We follow the training process to obtain a global weight of all silos. The training process is conducted with $3000$ communication rounds and each silo has one NVIDIA 1080 11 GB GPU for training. Note that, one communication round is counted each time all silos have finished updating their model weights. 

**Baselines.** We compare our results with various recent methods, including Random baseline and Constant baseline, Inception-V3, MobileNet-V2, VGG-16, and Dronet. All these methods use the Centralized Local Learning (CLL) strategy (i.e., the data are collected and trained in one local machine.) For distributed learning, we compare our Deep Federated Learning (DFL) approach with the Server-based Federated Learning (SFL) strategy. As the standard practice, we use the root-mean-square error (RMSE) metric to evaluate the results.

## 2. Results
Table 2 summarises the performance of our method and recent state-of-the-art approaches. We notice that our FADNet is trained using the proposed peer-to-peer DFL using the Gaia topology with 11 silos. This table clearly shows our FDANet + DFL outperforms other methods by a fair margin. In particular, our FDANet + DFL significantly reduces the RMSE in Gazebo and Carla datasets, while slightly outperforms DroNet in the Udacity dataset. These results validate the robustness of our FADNet while is being trained in a fully decentralized setting. Table 3 also shows that with a proper deep architecture such as our FADNet, we can achieve state-of-the-art accuracy when training the deep model in FL. Fig. 2 illustrates the spatial support regions when our FADNet making the prediction. Particularly, we can see that FADNet focuses on the ``line-like" patterns in the input frame, which guides the driving direction. 

| Architecture                          | Learning Method | Udacity | Gazebo | Carla | #Params |
|---------------------------------------|-----------------|---------|--------|-------|---------|
| Random     | -               | 0.301   | 0.117  | 0.464 | -       |
| Constant  | -               | 0.209   | 0.092  | 0.348 | -       |
| Inception| CLL             | 0.154   | 0.085  | 0.297 | 21,787,617 |
| MobileNet| CLL             | 0.142   | 0.083  | 0.286 | 2,225,153 |
| VGG-16         | CLL             | 0.121   | 0.083  | 0.316 | 7,501,587 |
| DroNet    | CLL             | 0.110   | 0.082  | 0.333 | 314,657   |
| FADNet (ours)                          | DFL             | **0.107** | **0.069** | **0.203** | 317,729 |
*<center>**Table 2**. Performance comparison of different architectures on the Udacity, Gazebo, and Carla datasets. The number of parameters (#Params) is also provided. </center>*


![](https://drive.google.com/uc?id=1qDcIX7XKWfrpyNREgMeO4AdiizMvB9vy)*<center>**Figure 2**. Spatial support regions for predicting steering angle in three datasets. In most cases, we can observe that our FADNet focuses on ``line-like” patterns to predict the driving direction. </center>*

## 3. Ablation Studies
### Effectiveness of our DFL.
Table 3 summarises the accuracy of DroNet and our FADNet when we train them using different learning methods: CLL, SFL, and our peer-to-peer DFL. From this table, we can see that training both DroNet and FADNet with our peer-to-peer DFL clearly improves the accuracy compared with the SFL approach. This confirms the robustness of our fully decentralized approach and removes a need of a central server when we train a deep network with FL. Compared with the traditional CLL approach, our DFL also shows a competitive performance. However, we note that training a deep architecture using CLL is less complicated than with SFL or DFL. Furthermore, CLL is not a federated learning approach and does not take into account the privacy of the user data.

| Architecture                          | Learning Method                    | Udacity | Gazebo | Carla |
|---------------------------------------|------------------------------------|---------|--------|-------|
| DroNet      | CLL   | 0.110   | 0.082  | 0.333 |
|                                       | SFL       | 0.176   | 0.081  | 0.297 |
|                                       | DFL (ours)                         | 0.152   | 0.073  | 0.244 |
| FADNet (ours)                          | CLL    | 0.142   | 0.081  | 0.303 |
|                                       | SFL      | 0.151   | 0.071  | 0.211 |
|                                       | DFL (ours)                         | **0.107** | **0.069** | **0.203** |
*<center>**Table 3**. Performance comparison of different methods. </center>*

### Effectiveness of our FADNet.
Table 3 shows that apart from the learning method, the deep architectures also affect the final results. This table illustrates that our FADNet combined with DFL outperforms DroNet in all configurations. We notice that DroNet achieves competitive results when being trained with CLL. However DroNet is not designed for federated training, hence it does not achieve good accuracy when being trained with SFL or DFL. On the other hand, our introduced FADNet is particularly designed with dedicated layers to handle the data imbalance and model convergence problem in federated training. Therefore, FADNet achieves new state-of-the-art results in all three datasets.

### Network Topology Analysis.
Table 4 illustrates the performance of DroNet and our FADNet when we train them using DFL under two distributed network topologies: Gaia and NWS. This table shows that the results of DroNet and FADNet under DFL are stable in both Gaia and NWS distributed networks. We note that the NWS topology has 22-silos while the Gaia topology has only 11 silos. This result validates that our FADNet and DFL do not depend on the distributed network topology. Therefore, we can potentially use them in practice with more silo data.

| Network Topology        | Architecture                      | Udacity | Gazebo | Carla |
|-------------------------|----------------------------------|---------|--------|-------|
| Gaia (11 silos)         | DroNet | 0.152   | 0.073  | 0.244 |
|                         | FADNet (ours)                     | 0.107   | 0.069  | 0.203 |
| NWS (22 silos)          | DroNet | 0.157   | 0.075  | 0.239 |
|                         | FADNet (ours)                     | 0.109   | 0.070  | 0.200 |
*<center>**Table 4**. Performance comparison of different network topologies. </center>*

### Convergence Analysis.
The effectiveness of federated learning algorithms is identified through the convergence ability, including accuracy and training speed, especially when dealing with the increasing number of silos in practice. Fig.3 shows the convergence ability of our FADNet with DFL using two topologies: Gaia with 11 silos, and NWS with 22 silos. This figure shows that our proposed DFL achieves the best results in Gaia and NWS topology and converges faster than the SFL approach in both Gazebo and Carla datasets. We also notice that the performance of our DFL is stable when there is an increase in the number of silos. Specifically, training our FADNet with DFL reaches the converged point after approximately $150$s, $180$s on the NWS and Gaia topology, respectively. Fig.3 validates the convergence ability of our FADNet and DFL, especially when dealing with the increasing number of silos.

In practice, compared with the traditional CLL approach, federated learning methods such as SFL or DFL can leverage more GPUs remotely. Therefore, we can reduce the total training time significantly. However, the drawback of federated learning is we would need more GPUs in total (ideally one for each silo), and deep architecture also should be carefully designed to ensure model convergence.


![](https://drive.google.com/uc?id=1W5J7W2hFqiknyTGPPIYI0wEQtFozQiOD)*<center>**Figure 3**. The convergence ability of our FADNet and DFL under Gaia and NWS topology. Wall-clock time or elapsed real-time is the actual time taken from the start of the whole training process to the end, including the synchronization time of the weight aggregation process. All experiments are conducted with $3,000$ communication rounds. </center>*

## Deployment
To verify the effectiveness of our FADNet in practice, we deploy the model trained on the Gazebo dataset on a mobile robot. The robot is equipped with a RealSense camera to capture the front RGB images. Our FADNet is deployed on a Qualcomm RB5 board to make the prediction of the steering angle for the robot. The processing time of our FADNet on the Qualcomm RB5 board is approximately $12$ frames per second. Overall, we observe that the robot can navigate smoothly in an indoor environment without colliding with obstacles. More qualitative results can be found in our supplementary material.


## Conclusion
We propose a new approach to learn an autonomous driving policy from sensory data without violating the user's privacy. We introduce a peer-to-peer deep federated learning (DFL) method that effectively utilizes the user data in a fully distributed manner. Furthermore, we develop a new deep architecture - FADNet that is well suitable for distributed training. The intensive experimental results on three datasets show that our FADNet with DFL outperforms recent state-of-the-art methods by a fair margin. Currently, our deployment experiment is limited to a mobile robot in an indoor environment. In the future, we would like to test our approach with more silos and deploy the trained model using an autonomous car on man-made roads.
