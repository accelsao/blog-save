---
title: OpenCV 入門 - IV
tags:
  - OpenCV C++

categories:
  - Computer Vision

date: 2018-07-21 19:43:56
---

OpenCV 續篇 IV

# Sobel Derivatives

是一種獲得影像一階的方法 常用於邊緣檢測

水平方向
$$G_{x} = \begin{bmatrix} -1 & 0 & +1 \\ -2 & 0 & +2 \\ -1 & 0 & +1 \end{bmatrix} * I$$
垂直方向
$$G_{y} = \begin{bmatrix} -1 & -2 & -1 \\ 0 & 0 & 0 \\ +1 & +2 & +1 \end{bmatrix} * I$$

計算有兩種方法
$$G = \sqrt{ G_{x}^{2} + G_{y}^{2} }$$
或是
$$G = |G_{x}| + |G_{y}|$$

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

int main(){
	Mat img=imread("../images/002.jpg", IMREAD_GRAYSCALE);
	
	GaussianBlur(img, img, Size(3, 3), 0, 0);
	Mat grad_x, grad_y;
	Mat abs_grad_x, abs_grad_y;
	Sobel(img, grad_x, CV_16S, 1, 0, 3, 1, 0, BORDER_DEFAULT);
	convertScaleAbs(grad_x, abs_grad_x);  //轉成CV_8U
	Sobel(img, grad_y, CV_16S, 0, 1, 3, 1, 0, BORDER_DEFAULT);
	convertScaleAbs(grad_y, abs_grad_y);
	Mat dst1, dst2;
	addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 0, dst1);
	imshow("origin", img);
	imshow("Sobel_1", dst1);
	imwrite("../images/sobel_002.jpg",dst1);
	waitKey(0);

}
```

<figure class="half">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="375"/>
<img src="https://i.imgur.com/Yi2Di0W.jpg" width="375"/>
</figure>

# Laplace Operator

Sobel是一階
Laplace就是求二階

$$Laplace(f) = \dfrac{\partial^{2} f}{\partial x^{2}} + \dfrac{\partial^{2} f}{\partial y^{2}}$$

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

int main(){
	Mat img=imread("../images/002.jpg", IMREAD_GRAYSCALE);
	int kernel_size = 3;
	int scale = 1;
	int delta = 0;
	int ddepth = CV_16S;
	GaussianBlur(img, img, Size(3, 3), 0, 0);
	Mat dst;
	Laplacian(img, dst, ddepth, kernel_size, scale, delta, BORDER_DEFAULT);
	//Convert output to a CV_8U image
	convertScaleAbs(dst, dst);
	
	imshow("origin", img);
	imshow("Laplacian", dst);
	imwrite("../images/Laplacian_002.jpg",dst);
	waitKey(0);

}
```

<figure class="half">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="375"/>
<img src="https://i.imgur.com/yzw0BAg.jpg" width="375"/>
</figure>

# Canny Edge Detector

1. 降噪
2. 找梯度
3. Non-maximum suppression
4. Two thresholds

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

const int kernel_size = 3;

