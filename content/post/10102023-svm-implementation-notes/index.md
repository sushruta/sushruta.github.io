

## Introduction

We will assume the following -

Number of Training Examples - $ne$
Number of Features for each example - $nf$
Number of Classes for classification - $nc$

## Loss Function

For every example, $\mathbf{x_{i}}$, we define its loss as -

$$
margin_{i} = \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta)
$$

Overall, margin is a scalar and we get the total margin across all examples by summing the margins. Therefore, total margin is -

$$
marginLoss = \sum_{i} \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta)
$$

In addition, we use regularization to avoid overfitting. Another nice thing about regularization is that it avoids too many variables (which would make the model complicated)

$$
regLoss = \lambda \begin{vmatrix}\mathbf{W}\end{vmatrix}^2
$$

Overall, total loss for the model is the sum of margin loss and regularization loss as given by -

$$
L = L_{m} + L_{r}
$$

## Gradient of the Loss Function

### Contribution of Regularization Loss

$$
\frac{dL_{r}}{dW} = 2 \lambda W
$$

### Contribution of marginLoss

$marginLoss$ is a scalar. It is defined in terms of $w_{j}$ and $w_{y_{i}}$. Both of these are vectors which means that the derivative of $marginLoss$ will be a vector. The derivative of marginLoss ($mL$) would be as follows -

$$
\nabla mL_{i} = \nabla_{w_{j}} mL_{i} + \nabla_{w_{y_{i}}} mL_{i}
$$

For the next section, we will be calling $mL_{i}$ as $L_{i}$ to write out things easier. Expanding out $\nabla L_{i}$, we get

$$
\nabla L_{i} = \nabla_{w_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta) + \nabla_{w_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$

#### Some Observations Before we Simplify Further

Before we go further, let us simplify a few small details and glue it back in the above expression for the final derivative

We see that the $l_{i}$ is being summed for all $j \neq y_{i}$. When it is infact equal to $y_{i}$, they both cancel out each other leaving $\Delta$. More concretely -

$$
L_{i} = \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \sum_{j} max(0, f_{j} - f_{y_{i}} + \Delta) - (f_{y_{i}} - f_{y_{i}} + \Delta) = \sum_{j} max(0, f_{j} - f_{y_{i}} + \Delta) - \Delta
$$

To further show the derivation, we briefly expand out all of $L_{i}$. If we have $nc$ as number of classes, we the loss defined as -

$$
L_{i} = max(0, f_{1} - f_{y_{1}} + \Delta) + max(0, f_{2} - f_{y_{2}} + \Delta) + ... + max(0, f_{nc} - f_{y_{nc}} + \Delta) - \Delta
$$

Further more, the derivative of $f$ w.r.t $w$ can be written like this -

$$
\nabla_{w_{j}}f_{j} = \nabla_{w_{j}}(w_{j}^Tx_{i}) = x_{i}
$$

Also, with the max in the picture, we should note that

$$
\nabla_{w}max(0, f_{j} - f_{y_{i}} + \Delta) = \begin{cases}
\nabla_{w}(f_{j} - f_{y_{i}} + \Delta) & f_{j} - f_{y_{i}} + \Delta \ge 0 \\
0 & f_{j} - f_{y_{i}} + \Delta \le 0
\end{cases}
$$

We can simplify the above by using an indicator variable -

$$
\nabla_{w} L_{i} = \nabla_{w}\sum_{j}max(0, f_{j} - f_{y_{i}} + \Delta)
$$

$$
\nabla_{w} L_{i} = \sum_{j}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w}(f_{j} - f_{y_{i}} + \Delta)
$$

$$
\nabla_{w} L_{i} = \sum_{j}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{j}}(f_{j} - f_{y_{i}} + \Delta) + \sum_{j}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{y_{i}}}(f_{j} - f_{y_{i}} + \Delta)
$$

$$
\nabla_{w} L_{i} = \sum_{j}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{j}}f_{j} + \sum_{j}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{y_{i}}}(-f_{y_{i}})
$$

For $\nabla_{w_{j}} L_{i}$, we have -

$$
\nabla_{w_{j}} L_{i} = \mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0)\nabla_{w_{j}}(f_{j}) = \mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) x_{i}
$$

and for $\nabla_{w_{y_{i}}} L_{i}$, we have

$$
\nabla_{w_{y_{i}}} L_{i} = \sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{y_{i}}}(-f_{y_{i}}) = \sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) (-x_{i}) = -\sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0)x_{i}
$$

Overall, the derivate of margin loss across all $w_{j}$ can be written as a vector. Please note that Loss is a scalar but when differentiated w.r.t to a vector, the derivative will be a vector.

$$
\nabla_{w} L = \begin{pmatrix} \nabla_{w_{1}}L & \nabla_{w_{2}}L & ... & \nabla_{w_{nc}}L \end{pmatrix}
$$

This can be understood as the following -

Every column in the above vector is a sum of two derivatives. Let us take the $j^{th}$ column. We will calculate $\nabla_{w_{j}}L$ by summing $\nabla_{w_{j}}L_{i}$ across all examples $x_{i}$. This will be the entry for the entire $j_{th}$ column.

Now, we will also add the contribution due to $\nabla_{w_{y_{i}}}L_{i}$. For this, take every example, $x_{i}$ and ask what is $y_{i}$, i.e., we ask what class does it belong to? It will be one of the classes from ${1, 2, ..., nc}$. So, $y_{i}$ becomes a number from ${1, 2, ..., nc}$. Let us say, it is $k$. Then we now have $\nabla_{w_{k}}L_{i}$. We go ahead and add the contribution to the $k^{th}$ column.

Once, we have done this, we have the derivative of the margin loss. Through some dimensional analysis, we see that derivative of regularization loss is of the same dimension as $\mathbf{W}$. Clearly, margin loss should also be of the same dimension. We can deduce that the derivative of the overall margin loss will also be the same as that, i.e., the dimension of $\mathbf{W}$.


## Implementation in NumPy (with loops)

Assume the training examples are in $\mathbf{X}$ and this is of size `ne X nf`.

Assume the training labels are in $y$ and this is of size `ne X 1`.

Assume the trained model has weights in $\mathbf{W}$ and this is of size `nf X nc`. Technically this should be `(nf+1) X nc` but that is a detail we can skip here in this discussion.

## Vectorized Implementation in Numpy

```
W = np.random.randn(nf, nc)
dW = np.zeros(nf, nc)

regularization_loss = 2 * lambda * W

y_preds = X @ W
y_preds_true = y_preds[range(ne), y].reshape((ne, 1))
margins = y_preds - y_preds_true + delta
margins = np.maximum(0, margins)

loss = margins.sum(axis=0) - delta
dloss = np.zeros((ne, nc))
dloss[margins > 0] = 1.0
dloss[range(ne), y] -= dloss.sum(axis=1)
dloss /= N

dW = X.T @ dloss
dW += reg_loss
```
