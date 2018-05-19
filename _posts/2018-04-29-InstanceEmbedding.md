---
layout: post
title: Instance Embedding for Proposal Free Object Detection
tags:
- deep learning
- reading
id: instance_embedding
author: Dushyant Mehta
---

*Summary of some ideas from current literature on proposal free object detection via instance embedding*

-----
  
As with the [previous post](/2018/04/25/DynamicRouting/), I read up on this while looking at ways to address the problem of [multi-person 3D pose estimation](https://arxiv.org/abs/1712.03453) [1]. The problem of estimating the 3D articulation of multiple subjects in a scene can be viewed through an object detection lens, where first individuals are localized in a scene and then their specific features used to predict their 3D articulation.

This post was inspired by [2].

Downsides of boudning-box proposal based object detection approaches such as [3] [4]:
1. Too many proposals are generated, many of which overlap and need a fusion step afterwards, which itself needs tuning.
2. If a bounding box has multiple objects, it is unclear which one should be selected. Objects under heavy occlusion, and objects with thin profiles are the most affected.
3. Feature pooling in the bounding box mixes up features of different objects  

### Semantic Instance Segmentation with a Discriminative Loss Function [5]
The approach produces a per pixel embedding that can be used in a post-processing step tp cluster pixels from the same object. The objective that is minimized is comprised of 3 terms: one to encourage intra pixel-cluster embeddings to be similar, one to separate inter pixel-cluster embeddings, and a regularization term.
The clustering takes place within a threshold margin around the cluster representative.

<figure class="figcenter">
  <img src="/assets/instaembed/embed_cluster.png" alt="Cluster evolution" width="80%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig1: Figure from [5] showing the evolution of the cluster embedding as training progresses. Top row shows the embeddings in 2D space colored by the ground truth labels. As training progresses the embedding separates the pixels into clear clusters. Middle row visualizes the embedding in the image space. Bottom row is the thresholded embedding showing the clusters.</figcaption>
</figure> 

The approach is very similar to Associative Embedding [6].

###  Semantic Instance Segmentation via Deep Metric Learning [7]
K pixel samples per instance are chosen and a pairwise similarity loss computed on the embedding based on whether the samples belong to the same instance or not.
Additionally, for each pixel a seediness score is predicted to indicate if the pixel would make a good seed for starting a mask. A predicted (selected) threshold value guides how similar pixel embeddings must be to the current seed to be clustered together. Seeds are sampled sufficiently far in embedding space from each other to ensure good spatial coverage. Importantly, the seediness score is actually trained by growing a mask from randomly selected seeds and using the classification score of the mask.

<figure class="figcenter">
  <img src="/assets/instaembed/embed_seed.png" alt="schema for instance embedding" width="80%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig2: Figure from [7] showing joint prediction of per-pixel embedding and a seediness score to determine where to start clustering from.</figcaption>
</figure> 

### Recurrent Pixel Embedding for Instance Grouping [8]
This has the same idea as the ones we have talked about before, producing an embedding for each pixel such that pixels from the same group have a high (cosine) similarity in the embedding space and those from different groups are separated by a margin. However, the instance grouping step from seeds is phrased as a recurrent task, and the whole setup trained end-to-end.


### Related Work
#### Uncategorized 
- [R-FCN-3000 at 30fps: Decoupling Detection and Classification](https://arxiv.org/abs/1712.01802)
- [DeepVoting: A Robust and Explainable Deep Network for Semantic Part Detection under Partial Occlusion](https://arxiv.org/abs/1709.04577)
- [Focal Loss for Dense Object Detection](https://arxiv.org/abs/1708.02002): Proposes to handle the class imbalance issue by downweighting easier examples

#### Recurrent Methods for Object Detection / Instance Segmentation
- [End to End People Detection in Crowded Scenes](https://arxiv.org/abs/1506.04878)
- [Recurrent Instance Segmentation](https://arxiv.org/abs/1511.08250)
- [End to End Instance Segmentation with Recurrent Attention](https://arxiv.org/abs/1605.09410)

### References
1. [Single-Shot Multi-Person 3D Pose Estimation From Monocular RGB Input](https://arxiv.org/abs/1712.03453)
2. [Instance Embedding: Segmentation Without Proposals](https://towardsdatascience.com/instance-embedding-instance-segmentation-without-proposals-31946a7c53e1)
3. [Faster-RCNN](https://arxiv.org/abs/1506.01497)
4. [Mask-RCNN](https://arxiv.org/abs/1703.06870)
5. [Semantic Instance Segmentation with a Discriminative Loss Function](https://arxiv.org/abs/1708.02551)
6. [Associative Embedding: End-to-End Learning for Joint Detection and Grouping](https://arxiv.org/abs/1611.05424)
7. [Semantic Instance Segmentation via Deep Metric Learning](https://arxiv.org/abs/1703.10277)
8. [Recurrent Pixel Embedding for Instance Grouping](https://arxiv.org/abs/1712.08273)
