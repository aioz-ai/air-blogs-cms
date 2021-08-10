---
last_modified_on: "2021-08-09"
title: A quick guide on principal component analysis
description: A study about principal component analysis
series_position: 2
author_github: https://github.com/trqminh
tags: ["type: tutorial", "level: medium"]
---

## Problem formulation
Let the data $X$,
$$

X = {x^{(1)}, x^{(2)}, ..., x^{(m)}}
x^{(i)} = {x_1^{(i)}, x_2^{(i)}, ..., x_n^{(i)}}
X \in \mathbb{R}^{m\times{n}}

$$
The result $C$ which we expect to have the smaller dimensions,

$$

C = {c^{(1)}, c^{(2)}, ..., c^{(m)}}
c^{(i)} = {c_1^{(i)}, c_2^{(i)}, ..., c_k^{(i)}}
C \in \mathbb{R}^{m\times{k}}

$$

## Intuition
PCA is an unsupervised machine learning algorithm as it learns from the data itself, or more concretely, it's based on the variance of the data in each dimension in order to remove those which have the smaller one.<br/>
The objective now is to find the new basis in which the variances of the data in some dimensions are small and we can skip them. With no loss of generality, we assume this basis is orthonormal for the purpose of making the optimization easier.

Let the orthonormal basis $U \in \mathbb{R}^{n\times{n}}$, $\widetilde{X}$ is the data $X$ perform in $U$, $D\in \mathbb{R}^{n\times{k}}$ is the first $k$ eigenvector of $U$ which have highest eigenvalues, $EY$ is the remainder, we have:

$$

X^{T} = U\widetilde{X}^{T}
X^{T} = DC^{T} + EY
X^{T}\approx{DC^{T}}

$$

## Optimization
**note**: in this part, I 'll skip the optimization step and only show the result.
Apparently, we need to solve this optimization problem, namely:
$$
\min_{C,D} ||X^{T} - DC^{T}||_2
$$

We can split it and deal with $C^{*}$ first:
$$
C^{*} = \arg\min_{C} ||X^{T} - DC^{T}||_2
$$

and the result is (the fact that columns in $D$ are orthonormal make the result more compact):

$$
C^{*^{T}} = D^{T}X^{T}
$$

then:
$$
X \approx DD^{T}X^{T}
$$
now let optimize $$D$$:
<!-- \\[ D^{*} = \arg \min_{D} \||X - DD^{T}X^{T}|\|_2 ~~~~~~(2) \\] -->
$$
D^{*} = \arg{\min_D} \||X - DD^{T}X^{T}|\|_2
$$

As a result, $D\in \mathbb{R}^{n\times{k}}$ has their columns as $k$ eigenvectors with the highest eigenvalues of the matrix $X^{T}X$.

## Step (code)
```python
# my code on understanding pca
import numpy as np
import numpy.linalg as linalg
import matplotlib.pyplot as plt


m = 100  # num of sample
n = 2  # num of dim (vector size)
k = 1  # num of dim to take down
x = np.random.randn(m, n)

# compute the S matrix
mean = np.mean(x, axis=0)
x_ = x - mean  # subtract mean
S = np.matmul(x_.transpose(), x_)  # the S matrix

# compute eigen vector of S
eig = linalg.eig(S)

# the decode matrix D
D = eig[1][:k]  # get first k eigenvector (nx1) with highest eigenvalue
D = D.reshape(-1, k)  # change the shape to n x k

# the encode E
E = D.transpose()
z = np.matmul(E, x.transpose()).transpose()  # z (m x k), the result of pca

# visualize
# get two eigenbasis in new basis
d1 = eig[1][0]
d2 = eig[1][1]

# check d1 and d2 are orthonormal
print(linalg.norm(d1))  # 1.0
print(linalg.norm(d2))  # 1.0
print(np.dot(d1, d2))  # 0.0

# plot the x
t = 4
plt.scatter(x[:, 0], x[:, 1])

plt.arrow(0, 0, t*d1[0], t*d1[1], fc='red', ec='red')
plt.arrow(0, 0, t*d2[0], t*d2[1], fc='blue', ec='blue')

plt.margins(x=0.5, y=0.5)
plt.show()
```
<img src="https://vision.aioz.io/thumbnail/8c9c196de3cb4a3a990f/1024/pca.png" class="fit image"/>
<br/>
*Fig. 1: Two eigenvectors, the red one has the higher eigenvalue, project the data to this vector to get the new data with the dimension k=1*

## Discussion
There are some questions that reader can working on to comprehend the algorithm:
- Why the orthogonal basis is chosen. Why not another basis? Whether or not that generality is guaranteed and other bases can give better results.

- Investigating the details of optimization procedure can help with solving the above question and gain more insights.

Hopefully that this tutorial can help.
