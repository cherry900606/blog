---
title: "Opencv 自救筆記"
date: 2022-02-26T11:46:36+08:00
draft: false
toc: true
images:
tags: 
  - openv
  - note
---

## week1 安裝測試
```cpp
#include<opencv2/opencv.hpp>
using namespace cv;
int main()
{
	Mat img = imread("Lenna.jpg");
	imshow("Hello World!", img);
	waitKey();
}
```
使用 `imread()` 來讀入圖片，存到名為 img 的 Mat 基本資料型態。
`imshow()` 把 img 顯示成圖片，並且設定視窗標題成特定的名稱。
`waitKey()` 會等使用者按下鍵盤後結束，不然會一直等。

## week2 RGB to Gray

方法一
```cpp
#include<opencv2/opencv.hpp>
using namespace cv;
int main()
{
	Mat img = imread("Lenna.jpg", 0);
	imshow("Hello World!", img);
	waitKey();
}
```
`imread()` 的第二個參數可以設定顏色，其中 0 代表灰階。

方法二
```cpp
#include <opencv2/opencv.hpp>
using namespace cv;

int main()
{
	Mat img = imread("Lenna.jpg"), grayMat;
	cvtColor(img, grayMat, COLOR_BGR2GRAY);
	imshow("Hello World!", grayMat);
	waitKey();
}
```
用 `cvtColor()` 把原本的彩色 Mat 轉成 灰色 Mat。

方法三
```cpp
#include <opencv2/opencv.hpp>
using namespace cv;

int main()
{
	Mat img = imread("Lenna.jpg");
	Mat img_gray = Mat::zeros(img.rows, img.cols, CV_8UC1);

	for (int i = 0; i < img.rows; i++)
	{
		for (int j = 0; j < img.cols; j++)
		{
			int b = img.at<Vec3b>(i, j)[0];
			int g = img.at<Vec3b>(i, j)[1];
			int r = img.at<Vec3b>(i, j)[2];

			img_gray.at<uchar>(i, j) = 0.299 * r + 0.587 * g + 0.114 * b;
		}
	}

	imshow("RGB to gray", img_gray);
	waitKey();
}
```
對每個 pixel 取出 R, G 跟 B 三色數值後套用公式轉換成灰色。

因為灰階圖只有一個 channel，所以宣告時是用 C1，並且是 uchar。

## week3 模糊

* 平滑法

不用寫好的函式實作平滑法，filter 的大小為 5 * 5，周圍不用處理。
```cpp
for (int i = 2; i < src.rows - 2; i++)
	{
		for (int j = 2; j < src.cols - 2; j++)
		{
			int value = 0;
			for (int x = -2; x <= 2; x++)
			{
				for (int y = -2; y <= 2; y++)
				{
					value += src.at<uchar>(i + x, j + y);
				}
			}
			value /= 25;
			dst.at<uchar>(i, j) = value;
		}
	}
```
最外層的 i, j 迴圈代表對照片中的每個 piexl 都看過一次，除了受限於 filter 本身限制而不會看的上下左右兩排。

要得到當下 piexl 經過平滑法後會變成甚麼數值，則要把九宮格範圍的數值總和取平均。

* 中值濾波法
```cpp
for (int i = 2; i < src.rows - 2; i++)
	{
		for (int j = 2; j < src.cols - 2; j++)
		{
			int arr[25] = { 0 }, flag = 0;
			for (int x = -2; x <= 2; x++)
			{
				for (int y = -2; y <= 2; y++)
				{
					//cout << flag << endl;
					arr[flag] = src.at<uchar>(i + x, y + j);
					flag++;
				}
			}
			sort(arr, arr + 25);
			dst.at<uchar>(i, j) = arr[13];
		}
	}
```
把九宮格的數字存入 5 * 5 大小的 array，透過 sort 將數字大小排序，取中位數作為新值。

這個方法把原本圖片的雜訊濾掉了。

## 其他

原本對 opencv 怎麼使用感到相當苦惱，才想把過程中不懂的記下來。幾周後發覺摸得差不多了，剩下都是程式實作的部分，決定不再紀錄每周內容，僅作零散的筆記內容。

* 寫檔
```cpp
imread("檔名.檔案格式", src);
```