---
title: OpenCV 入門 - II
tags:
  - OpenCV C++

categories:
  - Computer Vision

date: 2018-07-11 08:47:23
---
OpenCV 續篇 II

# Mask operations on matrices


## Sharpen

對每個pixel做以下轉換:
![](https://i.imgur.com/mu8IrdL.png)

### Code
手寫code vs built-in func
時間:0.06 vs 0.01
```cpp
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

void Sharpen(const Mat& img, Mat& Output) {
	CV_Assert(img.depth() == CV_8U);
	const int nChannels = img.channels();
	Output.create(img.size(), img.type());

	for (int i = 1;i < img.rows - 1;i++) {
		const uchar* pre = img.ptr<uchar>(i - 1);
		const uchar* cur = img.ptr<uchar>(i);
		const uchar* nxt = img.ptr<uchar>(i + 1);

		uchar* output=Output.ptr<uchar>(i);
		for (int j = nChannels;j < nChannels*(img.cols - 1);j++) {
			*output++ = saturate_cast<uchar>(5 * cur[j] - (cur[j - nChannels] + cur[j + nChannels] + pre[j] + nxt[j]));
		}
	}
	Output.row(0).setTo(Scalar(0));
	Output.row(Output.rows - 1).setTo(Scalar(0));
	Output.col(0).setTo(Scalar(0));
	Output.col(Output.cols - 1).setTo(Scalar(0));
}
int main(){
	
	Mat img = imread("../images/girl.jpg", IMREAD_COLOR);

	namedWindow("Input", WINDOW_AUTOSIZE);
	namedWindow("Output", WINDOW_AUTOSIZE);
	imshow("Input", img);
	Mat dst0,dst1;
	Sharpen(img, dst0);

	double t = (double)getTickCount();
	t = ((double)getTickCount() - t) / getTickFrequency();
	cout << "Hand written function time passed in seconds: " << t << endl;
	imshow("Output", dst0);
	waitKey();
	Mat kernel = (Mat_<char>(3, 3) << 0, -1, 0,
		-1, 5, -1,
		0, -1, 0);
	t = (double)getTickCount();
	filter2D(img, dst1, img.depth(), kernel);
	t = ((double)getTickCount() - t) / getTickFrequency();
	cout << "Built-in filter2D time passed in seconds:     " << t << endl;
	imshow("Output", dst1);
	waitKey();
}
```
原圖
![](https://i.imgur.com/VdaTDdg.jpg)
Sharpen
![](https://i.imgur.com/9KHdaBX.jpg)

# Adding (blending) two images using OpenCV

\\(g(x) = (1 - \alpha)f_{0}(x) + \alpha f_{1}(x)\\)
讓\\(\alpha\\)從0到1 可看出不同的變化
## Code
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;

// we're NOT "using namespace std;" here, to avoid collisions between the beta variable and std::beta in c++17
using std::cin;
using std::cout;
using std::endl;


int main(){
	
	double alpha = 0.5; double beta; double input;
	Mat src1, src2, dst;

	src1 = imread("../images/alpha.png", IMREAD_COLOR);
	src2 = imread("../images/beta.png", IMREAD_COLOR);
	
	if (src1.empty()) { cout << "Error loading src1" << endl; return -1; }
	if (src2.empty()) { cout << "Error loading src2" << endl; return -1; }
	
	beta = 1.0 - alpha;
	addWeighted(src1, alpha, src2, beta, 0.0, dst);
	imshow("Linear Blend", dst);

	imwrite("../images/blend.jpg",dst);

	waitKey();
}
```
Blend
![](https://i.imgur.com/NXzC4C1.png)
![](https://i.imgur.com/earO8EP.jpg)
![](https://i.imgur.com/N0crE60.jpg)

# Changing the contrast and brightness of an image!

## Image Processing
把一張或多張圖當作輸入輸出
* Point operators (pixel)
* Neighborhood  operators (area-based)

### Pixel Transforms
#### Brightness and contrast adjustments
\\(g(i,j) = \alpha \cdot f(i,j) + \beta\\)
* image.at<Vec3b>(y,x)[c] where y is the row, x is the column and c is R, G or B (0, 1 or 2)
* 因為 \\(\alpha \cdot f(i,j) + \beta\\)可能有非整數(if α is float), we use cv::saturate_cast 來保證合法

##### saturate_cast<uchar>
簡單說就是會自動調整到合法的值
```cpp
uchar a = saturate_cast<uchar>(-100); // a = 0 (UCHAR_MIN)
short b = saturate_cast<short>(33333.33333); // b = 32767 (SHRT_MAX)
```

##### Code
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;

// we're NOT "using namespace std;" here, to avoid collisions between the beta variable and std::beta in c++17
using std::cin;
using std::cout;
using std::endl;


int main(){
	
	double alpha = 2;
	int beta = 50;
	Mat image;

	image = imread("../images/beta.png", IMREAD_COLOR);
	Mat new_image = Mat::zeros(image.size(), image.type());
	for (int y = 0; y < image.rows; y++) {
		for (int x = 0; x < image.cols; x++) {
			for (int c = 0; c < 3; c++) {
				new_image.at<Vec3b>(y, x)[c] =
					saturate_cast<uchar>(alpha*(image.at<Vec3b>(y, x)[c]) + beta);
			}
		}
	}
	namedWindow("Original Image", WINDOW_AUTOSIZE);
	namedWindow("New Image", WINDOW_AUTOSIZE);
	imshow("Original Image", image);
	imshow("New Image", new_image);

	
	imwrite("../images/blend.jpg",new_image);

	waitKey();
}
```
![](https://i.imgur.com/yFx29UQ.jpg)
#### Gamma correction
\\(O = \left( \frac{I}{255} \right)^{\gamma} \times 255\\)
用來修正過度曝光造成的模糊
記得LUT嗎? 可以建table直接修改

##### Code

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;

// we're NOT "using namespace std;" here, to avoid collisions between the beta variable and std::beta in c++17
using std::cin;
using std::cout;
using std::endl;


int main(){
	
	double alpha = 2;
	int beta = 50;
	Mat image;

	image = imread("../images/beta.png", IMREAD_COLOR);
	Mat new_image = Mat::zeros(image.size(), image.type());

	/*
	for (int y = 0; y < image.rows; y++) {
		for (int x = 0; x < image.cols; x++) {
			for (int c = 0; c < 3; c++) {
				new_image.at<Vec3b>(y, x)[c] =
					saturate_cast<uchar>(alpha*(image.at<Vec3b>(y, x)[c]) + beta);
			}
		}
	}
	*/

	// 可用一行代替以上迴圈
	image.convertTo(new_image, -1, alpha, beta);

	// gamma修正
	Mat lookUpTable(1, 256, CV_8U);
	uchar* p = lookUpTable.ptr();
	double gamma_ = 0.4;
	for (int i = 0; i < 256; ++i)
		p[i] = saturate_cast<uchar>(pow(i / 255.0, gamma_) * 255.0);
	Mat res = image.clone();
	LUT(image, lookUpTable, res);

	namedWindow("Original Image", WINDOW_AUTOSIZE);
	namedWindow("New Image", WINDOW_AUTOSIZE);
	namedWindow("Gamma Image", WINDOW_AUTOSIZE);
	imshow("Original Image", image);
	imshow("New Image", new_image);
	imshow("Gamma Image", res);
	
	imwrite("../images/blend.jpg",new_image);
	imwrite("../images/gamma.jpg", res);

	waitKey();
}
```
![](https://i.imgur.com/w1qagcE.jpg)

# Discrete Fourier Transform
Fourier Transform 會把圖片分解成sin,cos 簡單說 他會把他轉到頻域 實現高通和低通濾波
\\(F(k,l) = \displaystyle\sum\limits_{i=0}^{N-1}\sum\limits_{j=0}^{N-1} f(i,j)e^{-i2\pi(\frac{ki}{N}+\frac{lj}{N})}\\)
* f is the image value in its spatial domain and F in its frequency domain

還可以利用矩陣的卷積等同在頻域的乘法 透過轉換到frequency domain -> 乘法 -> 再轉回spatial domain 來加速運算

## Code
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;

using std::cin;
using std::cout;
using std::endl;
int main(){
	
	Mat I = imread("../images/gamma.jpg", IMREAD_GRAYSCALE);
	if (I.empty()) {
		cout << "Error opening image" << endl;
		return -1;
	}

	//DFT的表現取決於imgae size 當他質因數只有2,3,5的時候會比較優 所以用getOptimalDFTSize()  來找最佳的size
	Mat padded;                            //expand input image to optimal size
	int m = getOptimalDFTSize(I.rows);
	int n = getOptimalDFTSize(I.cols); // on the border add zero values
	//擴充0邊界
	copyMakeBorder(I, padded, 0, m - I.rows, 0, n - I.cols, BORDER_CONSTANT, Scalar::all(0));


	//DFT結果是複數 所以每張圖有兩個val
	Mat planes[] = { Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F) };
	Mat complexI;
	merge(planes, 2, complexI);         // 複數 所以 2

	dft(complexI, complexI);
	// compute the magnitude and switch to logarithmic scale
	// => log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))
	// 把實部 虛部分開
	split(complexI, planes);                   // planes[0] = Re(DFT(I), planes[1] = Im(DFT(I))



	magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude
	Mat magI = planes[0];
	// +1 讓他範圍>=0
	magI += Scalar::all(1);                    // switch to logarithmic scale 
	log(magI, magI);

	// crop the spectrum, if it has an odd number of rows or columns
	// 如果row or col 是odd 就不會對稱
	// x&-2 的作用會捨棄掉末位 所以都會變偶數 等價於 x/2*2
	magI = magI(Rect(0, 0, magI.cols & -2, magI.rows & -2));

	// rearrange the quadrants of Fourier image  so that the origin is at the image center
	int cx = magI.cols / 2;
	int cy = magI.rows / 2;

	// 還記得我們一開始擴充的一些多的部分嗎? 
	//現在要還原size 還要讓原本的center繼續保持

	Mat q0(magI, Rect(0, 0, cx, cy));   // Top-Left - Create a ROI per quadrant
	Mat q1(magI, Rect(cx, 0, cx, cy));  // Top-Right
	Mat q2(magI, Rect(0, cy, cx, cy));  // Bottom-Left
	Mat q3(magI, Rect(cx, cy, cx, cy)); // Bottom-Right
	Mat tmp;                           // swap quadrants (Top-Left with Bottom-Right)
	q0.copyTo(tmp);
	q3.copyTo(q0);
	tmp.copyTo(q3);
	q1.copyTo(tmp);                    // swap quadrant (Top-Right with Bottom-Left)
	q2.copyTo(q1);
	tmp.copyTo(q2);
	normalize(magI, magI, 0, 1, NORM_MINMAX); // Transform the matrix with float values into a viewable image form (float between values 0 and 1).
	imshow("Input Image", I);    // Show the result
	imshow("spectrum magnitude", magI);
	waitKey();
}
```