int main(){
	Mat src=imread("../images/002.jpg", IMREAD_COLOR);
	Mat detected_edges, dst;
	dst.create(src.size(), src.type());
	cvtColor(src, src, COLOR_BGR2GRAY);
	blur(src, detected_edges, Size(3, 3));
	int lowThreshold = 10;
	int ratio = 3;
	Canny(detected_edges, detected_edges, lowThreshold, lowThreshold*ratio, kernel_size);
	dst = Scalar::all(0);
	src.copyTo(dst, detected_edges);
	imshow("Canny Edge Detector", dst);
	imwrite("../images/Canny Edge Detector_002.jpg",dst);
	waitKey();
}
```

<figure class="half">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="375"/>
<img src="https://i.imgur.com/emOtf8b.jpg" width="375"/>
</figure>

# Hough Line Transform

用Canny找Edge 
然後再找Line

有兩種方法
1. The Standard Hough Transform
	* dst:output
	* line:store in (r,\\(\theta\\))
	* rho:the resolution of the parameter r in pixel
	* theta
	* threshold 成line的最小交集數
2. The Probabilistic Hough Line Transform (較有效率)
	* dst
	* lines
	* rho
	* theta
	* threshold
	* minLinlength:少於這數字的線會被丟棄
	* maxLineGap:兩點被視為同線的最大間距
	
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

const int kernel_size = 3;

int main(){

	Mat dst, cdst, cdstP;
	Mat src = imread("../images/002.jpg", IMREAD_GRAYSCALE);


	Canny(src, dst, 50, 200, 3);
	cvtColor(dst, cdst, COLOR_GRAY2BGR);
	cdstP = cdst.clone();


	vector<Vec2f> lines;
	HoughLines(dst, lines, 1, CV_PI / 180, 200, 0, 0);
	// Draw the lines
	for (size_t i = 0; i < lines.size(); i++)
	{
		float rho = lines[i][0], theta = lines[i][1];
		Point pt1, pt2;
		double a = cos(theta), b = sin(theta);
		double x0 = a * rho, y0 = b * rho;
		pt1.x = cvRound(x0 + 1000 * (-b));
		pt1.y = cvRound(y0 + 1000 * (a));
		pt2.x = cvRound(x0 - 1000 * (-b));
		pt2.y = cvRound(y0 - 1000 * (a));
		line(cdst, pt1, pt2, Scalar(0, 0, 255), 3, LINE_AA);
	}

	// Probabilistic Line Transform
	vector<Vec4i> linesP; // will hold the results of the detection
	HoughLinesP(dst, linesP, 1, CV_PI / 180, 50, 60, 10); // runs the actual detection
														  // Draw the lines
	for (size_t i = 0; i < linesP.size(); i++)
	{
		Vec4i l = linesP[i];
		line(cdstP, Point(l[0], l[1]), Point(l[2], l[3]), Scalar(0, 0, 255), 3, LINE_AA);
	}

	imshow("Source", src);
	imshow("Detected Lines (in red) - Standard Hough Line Transform", cdst);
	imshow("Detected Lines (in red) - Probabilistic Line Transform", cdstP);

	imwrite("../images/ Standard Hough Line Transform_002.jpg",cdst);
	imwrite("../images/ Probabilistic Line Transform_002.jpg", cdstP);
	waitKey();
}
```
原圖->Standard->Probabilistic
<figure class="third">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="262"/>
<img src="https://i.imgur.com/Ol3zU7K.jpg" width="262"/>
<img src="https://i.imgur.com/FCi8C3d.jpg" width="262"/>
</figure>

# Remapping

簡而言之就是調整piexl的位置
以下是左右相反的例子
```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

int main(){

	Mat src = imread("../images/002.jpg", IMREAD_COLOR);
	imshow("origin",src);
	Mat dst(src.size(), src.type());
	Mat map_x(src.size(), CV_32FC1);
	Mat map_y(src.size(), CV_32FC1);
	for (int i = 0; i < map_x.rows; i++)
	{
		for (int j = 0; j < map_x.cols; j++)
		{
			map_x.at<float>(i, j) = (float)(map_x.cols - j);
			map_y.at<float>(i, j) = (float)i;
		}
	}
	remap(src, dst, map_x, map_y, INTER_LINEAR, BORDER_CONSTANT, Scalar(0, 0, 0));
	imshow("remap_window", dst);
	
	waitKey();
	
	imwrite("../images/Remap_002.jpg",src);

}
```
<figure class="Half">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="395"/>
<img src="https://i.imgur.com/6bFonzr.jpg" width="395"/>
</figure>

# Affine Transformations

簡單Affine變換

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

