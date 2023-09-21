---
title: "Pytorch Blitz - 01 - Run a minimal 2-layer network"
description: How to run a minimal 2 level neural network in pytorch
slug: pytorch-blitz-01
date: 2023-09-20T22:29:12-07:00
categories:
  - Machine Learning
  - Deep Learning
  - Pytorch
tags:
  - Machine Learning
  - Deep Learning
  - Pytorch
---

### Pytorch in a hurry - Part 1

Learning about `numpy` while going through [cs231n's first few notes](https://cs231n.github.io/) was exciting. The best part was figuring out the code that would run in a [vectorized manner](https://www.pythonlikeyoumeanit.com/Module3_IntroducingNumpy/VectorizedOperations.html).

In this code walkthrough, we will attempt to write a simple neural network that trains on GPU. The neural network is extremely simple -

(a) no regularization - this is bad because the model will be susceptible to overfitting
(b) no bias vector - this is bad as the layer can no longer capture affine properties
(c) abusing GPU - this is a really small example and can be done on CPU with ease but we want to demonstrate simple GPU use

But this is just a walkthrough and we will probably be okay inspite of all the above downsides.

### Jupyter Notebook and Annotations

The jupyter notebook can be accessed here.

### Credits

The entire structure has been heavily borrowed from Stanford's cs231n course. The code here is specially borrowed from [Justin Johnson](https://github.com/jcjohnson)'s github repository of [pytorch examples](https://github.com/jcjohnson/pytorch-examples). The course and various repositories and videos on youtube have been extremely helpful in paving the way to understanding these concepts!

### Appendix

#### What is vectorized code?

There are simpler examples to motivate vectorized code in numpy but here's an example that I would like to show. It involves computing the gradient of a `ReLU` function. So, how is ReLU function defined?

$$
ReLU(x) = max(0.0, x)
$$

That is it. It is one of the most popular activation functions in Neural Networks. Others being `Leaky ReLU`, `tanh`, `sigmoid`, etc.

Since, this is a neural network, we also have to worry about backpropagation. That means, we would like to compute the derivative of `ReLU(x)`.

$$
\dfrac{dReLU(x)}{dx}=
\begin{cases}
1.0, & \text{ if } x >= 0 \\
0.0, & \text{ if } x < 0 \\
\end{cases}
$$

Remember that `x` is a vector of size `Nx1`. Now, the easy and naive way of executing the deritvative would be -

```
for i in range(N):
  dx_ReLU[i] = 1.0 if x[i] > 0 else 0
```

CPUs and GPUs usually do not like sequential instructions like these. It is like sending `N` instructions one at a time on a 4 lane highway. We are wasting 3 lanes while only using 1 lane. Now, numpy has implemented many of its methods in a optimized way which means they can take advantage of all the lanes in a highway. For that, we need to throw away the for loop and instead write it this way -

```
dx_ReLU = np.ones(size=(N, 1))
dx_ReLU[x < 0] = 0
```
