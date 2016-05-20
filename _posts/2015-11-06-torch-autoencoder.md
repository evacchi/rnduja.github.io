---
layout:     post
title:      "A Torch autoencoder example"
subtitle:   "Extracting features from MNIST digits"
date:       2015-11-06 12:00:00
author:     "Andrea Ferretti"
header-img: "img/2015-11-06-torch-autoencoder/cover.jpg"
comments: true
tags:       [torch,neural-networks]
---

In the [previous example](/2015/10/13/torch-mnist), we have seen how to set up a simple Torch application to recognize digits from the MNIST set.

In this [iTorch](https://github.com/facebook/iTorch) notebook, we will see how neural networks can be used to extract features out of a dataset in a completely
unsupervised way.

A simple way to do this is with an *autoencoder*, which is just a neural network with just a hidden layer, where the output and the input layer are the same. So we
are asking of a way to to factorize the identity through a nonlinear layer. The nonlinearity guarantees that the network will not just learn a factorization of the
identity matrix.

Moreover, one can force the hidden layer to be smaller than the other one, so that the factorization has to pass through a lower dimension,
further enhancing the robustness of the learned model. Also, [dropout](https://www.cs.toronto.edu/~hinton/absps/JMLRdropout.pdf) can be used
to guarantee that hidden neurons will learn independent features.

The features that we learn in this way are nonlinear functions of the input. We can see them explicitly by taking a basis vector in the hidden layer and
applying the transformation to the output layer. In other words, features correspond to column vectors of the hidden/output matrix. This is particularly
convenient when working with image datasets, because such vectors live in the same space as the original images, and as such we can plot them to see
visually what we have extracted.

The features that we get here are not very good! Try to play around with the network architecture and parameters to extract more significant traits.

<p>
  <a href="/files/autoencoder.ipynb"> download notebook</a>
</p>

<iframe width="100%" height="500px" frameborder="0" src="http://nbviewer.ipython.org/url/rnduja.github.io/files/autoencoder.ipynb"></iframe>