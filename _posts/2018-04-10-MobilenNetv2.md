---
layout: post
title: Reducing MobileNetV2 Parameter Count by 30%
tags:
- deep learning
- benchmark
- ideas
id: mobilenet
author: Dushyant Mehta
---

*Some ideas on reducing the number of parameters in MobilenetV2 by replacing 1x1 expansion convolution with replication with minimal loss of accuracy*

-----
  
Before we begin, as with most ideas, I am quite sure many variants on the same theme already exist, such as [CReLU](https://arxiv.org/abs/1603.05201). 

MobileNetV2 [1], a recently proposed architecture for use on mobile phones uses bottleneck modules with depthwise 3x3 convolutions to save on computation. The bottleneck module differs from that found in ResNets [2] in that the first 1x1 layer acts to expand the number of channels and the 3x3 convolution (depthwise) acts on this expanded number of channels. Another 1x1 convolution layer then contracts the number of channels. Importantly, this final 1x1 convolution does not have a non-linearity attached. See Fig1 (a).

Incoming *c* channels get expanded to *kxc* channels by using **kxcxc** parameters, which is much more than the **kxcx9** parameters being used by the 3x3 layers. The question I wanted to address was if this expense for expanding the layer is necessary, or whether simply concatenating the input k times (also works if k is fractional) and learning separate scales and biases would be sufficient. Turns out it can bring approx. *30%* savings in the number of parameters without much degredation in performance!! Although it only translates into a modest speedup on the tested hardware. 

Fig1. below shows the various bottleneck schemes tested. The affine parameters of the BatchNorm layer after the replication layer are randomly initialized instead of a constant initialization. The replication layer is almost parameter free, except the BatchNorm and affine parameters involved which can't be trivially folded into the preceding layer for inference.

<figure class="figcenter">
  <img src="/assets/mobres/netstruct.png" alt="Proposed modifications to " width="80%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig1: Bottleneck module used in MobileNetV2 (a) and the proposed modifications (b), (c), (d). The parameter count is for 32x32 CIFAR10/100 images. The number of parameters at each layer are marked in blue.</figcaption>
</figure> 

### Experiments 
The various schemes are evaluated on CIFAR 10 and CIFAR 100, with the core network structure as described in Tab1. The networks are trained in Pytorch using SGD with a mini-batch size of 50, weight decay of 0.0001, and learning rates given in Tab2. The networks parameters are randomly initialized and average of 5 runs reported.

<table class="figtablestyle"> 
<tr>
<td>
<figure class="figcenter">
  <img src="/assets/mobres/netconfig.png" alt="Network configuration" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Tab1: MobileNetV2 configuration used for CIFAR10/100. Refer to the paper for the meaning of the column headers.</figcaption>
</figure> 
</td>

<td>
<figure class="figcenter">
  <img src="/assets/mobres/lrscheme.png" alt="learning rate scheme" width="40%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Tab2: Learning rate scheme used for the experiments. Mini-batch size of 50, weight decay of 0.0001.</figcaption>
</figure> 
</td>
</tr>
</table>

Tab3. shows the relative performance of the networks on CIFAR 10. As the parameter count goes down from 2.3M to 1.6M, the test error goes up from 8.65% (a) to 9.37% (b). Removing Batch Norm from the last 1x1 layer sees a drop of 0.1M in the parameter count with the test error increasing to 9.54% (c). Swapping the order of the 3x3 convolution and the replication layer sees a significant performance drop to 11.15% (d) test error. Tab4. shows a similar trend for CIFAR 100. Also see Fig4. and Fig5.

<table class="figtablestyle"> 
<tr>
<td>
<figure class="figcenter">
  <img src="/assets/mobres/cifar10.png" alt="CIFAR 10 performance" width="95%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Tab3: Train loss, test loss, and test error for the analysed network configurations on CIFAR 10. Checkpoint at epoch 250, averaged over 5 runs.</figcaption>
</figure> 
</td>
<td>
<figure class="figcenter">
  <img src="/assets/mobres/cifar100.png" alt="CIFAR 100 performance" width="95%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Tab4: Train loss, test loss, and test error for the analysed network configurations on CIFAR 100. Checkpoint at epoch 450, averaged over 5 runs.</figcaption>
</figure> 
</td>
</tr>
</table>


<figure class="figcenter">
  <img src="/assets/mobres/cifar10_all.png" alt="CIFAR 10 All" width="90%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig4: The proposed network schemes evaluated on CIFAR 10. Visualized are the best, mean, and worst of the 5 runs per network scheme. The horizontal axis marks the epochs.</figcaption>
</figure> 
<figure class="figcenter">
  <img src="/assets/mobres/cifar100_all.png" alt="CIFAR 100 All" width="90%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig5: The proposed network schemes evaluated on CIFAR 100. Visualized are the best, mean, and worst of the 5 runs per network scheme. The horizontal axis marks the epochs.</figcaption>
</figure> 

### Timing Benchmarks 
<figure class="figcenter">
  <img src="/assets/mobres/benchmark.png" alt="Timing benchmarks" width="60%" display="block" margin-left="auto" margin-right="auto">
  <figcaption>Fig6: Forward pass and backward pass (in brackets) timing of ResNet50, MobileNetV2, and MobileNetV2 with expansion (b) on various hardware and for various different image resolutions. The timings are in milliseconds. The performance is comparable for small to moderate image resolutions, but the proposed modifications are faster for large image resolutions.</figcaption>
</figure> 

### Conclusion
The proposed modifications to MobileNetV2 lead to significant savings in the number of parameters, and are shown to be promising on CIFAR10/100. These modifications remain to be tested on other tasks covered in the paper, as well as on mobile hardware so as to gauge performance in the real world. 

### References
1. [Inverted Residuals and Linear Bottlenecks: Mobile Networks for Classification, Detection and Segmentation](https://arxiv.org/abs/1804.07573)
2. [Deep residual learning for image recognition](https://arxiv.org/abs/1512.03385)
