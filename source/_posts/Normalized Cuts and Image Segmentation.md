---
title: Normalized Cuts and Image Segmentation
tags:
  - Image Segmentation
categories:
  - Computer Vision
date: 2018-08-04 21:41:38
---

Normalized Cuts and Image Segmentation 論文筆記(待補)

<!--more-->

從**圖像**轉成**圖**
問題轉成Graph Partition

Normalized Cut
在圖上怎樣定義比較好
最直觀的方法找出K個不相交的頂點集
頂點集內權重和最小(邊的weight可以是相似度)
這就是Min Cut問題
但這個準則不好 我們希望這K個Set也可以差不多大小

方法就是Normalized Cuts

$$Ncut(A,B)=\frac{cut(A,B)}{assoc(A,V)}+\frac{cut(A,B)}{assoc(B,V)}$$

然而問題變成了NP-Hard
把正交約束保留,離散轉成連續,聚類就變成找正交陣優化問題,SVD處理


[參考](http://www.cis.upenn.edu/~jshi/papers/pami_ncut.pdf)