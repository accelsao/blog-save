---
title: Recurrent neural network
tags:
  - CS231n
  - Neural Networks
categories:
  - Computer Vision
  - Natural Language Processing
date: 2018-07-07 09:15:01
---

`RNN`和`CNN`一樣可以分類圖片
簡單描述一下RNN是甚麼

<!--more-->

# RNN簡介

其實RNN裡面是`Sequence Classification`

![](https://i.imgur.com/aaYUuHK.jpg)

**例子**
* one to one: Vanilla Neural Networks
* one to many: Image Captioning (image -> sequence of words)
* many to one: Sentiment Classification (sequence of words -> sentiment)
* many to many(left):  Machine Translation (sseq of words -> seq of words)
* many to many(right):  Video classification on frame level

---

RNN 有loops 讓訊息可以保存
![](https://i.pinimg.com/564x/68/46/01/684601aa63886d86a1b4dafcf8ab079c.jpg)

---
對於上面的圖可以這樣想 把他展開
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-unrolled.png)

---
目前RNN成功應用在:
* Speech recognition
* Language modeling
* Machine Translation
* Image Captioning


![](https://i.imgur.com/rXTDGt8.png)

---
![](https://i.imgur.com/l4900y7.jpg)
![](https://i.imgur.com/YRFnVLf.jpg)
![](https://i.imgur.com/GTT3Su9.jpg)

# Vallina RNN Gradient Flow

傳統RNN在Backprop問出現 Gradient explode/vanish

![](https://i.imgur.com/YTzKJlS.jpg)

![](https://pic2.zhimg.com/80/v2-0634248def9b3dcfb4c6afa9c4d302b4_hd.jpg)

如上圖 在Backprop時 會乘4次W(和重複的tanh)直接爆炸
最大的singular value>1:Exploding Gradient
最大的singular value<1:Vanishing Gradient


對於Exploding Gradient的解決方法:
**Gradient Clipping**
簡單說就是rescale gradient
![](https://i.imgur.com/aBkb0Rk.jpg)

# LSTM

對於Vanishing Gradient的解決方法就是LSTM(long short time memory)了

![](https://pic3.zhimg.com/80/v2-49d5686a12e59bc8e5ba9d95593d3f2c_hd.jpg)

細看與RNN的差別
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-SimpleRNN.png)
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-chain.png)
Notation:
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM2-notation.png)

## Core Idea

關鍵點是`cell state` 
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-C-line.png)
就像是傳輸帶一樣 只有一些`linear interactions`

除了 `hidden state`外 還增加了 Gates

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-gate.png)

* i：Input gate  決定是否寫入cell (sigmoid)
* f: Forget gate 決定是否清除cell (sigmoid)
* o: Output gate 決定反映多少訊息 (sigmoid)
* g：Gate gate (沒特定名稱) 決定寫入多少訊息 (tanh)


## Step-by-Step Walk Through

LSTM的第一步是要決定我們要從`cell state`丟棄什麼訊息
這是由sigmoid layer`Forget gate`來決定

![](https://img-blog.csdn.net/20151116194536194)
輸入: \\(h_{t-1}\\)和\\(x_t\\)
輸出: \\(f_t\\) 範圍[0,1]到 \\(C_{t-1}\\) 其中1代表**完全保留**,0代表**完全丟棄**

下一步是決定哪些新的訊息要保留到`cell state`
分成兩部分
1. sigmoid layer `Input gate` 決定需要更新的值
2. tanh layer `Gate gate` 創一個新的vector \\(\widetilde{C_t}\\) 會被加到狀態裡
3. 把這兩個東西組合起來 產生新的狀態
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-i.png)

接下來把剛剛那兩個東西合併
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-C.png)

最後要決定output甚麼東西 根據 `cell state` 不過是過濾後的版本
先經過 sigmoid layer 決定那些部分要輸出(過濾)
再經過 tanh layer 讓值在[-1,1]
然後乘上 sigmoid layer `Output gate`
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-o.png)

CS231n的圖
![](https://i.pinimg.com/564x/6e/e1/4d/6ee14da77f1e9824e0427482b0191ef9.jpg)


## Variants on Long Short Term Memory

一種很流行的變形,它增加了`peephole connections`也就是讓 `Gates`也接收`cell state`為輸入

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-peepholes.png)

另外一種是用多個forget and input gates
和上面分別考慮甚麼要forget甚麼要input不同
這邊一起考慮
要輸入一些東西了才丟棄
舊的東西丟棄後 才輸入一些新的東西
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-tied.png)


另個更大變化的是 Gated Recurrent Unit(GRU)
把forget+input 合併成 `update gate`
然後 `cell state`換成`hidden state`
和一些小改變
相對標準LSTM更加簡化
![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-GRU.png)

# Conclusion
1. RNN適用於 Sequence data
2. RNN 梯度爆炸->Gradient Clipping 梯度消失->LSTM
3. LSTM下一步是`attention` ,讓他從一堆訊息裡面去**挑選**部分


# Code
```py
def rnn_step_forward(x, prev_h, Wx, Wh, b):
    """
    Run the forward pass for a single timestep of a vanilla RNN that uses a tanh
    activation function.

    The input data has dimension D, the hidden state has dimension H, and we use
    a minibatch size of N.Wh_

    Inputs:
    - x: Input data for this timestep, of shape (N, D).
    - prev_h: Hidden state from previous timestep, of shape (N, H)
    - Wx: Weight matrix for input-to-hidden connections, of shape (D, H)
    - Wh: Weight matrix for hidden-to-hidden connections, of shape (H, H)
    - b: Biases of shape (H,)

    Returns a tuple of:
    - next_h: Next hidden state, of shape (N, H)
    - cache: Tuple of values needed for the backward pass.
    """
    next_h, cache = None, None
    ##############################################################################
    # TODO: Implement a single forward step for the vanilla RNN. Store the next  #
    # hidden state and any values you need for the backward pass in the next_h   #
    # and cache variables respectively.                                          #
    ##############################################################################
    h_raw=np.dot(x,Wx)+np.dot(prev_h,Wh)+b
    next_h=np.tanh(h_raw)
    cache=(x,prev_h,Wx,Wh,h_raw,next_h)
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return next_h, cache


def rnn_step_backward(dnext_h, cache):
    """
    Backward pass for a single timestep of a vanilla RNN.

    Inputs:
    - dnext_h: Gradient of loss with respect to next hidden state, of shape (N, H)
    - cache: Cache object from the forward pass

    Returns a tuple of:
    - dx: Gradients of input data, of shape (N, D)
    - dprev_h: Gradients of previous hidden state, of shape (N, H)
    - dWx: Gradients of input-to-hidden weights, of shape (D, H)
    - dWh: Gradients of hidden-to-hidden weights, of shape (H, H)
    - db: Gradients of bias vector, of shape (H,)
    """
    dx, dprev_h, dWx, dWh, db = None, None, None, None, None
    ##############################################################################
    # TODO: Implement the backward pass for a single step of a vanilla RNN.      #
    #                                                                            #
    # HINT: For the tanh function, you can compute the local derivative in terms #
    # of the output value from tanh.                                             #
    ##############################################################################
    x, prev_h, Wx, Wh, h_raw, next_h = cache

    # d(tanh) = 1 - tanh * tanh
    dh_raw = (1 - next_h * next_h) * dnext_h # (N,H)
    dx = dh_raw.dot(Wx.T)
    dWh = prev_h.T.dot(dh_raw)
    dprev_h = dh_raw.dot(Wh.T)
    db = np.sum(dh_raw, axis=0)
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return dx, dprev_h, dWx, dWh, db


def rnn_forward(x, h0, Wx, Wh, b):
    """
    Run a vanilla RNN forward on an entire sequence of data. We assume an input
    sequence composed of T vectors, each of dimension D. The RNN uses a hidden
    size of H, and we work over a minibatch containing N sequences. After running
    the RNN forward, we return the hidden states for all timesteps.

    Inputs:
    - x: Input data for the entire timeseries, of shape (N, T, D).
    - h0: Initial hidden state, of shape (N, H)
    - Wx: Weight matrix for input-to-hidden connections, of shape (D, H)
    - Wh: Weight matrix for hidden-to-hidden connections, of shape (H, H)
    - b: Biases of shape (H,)

    Returns a tuple of:
    - h: Hidden states for the entire timeseries, of shape (N, T, H).
    - cache: Values needed in the backward pass
    """
    h, cache = None, None
    ##############################################################################
    # TODO: Implement forward pass for a vanilla RNN running on a sequence of    #
    # input data. You should use the rnn_step_forward function that you defined  #
    # above. You can use a for loop to help compute the forward pass.            #
    ##############################################################################
    N, T, D = x.shape
    N, H = h0.shape
    
    h = np.zeros((N,T,H))
    prev_h=h0
    cache={}
    for i in range (T):
        prev_h,cache_i=rnn_step_forward(x[:, i, :], prev_h, Wx, Wh, b)
        h[:, i, :] = prev_h
        cache[i] = cache_i
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return h, cache


def rnn_backward(dh, cache):
    """
    Compute the backward pass for a vanilla RNN over an entire sequence of data.

    Inputs:
    - dh: Upstream gradients of all hidden states, of shape (N, T, H). 
    
    NOTE: 'dh' contains the upstream gradients produced by the 
    individual loss functions at each timestep, *not* the gradients
    being passed between timesteps (which you'll have to compute yourself
    by calling rnn_step_backward in a loop).

    Returns a tuple of:
    - dx: Gradient of inputs, of shape (N, T, D)
    - dh0: Gradient of initial hidden state, of shape (N, H)
    - dWx: Gradient of input-to-hidden weights, of shape (D, H)
    - dWh: Gradient of hidden-to-hidden weights, of shape (H, H)
    - db: Gradient of biases, of shape (H,)
    """
    dx, dh0, dWx, dWh, db = None, None, None, None, None
    ##############################################################################
    # TODO: Implement the backward pass for a vanilla RNN running an entire      #
    # sequence of data. You should use the rnn_step_backward function that you   #
    # defined above. You can use a for loop to help compute the backward pass.   #
    ##############################################################################
    N, T, H = dh.shape
    x = cache[0][0]
    N, D = x.shape
    
    dx = np.zeros((N, T, D))
    dh0 = np.zeros((N, H))
    dWx = np.zeros((D, H))
    dWh = np.zeros((H, H))
    db = np.zeros(H)
    
    dprev_h = np.zeros((N, H))
    
    for i in reversed(range(T)):
        dnext_h=dh[:,i,:]+dprev_h
        dx[:, i, :], dprev_h, dWx_tmp, dWh_tmp, db_tmp = rnn_step_backward(dnext_h, cache[i])
        dWx+=dWx_tmp
        dWh += dWh_tmp
        db += db_tmp
    dh0=dprev_h
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return dx, dh0, dWx, dWh, db


def word_embedding_forward(x, W):
    """
    Forward pass for word embeddings. We operate on minibatches of size N where
    each sequence has length T. We assume a vocabulary of V words, assigning each
    word to a vector of dimension D.

    Inputs:
    - x: Integer array of shape (N, T) giving indices of words. Each element idx
      of x muxt be in the range 0 <= idx < V.
    - W: Weight matrix of shape (V, D) giving word vectors for all words.

    Returns a tuple of:
    - out: Array of shape (N, T, D) giving word vectors for all input words.
    - cache: Values needed for the backward pass
    """
    out, cache = None, None
    ##############################################################################
    # TODO: Implement the forward pass for word embeddings.                      #
    #                                                                            #
    # HINT: This can be done in one line using NumPy's array indexing.           #
    ##############################################################################
    # 直接從W裡面拿 
    out = W[x, :]
    cache = x, W
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return out, cache


def word_embedding_backward(dout, cache):
    """
    Backward pass for word embeddings. We cannot back-propagate into the words
    since they are integers, so we only return gradient for the word embedding
    matrix.

    HINT: Look up the function np.add.at

    Inputs:
    - dout: Upstream gradients of shape (N, T, D)
    - cache: Values from the forward pass

    Returns:
    - dW: Gradient of word embedding matrix, of shape (V, D).
    """
    dW = None
    ##############################################################################
    # TODO: Implement the backward pass for word embeddings.                     #
    #                                                                            #
    # Note that words can appear more than once in a sequence.                   #
    # HINT: Look up the function np.add.at                                       #
    ##############################################################################
    x , W = cache
    dW = np.zeros_like(W) # (V,D)
    
    np.add.at(dW, x, dout)
    
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################
    return dW


def sigmoid(x):
    """
    A numerically stable version of the logistic sigmoid function.
    """
    pos_mask = (x >= 0)
    neg_mask = (x < 0)
    z = np.zeros_like(x)
    z[pos_mask] = np.exp(-x[pos_mask])
    z[neg_mask] = np.exp(x[neg_mask])
    top = np.ones_like(x)
    top[neg_mask] = z[neg_mask]
    return top / (1 + z)


def lstm_step_forward(x, prev_h, prev_c, Wx, Wh, b):
    """
    Forward pass for a single timestep of an LSTM.

    The input data has dimension D, the hidden state has dimension H, and we use
    a minibatch size of N.

    Note that a sigmoid() function has already been provided for you in this file.

    Inputs:
    - x: Input data, of shape (N, D)
    - prev_h: Previous hidden state, of shape (N, H)
    - prev_c: previous cell state, of shape (N, H)
    - Wx: Input-to-hidden weights, of shape (D, 4H)
    - Wh: Hidden-to-hidden weights, of shape (H, 4H)
    - b: Biases, of shape (4H,)

    Returns a tuple of:
    - next_h: Next hidden state, of shape (N, H)
    - next_c: Next cell state, of shape (N, H)
    - cache: Tuple of values needed for backward pass.
    """
    next_h, next_c, cache = None, None, None
    #############################################################################
    # TODO: Implement the forward pass for a single timestep of an LSTM.        #
    # You may want to use the numerically stable sigmoid implementation above.  #
    #############################################################################
    N, H = prev_h.shape
    gate = np.dot(x,Wx) + np.dot(prev_h,Wh) + b # (N,4H)
    
    # 這邊我怎麼看都不懂ifog的順序在哪...反正只要對到那一層就好了
    i = sigmoid(gate[:,:H])
    f = sigmoid(gate[:,H:2*H])
    o = sigmoid(gate[:,2*H:3*H])
    g = np.tanh(gate[:,3*H:])
    
    next_c = f * prev_c + i * g
    next_h = o * np.tanh(next_c)
    
    cache = (x, prev_h, prev_c, Wx, Wh, gate, i, f, o, g, next_c, next_h)
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################

    return next_h, next_c, cache


def lstm_step_backward(dnext_h, dnext_c, cache):
    """
    Backward pass for a single timestep of an LSTM.

    Inputs:
    - dnext_h: Gradients of next hidden state, of shape (N, H)
    - dnext_c: Gradients of next cell state, of shape (N, H)
    - cache: Values from the forward pass

    Returns a tuple of:
    - dx: Gradient of input data, of shape (N, D)
    - dprev_h: Gradient of previous hidden state, of shape (N, H)
    - dprev_c: Gradient of previous cell state, of shape (N, H)
    - dWx: Gradient of input-to-hidden weights, of shape (D, 4H)
    - dWh: Gradient of hidden-to-hidden weights, of shape (H, 4H)
    - db: Gradient of biases, of shape (4H,)
    """
    dx, dprev_h, dprev_c, dWx, dWh, db = None, None, None, None, None, None
    #############################################################################
    # TODO: Implement the backward pass for a single timestep of an LSTM.       #
    #                                                                           #
    # HINT: For sigmoid and tanh you can compute local derivatives in terms of  #
    # the output value from the nonlinearity.                                   #
    #############################################################################
    x, prev_h, prev_c, Wx, Wh, gate, i, f, o, g, next_c, next_h = cache
    
    N, H = dnext_h.shape
    dgate = np.zeros((N, 4*H))
    
    # np.tanh(next_c).shape = (N,H) = o.shape
    # next_h = o * tanh(next_c)
    do = np.tanh(next_c) * dnext_h
    # dc = o * dnext_h * dtanh(c)
    # dc = 1 - tanh(c)**2 + dc + o * dnext_h
    dnext_c = 1 - np.tanh(next_c)**2 + dnext_c + o * dnext_h
    
    # next_c = f * prev_c + i * g
    # d(f*prev_c) = dnext_c ->backprop
    df = prev_c * dnext_c
    dprev_c = f * dnext_c
    # i * g 同理
    di = g * dnext_c
    dg = i * dnext_c
    
    # g = tanh(gate[:, 3H:])
    dgate[:, 3H:] = dg * (1 - np.tanh(gate[:, 3*H:]) ** 2)
    
    # o = sigmoid(gate[:,2*H:3*H])
    dgate[:,2*H:3*H] = do * (1 - gate[:, 3*H:]) * gate[:, 3*H:]
    # i ,f 同理
    dgate[:,H:2*H] = dF * (1 - gate[:,H:2*H]) * gate[:,H:2*H]
    dgate[:,:H] = di * (1 - gate[:,:H]) * gate[:,:H]
    
    
    # gate = xWx + prev_hWh + b
    # gate=(N,4H) Wx=(D,4H) Wh=(H,4H) b=(4H,) prev_h=(N,H) x=(N,D)
    dx = np.dot(dgate , dWx.T)
    dWx = np.dot(x , dgate) # (D,N X N,4H -> D,4H )
    dprev_h = np.dot(dgate , dWh.T)
    dWh = np.dot(dprev_h.T,dgate) # H,4H
    db = np.sum(dgate,axis=0)
    
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################

    return dx, dprev_h, dprev_c, dWx, dWh, db


def lstm_forward(x, h0, Wx, Wh, b):
    """
    Forward pass for an LSTM over an entire sequence of data. We assume an input
    sequence composed of T vectors, each of dimension D. The LSTM uses a hidden
    size of H, and we work over a minibatch containing N sequences. After running
    the LSTM forward, we return the hidden states for all timesteps.

    Note that the initial cell state is passed as input, but the initial cell
    state is set to zero. Also note that the cell state is not returned; it is
    an internal variable to the LSTM and is not accessed from outside.

    Inputs:
    - x: Input data of shape (N, T, D)
    - h0: Initial hidden state of shape (N, H)
    - Wx: Weights for input-to-hidden connections, of shape (D, 4H)
    - Wh: Weights for hidden-to-hidden connections, of shape (H, 4H)
    - b: Biases of shape (4H,)

    Returns a tuple of:
    - h: Hidden states for all timesteps of all sequences, of shape (N, T, H)
    - cache: Values needed for the backward pass.
    """
    h, cache = None, None
    #############################################################################
    # TODO: Implement the forward pass for an LSTM over an entire timeseries.   #
    # You should use the lstm_step_forward function that you just defined.      #
    #############################################################################
    N, T, D = x.shape
    N, H = h0.shape
    h = np.zeros((N, T, H))
    prev_h=h0
    
    cache={}
    next_c = np.zeros((N, H))
    for i in range(T):
        prev_h, next_c, cache_i = lstm_step_forward(x[:, i, :], prev_h, next_c, Wx, Wh, b)
        h[:,i,:]=prev_h
        cache[i] = cache_i
        
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################

    return h, cache


def lstm_backward(dh, cache):
    """
    Backward pass for an LSTM over an entire sequence of data.]

    Inputs:
    - dh: Upstream gradients of hidden states, of shape (N, T, H)
    - cache: Values from the forward pass

    Returns a tuple of:
    - dx: Gradient of input data of shape (N, T, D)
    - dh0: Gradient of initial hidden state of shape (N, H)
    - dWx: Gradient of input-to-hidden weight matrix of shape (D, 4H)
    - dWh: Gradient of hidden-to-hidden weight matrix of shape (H, 4H)
    - db: Gradient of biases, of shape (4H,)
    """
    dx, dh0, dWx, dWh, db = None, None, None, None, None
    #############################################################################
    # TODO: Implement the backward pass for an LSTM over an entire timeseries.  #
    # You should use the lstm_step_backward function that you just defined.     #
    #############################################################################
    N, T, H = dh.shape
    x = cache[0][0]
    N, D = x.shape

    dx = np.zeros((N, T, D))
    dh0 = np.zeros((N, H))
    dWx = np.zeros((D, 4*H))
    dWh = np.zeros((H, 4*H))
    db = np.zeros(4*H)
    dprev_h = np.zeros((N, H))
    dprev_c = np.zeros((N, H))
    
    for i in range(T):
        dnext_h = dh[:, i, :] + dprev_h
        dnext_c = dprev_c
        dx[:, i, :], dprev_h, dprev_c, dWx_tmp, dWh_tmp, db_tmp = lstm_step_backward(dnext_h, dnext_c, cache[i])
        dWx += dWx_tmp
        dWh += dWh_tmp
        db += db_tmp

    dh0 = dprev_h
    ##############################################################################
    #                               END OF YOUR CODE                             #
    ##############################################################################

    return dx, dh0, dWx, dWh, db
```

# 參考
[Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)