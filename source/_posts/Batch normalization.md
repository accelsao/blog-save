---
title: Batch Normalization
tags:
  - CS231n
  - Normalization
categories:
  - Computer Vision
date: 2018-07-05 15:55:42
---
該算法解決了如何合理初始化神經網路的問題
對於梯度消失/爆炸的問題
[Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/pdf/1502.03167.pdf) 提供了一個解決辦法


<!--more-->

## BN算法簡介

BN的想法是讓網路中**每一層**的數據都歸一化

### 訓練階段
因為隨意的更改每一層的數據可能影響後面的結果 所以在歸一化後 又進行了**還原**操作
* 歸一化
	1. 減去均值
	2. 除標準差
* 還原
	1. 縮放尺寸
	2. 平移


	
引入兩個參數
1. 縮放因子 \\(\gamma\\)
2. 平移距離 \\(\beta\\)


![](http://jermmy.xyz/images/2017-9-2/BN%20algorithm.png)

計算 Backprop

![](http://jermmy.xyz/images/2017-9-2/BN%20gradient.png)

### 預測階段
我們沒辦法透過`batch`計算出\\(\mu_\beta\\)和\\(\sigma_\beta\\)的值

具體作法如下:
\\(E[x] \leftarrow E_\beta [\mu_\beta]  \tag{4}\\)
\\(Var[x] \leftarrow \frac{m}{m-1}E_\beta [{\sigma_\beta} ^2] \tag{5}\\)

\\(\mu_\beta\\) mini-batch的均值
\\(\sigma_\beta\\) mini-batch的標準差

不過在預測階段不需要歸一化加速,但因為訓練時有BN層不能直接去掉
作用用了一點`trick`
去掉歸一化
並更改`還原`步驟
\\(y=\frac{\gamma}{\sqrt{Var[x]+\epsilon}}x+(\beta - \frac{\gamma E[x]}{\sqrt{Var[x]+\epsilon}}) \tag{6}\\)

結果就等同於 `y=x`

* BN算法總體流程
![](http://jermmy.xyz/images/2017-9-2/BN%20algorithm%20all.png)


## Computational Graph of Batch Normalization Layer
![](https://kratzert.github.io/images/bn_backpass/BNcircuit.png)
如一般的圖一樣
黑色:forward
紅色:backforward

### Simple forwardpass
```py
def batchnorm_forward(x, gamma, beta, eps):

  N, D = x.shape

  # Step1: 求均值
  mu = np.mean(x, axis = 0)

  # Step2: 平移
  xmu = x - mu

  # Step3: 計算分母
  sq = xmu ** 2

  # Step4: 標準差
  # var = 1./N * np.sum(sq, axis = 0)
  var = np.var(x, axis=0)

  # Step5: add eps 避免除0
  sqrtvar = np.sqrt(var + eps)

  # Step6: 倒數
  ivar = 1./sqrtvar

  # Step7: 計算標準化
  xhat = xmu * ivar

  # Step8: 縮放
  gammax = gamma * xhat

  # Step9: 平移
  out = gammax + beta

  cache = (xhat,gamma,xmu,ivar,sqrtvar,var,eps)

  return out, cache
```
### Backprop
利用`chainrule`
從最後步驟一步一步往前推

---
#### Step 9:
![](https://kratzert.github.io/images/bn_backpass/step9.png)
---
\\(\frac{\partial f}{\partial x}=\frac{\partial f}{\partial y}=1\\)
#### Step 8:
---
![](https://kratzert.github.io/images/bn_backpass/step8.png)
\\(\frac{\partial f}{\partial x}=y,\frac{\partial f}{\partial y}=x\\)
#### Step 7:
---
![](https://kratzert.github.io/images/bn_backpass/step7.png)
同Step 8
#### Step 6:
---
![](https://kratzert.github.io/images/bn_backpass/step6.png)
\\(f=x^{-1},\partial f=-x^{-2}\\)
#### Step 5:
---
![](https://kratzert.github.io/images/bn_backpass/step5.png)
\\(f=(x+\varepsilon)^{\frac{1}{2}} ,\partial f=\frac{1}{2}*(x+\varepsilon)^{-\frac{1}{2}}\\)
#### Step 4:
---
![](https://kratzert.github.io/images/bn_backpass/step4.png)
\\(f=x,\frac{\partial f}{\partial x}=1\\)
#### Step 3:
---
![](https://kratzert.github.io/images/bn_backpass/step3.png)
\\(f=x^{2},\partial x=2x\\)
#### Step 2:
---
![](https://kratzert.github.io/images/bn_backpass/step2.png)
#### Step 1:
---
![](https://kratzert.github.io/images/bn_backpass/step1.png)
#### Step 0:
---
![](https://kratzert.github.io/images/bn_backpass/step0.png)

最後目標`dx`

```py
def batchnorm_backward(dout, cache):

  #unfold the variables stored in cache
  xhat,gamma,xmu,ivar,sqrtvar,var,eps = cache

  #get the dimensions of the input/output
  N,D = dout.shape

  #step9
  dbeta = np.sum(dout, axis=0)
  dgammax = dout #not necessary, but more understandable

  #step8
  dgamma = np.sum(dgammax*xhat, axis=0)
  dxhat = dgammax * gamma

  #step7
  divar = np.sum(dxhat*xmu, axis=0)
  dxmu1 = dxhat * ivar

  #step6
  dsqrtvar = -1. /(sqrtvar**2) * divar

  #step5
  dvar = 0.5 * 1. /np.sqrt(var+eps) * dsqrtvar

  #step4
  dsq = 1. /N * np.ones((N,D)) * dvar

  #step3
  dxmu2 = 2 * xmu * dsq

  #step2
  dx1 = (dxmu1 + dxmu2)
  dmu = -1 * np.sum(dxmu1+dxmu2, axis=0)

  #step1
  dx2 = 1. /N * np.ones((N,D)) * dmu

  #step0
  dx = dx1 + dx2

  return dx, dgamma, dbeta
```

### Backprop Alternative


再次回顧:
![](https://kevinzakka.github.io/assets/batch_norm/alg1.png)

![](https://kevinzakka.github.io/assets/batch_norm/table0.png)
目標: 找出以下
\\(\dfrac{\partial f}{\partial \gamma}\\) , \\(\dfrac{\partial f}{\partial \beta}\\) , \\(\dfrac{\partial f}{\partial x_i}\\)


---
Cell 1:
![](https://kevinzakka.github.io/assets/batch_norm/table1.png)
\\(\dfrac{\partial f}{\partial y_i}\\) 不用計算 因為就是`dout`了

---
Cell 2:
![](https://kevinzakka.github.io/assets/batch_norm/table2.png)


計算 \\(\gamma\\)
\begin{eqnarray}
\frac{\partial f}{\partial \gamma} &=& \frac{\partial f}{\partial y_i} \cdot \frac{\partial y_i}{\partial \gamma} \qquad \\
&=& \boxed{\sum\limits_{i=1}^m \frac{\partial f}{\partial y_i} \cdot \hat{x}_i}
\end{eqnarray} 

\\(y=\gamma \hat{x}+\beta,\frac{\partial y_i}{\partial \gamma}=\hat{x_i}\\)
從1加到m 是因為我們在處理batches

---
計算 \\(\beta\\)
\begin{eqnarray}
\frac{\partial f}{\partial \beta} &=& \frac{\partial f}{\partial y_i} \cdot \frac{\partial y_i}{\partial \beta} \qquad \\
&=& \boxed{\sum\limits_{i=1}^m \frac{\partial f}{\partial y_i}}
\end{eqnarray}

\\(y=\gamma \hat{x}+\beta,\frac{\partial y_i}{\partial \beta}=1\\)

---
計算 \\(\hat{x_i}\\)
\begin{eqnarray}
\frac{\partial f}{\partial \hat{x}_i} &=& \frac{\partial f}{\partial y_i} \cdot \frac{\partial y_i}{\partial \hat{x}_i} \qquad \\
&=& \boxed{\frac{\partial f}{\partial y_i} \cdot \gamma}
\end{eqnarray}

\\(y=\gamma \hat{x}+\beta,\frac{\partial y_i}{\partial \hat{x_i}}=\gamma\\)

---

Cell 3:
![](https://kevinzakka.github.io/assets/batch_norm/table3.png)

\\(\dfrac{\partial f}{\partial \mu} = \frac{\partial f}{\partial \hat{x}_i} \cdot \color{red}{\frac{\partial \hat{x}_i}{\partial \mu}} + \color{red}{\frac{\partial f}{\partial \sigma^2}} \cdot \color{red}{\frac{\partial \sigma^2}{\partial\mu}}\\)

紅色部分是需要計算的


\\(\hat{x}_i = \frac{(x_i - \mu)}{\sqrt{\sigma^2 + \epsilon}}\\)


\\(\boxed{\dfrac{\partial \hat{x}_i}{\partial \mu} = \frac{1}{\sqrt{\sigma^2 + \epsilon}} \cdot (-1)}\\)


\\(\sigma^2 = \frac{1}{m} \sum\limits_{i=1}^m (x_i - \mu)^2\\)


\\(\boxed{\dfrac{\partial \sigma^2}{\partial \mu} = \frac{1}{m} \sum\limits_{i=1}^m 2 \cdot (x_i - \mu)\cdot (-1)}\\)

再來`chainrule`

\begin{eqnarray}
\frac{\partial f}{\partial \sigma^2} &=& \frac{\partial f}{\partial \hat{x}} \cdot \frac{\partial \hat{x}}{\partial \sigma^2} \qquad \\
\end{eqnarray}

先改寫\\(\hat{x_i}\\)
\\(\hat{x}_i = (x_i - \mu)(\sigma^2 + \epsilon)^{-0.5}\\)


\begin{eqnarray}
\dfrac{\partial \hat{x}}{\partial \sigma^2} &=& \sum\limits_{i=1}^m (x_i - \mu) \cdot (-0.5) \cdot (\sigma^2 + \epsilon)^{-0.5 - 1} \qquad \\
&=& -0.5 \sum\limits_{i=1}^m (x_i - \mu) \cdot (\sigma^2 + \epsilon)^{-1.5}
\end{eqnarray}

最後 全部帶入

\begin{eqnarray}
\frac{\partial f}{\partial \mu} &=& \bigg(\sum\limits_{i=1}^m  \frac{\partial f}{\partial \hat{x}_i} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}} \bigg) + \bigg( \frac{\partial f}{\partial \sigma^2} \cdot \frac{1}{m} \sum\limits_{i=1}^m -2(x_i - \mu)   \bigg) \qquad \\
&=& \bigg(\sum\limits_{i=1}^m  \frac{\partial f}{\partial \hat{x}_i} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}} \bigg) + \bigg( \frac{\partial f}{\partial \sigma^2} \cdot (-2) \cdot \bigg( \frac{1}{m} \sum\limits_{i=1}^m x_i - \frac{1}{m} \sum\limits_{i=1}^m \mu   \bigg) \bigg) \qquad \\
&=& \bigg(\sum\limits_{i=1}^m  \frac{\partial f}{\partial \hat{x}_i} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}} \bigg) + \bigg( \frac{\partial f}{\partial \sigma^2} \cdot (-2) \cdot \bigg( \mu - \frac{m \cdot \mu}{m} \bigg) \bigg) \qquad \\
&=& \sum\limits_{i=1}^m  \frac{\partial f}{\partial \hat{x}_i} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}} \qquad \\
\end{eqnarray}

得到
\\(\boxed{\frac{\partial f}{\partial \mu} = \sum\limits_{i=1}^m  \frac{\partial f}{\partial \hat{x}_i} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}}}\\)



---
\\(\dfrac{\partial f}{\partial x_i} = \frac{\partial f}{\partial \hat{x}_i} \cdot \color{red}{\frac{\partial \hat{x}_i}{\partial x_i}} + \frac{\partial f}{\partial \mu} \cdot \color{red}{\frac{\partial \mu}{\partial x_i}} + \frac{\partial f}{\partial \sigma^2} \cdot \color{red}{\frac{\partial \sigma^2}{\partial x_i}}\\)

同樣求紅色的部分

\\(\dfrac{\partial \hat{x}_i}{\partial x_i} = \dfrac{1}{\sqrt{\sigma^2 + \epsilon}}\\)
\\(\dfrac{\partial \mu}{\partial x_i} = \dfrac{1}{m}\\)
\\(\dfrac{\partial \sigma^2}{\partial x_i} = \dfrac{2(x_i - \mu)}{m}\\)

結果
\\(\frac{\partial f}{\partial x_i} = \bigg(\frac{\partial f}{\partial \hat{x}_i} \cdot \dfrac{1}{\sqrt{\sigma^2 + \epsilon}}\bigg) + \bigg(\frac{\partial f}{\partial \mu} \cdot \dfrac{1}{m}\bigg) + \bigg(\frac{\partial f}{\partial \sigma^2} \cdot \dfrac{2(x_i - \mu)}{m}\bigg)\\)
全部帶入整理

\begin{eqnarray}
\frac{\partial f}{\partial x_i} &=& \bigg(\frac{\partial f}{\partial \hat{x}_i} \cdot \dfrac{1}{\sqrt{\sigma^2 + \epsilon}}\bigg) + \bigg(\frac{\partial f}{\partial \mu} \cdot \dfrac{1}{m}\bigg) + \bigg(\frac{\partial f}{\partial \sigma^2} \cdot \dfrac{2(x_i - \mu)}{m}\bigg) \qquad \\
&=& \bigg(\frac{\partial f}{\partial \hat{x}_i} \cdot \dfrac{1}{\sqrt{\sigma^2 + \epsilon}}\bigg) + \bigg(\frac{1}{m} \sum\limits_{j=1}^m  \frac{\partial f}{\partial \hat{x}_j} \cdot \frac{-1}{\sqrt{\sigma^2 + \epsilon}}\bigg) - \bigg(0.5 \sum\limits_{j=1}^m \frac{\partial f}{\partial \hat{x}_j} \cdot (x_j - \mu) \cdot (\sigma^2 + \epsilon)^{-1.5} \cdot \dfrac{2(x_i - \mu)}{m} \bigg) \qquad \\
&=& \bigg(\frac{\partial f}{\partial \hat{x}_i} \cdot (\sigma^2 + \epsilon)^{-0.5} \bigg) - \bigg(\frac{(\sigma^2 + \epsilon)^{-0.5}}{m} \sum\limits_{j=1}^m  \frac{\partial f}{\partial \hat{x}_j} \bigg) + \bigg(\frac{(\sigma^2 + \epsilon)^{-0.5}}{m} \cdot \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} \sum\limits_{j=1}^m \frac{\partial f}{\partial \hat{x}_j} \cdot \frac{(x_j - \mu)}{\sqrt{\sigma^2 + \epsilon}} \bigg )\qquad \\
&=& \bigg(\frac{\partial f}{\partial \hat{x}_i} \cdot (\sigma^2 + \epsilon)^{-0.5} \bigg) - \bigg(\frac{(\sigma^2 + \epsilon)^{-0.5}}{m} \sum\limits_{j=1}^m  \frac{\partial f}{\partial \hat{x}_j} \bigg) + \bigg(\frac{(\sigma^2 + \epsilon)^{-0.5}}{m} \cdot \hat{x}_i \sum\limits_{j=1}^m \frac{\partial f}{\partial \hat{x}_j} \cdot \hat{x}_j \bigg )\qquad \\
\end{eqnarray}

非常的醜
裡面用到一個tricky的東西(根本黑魔法)

\\(\boxed{(\sigma^2 + \epsilon)^{-1.5} = (\sigma^2 + \epsilon)^{-0.5}(\sigma^2 + \epsilon)^{-1} = (\sigma^2 + \epsilon)^{-0.5} \frac{1}{\sqrt{\sigma^2 + \epsilon}}\frac{1}{\sqrt{\sigma^2 + \epsilon}}}\\)

\\(\boxed{\frac{\partial f}{\partial x_i} = \frac{(\sigma^2 + \epsilon)^{-0.5}}{m} \bigg [\color{red}{m \frac{\partial f}{\partial \hat{x}_i}} - \color{blue}{\sum\limits_{j=1}^m  \frac{\partial f}{\partial \hat{x}_j}} - \color{green}{\hat{x}_i \sum\limits_{j=1}^m \frac{\partial f}{\partial \hat{x}_j} \cdot \hat{x}_j}\bigg ]}\\)


記得前面有計算過的東西嗎?

\\(\boxed{\dfrac{\partial f}{\partial \hat{x}_i} = \dfrac{\partial f}{\partial y_i} \cdot \gamma}\\)

\\(\boxed{\color{red}{\frac{\partial f}{\partial \beta} = \sum\limits_{i=1}^m \frac{\partial f}{\partial y_i}}}\\)

\\(\boxed{\color{blue}{\frac{\partial f}{\partial \gamma} = \sum\limits_{i=1}^m \frac{\partial f}{\partial y_i} \cdot \hat{x}_i}}\\)

```py
def batchnorm_backward(dout, cache):

	N, D = dout.shape
	x_mu, inv_var, x_hat, gamma = cache

	# intermediate partial derivatives
	dxhat = dout * gamma

	# final partial derivatives
	dx = (1. / N) * inv_var * (N*dxhat - np.sum(dxhat, axis=0) 
		- x_hat*np.sum(dxhat*x_hat, axis=0))
	dbeta = np.sum(dout, axis=0)
	dgamma = np.sum(x_hat*dout, axis=0)

	return dx, dgamma, dbeta
```
實測: 速度6.92x 相對於一般版本

[Deriving the Gradient for the Backward Pass of Batch Normalization](https://kevinzakka.github.io/2016/09/14/batch_normalization/)
[Understanding the backward pass through Batch Normalization Layer](https://kratzert.github.io/2016/02/12/understanding-the-gradient-flow-through-the-batch-normalization-layer.html)
[论文笔记：Batch Normalization](http://jermmy.xyz/2017/09/02/2017-9-2-paper-notes-batch-normalization/)