---
layout: post
title: Dynamic Routing in NNs and Related Ideas
tags:
- deep learning
- reading
id: dynamic_routing
author: Dushyant Mehta
---

*Summarization of some ideas on dynamic routing and related approaches from literature.*

-----

There are a couple of hinderances to real-time multi-person 3D pose estimation that I am looking to address, namely data imbalance and non-judicious use of computation. 
1. Data imbalance is a double whammy because it is usually the difficult to predict cases with complex interaction and occlusion that also have the fewest exemplars. 
2. Current NN architectures give the benefit of the complete network depth equally to all pixels, which is wasteful and slows things down substantially when full frames need to be processed. See Fig1.

 <figure class="figcenter">
  <img src="/assets/dynaRoute/multi_person.jpg" alt="Multi-person pose example" width="40%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 1: Multiperson pose estimation example from [1] using a fully convolutional network. It would be prudent to not spend as much computation on empty regions in the scene as on the regions containing people. </figcaption>
</figure> 
I came up with an idea to address both simultaneously through a combination of spatially selective allocation of computation and difficulty aware allocation of layers to the task. As is the case with all ideas, there has been plenty of work in this direction. Here is a quick summary of a few:

### Not All Pixels are Created Equal [2]
Li et al. propose to speed up and improve the quality of semantic segmentation by allocating computation in accordance with the difficulty of segmentation of various spatial regions in the image. See Fig2.
 <figure class="figcenter">
  <img src="/assets/dynaRoute/napce.jpg" alt="Not all pixels are created equal" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 2: Easy, moderate, and difficult to segment regions in an image. Figure from [2]. </figcaption>
</figure>
They achieve this through an NN cascade which focuses on progressively difficult regions through the use of region convolutions. See Fig3 and Fig4.
 <figure class="figcenter">
  <img src="/assets/dynaRoute/napce2.jpg" alt="NN cascade" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 3: NN cascade with more computation/depth allocated to difficult regions. Figure from [2]. </figcaption>
</figure>
The masked cascade design implicitly handles data imbalance issues, with Stage2 and 3 not seeing the loss gradients of difficult examples being drowned out by loss gradients of many more easier examples.
 <figure class="figcenter">
  <img src="/assets/dynaRoute/napce3.jpg" alt="Region Convolution" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 4: Region convolution to focus computation on the masked region. Figure from [2]. </figcaption>
</figure>
Computational savings come from region convolution (Fig4), which doesn't spend computations on regions that would be masked out. The approach uses per pixel classification confidence at each stage to decide whether to engage the subsequent stage. The choice of which stage processes a particular pixel is implicitly dependent on the apparent difficulty of classifying the pixel, and not learned through an explicitly defined objective.

It remains an open problem to extend such an approach to regression problems, and problems where there isn't an image-to-image mapping.


### I Don't Know Cascades [3]
Wang et al. have a similar difficulty based computation allocation idea as [2], but instead of per-pixel decisions, the decisions are taken per example.
The idea of cascades is nothing new, being famously used in [Viola-Jones detector](https://en.wikipedia.org/wiki/Viola%E2%80%93Jones_object_detection_framework) for early rejection of negatives. 
 <figure class="figcenter">
  <img src="/assets/dynaRoute/idk.png" alt="IDK Cascade" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 5: IDK cascade design, engaging more computation if a simpler model was not confident in its prediction. Figure from [3]. </figcaption>
</figure>

Each stage in an IDK cascade outputs a target prediction and an uncertainty signal (Fig5). If the uncertainty signal exceeds a threshold, the subsequent stage is engaged. This cascaded design does not implicitly addresses data imbalance between easy and difficult targets, though it results in amortized computational savings. Things could be improved though.

The models in the cascade are pre-trained, and various learned selection approached proposed to produce the IDK signal, to jointly optimize the accuracy and the computation cost. This optimization does not modify the models the cascade is composed of.

### SkipNet [4]
From the same first author as [3], this work looks at learning to dynamically engage different layers of the network on a per-input basis. The motivation goes beyond saving computation, and can have the training examples split between combinatorially many implicit networks. The decision to bypass a particular layer is based on the outputs of the preceding layers.

 <figure class="figcenter">
  <img src="/assets/dynaRoute/skipnet.png" alt="SkipnNet" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 6: SkipNet, with input dependent activation paths in the network. Figure from [4]. </figcaption>
</figure>

Since hard gating of layers/blocks is required to be able to see any computational savings, re-inforcement learning is employed. The paper claims that approximating hard gating using soft gating while training results in low test accuracy. Interestingly, the paper shows that it is hard to train the gating using re-informcement learning when the network starts from a random initialization. Hence the network is pretrained with soft-gating. Also, the training signal is not entirely through REINFORCE, and the main network get its supervision signal through the task objective loss (classification in this case).

It is observed that the network routes difficult examples through more layers, as compared to easier examples.

### Deciding How to Decide [5]
McGill and Perona explore a similar idea as [3] and evaluate different architectural designs and training policies towards a similar end. See Fig7.
 <figure class="figcenter">
  <img src="/assets/dynaRoute/dynaroute.jpg" alt="Dynamic Routing" width="70%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig 7: Learned dynamic routing. Figure from [5]. </figcaption>
</figure>
There are various training policies evaluated to balance the cost of computation and prediction accuracy. An interesting obervation from the paper is that in the beginning of training, the policy chooses shorter paths, going deeper as the training progresses. This seems to have some parallels with [Looks Linear](http://proceedings.mlr.press/v70/balduzzi17b/balduzzi17b.pdf) initialization.

### .. and more
There are a few more that are worth mentioning, which I haven't had the time to read in detail yet. We have seen examples with per-example and per-pixel dynamic computation allocation. See [6] [7] for examples of the same in the temporal dimension, and [8] for spatio-temporal routing.

### References
1. [Single-Shot Multi-Person 3D Pose Estimation From Monocular RGB Input](https://arxiv.org/abs/1712.03453)
2. [Not All Pixels Are Created Equal: Difficulty-Aware Semantic Segmentation via Deep Layer Cascade](https://liuziwei7.github.io/projects/LayerCascade.html)
3. [IDK Cascades: Fast Deep Learning by Learning Not to Overthink](https://arxiv.org/abs/1706.00885)
4. [SkipNet: Learning Dynamic Routing in Convolutional Networks](https://arxiv.org/abs/1711.09485)
5. [Deciding How to Decide: Dynamic Routing in Artificial Neural Networks](https://arxiv.org/abs/1703.06217)
6. [Clockwork Convnets for Video Semantic Segmentation](https://arxiv.org/abs/1608.03609)
7. [Deep Feature Flow for Video Recognition](https://arxiv.org/abs/1611.07715)
8. [Dynamic Video Segmentation Network](https://arxiv.org/abs/1804.00931)



