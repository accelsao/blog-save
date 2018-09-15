---
title: Gradient descent optimization algorithms
tags:
  - CS231n
  - Neural Networks
categories:
  - Computer Vision
date: 2018-07-04 19:40:24
---

簡單紀錄一些 梯度下降優化
<!--more-->

### Simple update
The simplest form of update
Assuming a vector of parameters `x` and the gradient `dx`
```py
x += - learning_rate * dx
```
where `learning_rate` is a hyperparameter (超參數)


### Momentum update

Better converge rates


```py
v = mu * v - learning_rate * dx # integrate velocity
x += v # integrate position
```
`v`初始化0
超參數`mu`一般被當成`動量`(常設為0.9) 物理意義=摩擦係數
抑制速度,降低動能
一般典型設置
[0.5,0.9,0.95,0.99] 漸漸增加
>通過動量的變化 參數會在梯度上增加速度


### Nesterov Momentum
核心想法如下圖
<hr>
![](http://cs231n.github.io/assets/nn3/nesterov.jpeg)

`x`會通過`mu * v`來改變位置
那麼直接計算`x + mu * v`的梯度而不是`x`的
```py
x_ahead = x + mu * v
# 算 x_ahead 的梯度 dx_ahead
v = mu * v - learning_rate * dx_ahead
x +=v
```

```py
v_prev = v # Save v
v = mu * v - learning_rate * dx
```

另外一種
```
v_prev = v # back this up
v = mu * v - learning_rate * dx # velocity update stays the same
x += -mu * v_prev + (1 + mu) * v # position update changes form
```
來源公式推導
* [Advances in optimizing Recurrent Networks by Yoshua Bengio, Section 3.5](https://arxiv.org/pdf/1212.0901v2.pdf)
* [Ilya Sutskever’s thesis (pdf) contains a longer exposition of the topic in section 7.2](http://www.cs.utoronto.ca/~ilya/pubs/ilya_sutskever_phd_thesis.pdf)

#### Per-parameter adaptive learning rate methods

### Adagrad
adaptive learning rate method
```
# Assume the gradient dx and parameter vector x
cache += dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
```
`cache` size 和 梯度矩陣一樣
跟蹤每個參數的梯度平方和
收到高梯度時的更新效果減弱,
低梯度效果增強
eps 防止除0
缺點:過於激進且過早停止學習

### RMSprop
Adagrad修改版
讓Adagrad不那麼激進,單調降低學習率
用梯度平方的平均
```
cache = decay_rate * cache + (1 - decay_rate) * dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
```
`decay_rate` 超參數 常用[0.9,0.99,0.999]
比起Adagrad更新不會單調降低

### Adam
類似RMSProp 動量版

```
# 簡化版
m = beta1*m + (1-beta1)*dx
v = beta2*v + (1-beta2)*(dx**2)
x += - learning_rate * m / (np.sqrt(v) + eps)
```
整體來說比 RMSprop 優
但也可以使用 SGD+Nesterov
論文建議值
`eps=1e-8` `beta1=0.9` `beta2=0.999`
完整版有`bias` 因為`m` `v`初始0


[An overview of gradient descent optimization algorithms](http://ruder.io/optimizing-gradient-descent/index.html#nesterovacceleratedgradient)
[CS231n课程笔记翻译：神经网络笔记3（下）](https://zhuanlan.zhihu.com/p/21798784?refer=intelligentunit)
[First-order (SGD), momentum, Nesterov momentum](http://cs231n.github.io/neural-networks-3/#sgd)
