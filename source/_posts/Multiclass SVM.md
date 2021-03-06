﻿---
title: Multiclass SVM
tags:
  - CS231n
  - Loss function
categories:
  - Computer Vision
date: 2018-06-26 20:07:16
---



# Linear classification: Support Vector Machine

Multiclass Support Vector Machine loss

There are several ways to define the details of the loss function. As a first example we will first develop a commonly used loss called the Multiclass Support Vector Machine (SVM) loss

<!--more-->

## Loss function
The SVM loss is set up so that the SVM “wants” the correct class for each image to a have a score higher than the incorrect classes by some fixed margin Δ

Define: \\(s_j = f(x_i, W)_j\\)

SVM loss: \\(L_i = \sum_{j\neq y_i} \max(0, s_j - s_{y_i} + \Delta)\\)


Ex: s = [13,−7,11]

\\(L_i = \max(0, -7 - 13 + 10) + \max(0, 11 - 13 + 10)\\)

We get zero loss for this pair because the correct class score (13) was greater than the incorrect class score (-7) by at least the margin 10. Though the dif = -7-13 = -20 , but we only care about the dif at least 10.

Note that \\(f(x_i; W) = W x_i\\)
Rewrite it to : \\(L_i = \sum_{j\neq y_i} \max(0, w_j^T x_i - w_{y_i}^T x_i + \Delta)\\)



### Regularization
The bug above is that W is not necessary unique.
Ex: If W correctly classify, then the loss is 0. Then \\(\lambda W\\) where \\(\lambda >1\\) will also give zero loss 

$$R(W) = \sum_k\sum_l W_{k,l}^2$$

$$ L = \underbrace{ \frac{1}{N} \sum_i L_i }_\text{data loss} + \underbrace{ \lambda R(W) }_\text{regularization loss} \\\\ $$

or
$$ L = \frac{1}{N} \sum_i \sum_{j\neq y_i} \left[ \max(0, f(x_i; W)_j - f(x_i; W)_{y_i} + \Delta) \right] + \lambda \sum_k\sum_l W_{k,l}^2 $$

and it improve generalization.

## Code

```py
import numpy as np
from random import shuffle

def svm_loss_naive(W, X, y, reg):
    """
    Structured SVM loss function, naive implementation (with loops).

    Inputs have dimension D, there are C classes, and we operate on minibatches
    of N examples.

    Inputs:
    - W: A numpy array of shape (D, C) containing weights.
    - X: A numpy array of shape (N, D) containing a minibatch of data.
    - y: A numpy array of shape (N,) containing training labels; y[i] = c means
    that X[i] has label c, where 0 <= c < C.
    - reg: (float) regularization strength

    Returns a tuple of:
    - loss as single float
    - gradient with respect to weights W; an array of same shape as W
    """
    dW = np.zeros(W.shape) # initialize the gradient as zero

    # compute the loss and the gradient
    num_classes = W.shape[1]
    num_train = X.shape[0]
    loss = 0.0
    for i in range(num_train):
        scores = X[i].dot(W)
        correct_class_score = scores[y[i]]
        for j in range(num_classes):
            if j == y[i]:
                continue
            margin = scores[j] - correct_class_score + 1 # note delta = 1
            if margin > 0:
                loss += margin

        # Right now the loss is a sum over all training examples, but we want it
        # to be an average instead so we divide by num_train.
        loss /= num_train

        # Add regularization to the loss.
        loss += reg * np.sum(W * W)

    #############################################################################
    # TODO:                                                                     #
    # Compute the gradient of the loss function and store it dW.                #
    # Rather that first computing the loss and then computing the derivative,   #
    # it may be simpler to compute the derivative at the same time that the     #
    # loss is being computed. As a result you may need to modify some of the    #
    # code above to compute the gradient.                                       #
    #############################################################################
    dW=dW/num_train+2*reg*W

    return loss, dW


def svm_loss_vectorized(W, X, y, reg):
    """
    Structured SVM loss function, vectorized implementation.

    Inputs and outputs are the same as svm_loss_naive.
    """
    loss = 0.0
    dW = np.zeros(W.shape) # initialize the gradient as zero

    #############################################################################
    # TODO:                                                                     #
    # Implement a vectorized version of the structured SVM loss, storing the    #
    # result in loss.                                                           #
    #############################################################################
    # s: A numpy array of shape (N, C) containing scores

    
    num_train=X.shape[0]
    scores=np.dot(X,W)

    # scores.T shape= C,N , y shape= N,
    # np choose (N,) = (N,:)
    correct_class_scores = np.choose(y, scores.T) 

    mask = np.ones(scores.shape, dtype=bool)
    mask[range(num_train), y] = False
    scores_ = scores[mask].reshape(num_train, -1)

    margin = scores_ - correct_class_scores[..., np.newaxis] + 1

    # discard margin < 0
    margin[margin < 0] = 0

    # Average our data loss over the size of batch and add reg. term to the loss.
    loss = np.sum(margin) / num_train
    loss += reg * np.sum(W * W)
  
    #############################################################################
    #                             END OF YOUR CODE                              #
    #############################################################################


    #############################################################################
    # TODO:                                                                     #
    # Implement a vectorized version of the gradient for the structured SVM     #
    # loss, storing the result in dW.                                           #
    #                                                                           #
    # Hint: Instead of computing the gradient from scratch, it may be easier    #
    # to reuse some of the intermediate values that you used to compute the     #
    # loss.                                                                     #
    #############################################################################
    original_margin = scores - correct_class_scores[...,np.newaxis] + 1

    # Mask to identiy where the margin is greater than 0 (all we care about for gradient).
    pos_margin_mask = (original_margin > 0).astype(float)

    # Count how many times >0 for each image but dont count correct class hence -1
    sum_margin = pos_margin_mask.sum(1) - 1

    # Make the correct class margin be negative total of how many > 0
    pos_margin_mask[range(pos_margin_mask.shape[0]), y] = -sum_margin

    # Now calculate our gradient.
    dW = np.dot(X.T, pos_margin_mask)

    # Average over batch and add regularisation derivative.
    dW = dW / num_train + 2 * reg * W
	#############################################################################
	#                             END OF YOUR CODE                              #
	#############################################################################

    return loss, dW

```
