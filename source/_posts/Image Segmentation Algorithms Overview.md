---
title: Image Segmentation Algorithms Overview
tags:
  - Image Segmentation
categories:
  - Computer Vision
date: 2018-08-11 11:09:26
---
Image Segmentation Algorithms Overview 論文筆記(待補)

# Introdution
Image Segmentation 可以把圖像中我們有興趣的部分"分割"出來
這篇提出5個算法來簡單分析
1. Threshold segmentation method
基於區域最常見的算法,它的核心想法是基於一個最佳的灰度值界線來區分圖像
第二種是基於clustering的分割
將相近性質的區域合併,反之區分出來
最後一種是CNN裡的weakly-supervised learning來分割
把區分出來的東西添上label

# Region-based Segmentation
## Threshold Segmentation
Threshold segmentation 是最簡單也是平行分割最常見的

global threshold method 以單一閾值界定
local threshold method 選擇多個分割閾值處理

最常見的方法是  largest interclass variance method(Otsu) 選擇最佳的threshold來最大化class之間差異
除此之外還有 基於信息熵(entropy-based),最小化誤差,Co-occurrence matrix(共生矩陣),動量守恆(moment preserving),簡單統計(simple statistical),機率鬆弛(probability relaxation),模糊集(fuzzy set),和合併其他算法的方法

threshold method的優點是計算簡單又快速
當圖像有高對比,效果就很顯著
缺點則是正確率低因為沒有顯著的灰度差或高重複灰度值

## Regional Growth Segmentation
這個算法是基於"連續區域"將"相似"的部分合併
這個算法須要先找出"種子pixel"然後向外"生長"來分割圖
比如說種子旁的pixel灰度值和種子的差<threshold T就包含於種子
T的選擇很重要 不好的選擇可能把有分等於沒分
優點:
1. 通常能夠把相近特色的區域連接然後提供好的邊界資訊和分割結果
2. 生長標準自由
3. 可以同時擁有生長標準

缺點:
1. 計算量很大
2. noise和灰度不均勻會導致voids和over-division
3. 陰影效果差

# Edge Detection Segmentation
物體的邊緣是由不連續的圖像特徵組成如亮度變化,顏色變化,材質變化來達成分割
兩個相鄰區域不同灰度值的灰邊緣一定會有不連續的灰值 通常可用differential operators找出
並行邊緣檢測(Parallel edge detection)通常 done by means of a spatial domain differential operator to perform image segmentation by convoluting its template and image
常見的一階微分算子有 Prewitt operator, Roberts operator and Sobel operator
二階有Laplacian, Kirsch operator and Wallis operator

## Sobel Operator
一階邊緣檢測算子
主要用於邊緣檢測,離散微分算子用來計算圖像亮度梯度的接近值
noise有平滑效果且可有效去除noise造成的影響
Sobel operator 是有權重的 優於 Prewitt operator and the Roberts operator.

## Laplacian Operator
二階微分算子
只考慮位置忽略周圍灰度差更為適合
他對孤立pixel的反應大於邊緣和線,因此只應用於無噪音圖像

# Segmentation based on clustering
K-means
# Segmentation based on weakly-supervised
the use of bounding box and image-level labels as markup training data
on the basis of DeepLab and used the expected maximization
algorithm (EM) to estimate unmarked Pixel class and CNN parameters
DeepLab method 2 Step

1. FCN to get coarse score map and interpolate to the original image size
2. borrow the fully connected CRF from the FCN to get the details of the segmentation refinement




