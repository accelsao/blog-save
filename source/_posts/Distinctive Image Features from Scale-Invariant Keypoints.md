---
title: Distinctive Image Features from Scale-Invariant Keypoints
tags:
  - Feature Extraction
categories:
  - Computer Vision

date: 2018-08-04 14:34:00
---

論文筆記(待補)
<!--more-->

# Introduction

圖像匹配是一個基本的電腦視覺問題
圖像特徵包含尺度,旋轉,亮度,3D視角

以下是Scale Invariant Feature Transform(SIFT)的步驟
1. Scale-space extrema detection
第一步找所有位置 用difference-of-Gaussian(DoG)來找出**可能的點**(potential interest points)很有效
The first stage of computation searches over all scales
and image locations. It is implemented efficiently by using a difference-of-Gaussian
function to identify potential interest points that are invariant to scale and orientation.

2. Keypoint localization
在所有候選點,一個詳細的模型適用於決定位置和規模 Keypoint基於穩定性(stability)來選擇
At each candidate location, a detailed model is fit to determine
location and scale. Keypoints are selected based on measures of their stability

3. Orientation assignment
對於一個keypoint用一或多個梯度方向來指定趨勢,之後的操作都是基於這些指定的趨勢,規模和位置
One or more orientations are assigned to each keypoint location
based on local image gradient directions. All future operations are performed
on image data that has been transformed relative to the assigned orientation, scale, and
location for each feature, thereby providing invariance to these transformations.

4. Keypoint descriptor
本地的圖像梯度在一個區域內的keypoint以選擇的規模來測量
用來表示允許顯著程度的本地變形和亮度變化
The local image gradients are measured at the selected scale
in the region around each keypoint. These are transformed into a representation that
allows for significant levels of local shape distortion and change in illumination.

這方法的一個重要方向是他生成大量特徵密集的覆蓋整張圖

# Detection of scale-space extrema

關鍵點檢測的第一步是找出可以在不同視角下重複指定的位置和尺度
可以使用**scale space**(一個連續的尺度函數)找出所有的可能平穩特徵來實現
Gaussian function(高斯函數)是唯一的可行的scale-space kernel
尺度空間被定義為由一個可變尺度的高斯和輸入卷積
$$L(x, y, \sigma)=G(x, y, \sigma)*I(x, y)$$
where, 
$$G(x, y, \sigma)=\frac{1}{2\pi\sigma^{2}}e^{-(x^2+y^2)/2\sigma^2}$$

為了有效在尺度空間檢測到穩定的關鍵點,提出了高斯差分尺度空間(DoG)


\begin{split}
D(x, y, \sigma)&=(G(x, y, k\sigma)-G(x, y, \sigma))*I(x,y) \\
&=L(x, y, k\sigma)-L(x, y, \sigma)
\end{split}

挑選這個函數有一些原因
1. 計算有效**D**只要相減即可**L**只是平滑
2. DoG函數提供接近scale-normalized Laplacian的高斯,\\(\sigma^2\nabla^2G\\)
	這個函數的極值相較其他點更為平穩
	D和上式的關係從這來\\(\frac {\partial G}{\partial \sigma}=\sigma \nabla ^{2}G\\)
	\\(\sigma \nabla ^{2}G=\frac {\partial G}{\partial \sigma}\approx\frac{G(x,y,k\sigma)-G(x,y,\sigma)}{k\sigma-\sigma}\\)
	因此\\(G(x,y,k\sigma)-G(x,y,\sigma)\approx(k-1)\sigma^2 \nabla ^{2}G\\)
	由此發現當k接近1的時候 誤差趨近於0 然而實作上發現k並沒有影響
3. LoG(Laplacian of Gaussian)提取的特徵穩定性最高 但計算效率不高

## Local extrema detection
為了檢測\\(D(x, y, \sigma)\\)極值每個點和他3D相鄰點(26點)比較,當他大於26個點或小於才會被選上,因為前幾次的比較所以後面的計算量會大幅降低
一個重要的問題是決定取樣頻率和尺度用來檢測極值域,不幸的是沒有最小值的樣本可以檢測到所有極值

#  Accurate keypoint localization
找到這些key point後把低對比和鄰近邊緣的點去除
用Tayor展開\\(
D(x) = D +\frac{\partial D^T}{\partial x}x+\frac{1}{2}x^T\frac{\partial^2 D}{\partial x^2}x\\)
\\(x = (x, y, \sigma)^T\\)是差值中心,\\(\hat x =-\frac{\partial^2 D^{-1}}{\partial x^2}\frac{\partial D}{\partial x}\\)

## Eliminating edge responses
有些極值點在邊緣位置,難定位又易受**noise**干擾,視為不穩定點,要去除
一般邊緣附近主曲率( principal curvature)會較高
主曲率可由2X2 Hessian Matrix得到
$$
H = \left[\begin{matrix}D_{xx}(x,y)&D_{xy}(x,y)\\D_{xy}(x,y)&D_{yy}(x,y)\end{matrix}\right]
$$
我們可以避免計算它的特徵值而只關心他的比例
設\\(\alpha\\)為高強度,\\(\beta\\)為低強度
$$Tr(H) = D_{xx} + D_{yy}= \alpha + \beta$$
$$Det(H) = D_{xx}D_{yy}-(D_{xy})^2= \alpha\beta$$
當determinant是負的時候,因為它非極值,去除
$$\frac{Tr(H)^2}{Det(H)} = \frac{(\gamma\beta+\beta)^2}{\gamma\beta^2} = \frac{(\gamma+1)^2}{\gamma}$$
如此只和ratio有關
只需用以下式子判斷主曲率是否低於threshold,r
$$\frac{Tr(H)^2}{Det(H)} < \frac{(\gamma_0+1)^2}{\gamma_0}$$

# Orientation assignment
為了實現旋轉不變性,需要特徵點角度 在計算特徵點的方向時根據特徵點所在的高斯尺度圖像中的局部特徵計算出
局部特徵包含gradient magnitude和orientation
$$m(x,y) = \sqrt{(L(x+1,y) - L(x-1,y))^2 + (L(x,y+1)-L(x,y-1))^2}$$
$$\theta(x,y) = arctan\left(\frac{L(x,y+1)-L(x,y-1)}{L(x+1,y)-L(x-1,y)}\right)$$
方向直方圖從樣本周遭的領域內分成36bins包含360度 1bin/10度 因為不是所有像素權重都相同所以要進行加權
論文裡用\\(\sigma\\)1.5來加權
為了防止噪音干擾 做一次平滑

To be continued...
