---
title: Motion
tags:
  - CS131
categories:
  - Computer Vision
date: 2018-07-10 11:37:31
---

簡述CS131的課程-Motion

原本圖片的data是二維的\\(I(x,y)\\)

一堆圖片加上時間維度就成了影片
變成 Image date=\\(I(x,y,t)\\)

# Optical flow

光流(Optical flow or optic flow)
關於視域中的物體運動檢測中的概念
描述相對**觀察者**目標的運動
## Definition

基本上是通過圖片中的pixel在時間上的各種變化來推測物體速度,方向
光流是由於**觀察者**和**被觀察者**之間的相互運動產生
利用圖片中pixel強度的時間變化和關聯性來推測物體的**運動**

## 前提假設
1. Brightness constancy 亮部不變
2. Small motion 運動變化微小
3. Spatial coherence 同張圖的pixel做一樣的運動

## Brightness	Constancy	Equation
假設目前亮度\\(I(x,y,t)\\)移動後 可以得出亮度相同的結論:
* Brightness	Constancy	Equation
	\\(I(x,y,t-1)=I(x+u(x,y),y+v(x,y),t)\\)
	把右邊泰勒展開
	\begin{equation} 
	\begin{split}
	I(x+u,y+v,t) & \approx I(x,y,t-1)+I_xu(x,y)+I_yv(x,y)+I_t \\
	I(x+u,y+v,t) & -I(x,y,t-1)=I_xu(x,y)+I_yv(x,y)+I_t \\
	Hence,I_xu(x,y) & +I_yv(x,y)+I_t \approx 0 \rightarrow \nabla I \cdot \begin{bmatrix}
	u\\
	v
	\end{bmatrix}^{T}+I_t=0
	\end{split}
	\end{equation}
	
# Lucas-Kanade method
由上述可知

\\(I_{x}(q_{1})V_{x}+I_{y}(q_{1})V_{y}=-I_{t}(q_{1})\\)
\\(I_{x}(q_{2})V_{x}+I_{y}(q_{2})V_{y}=-I_{t}(q_{2})\\)
.
.
.
\\(I_{x}(q_{n})V_{x}+I_{y}(q_{n})V_{y}=-I_{t}(q_{n})\\)

where \\(q_1\\),\\(q_2\\)..\\(q_n\\)都是pixels在window裡面 其他跟上述公式一樣
寫成矩陣\\(Av=b\\)

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/4dba1d119546e8fe15ad701c99d0e32595f9a6c8)

用least squares 改寫成
\\(A^{T}Av=A^{T}b\\)

![](https://i.imgur.com/8N5MvU4.jpg)

## Harris corner detection
先前所述\\(A^{T}A\\) 是 second moment Matrix 也是 Harris corner detection的 M
![](https://i.imgur.com/omXgypM.png)
我們來考慮 eigenvalues 的關係
![](https://i.imgur.com/wvFk4i7.jpg)

# Horn-Schunk method
這個算法用global constrint of smoothness 來解決Aperture problem
假設整張圖平滑,灰度不變

制定以下公式,最小化其公式
\\(E=\iint [(I_{x}u+I_{y}v+I_{t})^{2}+\alpha ^{2}(\lVert \nabla u\rVert ^{2}+\lVert \nabla v\rVert ^{2})]dxdy\\)

\\(I_{x}u+I_{y}v+I_{t}\\)是	brightness consistency
\\(\lVert \nabla u\rVert ^{2}+\lVert \nabla v\rVert ^{2}\\) smoothness constraint 用來保證frame變化很小
\\(\alpha\\)是平滑因子 越大越平滑

其中\\((I_{x}u+I_{y}v+I_{t})^{2}+\alpha ^{2}(\lVert \nabla u\rVert ^{2}+\lVert \nabla v\rVert ^{2})\\)求噵
\\(I_x(I_xu+I_yv+I_t)-\alpha ^2 \Delta u=0 \\
I_y(I_xu+I_yv+I_t)-\alpha ^2 \Delta v=0\\)

Laplace算子:
\\(\Delta=\frac{\partial^{2}}{\partial x^{2}}+\frac{\partial^{2}}{\partial y^{2}}\\)

實作中用:
\\(\Delta u(x,y)=\bar{u}(x,y)-u(x,y)\\)
\\(\bar{u}(x,y)\\)相當於在(x,y)的重心

替換後
\\((I_x^2+\alpha ^2)u+I_xI_yv=\alpha^2 \bar u-I_xI_t \\
I_xI_yu+(I_y^2+\alpha ^2)v=\alpha^2 \bar v-I_yI_t \\)
線性方成組,但由於要計算\\(\bar{u},\bar{v}\\)所以用疊代法來求解
$$
u^{k+1}=\bar u^k-\frac{I_x(I_x\bar u^k+I_y\bar v^k+I_t)}{\alpha ^2+I_x^2+I_y^2} \\
v^{k+1}=\bar v^k-\frac{I_y(I_x\bar u^k+I_y\bar v^k+I_t)}{\alpha ^2+I_x^2+I_y^2}
$$

## Pyramids	for	large motion

記得前面要求 small motion嗎?
那如果他很大怎麼解決?
**Reduce the resolution!**
每一層warp&unsample就好了
![](https://i.imgur.com/UnNygbA.jpg)

## Common fate
也記得 Spatial	coherence 吧?
大家的**運動模式**都要相同
那要怎麼在圖景中表示**Motion**

把每個張圖片分多個layers,同個affine motion放一起

![](https://slideplayer.com/13284810/79/images/42/How+do+we+estimate+the+layers.jpg)

# 參考
[Course pdf](http://vision.stanford.edu/teaching/cs131_fall1718/files/17_motion.pdf)
[Course Notes](http://vision.stanford.edu/teaching/cs131_fall1718/files/17_notes.pdf)