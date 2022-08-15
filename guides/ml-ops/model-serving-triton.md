---
last_modified_on: "2022-08-13"
title: Model Serving and NVIDIA Triton Inference Server.
description: The need for model serving tools and NVIDIA Triton Inference Server.
series_position: 1
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';

Billions of predictions made by machine learning models are used daily when one unlocks their iPhone with FaceID, refreshes their TikTok "For You" page, or checks for ETA time for their Grab order. At the same time, thousands, if not millions other people are doing the exact same. Indeed, in cases where instant predictions are expected, latency and throughput problems quickly magnify. Having engineers build and maintain their own serving systems can become inefficient in the long run. Therefore, a standardised deployment solution is very much in need. In this article, we will discuss model serving, why we need ML-specific serving tools and a framework-agnostic, quick-to-production solution, NVIDIA Triton Inference Server.


## Introduction

After a model is trained, one may wish to share the model with other users to either showcase the capability of the model or integrate the model's learned features into an existing production environment. This process is called **model deployment**, which can be done via **embedding** or **serving**. Model embedding is where the entire trained model is copied onto the device and loaded into memory when run. While it is suitable for latency-critical problems, it is not scalable and requires every application to have a version of the model, which quickly poses the problems of monitoring and versioning.

For environments where latency is less of an issue, we can consider porting the trained model on a server and making it accessible to other applications as a **client-server architecture**. This approach, which is called **model serving**, promotes the transactional nature of the model inference process and allows applications to interact with ML models similar to how they interact with database servers, where the trained weights and computation are kept and done in a separate server, and HTTP/gRPC calls will be used to send the prediction requests to said server.