int main(){

	Mat src = imread("../images/002.jpg", IMREAD_COLOR);
	imshow("origin",src);

	Point2f srcTri[3];
	srcTri[0] = Point2f(0.f, 0.f);
	srcTri[1] = Point2f(src.cols - 1.f, 0.f);
	srcTri[2] = Point2f(0.f, src.rows - 1.f);

	Point2f dstTri[3];
	dstTri[0] = Point2f(0.f, src.rows*0.33f);
	dstTri[1] = Point2f(src.cols*0.85f, src.rows*0.25f);
	dstTri[2] = Point2f(src.cols*0.15f, src.rows*0.7f);

	Mat warp_mat = getAffineTransform(srcTri, dstTri);
	Mat warp_dst = Mat::zeros(src.rows, src.cols, src.type());
	warpAffine(src, warp_dst, warp_mat, warp_dst.size());

	Point center = Point(warp_dst.cols / 2, warp_dst.rows / 2);
	double angle = -50.0;
	double scale = 0.87;

	Mat rot_mat = getRotationMatrix2D(center, angle, scale);
	Mat warp_rotate_dst;
	warpAffine(warp_dst, warp_rotate_dst, rot_mat, warp_dst.size());

	imshow("Source image", src);
	imshow("Warp", warp_dst);
	imshow("Warp + Rotate", warp_rotate_dst);
	waitKey();
	
	imwrite("../images/Warp_002.jpg", warp_dst);
	imwrite("../images/Warp + Rotate_002.jpg", warp_rotate_dst);
}
```

src -> wrap -> wrap+rotate
<figure class="third">
<img src="https://i.imgur.com/RGw1sPy.jpg" width="262"/>
<img src="https://i.imgur.com/XAh1EQZ.jpg" width="262"/>
<img src="https://i.imgur.com/Te1gFv5.jpg" width="262"/>
</figure>

# Histogram Equalization
這種方法通常用來增加許多圖像的全局對比度
尤其當圖像對比接近時可以用來加強亮度對比

考慮一個離散的灰階影像 {x} 讓 \\(n_i\\) 表示灰階 i 出現的次數，這樣圖像中灰階為 i 的像素的出現機率是
$$ p_x(i) = p(x=i) = \frac{n_i}{n},\quad 0 \le i < L $$
L 通常是256
n 是圖像中所有的pixel數量
\\(P_x(i)\\)實際上是像素值為 i 的圖像的直方圖，歸一化到 [0,1]

定義累積函數:
$$ cdf_x(i) = \sum_{j=0}^i p_x(j)$$

$$equalized( x, y ) = cdf_x(src(x,y))$$

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){

	Mat src = imread("../images/002.jpg", 0);
	Mat dst;
	equalizeHist(src, dst);
	imshow("Source image", src);
	imshow("Equalized Image", dst);
	waitKey();
	
	imwrite("../images/Equalized Image_002.jpg", dst);
}
```
<figure class="Half">
<img src="https://i.imgur.com/45LHFXn.jpg" width="395"/>
<img src="https://i.imgur.com/MjUTzek.jpg" width="395"/>
</figure>

# Finding contours in your image

就結果來說 contours 和 edge 是一樣的

細說一下他們的差別
Edges是一堆影像梯度的極值點
他就是找出所有和**相鄰**點有較大變化的這些點

Contours通常從edges來 但他是找出閉曲線 類似**邊界**的感覺
把這些edges連起來形成閉曲線

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;


RNG rng(12345);

int main(){

	Mat src = imread("../images/002.jpg", 0);
	
	blur(src, src, Size(3, 3));
	imshow("src", src);

	int thresh = 100;
	Mat canny_output;
	Canny(src, canny_output, thresh, thresh * 2);
	vector<vector<Point> > contours;
	vector<Vec4i> hierarchy;
	findContours(canny_output, contours, hierarchy, RETR_TREE, CHAIN_APPROX_SIMPLE);
	Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);
	for (size_t i = 0; i< contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 256), rng.uniform(0, 256), rng.uniform(0, 256));
		drawContours(drawing, contours, (int)i, color, 2, LINE_8, hierarchy, 0);
	}
	imshow("Contours", drawing);
	waitKey();
	
	imwrite("../images/Contours_002.jpg", drawing);
}
```
<figure class="Half">
<img src="https://i.imgur.com/45LHFXn.jpg" width="395"/>
<img src="https://i.imgur.com/bI2Rqwg.jpg" width="395"/>
</figure>

# Convex Hull

把contours的凸包連線

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;


RNG rng(12345);

int main(){

	Mat src = imread("../images/002.jpg", 0);
	
	blur(src, src, Size(3, 3));
	imshow("src", src);

	int thresh = 100;
	Mat canny_output;
	Canny(src, canny_output, thresh, thresh * 2);
	vector<vector<Point> > contours;
	findContours(canny_output, contours, RETR_TREE, CHAIN_APPROX_SIMPLE);
	vector<vector<Point> >hull(contours.size());
	for (size_t i = 0; i < contours.size(); i++)
	{
		convexHull(contours[i], hull[i]);
	}

	Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);
	for (size_t i = 0; i< contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 256), rng.uniform(0, 256), rng.uniform(0, 256));
		drawContours(drawing, contours, (int)i, color);
		drawContours(drawing, hull, (int)i, color);
	}
	imshow("Hull demo", drawing);
	waitKey();
	
	imwrite("../images/Hull demo_002.jpg", drawing);
}
```
<figure class="Half">
<img src="https://i.imgur.com/45LHFXn.jpg" width="395"/>
<img src="https://i.imgur.com/dzU1iAY.jpg" width="395"/>
</figure>

