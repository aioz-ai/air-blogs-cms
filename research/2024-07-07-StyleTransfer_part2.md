---
last_modified_on: "2024-07-07"
id: Style2
title: Style Transfer for 2D Talking Head Generation (Part 2)
description: Style-Aware Talking Head Generator
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---
In the previous part, we have introduced an interesting problem in 2D animation and provided an overview of our system in order to achieve personalized talking head creation via style transfer. We also investigated existing research and identified crucial problems in the topic. Now we can dive into more details of our proposed method, firstly, how to generate a talking head animation from an audio input.


![image](https://vision.aioz.io/f/c1a222c66dec4a26b3dc/?dl=1)


## Style-Aware Talking Head Generator

Our Style-Aware Talking Head Generator is illustrated in Figure 1. It first takes the an audio sequence (such as speech or singing) to synthesize the corresponding Intermediate Audio-driven Motion. The Style Reference is the learned by the Style Mapping to capture the personalized talking style of any arbitrary character, which is then combined with the intermediate motion to create talking head nimation with desirable style .

### Audio Stream Representation and Motion Generator

**Audio Stream Representation** Representing audio for learning is essential in a talking head generator. Different speaking styles, referred to as individualized styles, can present challenges when using deep speech representation directly, resulting in suboptimal outcomes, particularly when dealing with distant variations in speech features. To improve the generalization of the audio stream extractor, we incorporate Lu et al.’s manifold projection technique.


**Motion generator** Given the extracted audio features, this step generates audio-driven motions in our framework. In practice, the character’s style is mainly defined by the mouth, eye, head, and torso movement. Therefore, we consider the motion around these regions of the face in our work.

![styletransfer1](https://vision.aioz.io/f/cd67b565d7b84df0ba0b/?dl=1)*<center>**Figure 1**. A detailed illustration of our Style-aware Talking Head Generator.</center>*



### Style Reference Images
To learn the character’s styles more effectively, we define the Style Reference Images as a set of images retrieved from a video of a specific character by using the key motion templates. Inspired by Lu et al. and music theory about rhythm, we use four key motion templates that contain popular motion range and behavior. Each behavior is then plotted as a reference style pattern, which is used to retrieve the ones that are most similar in each video in the dataset. To retrieve similar patterns, we apply similarity search for each image in the video of the character. The result image set is called the Style Reference Images and is used to provide character’s styles information in our framework.


![image](https://vision.aioz.io/f/1184fb1bd20f4afaa7af/?dl=1)*<center>**Figure 1**. An Illustration of four key motion templates..</center>* 




The style reference is expected to capture the personalized spotlight of the characters when they are talking or singing. To learn and capture the style information from a target character, we need to use the key motion templates that match with the syllable. According to music theory about rhythm, a word can have many syllables and one syllable can have more than one vowel. Vowels are a, e, i, o, u. The other letters (like b, c, d, f ) are consonants. However, each word can be split into single syllables and follow open and closed syllable patterns. A closed syllable has a short vowel ending in a consonant. It currently matches with the ‘None’ case and ‘M’ case, which are split based on the differences in mouth shape. An open syllable ends with a vowel sound that is spelled with a single vowel letter. ‘R’ case and ‘O’ case are two cases of the open syllable that have high differences in mouth motions. Each word can be formed by more than one vowel and there are seven syllable types in total for English. A visualization of the motion templates is shown in Figure 2.

### Style Mapping


The Style Mapping is designed to disentangle the style in the reference images and then map the extracted style to the neutral image.
Then, the input of this module is a pair of two images: a neutral image $\mathbf{\textit{I}}_{s}$, and a style reference image $\mathbf{\textit{I}}_{r}$. The output is an Intermediate Style Pattern (ISP - an image) which has the identity that comes from $\mathbf{\textit{I}}_{s}$ and the style represented in $\mathbf{\textit{I}}_{r}$. ISP has the visual information of the neutral image but the style is from the style reference image. In practice, we first disentangle the style information encoded in the pose and expression of both the neutral and reference image, then map the style from the reference image into the neutral image to generate the output ISP image $\mathbf{\textit{I}}_{o}$.




**Disentangling Neutral Image** Since the head pose, expression, and keypoints from the neutral image contain the style information of a specific character, they need to be disentangled to learn the style information. In this step, given an input image $\mathbf{\textit{I}}_{s}$, a set of $\rm{k}$ number of keypoints $\rm{c}_k$ is first disentangled to store the geometry signature via a Keypoint Extractor network. Then, we extract the pose, which is parameterized by a translation vector $\tau \in \mathbb{R}^3$ and a rotation matrix $\rm{R} \in \mathbb{R}^{3\times 3}$, and expression information $\varepsilon_k$ from the image by a Pose Expression network. The extracted keypoints maintain the geometry signature and style information of the head in the neutral image. 
$$
    \rm{C}_k = \rm{c}_k \times \rm{R} +\tau + \varepsilon_k \tag{1}
$$

**Disentangling Style Reference Image** Similar to the neutral image, we use two deep networks to disentangle and extract the head pose and keypoints from the style reference image. However, instead of extracting new keypoints from the reference images, we reuse the extracted ones $\rm{c}_k$ from the neutral image, which contains the identity-specific geometry signature of the neutral image. The final keypoints $\bar{\rm{C}}_k$ of the style reference image are computed as:
$$
\bar{\rm{C}}_k = \rm{c}_k \times \bar{\rm{R}} +\bar{\tau} + \bar{\varepsilon}_k \tag{2}
$$
where $\bar{\tau} \in \mathbb{R}^3$, $\bar{\rm{R}} \in \mathbb{R}^{3\times 3}$ and $\bar{\varepsilon}$ are translation vector, rotation matrix, and expression information extracted from the style reference image, respectively.

**Style Mapping** To construct the Intermediate Style Pattern $\mathbf{\textit{I}}_{o}$, we first extract two keypoints sets $\rm{C_k}$ and $\bar{\rm{C}}_k$ from the neutral image and the style reference image. We then estimate the warping function based on the two keypoints sets to warp the encoded features of the source (neutral image) to the target so that it can represent the style of the reference image. Then, we feed the warped version of the source encoded features and the extracted style information into an Intermediate Generator to obtain the ISP image. In practice, we choose the neutral image as a general image in Obama Weekly Address dataset, while the style reference image is one of the four images in the Style Reference Images set. By applying the style mapping process for all four images in the Style Reference Images, we obtain a set of four ISP images. This set (the Intermediate Style Pattern - ISP) is used as the input for the Style-Aware Generator in the next section.

### Style-Aware Generator
This module generates a 2D talking head from a source image, the generated intermediate motion, and the style information represented in the Intermediate Style Pattern. In this module, the facial map plays an essential role in explicitly identifying groups of facial keypoints, which makes the style-aware learning process easier to converge. In our experiment, the facial map has the size of $512 \times 512$ and can be obtained by connecting consecutive keypoints in a preset semantic sequence and projecting it onto the 2D image plane using a pre-computed camera matrix.

**Style-Aware Loss** To learn the style-aware loss, we introduce the style-aware photometric loss $\mathcal{L}_{\rm{sp}}$. This loss is combined with the generator loss $\mathcal{L}_{\mathbf{G}}$ to improve the generation quality and penalize the generated output that has a high deviation from the reference style patterns. The style-aware photometric loss is formulated as the pixel-wise error between the generated image $\mathbf{\textit{I}}'$ and the matched style pattern image $\mathbf{\textit{I}}_{\rm {m}}$:
$$
    \mathcal{L}_{\rm{sp}}=\Vert \mathbf{W} \odot (\mathbf{\textit{I}}' - \mathbf{\textit{I}}_{\rm {m}}) \Vert_{1} \tag{3}
$$

where $\mathbf{W}$ is the weighting mask which has values depending on different face regions; $\odot$ denotes the Hadamard product; the matched style pattern image $\mathbf{\textit{I}}_{\rm {m}}$ is obtained by using a pre-trained deep feature extractor to retrieve the best-matched image corresponding to one of the style reference images. To acquire $\mathbf{W}$, we first use an off-the-shelf face parsing method to generate the segmentation mask of the face. To achieve high fidelity image generation, we want the network to focus more on each facial region. Specifically, the corresponding weight of $\mathbf{W}$ according to mouth, eyes, and skin regions are set to  $5.0, 3.0, 1.0$, respectively. Note that weights for other regions in the weighting mask $\mathbf{W}$, e.g. background, are set to $0$.