---
layout:     post
title:      "A complete Torch example"
subtitle:   "Recognizing digits from MNIST"
date:       2015-10-13 12:00:00
author:     "Andrea Ferretti"
header-img: "img/2015-10-13-torch-mnist/cover.jpg"
comments: true
---

If you have followed [Fabio's posts](/2015/10/01/deep_learning_with_torch) up until now, you should be familiar with the basic blocks of a Torch application.

In this [iTorch](https://github.com/facebook/iTorch) notebook, we will see a simple but complete example of neural network for the classification problem of recognizing handwritten digits. We use the standard [MNIST](http://yann.lecun.com/exdb/mnist/) dataset, which consists of 60000 grayscale images of size 28x28. We will build a recognizer as a neural network with an input of 28x28=764 neurons and an output of 10 numbers representing the (log-)probabilities that we assign to the 10 digits.

The model, as implemented here, is very shallow and only has 30 neurons in the (only) hidden layer; hence the precision achieved is very low. Try experimenting by changing the size and architecture of the network, the training parameters and the initialization of the layers to get a feeling of how the training time changes, and how the precision is affected.

<p>
  <a href="/files/digit-classification.ipynb"> download notebook</a>
</p>

<iframe width="100%" height="500px" frameborder="0" src="http://nbviewer.ipython.org/url/rnduja.github.io/files/digit-classification.ipynb"></iframe>