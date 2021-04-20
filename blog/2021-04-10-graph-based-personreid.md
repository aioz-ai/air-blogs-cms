---
last_modified_on: "2021-04-12"
id:  gps-reid
title: Graph-based Person Signature for Person Re-Identifications
description: "We introduce a new Light-weight Deformable Registration network."
author_github: https://github.com/xuanbinh-nguyen96
tags: ["type: post", "AI", "Computer Vision", "Person ReID", "Re-identification", "Graph-based"]
---

# Abstract
The task of person re-identification (ReID) is to match images of the same person over multiple non-overlapping camera views. Due to the variations in visual factors, previous works have investigated how the person identity, body parts, and attributes benefit the person ReID problem. However, the correlations between attributes, body parts, and within each attribute are not fully utilized. In this paper, we propose a new method to effectively aggregate detailed person descriptions (attributes labels) and visual features (body parts and global features) into a graph, namely Graph-based Person Signature, and utilize Graph Convolutional Networks to learn the topological structure of the visual signature of a person. The graph is integrated into a multi-branch multi-task framework for person re-identification. The extensive experiments are conducted to demonstrate the effectiveness of our proposed approach on two large-scale datasets, including Market-1501 and DukeMTMC-ReID. Our approach achieves competitive results among the state of the art and outperforms other attribute-based or mask-guided methods.
