---
title: Generative Models
tags:
  - CS231n
categories:
  - Computer Vision
date: 2018-07-09 08:28:09
---

Talk about Generative Models from cs231n 2018 lecture 12

# Generative Models


![](https://i.imgur.com/zvwaKBG.png)
![](https://i.imgur.com/vEvSjsm.png)

![](https://i.imgur.com/z1aJwnh.png)


## PixelRNN/CNN

![](https://i.imgur.com/slPdqlC.png)

![](https://i.imgur.com/A5CiaSg.png)

![](https://i.imgur.com/oGgXdy8.png)
![](https://i.imgur.com/ll2pwjm.png)
![](https://i.imgur.com/KQFMeyp.png)
幾張圖片就講得很好了不需多言

## VAE(Variational Autoencoder)

![](https://i.imgur.com/JIkxQA2.png)

![](https://i.imgur.com/GjFYMLp.png)
![](https://i.imgur.com/xLYtVgJ.png)
Decoder 讓他還原成高維度
![](https://i.imgur.com/RbgXsQK.png)
訓練完之後 我們可以把Decoder 丟掉
放入分類器
![](https://i.imgur.com/micqnzE.png)

![](https://i.imgur.com/RuB0y2J.png)
![](https://i.imgur.com/dmOjw1n.png)
![](https://i.imgur.com/te4hdBh.png)


![](https://i.imgur.com/m6Psy0Z.png)
\\(P_{\theta }(z)\\),simple Gaussian prior
\\(P_{\theta }(x|z)\\),decoder neural network
這兩個都可以算
但沒辦法算所有\\(\int dz\\)
Date likelihood fail
\\(P_{\theta }(x)\\),不正是上面那個不能算的嗎
Posterior density also fail

![](https://i.imgur.com/XmXHZrA.png)
![](https://i.imgur.com/H7bnpJY.png)

![](https://i.imgur.com/H5MQYvG.png)
![](https://i.imgur.com/1vR6Z3S.png)
![](https://i.imgur.com/wziR3Bt.png)
![](https://i.imgur.com/PUYGRlS.png)
![](https://i.imgur.com/MTggorB.png)


![](https://i.imgur.com/8RnhZBj.png)
![](https://i.imgur.com/E65o6xo.png)
Vary z2 看左到右
Vary z1 不高興到高興
![](https://i.imgur.com/xZKUtib.png)
缺點是還有一些模糊的圖片
![](https://i.imgur.com/sVpew6D.png)

## GAN
![](https://i.imgur.com/VcDdtaE.png)
![](https://i.imgur.com/36Od20S.png)

![](https://i.imgur.com/xoXbTlH.png)
![](https://i.imgur.com/0jJRJ6D.png)
![](https://i.imgur.com/SBdWGpc.png)
generator上比起求min 改成求max
![](https://i.imgur.com/Dmo7GVb.png)
![](https://i.imgur.com/xf522XR.png)
## Conclusion
![](https://i.imgur.com/UtkCEWN.png)
抱歉只有圖 ㄎㄎ