---
last_modified_on: "2023-04-01"
title: Introduction to Diffusion Models (Part 3)
description: A kind introduction to Diffusion Models and its underlying theory.
series_position: 
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
In the previous posts ([Part 1](https://ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_31/), [Part 2](https://ai.aioz.io/guides/computer-vision/IntroductiontoDiffusionModels_37/)). We have introduced fundamental principles of diffusion models, how to train as well as to sample from diffusion model through the learned reverse process. However, the posts only covered standard diffusion mechanism under unconditional scenarios, i.e., we only generate data samples in a purely random manner without any desire for the generated outputs. In this blog post, we will demonstrate how to generate data according to some input conditions.

## Conditional Generation
Literature has found the connection between diffusion models and denoising score matching models [1], in which samples are produced via an iterative process using gradients of the data distribution estimated with score matching. The score of each sample $x$’s probability density $p(x)$ is defined as the gradient $\nabla_{x}\log p(x)$ of its log-likelihood  (i.e., the steepest increasing direction of the log-likelihood). Specifically, it is shown that training the backward process of denoising diffusion (i.e., predicting the posterior mean) is equivalent to estimating the score function: 
$$
\nabla_{x_t}\log p(x_t) \propto \epsilon_\theta(x_t,t) \tag{1}
$$

For conditional generation, a naive and straightforward approach is to append the condition $y$ into the network's inputs as a separate channel, i.e., $\epsilon_\theta(x_t,t|y)$. However, this strategy may not work well since the network may ignore this new input channel, which can result in some arbitrarily generated outputs not following the condition. Instead, we can use Bayes rule to decompose the conditional probability $p(x|y)$ into the combination of unconditional model $p(x)$, and a discriminative model $p(y|x)$ that predicts the probability of the condition (e.g., class label) given the data sample (e.g., image):
$$
p(x | y) = \frac{p(y | x) \cdot p(x)}{p(y)}\\
\implies \log p(x | y) = \log p(y | x) + \log p(x) - \log p(y) 
$$
By differentiating both sides of the above equation with respect to $x$, we obtain the following formula (note that $\nabla_x\log p(y)=0$ since $p(y)$ does not depend on $x$):

$$
\nabla_x \log p(x | y) =  \nabla_x \log p(x) + \nabla_x \log p(y | x) \tag{2}
$$


### Classifier Guidance

In order to incorporate the class label into the diffusion process, Dhariwal and Nichol [2] proposed a classifier guidance strategy that uses gradients from a classifier to guide the diffusion model during sampling. However, the classifier should be trained on noised images (with respect to different diffusion timesteps) to obtain the correct gradient to guide the sampling process toward the conditioning class. Otherwise, it can adversely affect the sample quality because the noised intermediate images encountered during sampling are out-of-distribution for the classifier.



Recall that the score function is proportional to the output of the denoising network (Equation 1) the classifier guidance has a very similar formula to Equation 2:
$$
\nabla_x \log p(x | y) = \nabla_x \log p(x) + s \cdot \nabla_x \log p(y | x) \\
\implies \bar{\epsilon}_\theta(x_t,t) \approx \epsilon_\theta(x_t,t) + s\cdot\nabla_{{x}_t} \log f_\phi(y \vert {x}_t)
$$

where $\bar{\epsilon}_\theta(x_t,t)$ is the newly shifted output of the model guided by the classifier $f_\phi(y | {x}_t)$ toward the target conditioning class $y$. The author also found that multiplying the condition with a scaling coefficient $s$ (called the guidance scale and typically $>1$) can amplify the influence of the conditioning signal. Thus, they can trade off the sample diversity for sample quality adhering to the target class. The figure below summarizes the sampling algorithm of diffusion model with classifier guidance.


![](https://vision.aioz.io/f/746c072a0b5c4ed3a871/?dl=1)*<center>**Figure 1**. Sampling algorithm of classifier guided diffusion in conditional generation. ([source](https://arxiv.org/abs/2105.05233))</center>* 

Figure 2 demonstrates the generated samples of unconditional diffusion model with classifier guidance trained on the ImageNet dataset. The conditioning class in the generation is "Welsh corgi". It can be seen that when using the guidance scale of 1.0 (left), the samples do not match very well with the target class upon visual inspection, whereas the guidance of scale 10.0 (right) produces much more convincing images that are well-aligned with the desired class. Additionally, the FID (a metric for measuring the fidelity of generated samples against the ground-truth data distribution) of the right images (FID: 12.0) is much lower than the left images (FID: 33.0). This indicates that the classifier-guidance model with high guidance scale can better capture the distribution of images belonging to this "corgi" class. The author also showed that diffusion models yield better performance than other generative models such as GANs.

![](https://vision.aioz.io/f/d4772ab6fd024aa7a74f/?dl=1)*<center>**Figure 2**. Comparisons of samples from an unconditional diffusion model with classifier guidance to condition on class "Pembroke Welsh corgi", with the guidance scale 1.0 (left) and 10.0 (right). ([source](https://arxiv.org/abs/2105.05233))</center>* 

### Classifier-Free Guidance
Although the classifier guidance can have several benefits when we want to control the generation with some conditioning information, there are unfortunately a few challenges that limit its practical utility. Due to the nature of the diffusion process, in which the data are gradually denoised in an iterative manner, the classifier used for guidance is required to be able to deal with high noise levels. Consequently, the training of this external classifier can be quite problematic and cumbersome to the overall system.

Moreover, even if we already had a classifier that is robust to noise, another problem may arise, that is not all the information of the input $x$ is useful in predicting $y$. As a result, the gradient of the classifier with respect to the whole input can result in arbitrary and even adversarial guiding signals in the data space, which may lead to undesired behavior of the model.

To alleviate the need for training a cumbersome classifier, Ho & Salimans [3] proposed the Classifier-Free Guidance technique, which performs guidance by combining both the conditional and unconditional diffusion model. Firstly, let us re-express the class probability $p(y|x)$ in Equation 2 using the Bayes rule again:
$$
p(y | x) = \frac{p(x | y) \cdot p(y)}{p(x)} \notag \\
\implies \log p(y|x) = \log p(x|y) + \log p(y) - \log p(x) \notag \\
\implies \nabla_x\log p(y|x) = \nabla_x\log p(x|y) - \nabla_x\log p(x)
$$
We have broken down the conditioning probability term into a combination of the conditional and unconditional model. Now, let's multiply this term with a scale factor $s$ and plug it into Equation 2:
$$
\nabla_x \log p(x | y) =  \nabla_x \log p(x) + s\cdot( \nabla_x\log p(x|y) - \nabla_x\log p(x))
$$
Or equivalently, we can obtain the formulation for classifier-guidance:
$$
\implies \bar{\epsilon}_\theta(x_t,t|y) \approx \epsilon_\theta(x_t,t|\emptyset) + s \cdot \left[ \epsilon_\theta(x_t,t|y) - \epsilon_\theta(x_t,t|\emptyset) \right] \tag{3}
$$
Interestingly, we can unify both the unconditional/conditional diffusion model into a single model by a special training strategy. Specifically, when training a conditional diffusion model $\epsilon_\theta(x_t|y)$), the condition is randomly masked out (dropped out) by the null condition $\emptyset$ with some probabilities. The resulting model can both represent conditional model $p(x|y)$ and unconditional model $p(x|\emptyset)$ since it is trained with either condition information or non-condition. 

We can see that the setting for $s=0$ is equivalent to the unconditional model (i.e., no guidance), and for $s=1$, we get the standard conditional model $\epsilon_\theta(x_t,t|y)$. Intuitively, when $s>1$, the classifier-guidance formula extrapolates the output of the model by taking a bigger step along the direction starting from the non-condition $\epsilon_\theta(x_t|\emptyset)$ towards the condition $\epsilon_\theta(x_t|y)$. This can yield useful and robust signals from the network's knowledge to guide the process itself without relying on a separate classifier (which is usually tricky to train). Moreover, this technique is very effective in scenarios when we want to perform guidance on complicated conditioning information such as text or audio, which is very difficult to estimate the probability with a classification model.

![](https://vision.aioz.io/f/bf9eaafe83ec4357a166/?dl=1)*<center>**Figure 3**. Comparisons of GLIDE model with CLIP guidance and classifier-free guidance on MS-COCO text prompts. ([source](https://arxiv.org/abs/2112.10741))</center>* 

The authors of the GLIDE paper [4] experimented with both classifier guidance and classifier-free guidance strategy in text-condition image generation. For classifier guidance, they use the CLIP model [5], a model trained on paired text/image and can provide a score of how close an image is to a text caption, to guide the generated image towards the input text (CLIP guidance). They found that classifier-free guidance performs more favorably against CLIP guidance. Figure 3 shows the visual comparisons between CLIP guidance (first row) to classifier-free guidance (second row). They observed that classifier-free guidance often produces more realistic samples than those using CLIP guidance. The former model is also capable of generalizing to a wide variety of prompts. 

![](https://vision.aioz.io/f/1f65b31b7be44dac953f/?dl=1)*<center>**Figure 4**. Comparing the diversity-fidelity trade-off between CLIP guidance and classifier-free guidance. ([source](https://arxiv.org/abs/2112.10741))</center>* 


As we have mentioned, guidance introduces a trade-off: it significantly enhances adherence to the conditioning signal and overall sample quality but at the cost of diversity. The GLIDE's experiments showed that classifier-free guidance can provide a good balance between FID score (measuring realism of the generated compared to the ground-truth images) and IS score (indicating quality and diversity of the samples), see Figure 4. This is usually an acceptable trade-off in conditional generative modeling. The conditioning signal often captures the desired information that we are interested in, and otherwise, if we favor diversity, we can adjust the provided conditioning signal accordingly.



 


















## References
[1] Song Y, Ermon S. Generative modeling by estimating gradients of the data distribution. In NIPS 2019.

[2] Dhariwal P, Nichol A. Diffusion models beat gans on image synthesis. In NIPS 2021.

[3] Ho J, Salimans T. Classifier-free diffusion guidance. arXiv preprint arXiv:2207.12598, 2022.

[4] Nichol A, Dhariwal P, Ramesh A, Shyam P, Mishkin P, McGrew B, Sutskever I, Chen M. Glide: Towards photorealistic image generation and editing with text-guided diffusion models. In ICML 2022.

[5] Radford A, Kim JW, Hallacy C, Ramesh A, Goh G, Agarwal S, Sastry G, Askell A, Mishkin P, Clark J, Krueger G. Learning transferable visual models from natural language supervision. In ICML 2021.