![Fig-1](https://drive.google.com/uc?id=1fCHTSEdMOYWoUWJXEi_vDQP_vd15vlq5)
*<center>**Figure 1**:  Model Server Architecture. [1]</center>*

In previous years, ML engineers would create prediction services by using general APIs such as Flask or FastAPI to wrap their models in web servers. Although this is straightforward to incorporate into existing environments, it can be inefficient, inconsistent across developers, and requires lots of customisation to meet the infrastructure needs. Therefore, more ML-specific serving tools have been introduced to streamline and standardise the deployment process, as well as provide ease of scaling, versioning, and monitoring post-deployment.

## The need for model serving frameworks
To deploy a model for online-prediction, API services are often used by creating a server with an ability to listen and respond to prediction requests, as illustrated in **Figure 1**. Through HTTP or gRPC interface, APIs can be easily integrated into existing production environments and platforms (e.g., web apps, mobile apps) [2]. Technology stacks like Flask and Docker are often used for creating APIs and containerising Python models. Afterwards, to keep the server running 24/7, companies can decide to host the server on the cloud or on-premise. This high-level process is true whether we use serving tools or not.  

The detailed implementation, however, is often specific to the project as well as the engineering team. For instance, there are often no consistent APIs in how to deal with requests, cache and transition between model versions, etc. Indeed, in most cases, many teams build and maintain their own serving solutions [3], and each big companies have its own in-house serving infrastructure [3,4].       

Furthermore, with larger models and more inference requests, serving solutions can quickly become complicated [5]. Backend developers will need to write new codes for any added features, development-wise (model versioning, model ensembling) or orchestration-wise (load balancing, hardware allocating, scaling, performance monitoring, logging, etc.). This serving architecture can become restrictive to the mentioned issues, the business, and the framework used by the developers, hence rendering it often not reusable.  

![Fig-2](https://drive.google.com/uc?id=1uiJty8LLFNeOSdwLkFCQ_k1mxWkFKSl3)
*<center>**Figure 2**:  Expectation vs. Reality in serving models as APIs. [6]</center>*

Model serving tools simplify this process by eliminating all the supposed codes that developers need to write as well as standardising the ML deployment workflow. This allows ML engineers to focus more on developing and maintaining models, promotes faster CI/CD integration, all the while helps to balance the three model deployment criteria - latency, throughput, and cost.

Examples of model serving frameworks include, but not limited to, Clipper, Tensorflow Serving, KLServing, TorchServe, NVIDIA Triton Inference Server, etc.

### How do these frameworks make model deployment easier?
Major serving frameworks were developed with the following goals in mind:  
1. Shorten time from development to production,   
2. Standardise deployment,   
3. Deliver reliable ML products at scale with high throughput and low latency predictions.  

To achieve these goals, some common features provided by major serving tools include [7]:  
- **Automate model initialisation**: users only have to specify the models they want to serve in a model repository. The serving tool will automatically collect the models' artifact and associate resources for the model. Once loaded, the model server is ready to accept inference requests.    
- **Support standard HTTP/gRPC interface**: Servers created by these serving frameworks will be automatically exposed through an API endpoint(s) with ports assigned for HTTP, gRPC, and health check calls. Direct communication with clients is possible as well as connection with other applications such as load balancers.    
- **Model versioning**: Newest updates are checked for periodically in the model repository so that newer model versions can be deployed immediately, without extra coding or shutting down the server.       
- **Multi-model serving**: For safe transitions between different model versions, multiple models can be served at the same time for A/B testing or canary deployment.   
- **Real-time metrics** for monitoring hardware utilisation, latencies, errors, prediction qualities, etc.,  
- **Integrates with orchestration services** such as Kubernetes for scheduling tasks, metrics, and autoscaling.  

## NVIDIA Triton Inference Server

### What is Triton?

The question of which serving tool to use still remains. For available serving frameworks, there were still problems of framework lock-in (Tensorflow Serving can only serve Tensorflow models and TorchServe can only serve PyTorch models) and training and serving divergence (where models have to be converted to a compatible format such as ONNX or PMML) [8]. Therefore, a platform-independent model deployment mechanism was very much in need. With this in mind and being a company that often works with large scale data, NVIDIA introduced the Triton Inference Server, an open-source software that streamlines AI inferencing, optimised for cases where hundreds of thousands real-time predictions are expected.

Aside from the main features as mentioned above, Triton prides itself on being a **single standardised inference platform** which supports deploying models trained on different frameworks, thus allowing data scientists the freedom to work with their most fluent ML framework. Moreover, Triton supports handling different types of prediction queries and live updating of newer model versions without any changes in the server architecture. Lastly, it is also available as a Docker container makes it incredibly easy to set up and scale.   

### Triton Architecture

![Fig-3](https://drive.google.com/uc?id=1PoruVGwEDdyYAotgLLRigRIpWEAqExM1)
*<center>**Figure 3**:  NVIDIA Triton Inference Server Architecture. [9]</center>*

The full workflow of Triton can be seen in **Figure 3** and described as follows:  
1. Client sends a request through HTTP or gRPC.  
2. The request goes to the Request Handler.  
3. Then, the request goes to a Per Model Scheduler Queue.   
4. When it’s ready, the schedule will send the request to the worker backend for computation (GPU/CPU).  
5. At the same time, the health metrics of the server are exposed through another port.   
6. Once the computation is complete, the response goes to the Response Handler.   
7. The response is then sent back to the client.  

### How is Triton different?
#### 1. Model Ensemble
Many ML tasks require two or more steps together such as image captioning, question answering, etc. Triton allows a set of models to be applied to one input sequentially and lets engineers define the pipeline through a configuration file, illustrated in **Figure 4**. Taking advantage of the GPUs run by the server, we can add the preprocessing and postprocessing steps into the pipeline, reducing the computation costs on the client side and the data transfer between client and server.

![Fig-4](https://drive.google.com/uc?id=1Lj2JUfuEAA83dzz79CnWiabACSBpa2-W)
*<center>**Figure 4**:  Server-side Processing. [10]</center>*

On the other hand, developers can also choose to host different modules on different servers and orchestrate their operation through tools such as Kubernetes. This approach is suitable for cases where one module requires scaling while others do not.

#### 2. Concurrent Model Runs
During the server initialisation, Triton creates multiple instances of the same model on different GPU/CPU(s). In the case of multiple models, each model will be duplicated and run in parallel. This ensures higher throughput and that all resources are utilised under high demand. Furthermore, as NVIDIA houses the fastest suite of accelerators in the AI industry, Triton also provides hardware optimisation specific to NVIDIA's GPUs.

#### 3. Dynamic Batching

Regardless of the deployment platform, the two main desired characteristics of a deployed model are low latency and high throughput. GPUs achieve higher throughput with higher batch sizes. However, for a typical client-server architecture, requests are often processed sequentially which would easily affect throughput. Under higher demand and specific requirements for an application, engineers can balance latency and throughput for an optimised user experience [11]. Incoming requests within a certain time unit (e.g. 30ms) will be batched dynamically by Triton, utilising the parallel computing power of GPUs. An example is depicted in **Figure 5**, where eight different clients put in requests for an image classification concurrently, and the requests are batched on the server side and computed together. As can be seen on the right, by increasing the number of batched requests from 1 to 16, we experienced a 3ms delay in latency, while throughput performance increased by 3x.

![Fig-5](https://drive.google.com/uc?id=1tubhmMtotun_1n444Rq8MrvJK-oSDstb)
*<center>**Figure 5**: Triton Dynamic Batching illustration (left) and results (right) [11]</center>*

## Conclusion

In this article, we have explored the less talked about aspect of machine learning - model serving, its status quo, how it reflects the need for more ML-specific solutions, and an example of these tools, NVIDIA Triton. Triton has proved to simplify the deployment process and is used by many large companies  such as Salesforce, Volkswagen, Hugging Face, etc. in bringing ML models to end-users [12].

## Reference

[1] A. Vikati, "The Rise of the Model Servers", *Medium*, 2018. [Online]. Available: https://medium.com/@vikati/the-rise-of-the-model-servers-9395522b6c58.   
[2] T. Teh, "How companies deploy machine learning models to production today", *Linkedin*, 2019. [Online]. Available: https://www.linkedin.com/pulse/how-companies-deploy-machine-learning-models-production-teren-teh.  
[3] J. Hermann and M. Balso, "Meet Michelangelo: Uber's Machine Learning Platform", *Uber Engineering Blog*, 2017. [Online]. Available: https://eng.uber.com/michelangelo-machine-learning-platform/.   
[4] C. Zeng, "Meet Sibyl - DoorDash’s New Prediction Service - Learn about its Ideation, Implementation and Rollout", *DoorDash Engineering Blog*, 2020. [Online]. Available: https://doordash.engineering/2020/06/29/doordashs-new-prediction-service/.  
[5] C. Olston et al., "TensorFlow-Serving: Flexible, High-Performance ML Serving", *NIPS 2017 Workshop on ML Systems*, 2017. Available: https://arxiv.org/abs/1712.06139.  
[6] M. Schmitt, "Why you need a model serving tool", *Data Revenue*, 2020. [Online]. Available: https://www.datarevenue.com/en-blog/why-you-need-a-model-serving-tool-such-as-seldon.  
[7] P. Kijko, "Best Tools to Do ML Model Serving", *Neptune Blog*, 2022. [Online]. Available: https://neptune.ai/blog/ml-model-serving-best-tools.  
[8] S. Mo, "Machine Learning Serving is Broken", *Medium*, 2020. [Online]. Available: https://medium.com/distributed-computing-with-ray/machine-learning-serving-is-broken-f59aff2d607f.  
[9] S. Bhavani, E. Isaza, J. Liu, A. Ansari and Q. Li, "Deploy fast and scalable AI with NVIDIA Triton Inference Server in Amazon SageMaker", *AWS Machine Learning Blog*, 2021. [Online]. Available: https://aws.amazon.com/blogs/machine-learning/deploy-fast-and-scalable-ai-with-nvidia-triton-inference-server-in-amazon-sagemaker/.  
[10] R. Banas, K. Lecki and M. Szolucha, "Accelerating Inference with NVIDIA Triton Inference Server and NVIDIA DALI", *NVIDIA Technical Blog*, 2021. [Online]. Available: https://developer.nvidia.com/blog/accelerating-inference-with-triton-inference-server-and-dali/.  
[11] S. Chandrasekaran and M. Salehi, "Fast and Scalable AI Model Deployment with NVIDIA Triton Inference Server", *NVIDIA Technical Blog*, 2021. [Online]. Available: https://developer.nvidia.com/blog/fast-and-scalable-ai-model-deployment-with-nvidia-triton-inference-server/.  
[12] S. Chandrasekaran, "NVIDIA Triton Tames the Seas of AI Inference", *NVIDIA Blog*, 2021. [Online]. Available: https://blogs.nvidia.com/blog/2021/04/12/triton-ai-inference/.  
