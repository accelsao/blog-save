---
title: Group Normalization
tags:
  - CS231n
  - Normalization
categories:
  - Computer Vision
date: 2018-07-06 17:23:19
---

Group Normalization (GN) 是[BN](https://accelsao.github.io/2018/07/05/Batch%20normalization/)的一個簡單的變形
<!--more-->

# 介紹

BN的Normalization,過度依賴batch,過小的batch性能很低
他會在batch的維度normalize 但batch size 又不固定
訓練和測試不一樣

LN就對Channel方向做處理 , 在RNN上有顯著表現 ,CNN就沒有

GN是在Channel方向上分成Group來處理

![](http://localhost:8888/notebooks/assignment2/normalization.png)

簡單比較:
BN：batch方向,算(N*H*W)的mean
LN: channel方向,算(C*H*W)的mean
GN: channel方向分group,算(C//G)*H*W的mean ,C%G==0
LN可以說是GN的情況中`G==C`的特例


# Code

Forward pass

```py
def spatial_groupnorm_forward(x, gamma, beta, G, gn_param):
    """
    Computes the forward pass for spatial group normalization.
    In contrast to layer normalization, group normalization splits each entry 
    in the data into G contiguous pieces, which it then normalizes independently.
    Per feature shifting and scaling are then applied to the data, in a manner identical to that of batch normalization and layer normalization.

    Inputs:
    - x: Input data of shape (N, C, H, W)
    - gamma: Scale parameter, of shape (C,)
    - beta: Shift parameter, of shape (C,)
    - G: Integer mumber of groups to split into, should be a divisor of C
    - gn_param: Dictionary with the following keys:
      - eps: Constant for numeric stability

    Returns a tuple of:
    - out: Output data, of shape (N, C, H, W)
    - cache: Values needed for the backward pass
    """
    out, cache = None, None
    eps = gn_param.get('eps',1e-5)
    ###########################################################################
    # TODO: Implement the forward pass for spatial group normalization.       #
    # This will be extremely similar to the layer norm implementation.        #
    # In particular, think about how you could transform the matrix so that   #
    # the bulk of the code is similar to both train-time batch normalization  #
    # and layer normalization!                                                # 
    ###########################################################################
    N, C, H, W = x.shape
	# 調整size N*G 因為 C/G 
    x = np.reshape(x, (N*G, C//G*H*W))
	# 跟LN一樣 轉成channel方向
    x=x.T
	# 以下和BN一樣
    mu=np.mean(x,axis=0)
    xmu=x-mu
    sq=xmu**2
    var=np.var(x,axis=0)
    sqrtvar=np.sqrt(var+eps)
    ivar=1./sqrtvar
    xhat=xmu*ivar
	# 轉回來
    xhat=np.reshape(xhat.T,(N,C,H,W))
	# out = gamma * xhat + beta
    out = gamma[np.newaxis, :, np.newaxis, np.newaxis] * xhat + beta[np.newaxis, :, np.newaxis, np.newaxis]
    cache = (xhat, gamma, xmu, ivar, sqrtvar, var, eps, G)
    
    ###########################################################################
    #                             END OF YOUR CODE                            #
    ###########################################################################
    return out, cache
```
Backward pass

```py
def spatial_groupnorm_backward(dout, cache):
    """
    Computes the backward pass for spatial group normalization.

    Inputs:
    - dout: Upstream derivatives, of shape (N, C, H, W)
    - cache: Values from the forward pass

    Returns a tuple of:
    - dx: Gradient with respect to inputs, of shape (N, C, H, W)
    - dgamma: Gradient with respect to scale parameter, of shape (C,)
    - dbeta: Gradient with respect to shift parameter, of shape (C,)
    """
    dx, dgamma, dbeta = None, None, None

    ###########################################################################
    # TODO: Implement the backward pass for spatial group normalization.      #
    # This will be extremely similar to the layer norm implementation.        #
    ###########################################################################
    N, C, H, W = dout.shape
    xhat, gamma, xmu, ivar, sqrtvar, var, eps, G = cache
    dxhat = dout * gamma[np.newaxis, :, np.newaxis, np.newaxis]
    # keepdims size維持(1,C,1,1)
    dbeta = np.sum(dout, axis=(0, 2, 3), keepdims=True)
    dgamma = np.sum(dout*xhat, axis=(0, 2, 3), keepdims=True)
    
    # Reshape
    
    dxhat=np.reshape(dxhat, (N*G, C//G*H*W)).T
    xhat = np.reshape(xhat, (N*G, C//G*H*W)).T
    Nprime, Dprime = dxhat.shape
    dx = 1.0/Nprime * ivar * (Nprime*dxhat - np.sum(dxhat, axis=0) - xhat*np.sum(dxhat*xhat, axis=0))

    dx = np.reshape(dx.T, (N, C, H, W))
    ###########################################################################
    #                             END OF YOUR CODE                            #
    ###########################################################################
    return dx, dgamma, dbeta
```
