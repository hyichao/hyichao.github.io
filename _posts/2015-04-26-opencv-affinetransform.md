---
layout: post
title:  "OpenCV的仿射变换"
tags: [Computer Vision]
---


## 仿射变换
实现图像的仿射变换，关键是获得正确的仿射矩阵。
在OpenCV里面，有几个简单的API可以获得一个2*3的仿射矩阵，如例子里面的


### 关键函数
可以直接利用以下这个api

```
// opencv api  
void getRotationMatrix2D(center, angle, scale);  
```

传入的参数有三个:

* 第一个是仿射的中心，是一个cv::Point

* 第二个是变换的角度，如逆时针旋转45度，则是传入-45

* 第三个是变换的大小，大小不变是1

后面还有两三个参数，写入了默认值，可以不写，但是有一些场合可以使用。
例如仿射变换后矩阵会默认填入0到非图像区域，假如有需要，可以填入最后一个参数，让这个默认值变为其他RGB值


## Example

```
Mat affineTransform(Mat src, std::vector<float>& v)
{
    Mat rot_mat(2, 3, CV_32FC1);

    Mat dst = Mat::zeros(src.rows, src.cols, src.type());

    /** Rotating the image after Warp */
    /// Compute a rotation matrix with respect to the center of the image
    Point center = Point(dst.cols / 2, dst.rows / 2);
    double scale = 0.7;

    double angle = rand()%720;

    /// Get the rotation matrix with the specifications above
    rot_mat = getRotationMatrix2D(center, angle, scale);
    double a11 = rot_mat.at<double>(0,0);
    double a12 = rot_mat.at<double>(0,1);
    double a21 = rot_mat.at<double>(1,0);
    double a22 = rot_mat.at<double>(1,1);
    double b11 = rot_mat.at<double>(0,2);
    double b12 = rot_mat.at<double>(1,2);
    
    /// Rotate the warped image
    warpAffine(src, dst, rot_mat, dst.size());


    int ftx = v[0]*src.cols;;
    int fty = v[1]*src.rows;
    int bnx = v[2]*src.cols;
    int bny = v[3]*src.rows;

    //calculation of new label
    float cftx = (ftx*a11+fty*a12+b11)/src.cols;
    float cfty = (ftx*a21+fty*a22+b12)/src.rows;
    float cbnx = (bnx*a11+bny*a12+b11)/src.cols;
    float cbny = (bnx*a21+bny*a22+b12)/src.rows;

    //write vector
    v.erase(v.begin(),v.end());
    v.push_back(cftx);v.push_back(cfty);
    v.push_back(cbnx);v.push_back(cbny);

    return dst;
}

```

函数传入的vector里面是某些点坐标，利用rotationMatrix可以计算出仿射变换之后的点坐标。