---
title: '[Reading Notes] Deep Extraction of Manga Structural Lines'
date: 2018-09-07 20:10:07
tags: [deep learning, CG]
category: 学习一个
---
[This](http://delivery.acm.org/10.1145/3080000/3073675/a117-li.pdf?ip=137.189.90.241&id=3073675&acc=ACTIVE%20SERVICE&key=CDD1E79C27AC4E65%2E63D3CA449C1BD759%2E4D4702B0C3E38B35%2E4D4702B0C3E38B35&__acm__=1536341127_445dc05cd8d2a0df2cb4df34421b6d4a) is a work published on Tansaction on Graphics by Chengze Li et al. It aims to extract structural lines from pattern-rich manga. Its contributions are: (1)Novel CNN frame-work tailored for their extraction task; (2) An ingenious way of "reversely generating" data set, which well solved the requirement of demanding large amount of data by deep learning methods. (3) proposed some potential applications utilizing this work.

<!--more-->
## Purpose
the purpose can be intuitively presented by the cover picture of this paper:

{% asset_img cover_photo.png cover photo %}

## Highlights

### Comparison with related works
In section 2, Related Works, The author classify existing methods to 3 categories (edge detection, texture removal and CNN-based techniques), then compare each catgory method with theirs to distinguish this work, so this part is clear and well-organized.

### Training data generation
In section 4.1, instead of manually tracing structural lines in screen patterns decorated mangas, they generate data set inversely, i.e. sythesizing screen-rich manga from the screen-free line drawings, by laying a rich library of screen patterns.

{% asset_img dataset.png dataset sythsizing %}

To allign the real world distribution of patterns, they imposed some heuristic rules. 

### CNN frame-work design
{% asset_img network.png network structure %}

In section 4.2, the authors presented the network design. Although it's composed all by existing layers, they persuasivly illstrated why they design the network in this way
1. why three levels of downsampling: experiments show that increasing levels filters away important structural components and leads to blurry structural lines, decreasing levels cannot filter away textural pattern due to limited receptive field.
2. why use residual network(common sense): it allows direct information propagation between the input and output -- easy training, better quality.
3. why use convolutions/convolutions for downsampling/upsampling, instead of standard pooling/unpooling: max-pooling breaks spatial continuity.
4. why use skipping (common sense): information at certain level in the downsampling network can be directly passed to the corresponding layer of upscaling network without compression.
5. why use MSE instead of MAE as regularization: keep the tones of structural lines.

### Potential applications 
ommitted.

## My thoughts
1. **Really no assumption?** Does this method work when apply test on a picture with the screen patterns do not emerged in pattern library used to sythsize training data set? Especially I see on page 5, the paper mentions "At first glance, it seems...may not exist in real world." which implies the sensitivity?

2. **Can it outperform GAN-based methods?** e.g. [pix2pix](https://arxiv.org/abs/1611.07004)

3. **How to utilize the sparsity.** Notice the sparsity (most of the pixels are filled with pure white) of structural line drawings, how can we utilize the property? Maybe an unevenly weighted(larger coefficients on false-positive terms) l2 regularization? Or define a structural MAE with 8-connected region (to avoid losing tone but keep sparse output)?

4. Image to image translation can be regarded as a combination of CV and CG, the downscaling part is CV job, and upscaling part is for CG.


## Retrospect Some knowlege points

1. [Deconvolution](https://www.zhihu.com/question/43609045/answer/130868981) (precisely, transposed convolution) 
2. [Unpooling](https://blog.csdn.net/A_a_ron/article/details/79181108)