# Creating Bounding boxes and circles for contours

如其名 在畫上 圈圈和矩形

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;


RNG rng(12345);

int main(){

	Mat src = imread("../images/moon.jpg", 0);
	
	blur(src, src, Size(3, 3));
	imshow("src", src);

	int thresh = 100;
	Mat canny_output;
	Canny(src, canny_output, thresh, thresh * 2);
	vector<vector<Point> > contours;
	findContours(canny_output, contours, RETR_TREE, CHAIN_APPROX_SIMPLE);
	
	vector<vector<Point> > contours_poly(contours.size());
	vector<Rect> boundRect(contours.size());
	vector<Point2f>centers(contours.size());
	vector<float>radius(contours.size());

	for (size_t i = 0; i < contours.size(); i++)
	{
		approxPolyDP(contours[i], contours_poly[i], 3, true);
		boundRect[i] = boundingRect(contours_poly[i]);
		minEnclosingCircle(contours_poly[i], centers[i], radius[i]);
	}
	Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);


	for (size_t i = 0; i< contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 256), rng.uniform(0, 256), rng.uniform(0, 256));
		drawContours(drawing, contours_poly, (int)i, color);
		rectangle(drawing, boundRect[i].tl(), boundRect[i].br(), color, 2);
		circle(drawing, centers[i], (int)radius[i], color, 2);
	}
	imshow("Contours", drawing);
	waitKey();
	
	imwrite("../images/Contours_002.jpg", drawing);
}
```
<figure class="Half">
<img src="https://i.imgur.com/2WjQYW9.jpg" width="395"/>
<img src="https://i.imgur.com/zFV8vGr.jpg" width="395"/>
</figure>

# Image Moments

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;


RNG rng(12345);

int main(){

	Mat src = imread("../images/moon.jpg", 0);
	
	blur(src, src, Size(3, 3));
	imshow("src", src);

	int thresh = 100;
	Mat canny_output;
	Canny(src, canny_output, thresh, thresh * 2, 3);
	vector<vector<Point> > contours;
	findContours(canny_output, contours, RETR_TREE, CHAIN_APPROX_SIMPLE);
	
	vector<Moments> mu(contours.size());
	for (size_t i = 0; i < contours.size(); i++)
	{
		mu[i] = moments(contours[i]);
	}

	vector<Point2f> mc(contours.size());
	for (size_t i = 0; i < contours.size(); i++)
	{
		//add 1e-5 to avoid division by zero
		mc[i] = Point2f(static_cast<float>(mu[i].m10 / (mu[i].m00 + 1e-5)),
			static_cast<float>(mu[i].m01 / (mu[i].m00 + 1e-5)));
		cout << "mc[" << i << "]=" << mc[i] << endl;
	}
	Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);
	for (size_t i = 0; i< contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 256), rng.uniform(0, 256), rng.uniform(0, 256));
		drawContours(drawing, contours, (int)i, color, 2);
		circle(drawing, mc[i], 4, color, -1);
	}
	imshow("Contours", drawing);
	cout << "\t Info: Area and Contour Length \n";
	for (size_t i = 0; i < contours.size(); i++)
	{
		cout << " * Contour[" << i << "] - Area (M_00) = " << std::fixed << std::setprecision(2) << mu[i].m00
			<< " - Area OpenCV: " << contourArea(contours[i]) << " - Length: " << arcLength(contours[i], true) << endl;
	}


	waitKey();
	
	imwrite("../images/Contours_002.jpg", drawing);
}
```
<figure class="Half">
<img src="https://i.imgur.com/2WjQYW9.jpg" width="395"/>
<img src="https://i.imgur.com/c6gRi5x.jpg" width="395"/>
</figure>


# Image Segmentation with Distance Transform and Watershed Algorithm

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

