---
last_modified_on: "2023-15-04"
id: GDANCE
title: Music-Driven Group Choreography (Part 2)
description: Methoology part, a group dance generation baseline of AIOZ-GDANCE
author_github: https://github.com/aioz-ai
tags: ["type: research", "AI", "Computer Graphic"]
---
In the previous post, we have introduced AIOZ-GDANCE, a new largescale in-the-wild dataset for music-driven group dance generation. On the basis of the new dataset, we introduce the first strong baseline for group dance generation that can jointly generate multiple dancing motions expressively and coherently.

![](https://vision.aioz.io/f/779cd3f378f446b796be/?dl=1)*<center>**Figure 1**. The overall architecture of GDanceR. Our model takes in a music sequence and a set of initial positions, and then auto-regressively generates coherent group dance motions that are attuned to the input music. </center>* 



## Music-driven Group Dance Generation Method

### Problem Formulation
Given an input music audio sequence $\{m_1, m_2, ...,m_T\}$ with $t = \{1,..., T\}$ indicates the index of music segments, and the initial 3D positions of $N$ dancers $\{\tau^1_0, \tau^2_0, ..., \tau^N_0 \}$, $\tau^i_0 \in \mathbb{R}^{3}$, our goal is to generate the group motion sequences $\{y^1_1,..., y^1_T; ...;y^n_1,...,y^n_T\}$ where $y^i_t$ is the generated pose of $i$-th dancer at time step $t$. Specifically, we represent the human pose as a 72-dimensional vector $y = [\tau;  \theta]$ where $\tau$, $\theta$ represent the root translation and pose parameters of the SMPL model [1], respectively.

In general, the generated group dance motion should meet the two conditions: *(i)* consistency between the generated dancing motion and the input music in terms of style, rhythm, and beat; *(ii)* the motions and trajectories of dancers should be coherent without cross-body intersection between dancers. To that end, we propose the first baseline method, for group dance generation that can jointly generate multiple dancing motions expressively and coherently. Figure 1 shows the architecture of our proposed Music-driven 3D **G**roup **Dance** generato**R** (GDanceR), which consists of three main components:
* Transformer Music Encoder.
* Initial Pose Generator.
* Group Motion Generator. 

### Transformer Music Encoder


From the raw audio signal of the input music, we first extract music features using the available audio processing library Librosa. Concretely, we extract the mel frequency cepstral coefficients (MFCC), MFCC delta, constant-Q chromagram, tempogram, onset strength and one-hot beat, which results in a 438-dimensional feature vector. We then encode the music sequence $M =\{m_1, m_2, ...,m_T\}$, $m_t \in \mathbb{R}^{438}$ into a sequence of hidden representation $\{a_1, a_2,..., a_T\}$, $a_t \in \mathbb{R}^{d_a}$. In practice, we utilize the self-attention mechanism of transformer [2] to effectively encode the multi-scale information and the long-term dependency between music frames. The hidden audio at each time step is expected to contain meaningful structural information to ensure that the generated dancing motion is coherent across the whole sequence.

Specifically, we first embed the music features $m_t$ using a Linear layer followed by Positional Encoding to encode the time ordering of the sequence.
$$
U = \text{PE}({M} W^u_a)
$$
where $\text{PE}$ denotes the Positional Encoding, and $W^u_a \in \mathbb{R}^{438 \times d_a}$ is the parameters of the linear projection layer. Then, the hidden audio information can be calculated using self-attention mechanism:
$$
\\ \mathbb{A} = \text{FF}(\text{softmax}\left(\frac{(U^q U^k)^T}{\sqrt{d_{k}}} \right) U^v ), \\
U^q = U W^q_a, \quad U^k = U W^k_a, \quad U^v = UW^v_a 
$$

where $W^q_a, W^k_a \in \mathbb{R}^{d_a \times d_k}$, and  $W^v_a \in \mathbb{R}^{d_a \times d_v}$ are the parameters that transform the linear embedding audio $U$ into a query $U^q$, a key $U^k$, and a value $U^v$ respectively. $d_a$ is the dimension of the hidden audio representation while  $d_k$ is the dimension of the query and key, and $d_v$ is the dimension of value. $\text{FF}$ is a feed-forward neural network.

### Initial Pose Generator
![](https://vision.aioz.io/f/3155b7d233554fbfab73/?dl=1)*<center>**Figure 2**.The Transformer Music Encoder encodes the acoustic and rhythmic information to generate the initial poses from the input positions. </center>* 




Given the initial positions of all dancers, we generate the initial poses by combing the audio feature with the starting positions. We aggregate the audio representation by taking an average over the audio sequence. The aggregated audio is then concatenated with the input position and fed to a multilayer perceptron (MLP) to predict the initial pose for each dancer:
$$
y^i_0 = \text{MLP}\left( \left[\frac{1}{T}\sum_{t=1}^T a_t ; \tau^i_0 \right] \right),
$$
where $[;]$ is the concatenation operator, $\tau^i_0$ is the initial position of the $i$-th dancer.


### Group Motion Generator

![](https://vision.aioz.io/f/a79f22f06374466fb250/?dl=1)
*<center>**Figure 3**. The Group Motion Generator auto-regressively generates coherent group dance motions based on the encoded acoustic information. </center>* 


To generate the group dance motion, we aim to synthesize the coherent motion of each dancer such that it aligns well with the input music. Furthermore, we also need to maintain global consistency between all dancers. As shown in Figure 3, our Group Generator comprises a Group Encoder to encode the group sequence information and an MLP Decoder to decode the hidden representation back to the human pose space. To effectively extract both the local motion and global information of the group dance sequence through time, we design our **Group Encoder** based on two factors: Recurrent Neural Network [3] to capture the temporal motion dynamics of each dancer, and Attention mechanism [2] to encode the spatial relationship of all dancers. 


Specifically, at each time step, the pose of each dancer in the previous frame $y^i_{t-1}$ is sent to an LSTM unit to encode the hidden local motion representation $h^i_t$:
$$
{h^i_t=\text{LSTM}(y^i_{t-1},h^i_{t-1})}
$$
To ensure the motions of all dancers have global coherency and discourage strange effects such as cross-body intersection, we introduce the **Cross-entity Attention** mechanism. In particular, each individual motion representation is first linearly projected into a key vector $k^i$, a query vector $q^i$ and a value vector $v^i$ as follows: 
\begin{equation}
    k^i = h^i W^{k}, \quad q^i = h^i W^{q}, \quad v^i = h^i W^{v},
\end{equation}
where $W^q, W^k \in \mathbb{R}^{d_h \times d_k}$, and  $W^v \in \mathbb{R}^{d_h \times d_v}$ are parameters that transform the hidden motion $h$ into a query, a key, and a value, respectively. $d_k$ is the dimension of the query and key while $d_v$ is the dimension of the value vector. To encode the relationship between dancers in the scene, our Cross-entity Attention also utilizes the Scaled Dot-Product Attention as in the Transformer [3].

![](https://vision.aioz.io/f/ff5d7f4e917a47028e13/?dl=1)*<center>**Figure 4**. The Group Encoder learns to encode the relations among dancers through our proposed Cross-entity Attention mechanism. </center>* 

In practice, we find that people having closer positions to each other tend to have higher correlation in their movement. Therefore, we adopt **Spacial Encoding** strategy to encode the spacial relationship between each pair of dancers. The Spacial Encoding between two entities based on their distance in the 3D space is defined as follows:
$$
e_{ij} = \exp\left(-\frac{\Vert \tau^i - \tau^j \Vert^2}{\sqrt{d_{\tau}}}\right),
$$
where $d_{\tau}$ is the dimension of the position vector $\tau$. Considering the query $q^i$, which represents the current entity information, and the key $k^j$, which represents other entity information, we inject the spatial relation information between these two entities onto their cross attention coefficient: 
$$
\alpha_{ij} = \text{softmax}\left(\frac{(q^i)^\top k^j}{\sqrt{d_k}} + e_{ij}\right).
$$

To preserve the spatial relative information in the attentive representation, we also embed them into the hidden value vector and obtain the global-aware representation $g^i$ of the $i$-th entity as follows:
$$
g^i = \sum_{j=1}^N\alpha_{ij}(v^j +  e_{ij}\gamma),
$$
where $\gamma \in \mathbb{R}^{d_v}$ is the learnable bias and scaled by the Spacial Encoding. Intuitively, the Spacial Encoding acts as the bias in the attention weight, encouraging the interactivity and awareness to be higher between closer entities. Our attention mechanism can adaptively attend to each dancer and others in both temporal and spatial manner, thanks to the encoded motion as well as the spatial information.

We then fuse both the local and global motion representation by adding $h^i$ and $g^i$ to obtain the final latent motion $z^i$. Our final global-local representation of each entity is expected to carry the comprehensive information of their own past motion as well as the motion of every other entity, enabling the **MLP Decoder** to generate coherent group dancing sequences. Finally, we generate the next movement ${y}^i_t$ based on the final motion representation $z^i_t$ as well as the hidden audio representation $a_t$, and thus can capture the fine-grained correspondence between music feature sequence and dance movement sequence:
$$
y^i_t = \text{MLP}([z^i_t; a_t]).
$$

Built upon these components, our model can effectively learn and generate coherent group dance animation given several pieces of music. In the next part, we will go through the experiments and detailed studies of the method.



## References


[1] Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, and Michael J. Black. SMPL: A skinned multiperson linear model. ACM Trans. Graphics, 2015


[2] Vaswani A, Shazeer N, Parmar N, Uszkoreit J, Jones L, Gomez AN, Kaiser Ł, Polosukhin I. Attention is all you need. NIPS 2017.

[3] Hochreiter S, Schmidhuber J. Long short-term memory. Neural computation. 1997 Nov 15;9(8):1735-80.