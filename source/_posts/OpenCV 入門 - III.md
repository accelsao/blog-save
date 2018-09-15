---
title: OpenCV 入門 - III
tags:
  - OpenCV C++

categories:
  - Computer Vision

date: 2018-07-11 11:49:29
---
OpenCV 續篇 III

# Smoothing Images

*	**smoothing**有時叫做**blurring**,簡單常見的影像處理方法
*	smoothing理由很多 我們這邊會用來降噪,**reduce noice**
*	我們會用**filter**處理,常見的方法是linear的
	\\(g(i,j)\\) 是輸入權重和
	\\(g(i,j) = \sum_{k,l} f(i+k, j+l) h(k,l)\\)
	\\(h(k,l)\\) 是 kernel,filter的係數
	## Filter
	### Normalized Box Filter
	$$
	K = \dfrac{1}{K_{width} \cdot K_{height}} \begin{bmatrix} 1 & 1 & 1 & ... & 1 \\ 1 & 1 & 1 & ... & 1 \\ . & . & . & ... & 1 \\ . & . & . & ... & 1 \\ 1 & 1 & 1 & ... & 1 \end{bmatrix}
	$$
	#### Code
	```cpp
	#include <iostream>
	#include <opencv2/opencv.hpp>
	using namespace cv;
	using namespace std;
	int main(){
		namedWindow("Blur", WINDOW_AUTOSIZE);
		Mat img; Mat blur_img;
		img = imread("../images/blur.jpg", IMREAD_COLOR);
		blur(img, blur_img, Size(15, 15), Point(-1, -1));
		imshow("Blur", blur_img);
		imwrite("../images/blur_img.jpg", blur_img);
		blur(img, blur_img, Size(31, 31), Point(-1, -1));
		imshow("Blur_2", blur_img);
		imwrite("../images/blur_img_2.jpg", blur_img);
		waitKey();
	}
	```
	左到右是原圖,Size(15,15),Size(31,31)
	<figure class="third">
	<img src="https://i.imgur.com/8hJndbK.jpg" width="220"/>
	<img src="https://i.imgur.com/dlvCOmf.jpg" width="220"/>
	<img src="https://i.imgur.com/ZLH3uA5.jpg" width="220"/>
	</figure>
	
	### Gaussian Filter
	$$
	G_{0}(x, y) = A e^{ \dfrac{ -(x - \mu_{x})^{2} }{ 2\sigma^{2}_{x} } + \dfrac{ -(y - \mu_{y})^{2} }{ 2\sigma^{2}_{y} } }
	$$
	\\(\mu\\)相當於高斯分布的頂端(peak)
	
	GaussianBlur(input,output,Size ksize,sigmaX,sigmaY)
	ksize()兩個可以不同但都要是正odd or 0
	#### Code
	```cpp
	GaussianBlur(img, blur_img, Size(15, 15), 0, 0);
	GaussianBlur(img, blur_img, Size(31, 31), 0, 0);
	```
	原圖->(15,15)->(31,31)
	<figure class="third">
	<img src="https://i.imgur.com/MqSWD26.jpg" width="220"/>
	<img src="https://i.imgur.com/yJ532YM.jpg" width="220"/>
	<img src="https://i.imgur.com/gE3kupK.jpg" width="220"/>
	</figure>
	```cpp
	GaussianBlur(img, blur_img, Size(15, 15), 0, 0);
	GaussianBlur(img, blur_img, Size(15, 15), 15, 15);
	```
	原圖->(15,15,0,0)->(15,15,15,15)
	<figure class="third">
	<img src="https://i.imgur.com/MqSWD26.jpg" width="220"/>
	<img src="https://i.imgur.com/sZc2OsZ.jpg" width="220"/>
	<img src="https://i.imgur.com/2KwruvO.jpg" width="220"/>
	</figure>
	
	### Median Filter
	預先處理所有pixel相鄰點的中位數(mean)
	然後再替換
	medianBlur(in,out,int ksize)
	ksize 只能是>1的odd
	#### Code
	```cpp
	medianBlur(img, blur_img, 15);
	medianBlur(img, blur_img, 31);
	```
	原圖->15->31
	<figure class="third">
	<img src="https://i.imgur.com/MqSWD26.jpg" width="220"/>
	<img src="https://i.imgur.com/dr6IQM4.jpg" width="220"/>
	<img src="https://i.imgur.com/FjF73tP.jpg" width="220"/>
	</figure>
	
	### Bilateral Filter
	到目前為止的**filters**都用來降噪,但有時候連**邊緣(edge)**都給去掉,我不們希望發生
	所有就有了這個算法
	他也考慮了周圍的pixel
	結合兩個函數
	其一用**幾何空間距離**決定filter係數
	另一個用**像素色差**決定
	$$
	I^{\text{filtered}}(x)={\frac {1}{W_{p}}}\sum _{x_{i}\in \Omega }I(x_{i})f_{r}(\|I(x_{i})-I(x)\|)g_{s}(\|x_{i}-x\|)
	$$
	where the normalization term
	$$
	W_{p}=\sum _{x_{i}\in \Omega }{f_{r}(\|I(x_{i})-I(x)\|)g_{s}(\|x_{i}-x\|)}
	$$
	\\(I^{filterd}\\) 是過濾過的image
	\\(I\\)是輸入image
	\\(x\\)是要被filter的位置
	\\(\Omega \\) 是中心在x的windows
	\\(f_r\\) range kernal 像素色差
	\\(g_s\\) spatial kernal 幾何距離
	$$
	w(i,j,k,l)=\exp \left(-{\frac {(i-k)^{2}+(j-l)^{2}}{2\sigma _{d}^{2}}}-{\frac {\|I(i,j)-I(k,l)\|^{2}}{2\sigma _{r}^{2}}}\right)
	$$
	\\(\sigma\\) 是smoothing 參數
	$$
	I_{D}(i,j)={\frac {\sum _{k,l}I(k,l)w(i,j,k,l)}{\sum _{k,l}w(i,j,k,l)}}
	$$
	計算完weight後要再normalize
	
	bilateralFilter(in,out,int d,double sigmaColor,double sigmaSpace)
	* Sigma values: 可以設兩個值相同 如果很小(<10),相反的(>150) 效果就很顯著 效果像卡通

	* Filter size: Large filters (d > 5) 很慢, 建議用 d=5 來實時應用(real-time applications), 那些需要heavy noise filtering的用 d=9 來做離線應用(offline applications)
	#### Code
	```cpp
	bilateralFilter(img, blur_img, 9, 100, 100);
	bilateralFilter(img, blur_img, 9, 500, 500);
	```
	原圖->100->500
	<figure class="third">
	<img src="https://i.imgur.com/MqSWD26.jpg" width="220"/>
	<img src="https://i.imgur.com/zYL8o0h.jpg" width="220"/>
	<img src="https://i.imgur.com/ts7o6er.jpg" width="220"/>
	</figure>
	
