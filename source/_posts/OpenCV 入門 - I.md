---
title: OpenCV 入門 - I
tags:
  - OpenCV C++
categories:
  - Computer Vision
date: 2018-07-10 19:15:17
---
簡單紀錄OpenCV C++的入門過程
一如既往的被各種環境搞瘋 重點是我以前才做過 結果還是...乾

特別紀錄一下
[[圖文] OpenCV 3.4.1 安裝配置在 Visual Studio 2017](https://charlottehong.blogspot.com/2017/06/opencv-320-vc14-visual-studio-2017.html)
首先這篇是沒什麼問題 之前做也成功過
後來發現新開專案 還要再重新設定...
還好翻到之前的repo


# Load and Display an Image

再來image路徑問題...
不知道為甚麼我一直放在opencv旁邊
後來突然想到要放cpp附近...
結果忘了在哪 找半天差點放棄
滑鼠移到 .cpp上面 就會出現路徑了 vs2017應該也有其他方法找cpp路徑 總之我不想管了

圖片路徑C:\Users\owner\source\repos\R00\images

不過只要`String imageName("../images/Test.png");`就OK了 然後.jpg也沒問題
```cpp
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	String imageName("../images/Test.jpg");
	Mat image;
	image = imread(imageName, IMREAD_COLOR); // Read the file
	if (image.empty())                      // Check for invalid input
	{
		cout << "Could not open or find the image" << std::endl;
		return -1;
	}
	namedWindow("Display window", WINDOW_AUTOSIZE); // Create a window for display.
	imshow("Display window", image);                // Show our image inside it.
	waitKey(0); // Wait for a keystroke in the window
}
```

# Load, Modify, and Save an Image
很簡單 經實測 可以用.png 同檔名圖片會被覆寫
```cpp
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	String imageName("../images/test.png");
	Mat image,gray;
	image = imread(imageName, IMREAD_COLOR); // Read the file
	if (image.empty())                      // Check for invalid input
	{
		cout << "Could not open or find the image" << std::endl;
		return -1;
	}

	cvtColor(image, gray, COLOR_BGR2GRAY);

	imwrite("../images/Gray_Image-png.png",gray);

	namedWindow(imageName, WINDOW_AUTOSIZE);
	namedWindow("Gray image", WINDOW_AUTOSIZE);
	imshow(imageName, image);
	imshow("Gray image", gray);
	waitKey(0); // Wait for a keystroke in the window
}
```

# The Basic Image Container
Mat 基本操作

## Mat
```cpp
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
int main(){
	String imageName("../images/test.png");
	Mat A,C;
	A  = imread(imageName, IMREAD_COLOR); // Read the file
	
	//只copy header
	Mat B(A);
	C = A;

	//找ROI的方法 產生部分的headers
	Mat D(A, Rect(10, 10, 100, 100)); // using a rectangle
	Mat E = A(Range::all(), Range(1, 3)); // using row and column boundaries

	//copy data
	Mat F = A.clone();
	Mat G;
	A.copyTo(G);
}
```
## Storing methods

簡介一些Color system

* RBG 常見的不多說
* HSV and HLS  把顏色分成 色調,飽和度,亮度
* YCrCb 常用於 JPEG
* CIE L*a*b* 是一個感知上均勻的色彩空間 有用於了解色彩和色彩的**距離**

## Mat類別

### cv::Mat::Mat Constructor
```cpp
Mat M(2, 2, CV_8UC3, Scalar(0, 0, 255));
cout << "M = " << endl << " " << M << endl << endl;

M=
[0,0,255,0,0,255;
0,0,255,0,0,255]
```

其中 CV_8UC3 如下格式:
CV_[The number of bits per item][Signed or Unsigned][Type Prefix]C[The channel number]

### Use C/C++ arrays and initialize via constructor

如果想要矩陣維度>2的話
```cpp
int sz[3] = {2,2,2};
Mat L(3,sz, CV_8UC(1), Scalar::all(0));
```
只是沒辦法用cout輸出..

### cv::Mat::create function
`M.create(4,4, CV_8UC(2)); `
括號內個數寬個數量 如上式 會構成4*(4*2)的矩陣 都是205
這不能用來初始矩陣,只有新的尺寸不符合舊的 他才會重新分配data memory

### MATLAB style initializer: cv::Mat::zeros , cv::Mat::ones , cv::Mat::eye . Specify size and data type to use:
`Mat E = Mat::eye(4, 4, CV_64F);`:左上-右下對角線 都1
` Mat O = Mat::ones(2, 2, CV_32F);`:全1
`Mat Z = Mat::zeros(3,3, CV_8UC1);`:全0

### 小矩陣初始方法
`Mat C = (Mat_<double>(3,3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);`:3x3
`C = (Mat_<double>({0, -1, 0, -1, 5, -1, 0, -1, 0})).reshape(3);`:3x3 reshape第二維

### Create a new header for an existing Mat object and cv::Mat::clone or cv::Mat::copyTo it.
`Mat RowClone = C.row(1).clone();`
把上一個row1複製下來

### Randpm
```
Mat R = Mat(3, 2, CV_8UC3);
randu(R, Scalar::all(0), Scalar::all(255));
```

### 輸出格式
* default `cout << "R (default) = " << endl <<        R           << endl << endl;`
* python `cout << "R (python)  = " << endl << format(R, Formatter::FMT_PYTHON) << endl << endl;`
* 沒[]的default `cout << "R (csv)     = " << endl << format(R, Formatter::FMT_CSV   ) << endl << endl;`
* numpy `cout << "R (numpy)   = " << endl << format(R, Formatter::FMT_NUMPY ) << endl << endl;`
* []變成<> `cout << "R (c)       = " << endl << format(R, Formatter::FMT_C     ) << endl << endl;`

# How to scan images, lookup tables and time measurement with OpenCV

## 目標
* 如何訪問所有pixel
* OpenCV矩陣儲存方式
* 如何衡量算法的性能
* 甚麼是查詢表?如何用?

## 預處理

我們要考慮減少色彩的方法 不然計算量會爆大
有時候只有少少的顏色也能夠得到同樣的效果 計算量呢相對少很多

一般圖會有256個灰度級
我們可能要讓0~127給0,其他給1來加速運算
那我們考慮要此算法:
\\(I_{new} = (\frac{I_{old}}{D}) * D\\) , D是 divide_width
計算lookuptable 之後只要讀取值O(1)
```cpp
	int divideWith = 0; // convert our input string to number - C++ style
    stringstream s;
    s << argv[2];
    s >> divideWith;
    if (!s || !divideWith)
    {
        cout << "Invalid number entered for dividing. " << endl;
        return -1;
    }
    uchar table[256];
    for (int i = 0; i < 256; ++i)
       table[i] = (uchar)(divideWith * (i/divideWith));
```

### 測量時間的方法
```cpp
double t = (double)getTickCount();
// do something ...
t = ((double)getTickCount() - t)/getTickFrequency();
cout << "Times passed in seconds: " << t << endl;
```
## Mat內存方式
Gray_scale
![](https://docs.opencv.org/3.4.1/tutorial_how_matrix_stored_1.png)
RGB_scale
![](https://docs.opencv.org/3.4.1/tutorial_how_matrix_stored_2.png)

### 高效的方法
```cpp
Mat& ScanImageAndReduceC(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    int channels = I.channels();
    int nRows = I.rows;
    int nCols = I.cols * channels;
    if (I.isContinuous())
    {
        nCols *= nRows;
        nRows = 1;
    }
    int i,j;
    uchar* p;
    for( i = 0; i < nRows; ++i)
    {
        p = I.ptr<uchar>(i);
        for ( j = 0; j < nCols; ++j)
        {
            p[j] = table[p[j]];
        }
    }
    return I;
}
```
當儲存是連續的時候可以用這個
```cpp
uchar* p = I.data;
for( unsigned int i =0; i < ncol*nrows; ++i)
    *p++ = table[*p];
```

### The iterator (safe) method
```cpp
Mat& ScanImageAndReduceIterator(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    const int channels = I.channels();
    switch(channels)
    {
    case 1:
        {
            MatIterator_<uchar> it, end;
            for( it = I.begin<uchar>(), end = I.end<uchar>(); it != end; ++it)
                *it = table[*it];
            break;
        }
    case 3:
        {
            MatIterator_<Vec3b> it, end;
            for( it = I.begin<Vec3b>(), end = I.end<Vec3b>(); it != end; ++it)
            {
                (*it)[0] = table[(*it)[0]];
                (*it)[1] = table[(*it)[1]];
                (*it)[2] = table[(*it)[2]];
            }
        }
    }
    return I;
}
```
### 核心函數 LUT
Opencv提供方法讓你修改 就不用窮舉所有pixel了
先建立查詢表
```cpp
Mat lookUpTable(1, 256, CV_8U);
    uchar* p = lookUpTable.ptr();
    for( int i = 0; i < 256; ++i)
        p[i] = table[i];
```
再call func
`LUT（I，lookUpTable，J）；//I 輸入 J 輸出`

速度最快

## Code

```cpp
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
void CreateLookupTable(Mat &table,uchar divide_width) {
	table.create(1, 256, CV_8UC1);
	uchar *p = table.data;
	for (int i = 0;i < 256;i++)
		p[i] = divide_width * (i / divide_width);

}
int main(){
	
	Mat img = imread("../images/test-1.png");

	Mat table;//lookup table
	CreateLookupTable(table, 10);//用級別10的

	Mat out1, out2;
	LUT(img, table, out1);
	CreateLookupTable(table, 100);//100

	LUT(img, table, out2);

	namedWindow("origin", WINDOW_AUTOSIZE);
	imshow("origin", img);
	namedWindow("D=10", WINDOW_AUTOSIZE);
	imshow("D=10", out1);
	namedWindow("D=100", WINDOW_AUTOSIZE);
	imshow("D=100", out2);

	waitKey(0);
}
```
原圖
![](https://i.imgur.com/1PRVa0F.png)
Q=10
![](https://i.imgur.com/CHFx7H5.png)
Q=100
![](https://i.imgur.com/2vySgNg.png)

RBG圖的變化
更改以下
```cpp
table.create(1, 256, CV_8UC1); -> table.create(1, 256, CV_8UC3);
```
原圖
![](https://i.imgur.com/JF9j7DW.jpg)
Q=10
![](https://i.imgur.com/WE3IaUR.png)
Q=100
![](https://i.imgur.com/i28Jy2v.png)