

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

$marginLoss$ is a scalar. This means its derivative w.r.t to a vector $w_{j}$ would be a vector too. If we were to look at $\mathbf{W}$, the derivative of $marginLoss$ w.r.t to $\mathbf{W}$ would be a matrix too. It will be a matrix where the $m^{th}$ column of the matrix will hold $\mathbf{w_{m}}$

$$
\frac{d}{dW}mLoss = \begin{pmatrix} \frac{d}{dw_{1}}mLoss & \frac{d}{dW_{2}}mLoss & ... & \frac{d}{dW_ne}mLoss \end{pmatrix}
\frac{d}{dW} \sum_{j \neq y_{i}} max(0, f_{j} - f_{y_{i}} + \Delta) = \frac{d}{dw_{j}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta)) + \frac{d}{dw_{y_{i}}}\sum_{j \neq y_{i}}(max(0, f_{j} - f_{y_{i}} + \Delta))
$$



## Implementation in NumPy (with loops)

## Vectorized Implementation in Numpy
