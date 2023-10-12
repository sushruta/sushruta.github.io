

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

Expanding out $\nabla mL_{i}$, we get

$$
\nabla_{w_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta) + \nabla_{w_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$

Before we go further, let us solve a few smaller details and glue it back in the above expression for the final derivative -

$$
mL_{i} = \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \sum_{j} max(0, f_{j} - f_{y_{i}} + \Delta) - \Delta
$$

Further more

$$
\nabla_{w_{j}}f_{j} = \nabla_{w_{j}}(w_{j}^Tx_{i}) = x_{i}
$$

Also, with the max in the picture, we should note that

$$
\frac{d}{dx}max(\alpha(x), 0) = \begin{pmatrix} \frac{d\alpha(x)}{dx} if \alpha(x) > 0 & 0 \end{pmatrix}
$$

$$
\nabla L_{i} = \nabla_{w_{j}}L_{i} + \nabla_{w_{y_{i}}}L_{i} = \nabla_{w_{j}}(\sum_{j}max(0, f_{j} - f_{y_{i}} + \Delta) - \Delta) + \nabla_{w_{j}}(\sum_{j \neq y_{i}}max(0, f_{j} - f_{y_{i}} + \Delta))
$$

For further simplification, we should note that $\frac{d}{dx}max(0, \alpha(x))$ is $0$ if $\alpha(x) < 0$ otherwise it will be $\frac{d}{dx}\alpha(x)$

We can use the indicator function to express this as one expression like this - $\frac{d}{dx}max(0, \alpha(x)) = \mathbb{1}(\alpha(x) > 0)\alpha\upquote(x)$

$$
CE(p,y)=\left\{
\begin{array}{ll}
-\log(p) &\text{if }y=1 \\ 
-\log(1-p) &\text{otherwise}.
\end{array} 
\right.
$$

$$
\nabla L_{i} = \nabla_{w_{j}}\sum_{j}max(0, f_{j} - f_{y_{i}} + \Delta) + \nabla_{w_{y_{i}}}\sum_{j}max(0, f_{j} - f_{y_{i}} + \Delta) = \mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0) x_{i} - \sum_{j \neq w_{y_{i}}}\mathbb{1}(max(0, f_{j} - f_{y_{i}} + \Delta) > 0)x_{i}
$$

$$
\frac{d}{dW}mLoss = \begin{pmatrix} \frac{d}{dw_{1}}mLoss & \frac{d}{dw_{2}}mLoss & ... & \frac{d}{dw_{ne}}mLoss \end{pmatrix} + \begin{pmatrix} \frac{d}{dw_{y_{1}}}mLoss & \frac{d}{dw_{y_{2}}}mLoss & ... & \frac{d}{dw_{y_{ne}}}mLoss \end{pmatrix}
$$
$$
\frac{d}{dW} \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \frac{d}{dw_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta)) + \frac{d}{dw_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$



## Implementation in NumPy (with loops)

## Vectorized Implementation in Numpy
