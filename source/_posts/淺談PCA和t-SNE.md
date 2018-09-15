---
title: 淺談PCA和t-SNE
tags:
  - PCA
categories:
  - Machine learning
date: 2018-07-09 10:07:06
---

當特徵數過多時會有一些問題
1. overfitting
2. slow
3. 超過3維不好視覺化

所以我們需要一些降維方法

# PCA(Principal components analysis)

1. 標準化數據
2. 建立共變異數矩陣(covariance matrix)
3. 用奇異值分解(SVD)得到eigenvector和eigenvalue
4. 選k個特徵值和向量 將原本的數據映射到特徵向量上,得到新的數據

## SVD

\\(A=U\Sigma V^{T}\\)
其中 A 為一個 m x n 的矩陣,U和V都為正交矩陣,\\(\Sigma\\)為奇異值矩陣
A在PCA當中就叫做主成分

# t-SNE(t-distributed stochastic neighbor embedding)

PCA 是個相當直觀且有效的降維方式
不過在三維轉換為二維時我們可以看到
有些數據的集群完全被搗成一團
無法表現相似性和分布
且PCA是線性降維
可能會有underfitting問題

t-SNE把高維的數據用高斯分布的機率密度函數近似
低維度用t分佈來近似
然後用KL距離算相似度 再用 gradient descent求最佳

高維度數據表示:
![](https://cdn-images-1.medium.com/max/800/1*KxxzB_L8Wi1L-FgznjhkyA.png)

低維度數據表示:
![](https://cdn-images-1.medium.com/max/800/1*OwvAZgYHAfYvKehREGH9WQ.png)

## KL距離 (Kullback-Leibler Divergence)
用來表示兩個分布之間的相似度
也稱為相對熵(Relative Entropy)
![](https://cdn-images-1.medium.com/max/800/1*geKaV3PDC2idMGUCXroJzw.png)

t-SNE
還有使用困惑度(Perp)來當作超參
![](https://cdn-images-1.medium.com/max/800/1*BiQ-gYxdbZsGP5KfYUF9Kg.png)

## Cost function
用KL計算cost
![](https://cdn-images-1.medium.com/max/800/1*2XSnUoPaYB2_FMQ8xMzOuQ.png)

Gradient optimize
![](https://cdn-images-1.medium.com/max/800/1*MzouyFbnSKdeBA7nLJVCPw.png)

# PCA vs t-SNE

PCA
![](https://cdn-images-1.medium.com/max/1000/1*Be6oSLuXgG0cDi2gJEL-wA.jpeg)

t-SNE
![](https://cdn-images-1.medium.com/max/1000/1*5i8McPBKmFOMOcCDjl8w4Q.jpeg)


t-SNE缺點 比PCA慢

To be continued...