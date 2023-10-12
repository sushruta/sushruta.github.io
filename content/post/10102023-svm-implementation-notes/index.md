

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
Loss = marginLoss + regLoss
$$

## Gradient of the Loss Function

### Contribution of Regularization Loss

$$
\frac{d}{dW}regLoss = 2 \lambda W
$$

### Contribution of marginLoss

$marginLoss$ is a scalar. It is defined in terms of $w_{j}$ and $w_{y_{i}}$. Both of these are vectors which means that the derivative of $marginLoss$ will be a vector. The derivative of marginLoss ($mL$) would be as follows -

$$
\nabla mL_{i} = \frac{d}{dw_{j}}mL_{i} + \frac{d}{dw_{y_{i}}}mL_{i}
$$

Expanding out $mL_{i}$, we get

$$
\nabla mL_{i} =  \frac{d}{dw_{j}} \sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta)) + \frac{d}{dw_{y_{i}}} \sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$

Before we go further, let us solve a few smaller details and glue it back in the above expression for the final derivative -

$$
\frac{d}{dW}mLoss = \begin{pmatrix} \frac{d}{dw_{1}}mLoss & \frac{d}{dw_{2}}mLoss & ... & \frac{d}{dw_{ne}}mLoss \end{pmatrix} + \begin{pmatrix} \frac{d}{dw_{y_{1}}}mLoss & \frac{d}{dw_{y_{2}}}mLoss & ... & \frac{d}{dw_{y_{ne}}}mLoss \end{pmatrix}
$$
$$
\frac{d}{dW} \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \frac{d}{dw_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta)) + \frac{d}{dw_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$



## Implementation in NumPy (with loops)

## Vectorized Implementation in Numpy
