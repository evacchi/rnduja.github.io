---
layout:     post
title:      "Deep Learning with Torch: 2"
subtitle:   "Understand torch container class"
date:       2015-10-04 12:00:00
author:     "Fabio Fumarola"
header-img: "img/post.jpg"
comments: true
tags:       [torch,neural-networks]
---

##Abstract
In this post we analyze how to build complex neural networks using the container classes.

-------------------------------

[Container](https://github.com/torch/nn/blob/master/doc/containers.md#nn.Containers), similarly to [Module](https://github.com/torch/nn/blob/master/doc/module.md#nn.Module), is the abstract class defining the base methods inherited from concrete containers.

The main functions of `Container` are:

1. add(module): add a Module to the given container
2. get(index): get the module at index
3. size(): the size of the container

## Implemented Containers

### Sequential

Sequential provides a way to plug layers together in a feed-forward fully connected manner.

~~~lua

model = nn.Sequential()
model:add( nn.Linear(10, 25) ) -- 10 input, 25 hidden units
model:add( nn.Tanh() ) -- some hyperbolic tangent transfer function
model:add( nn.Linear(25, 1) ) -- 1 output

print(model:forward(torch.randn(10)))

~~~

which gives as output

~~~lua
-0.1815
[torch.Tensor of dimension 1]
~~~

Moreover this container offers a method to `insert` a module at `index` and `remove` a module at `index`.

### Parallel

It creates a container that allows to train in parallel different layers. For example we can define a model composed of two parallel layers with the same input size. Their output is concatenated together.

~~~lua
model = nn.Parallel(2,1)
model:add(nn.Linear(10,3))
model:add(nn.Linear(10,2))
print(model:forward(torch.randn(10,2)))
~~~

gives as output a Tensor `5x1`

### Concat

Concat concatenates the output of one layer of "parallel" modules along the provided dimension dim: they take the same inputs, and their output is concatenated.

~~~
model=nn.Concat(1);
model:add(nn.Linear(5,3))
model:add(nn.Linear(5,7))
print(model:forward(torch.randn(5)))
~~~

## Conclusion

With Module, Layers and Containers we have the basis to build Deep Neural Networks.