int main()
{
	
	Mat src = imread("../images/girl.jpg",1);
	// Show source image
	imshow("Source Image", src);
	// Change the background from white to black, since that will help later to extract
	// better results during the use of Distance Transform
	// 把白色的pixel換成黑色 當用Distance Transform的時候有用於分辨物體
	for (int i = 0; i < src.rows; i++) {
		for (int j = 0; j < src.cols; j++) {
			if (src.at<Vec3b>(i, j) == Vec3b(255, 255, 255))
			{
				src.at<Vec3b>(i, j)[0] = 0;
				src.at<Vec3b>(i, j)[1] = 0;
				src.at<Vec3b>(i, j)[2] = 0;
			}
		}
	}
	// Show output image
	imshow("Black Background Image", src);
	
	// 接下來要Sharpen圖片 用接近二階的Laplacian
	// Create a kernel that we will use to sharpen our image
	Mat kernel = (Mat_<float>(3, 3) <<
		1, 1, 1,
		1, -8, 1,
		1, 1, 1); // an approximation of second derivative, a quite strong kernel
				  // do the laplacian filtering as it is
				  // well, we need to convert everything in something more deeper then CV_8U
				  // because the kernel has some negative values,
				  // and we can expect in general to have a Laplacian image with negative values
				  // BUT a 8bits unsigned int (the one we are working with) can contain values from 0 to 255
				  // so the possible negative number will be truncated
	Mat imgLaplacian;
	filter2D(src, imgLaplacian, CV_32F, kernel);
	Mat sharp;
	src.convertTo(sharp, CV_32F);
	Mat imgResult = sharp - imgLaplacian;
	
	
	
	// convert back to 8bits gray scale
	imgResult.convertTo(imgResult, CV_8UC3);
	imgLaplacian.convertTo(imgLaplacian, CV_8UC3);
	// imshow( "Laplace Filtered Image", imgLaplacian );
	imshow("New Sharped Image", imgResult);
	// src -> gray -> binary
	// Create binary image from source image
	Mat bw;
	cvtColor(imgResult, bw, COLOR_BGR2GRAY);
	threshold(bw, bw, 40, 255, THRESH_BINARY | THRESH_OTSU);
	imshow("Binary Image", bw);
	// Perform the distance transform algorithm
	Mat dist;
	distanceTransform(bw, dist, DIST_L2, 3);
	// Normalize the distance image for range = {0.0, 1.0}
	// so we can visualize and threshold it
	normalize(dist, dist, 0, 1.0, NORM_MINMAX);
	imshow("Distance Transform Image", dist);
	// Threshold to obtain the peaks
	// This will be the markers for the foreground objects
	threshold(dist, dist, 0.4, 1.0, THRESH_BINARY);
	// Dilate 來提取出 Peaks
	// Dilate a bit the dist image
	Mat kernel1 = Mat::ones(3, 3, CV_8U);
	dilate(dist, dist, kernel1);
	imshow("Peaks", dist);
	// Create the CV_8U version of the distance image
	// It is needed for findContours()
	Mat dist_8u;
	dist.convertTo(dist_8u, CV_8U);
	// Find total markers
	vector<vector<Point> > contours;
	findContours(dist_8u, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
	// Create the marker image for the watershed algorithm
	Mat markers = Mat::zeros(dist.size(), CV_32S);
	// Draw the foreground markers
	for (size_t i = 0; i < contours.size(); i++)
	{
		drawContours(markers, contours, static_cast<int>(i), Scalar(static_cast<int>(i) + 1), -1);
	}
	// Draw the background marker
	circle(markers, Point(5, 5), 3, Scalar(255), -1);
	imshow("Markers", markers * 10000);

	// 最後套watershed algorithm
	watershed(imgResult, markers);
	Mat mark;
	markers.convertTo(mark, CV_8U);
	bitwise_not(mark, mark);
	//    imshow("Markers_v2", mark); // uncomment this if you want to see how the mark
	// image looks like at that point
	// Generate random colors
	vector<Vec3b> colors;
	for (size_t i = 0; i < contours.size(); i++)
	{
		int b = theRNG().uniform(0, 256);
		int g = theRNG().uniform(0, 256);
		int r = theRNG().uniform(0, 256);
		colors.push_back(Vec3b((uchar)b, (uchar)g, (uchar)r));
	}
	// Create the result image
	Mat dst = Mat::zeros(markers.size(), CV_8UC3);
	// Fill labeled objects with random colors
	for (int i = 0; i < markers.rows; i++)
	{
		for (int j = 0; j < markers.cols; j++)
		{
			int index = markers.at<int>(i, j);
			if (index > 0 && index <= static_cast<int>(contours.size()))
			{
				dst.at<Vec3b>(i, j) = colors[index - 1];
			}
		}
	}
	// Visualize the final image
	imshow("Final Result", dst);
	waitKey();
	return 0;
}
```

<figure class="Half">
<img src="https://i.imgur.com/H7biN89.jpg" width="395"/>
<img src="https://i.imgur.com/6EGhk9o.jpg" width="395"/>
</figure>

