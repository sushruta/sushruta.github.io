

## Introduction

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

and for $\nabla_{w_{y_{i}}}$, we have

$$
\nabla_{w_{y_{i}}} L_{i} = \sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) \nabla_{w_{y_{i}}}(-f_{y_{i}}) = \sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) (-x_{i}) = -\sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0)x_{i}
$$

------------------
$$
\frac{d}{dW}mLoss = \begin{pmatrix} \frac{d}{dw_{1}}mLoss & \frac{d}{dw_{2}}mLoss & ... & \frac{d}{dw_{ne}}mLoss \end{pmatrix} + \begin{pmatrix} \frac{d}{dw_{y_{1}}}mLoss & \frac{d}{dw_{y_{2}}}mLoss & ... & \frac{d}{dw_{y_{ne}}}mLoss \end{pmatrix}
$$
$$
\frac{d}{dW} \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \frac{d}{dw_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta)) + \frac{d}{dw_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$



## Implementation in NumPy (with loops)

## Vectorized Implementation in Numpy