# Morphological Operations
針對shape的處理
常見的有**Erosion** and **Dilation**
* Removing noise
* Isolation of individual elements and joining disparate elements in an image.
* Finding of intensity bumps or holes in an image

## Dilation & Erosion
### Dilation
會讓bright地方**grow**
\\(\texttt{dst} (x,y) = \max _{(x',y'): \, \texttt{element} (x',y') \ne0 } \texttt{src} (x+x',y+y')\\)
### Erosion
相反的他選min
\\(\texttt{dst} (x,y) = \min _{(x',y'): \, \texttt{element} (x',y') \ne0 } \texttt{src} (x+x',y+y')\\)
所以會讓bright**縮小**

### Code
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	
	Mat img; 
	Mat erosion_dst;
	Mat dilation_dst;
	img = imread("../images/girl.jpg", IMREAD_COLOR);
	imshow("Origin", img);
	int erosion_size = 5;
	int dilation_size = 5;
	Mat element = getStructuringElement(MORPH_RECT,
		Size(2 * erosion_size + 1, 2 * erosion_size + 1),
		Point(erosion_size, erosion_size));
	erode(img, erosion_dst, element);
	imshow("Erosion", erosion_dst);
	imwrite("../images/erosion_img.jpg", erosion_dst);
	element = getStructuringElement(MORPH_RECT,
		Size(2 * dilation_size + 1, 2 * dilation_size + 1),
		Point(dilation_size, dilation_size));
	dilate(img, dilation_dst, element);
	imshow("Dilate", dilation_dst);
	imwrite("../images/dilate_img.jpg", dilation_dst);
	waitKey();
}
```
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/rB6RHTC.jpg" width="220"/>
<img src="https://i.imgur.com/rnEvfyO.jpg" width="220"/>
</figure>

---
```cpp
int erosion_size = 15;
int dilation_size = 15;
```
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/pgIIKQq.jpg" width="220"/>
<img src="https://i.imgur.com/vTFL5sW.jpg" width="220"/>
</figure>

---
```cpp
int erosion_size = 5;
int dilation_size = 5;
MORPH_CROSS
```
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/GAaCCdV.jpg" width="220"/>
<img src="https://i.imgur.com/k2ouwJ7.jpg" width="220"/>
</figure>

---
```cpp
int erosion_size = 5;
int dilation_size = 5;
MORPH_ELLIPSE
```
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/VQ8SvhH.jpg" width="220"/>
<img src="https://i.imgur.com/ZgpA7N9.jpg" width="220"/>
</figure>

## More Morphology Transformations
### Opening
* \\(dst = open( src, element) = dilate( erode( src, element ) )\\)
* 有用於移除小白點(亮點)

### Closing
* \\(dst = close( src, element ) = erode( dilate( src, element ) )\\)
* 有用於移除小黑點(暗點)

### Morphological Gradient
* \\(dst = morph_{grad}( src, element ) = dilate( src, element ) - erode( src, element )\\)
* 有用於找出輪廓

### Top Hat
* \\(dst = tophat( src, element ) = src - open( src, element )\\)
* 加強比原輪廓亮的部分

### Black Hat
* \\(dst = blackhat( src, element ) = close( src, element ) - src\\)
* 加強比原輪廓暗的部分

### Code
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	
	Mat img; 
	Mat erosion_dst;
	Mat dilation_dst;
	img = imread("../images/girl.jpg", IMREAD_COLOR);
	imshow("Origin", img);
	int erosion_size = 5;
	int dilation_size = 5;
	Mat element = getStructuringElement(MORPH_ELLIPSE,
		Size(2 * erosion_size + 1, 2 * erosion_size + 1),
		Point(erosion_size, erosion_size));
	erode(img, erosion_dst, element);
	element = getStructuringElement( MORPH_ELLIPSE,
		Size(2 * dilation_size + 1, 2 * dilation_size + 1),
		Point(dilation_size, dilation_size));
	dilate(img, dilation_dst, element);
	//imshow("Dilate", dilation_dst);
	//imwrite("../images/dilate_img.jpg", dilation_dst);

	Mat open_dst, close_dst;
	dilate(erosion_dst, open_dst, element);
	erode(dilation_dst, close_dst, element);
	imshow("Open", open_dst);
	imwrite("../images/open_img.jpg", open_dst);
	imshow("Close", close_dst);
	imwrite("../images/close_img.jpg", close_dst);

	waitKey();
}
```
原圖->OPEN->CLOSE
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/aGbm8WI.jpg" width="220"/>
<img src="https://i.imgur.com/NJaT1Y0.jpg" width="220"/>
</figure>

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	
	Mat img,dst; 
	img = imread("../images/girl.jpg", IMREAD_COLOR);
	int morph_size = 10;
	Mat element = getStructuringElement(MORPH_RECT, Size(2 * morph_size + 1, 2 * morph_size + 1), Point(morph_size, morph_size));
	morphologyEx(img, dst, MORPH_GRADIENT, element);
	
	imshow("dst", dst);
	imwrite("../images/MORPH_GRADIEN_img.jpg",dst);

	waitKey();
}
```
Morphological Gradient
<figure class="half">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/hCqyOFG.jpg" width="220"/>
</figure>

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	
	Mat img,dst; 
	img = imread("../images/girl.jpg", IMREAD_COLOR);
	int morph_size = 10;
	Mat top, black;
	Mat element = getStructuringElement(MORPH_RECT, Size(2 * morph_size + 1, 2 * morph_size + 1), Point(morph_size, morph_size));
	morphologyEx(img, top, MORPH_TOPHAT, element);
	morphologyEx(img, black, MORPH_BLACKHAT, element);
	
	imshow("top", top);
	imwrite("../images/top_img.jpg",top);
	imshow("black", black);
	imwrite("../images/black_img.jpg", black);

	waitKey();
}
```
原圖->TOP->BLACK
<figure class="third">
<img src="https://i.imgur.com/1TgxVNP.jpg" width="220"/>
<img src="https://i.imgur.com/sWULK2Z.jpg" width="220"/>
<img src="https://i.imgur.com/WcuEUMg.jpg" width="220"/>
</figure>