---
layout:     post
title:      "Deep Learning with Torch"
subtitle:   "The Long way of Deep Learning with Torch"
date:       2015-10-01 12:00:00
author:     "Fabio Fumarola"
header-img: "img/post.jpg"
tags:       [torch,neural-networks]
---

## Deep Learning with Torch
I come to deep learning from [char-rnn project](https://github.com/karpathy/char-rnn). I didn't know lua nor torch and it was very hard to understand what was done in the code. I spent the first 2 weeks rewriting the project and doing experiments to get used to the language.
I was sure to be ready and I have implemented a complete revisitation of the original implementation.
However, I found the LSTM model such as gibberish, because I'm really not used to the language and to the torch packages. This is because I decided to write a blog series titled "The Long way of Deep Learning with Torch".

## What is Deep Learning

Broadly speaking, the goal of deep learning is to model complex, hierarchical features in data. "Deep learning" is not a particular type of algorithm, such as feedforward feedforward neural networks (FF nets for short) or SVMs, but rather a set of machine learning algorithms. In fact, any learning algorithm that learns a distributed representation of its input data could be considered a "deep learning algorithm". These algorithms can be used in both unsupervised and supervised learning.

- [FF nets](https://en.wikipedia.org/wiki/Feedforward_neural_network) are useful in deep learning problems because they consist of hidden units *h1…hn*, where each can learn an increasingly high-level representation of data. The input layer mapping from one layer to the next can vary from algorithm to algorithm.
- In [Deep belief networks](http://en.wikipedia.org/wiki/Deep_belief_network), the units are [Restricted Boltzmann machines](http://en.wikipedia.org/wiki/Restricted_Boltzmann_machine). A very powerful deep learning algorithm is the [Convolutional Neural Network](http://en.wikipedia.org/wiki/Convolutional_neural_network). This learning algorithm takes advantage of pooling layers, which combine the outputs of neuron clusters, and shared weights. These algorithms often are comparatively computationally efficient, while still having a low error rate:

> As of 2011, the state of the art in deep learning feedforward networks alternates convolutional layers and max-pooling layers, topped by several pure classification layers. Training is usually done without any unsupervised pre-training. Since 2011, GPU-based implementations of this approach won many pattern recognition contests, including the IJCNN 2011 Traffic Sign Recognition Competition, the ISBI 2012 Segmentation of neuronal structures in EM stacks challenge, and others.

(Source: [Deep learning](http://en.wikipedia.org/wiki/Deep_learning#Deep_learning_in_artificial_neural_networks) article from Wikipedia).

If you're looking for a technical introduction to deep architectures, I recommend:
- this review paper by Yoshua Bengio: [Page on iro.umontreal.ca](http://www.iro.umontreal.ca/~bengioy/papers/ftml.pdf), and
- the website [Deep Learning](http://deeplearning.net/).

## Definition of Deep Learning

As defined in Bengio et al. [Representation Learning: A Review and New
Perspectives](http://arxiv.org/pdf/1206.5538v3.pdf), deep Learning can be defined as constructing multiple levels of representation or learning a hierarchy of features.

## The Long way of Deep Learning with Torch

Here it is the list of arguments faced:

- [initial setup](/2015/10/02/deep_learning_with_torch_step_0_setup/)
- [NN Modules](/2015/10/03/deep_learning_with_torch_step_1_nn_module/)
- [NN Containers](/2015/10/04/deep_learning_with_torch_step_2_nn_containers/)
- [NN Criterions](/2015/10/05/deep_learning_with_torch_step_3_nn_criterions/)
- [NN Graph](/2015/10/07/deep_learning_with_torch_step_4_nngraph/)
- [RNN and LSTM](/2015/10/12/deep_learning_with_torch_step_5_rnn_lstm/)
- [Dropout](/2015/10/20/deep_learning_with_torch_step_6_dropout/)
- [Optim](/2015/10/26/deep_learning_with_torch_step_7_optim/)
- [Eval and Early Stop](/2015/11/03/deep_learning_with_torch_step_8_early_stop/)

Next Posts, every Monday

- [Cuda and OpenCL]()
- [Plotting]()
- [Convolutional]()
- [Other Models]()
