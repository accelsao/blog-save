---
title: Detection and Segmentation
date: 2018-07-08 15:42:05
tags:
  - CS231n
  - Object Detection
categories:
  - Computer Vision
---

簡述CS231n Lecture 11的內容


先說一下目前有哪些問題
![](https://i.imgur.com/C1bfkk6.jpg)
![](https://i.imgur.com/VTp8YIM.jpg)


# Semantic Segmentation 

## 簡述
只在乎pixel
Label圖片中所有的pixel來分類
![](https://i.imgur.com/FdygjfH.jpg)


![](https://i.imgur.com/TDk5J2e.jpg)
不可行...計算量太大

換個方法

![](https://i.imgur.com/5MCttJi.jpg)
一開始計算量很大


![](https://i.imgur.com/7SnFZWv.jpg)
問題來了
Upsampling怎麼做

![](https://i.imgur.com/bM31yNc.jpg)
有兩種 如圖
要碼複製 要碼放0

另外一種
![](https://i.imgur.com/4tzSCI1.jpg)
取最大


![](https://i.imgur.com/gee3hq1.jpg)
當做是Rescale 


![](https://i.imgur.com/FfwSe8r.jpg)

## FCN(Fully Convolutional networks)

綜述以上有點亂七八糟的東西

1. Convolutionalization
一般網路 會接FC層 再接 分類層
但因為Segmentation 需要分割圖 所以把FC層替換成ConvNet 就是Idea: Fully Convolutional 那邊
因為FC層會把圖片壓縮成一維的 好方便接後面的分類標籤
Conv則還是二維

2. Upsampling
一般CNN會有池化 把圖片Size縮小
但我們需要和原圖一樣的分割圖
所以Downsampling 再接 Upsampling
方法呢 就是上述所說的 **Transpose_Convolution**



# Object detection 傳統算法

## Sliding Window

很暴力的方法
![](https://i.imgur.com/LEyGDPa.jpg)

爆搜框框 說明這個框框是否是我要找的 是->yes 否->No
明顯困難點是怎麼選擇這個框框 因為物體在哪裡都有可能
運算花費極高

## Region proposal
![](https://i.imgur.com/GeWYEt8.jpg)
顧名思義是找一些候選位置 挑出比較有可能的地方來處理


# R-CNN

![](https://i.imgur.com/aSSVUW0.jpg)
先Region proposal挑出一些地方
但因為要放入ConvNet 所以要Resize
最後進入SVM 分類器

![](https://i.imgur.com/bxDd1Go.jpg)


## Fast-RCNN

![](https://i.imgur.com/LT1oY4m.jpg)
一開始不先找**候選區域**
直接ConvNet
然後再找**候選位置**
稱之為Rol(Regions of Interest)

![](https://i.imgur.com/Zou15pP.jpg)

然後進FC+分類器

整體大概是這樣
![](https://i.imgur.com/417BbLS.jpg)

## Faster-RCNN
![](https://i.imgur.com/kKgxLGT.jpg)
和Fast-RCNN
差別就只是多了一層RPN(區域候選網路)
從CNN出來的feature 直接在這邊找候選 同一個網路框架




## SPP-Net
這裡是CS231n沒有的 但還是記錄一些
![](https://i.imgur.com/ZCtiXav.jpg)

原本的CNN要Resize圖片 然後才進入Conv Layer
但SPP 可以輸入任意Size 然後 通過 Spatial pyramid pooling 來生成固定size的圖片

![](https://i.imgur.com/luPIE1W.png)
![](https://images2015.cnblogs.com/blog/1047939/201704/1047939-20170426134929553-607353721.png)

## 比較
![](https://i.imgur.com/P08IDWX.jpg)


## Mask R-CNN

[paper limk](https://arxiv.org/pdf/1703.06870.pdf)

![](https://img.itw01.com/images/2017/12/09/00/4503_Fode7X_2N4NEMF.jpg)

### 介紹

和Faster的差別

1. Roi pooling -> RoiAlign
2. 添加並列的FCN層(Lmask)

###	技術要點
1. 用ResNeXt-101 + FPN 
2. RoiAlign 引入插值過程 先通過雙線性插值到\\(14X14\\) 再pooling到 \\(7X7\\)
解決只通過pooling帶來的Misalignment問題
3. 訓練過程中 在每個sampled RoI定義 `multi-task loss` as \\(L = L_{cls}+L_{box}+L_{mask}\\)
The classification loss and bounding-box loss are identical to Fast RCNN
The mask branch has a (K , \\(m^{2}\\)) - dimensional output for each RoI, which encodes K binary masks of resolution m × m, one for each of the K classes.
分成K個classes 應用 per-pixel sigmoid 
\\(L_{mask}\\)定義為 average binary cross-entropy loss
這樣不同Classes之間就不會貢獻Loss給對方




# YOLO/SSD


![](https://i.imgur.com/7iLhksz.jpg)

前述所說是基於Region Proposal的方法
這兩個呢 則是基於迴歸法
由給定圖像的多個位置上迴歸出這個位置的目標邊框和類別來提升檢測速度

## YOLO

首先把圖片分割成 S X S
每一格會預測B個邊界框(bounding box)和邊界框的可信度(confidence score)
可信度包含兩個
* 可能性大小 \\(Pr(object)\\) 包含目標=1 否則=0
* 準確度

準確度可以用來預測和實際框(Ground truth)的IOU(intersection over union)
所以可信度(confidence score) 可以用\\(Pr(object)*IOU^{truth}_{pred}\\)表示
每個邊界框的預測值包含(x,y,w,h,c)
(x,y)位置 ,(w,h)大小 c可信度

每個單元格要預測(B*5+C)個值
最終預測量是(S,S,B*5+C)

![](https://i.imgur.com/1bm36Aq.jpg)


## SSD

### Abstract

SSD輸出一系列離散化(discretization)的bounding boxes
這些bounding boxes是在不同layers上的feature map 生成
在不同尺度上預測 增加準度還有不同aspect ratio結果

#### 預測階段
要計算出每個default box的物體,其屬於每種類別的可能性,也就是scores
對這些bounding box的shape使其符合物體的外接矩形
比起基於region proposal的模型
SSD沒**proposals generation**,**pixel resampling**或**feature resampling**這些階段
所以在保證精度時 速度也快上許多

### introduction

現今流行的檢測步驟
先生成一些bounding boxes
從中提取特徵 在經過分類器 判斷是否是物體
Faster R-CNN的 ResNet 為了精度犧牲不少時間
但從YOLO開始 消除了boungind box , pixel of feature resampling的過程
SSD比起YOLO又好很多

### Core Idea
Our improvements include using a small convolutional filter to predict object categories and offsets in bounding box locations, using separate predictors (filters) for different aspect ratio detections, and applying these filters to multiple feature maps from the later stages of a network in order to perform detection at multiple scales.

### 特點
1. 速度>YOLO 準度\\(\approx \\)Faster RCNN
2. SSD核心 predict **object** 和屬於哪種class的**score**
3. 在feature map 上使用小的Conv filter來predict一系列bounding box的**box offsets**
4. 在不同層次,尺度的feature map上predict **object**,**box offests**,得到不同**aspect ratio**
5. end-to-end 訓練也變簡單, 在檢測速度和精度取得好的**trade-off**

### Model
SSD 是基於前向CNN 產生一系列的 **fixed-size** bounding boxes和boxes的scores 配上 NMS(Non-maximum suppression)得到predicts

除了基礎結構外增加
* Multi-scale feature maps for detection 
額外的ConvNet layer 大小逐層遞減 來多尺度進行
* Convolutional predictors for detection 
每個增加的特徵層 用一系列的Convolutional filters,產生固定大小的predictions
* Default boxes and aspect ratios 
預測得到的box和default box的 offset(偏移值)
和每個box的scores(K個classes的機率都要算)
因此對於一個位置上的N個boxes中的每一個box,需要計算K-classes,每一個類的score還有這個box相對於他的默認box的4個offset
需要 (K+4) X N filters
對於h,w的size的特徵圖 就要 (K+4) X N X h X w 個predicts


### 比較圖
![](https://i.imgur.com/lRS9hMh.png)

# 其他
![](https://i.imgur.com/PDODwbS.jpg)

# 參考
![论文阅读：SSD: Single Shot MultiBox Detector](https://blog.csdn.net/u010167269/article/details/52563573)
![cs231n_2017_lecture11](http://cs231n.stanford.edu/slides/2017/cs231n_2017_lecture11.pdf)
