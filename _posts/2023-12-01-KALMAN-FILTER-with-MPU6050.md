---
layout: single
comments: true
author_profile: true
excerpt: "KALMAN-FILTER-with-MPU6050"
header:
  overlay_image: https://ddnotbook.files.wordpress.com/2023/12/image.png?w=1024
  teaser: https://ddnotbook.files.wordpress.com/2023/12/image.png?w=1024
classes:
  - landing
  - dark-theme
title:  "KALMAN-FILTER-with-MPU6050"
date:   2023-12-01 14:00:00 +0800
categories: KALMANFILTER
---
# 2023-12-01-KALMAN-FILTER-with-MPU6050.md

![https://ddnotbook.files.wordpress.com/2023/12/image.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image.png?w=1024)

## MPU6050

MPU6050 是一個可以量測加速度與角速度的物理量IC晶片，這篇就是要說明如何利用這個晶片配合卡爾曼濾波器算出正確的角度。

![https://ddnotbook.files.wordpress.com/2023/12/image-1.png?w=751](https://ddnotbook.files.wordpress.com/2023/12/image-1.png?w=751)

MPU6050會給予三軸加速度g以及三軸角速度ω ，假設我們要算出晶片的Y軸向翻轉角度有兩種方法。

- 利用x軸與z因為重力所產生的加速度gx與gz，計算arctan得θ

![https://ddnotbook.files.wordpress.com/2023/12/image-2.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-2.png?w=1024)

- 角速度*ω*以及時間*dt* 計算出，例如角速度為每秒0.1，*ω* =0.1/sec。我們計算1秒後就可以得到0.1度。也可以用積分表達如下：
    
    。
    

![https://ddnotbook.files.wordpress.com/2023/12/image-35.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-35.png?w=1024)

我們再將[上一篇文章](https://wordpress.com/post/ddnotbook.wordpress.com/86)中的卡爾曼濾波器的方塊圖來做濾波器，想要得到較好的θ。因為MPU6050會給我們由上述兩種方法得到的θ

![https://ddnotbook.files.wordpress.com/2023/12/image-4.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-4.png?w=1024)

## Step 1 . 先預估角度*angle*與速度偏移量*Q_bias*

![https://ddnotbook.files.wordpress.com/2023/12/image-5.png?w=741](https://ddnotbook.files.wordpress.com/2023/12/image-5.png?w=741)

![https://ddnotbook.files.wordpress.com/2023/12/image-6.png?w=1010](https://ddnotbook.files.wordpress.com/2023/12/image-6.png?w=1010)

其中我們定義

![https://ddnotbook.files.wordpress.com/2023/12/image-7.png?w=820](https://ddnotbook.files.wordpress.com/2023/12/image-7.png?w=820)

![https://ddnotbook.files.wordpress.com/2023/12/image-8.png?w=464](https://ddnotbook.files.wordpress.com/2023/12/image-8.png?w=464)

## Step 2 . 計算出變異數矩陣

![https://ddnotbook.files.wordpress.com/2023/12/image-9.png?w=571](https://ddnotbook.files.wordpress.com/2023/12/image-9.png?w=571)

![https://ddnotbook.files.wordpress.com/2023/12/image-10.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-10.png?w=1024)

其中Qk-1矩陣為雜變異數矩陣。如果要確實了解，需要研究一下變異數或是協方差的相關性，至於Q矩陣中為何要有一個*dt* ？這是我目前實驗測試比較好的結果，日後可以繼續研究這一項的必要性，理論上不用。

## Step 3 . 計算卡爾曼增益Optimal Gain K

![https://ddnotbook.files.wordpress.com/2023/12/image-12.png?w=416](https://ddnotbook.files.wordpress.com/2023/12/image-12.png?w=416)

![https://ddnotbook.files.wordpress.com/2023/12/image-13.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-13.png?w=1024)

其中

![https://ddnotbook.files.wordpress.com/2023/12/image-14.png?w=394](https://ddnotbook.files.wordpress.com/2023/12/image-14.png?w=394)

其中Rk越大則Kg就會越小，相反Rk越小Kg就會越大，在下一個步驟中會看到他影響預測值與實際值

## Step 4 . 修正預估角度與速度偏移量

![https://ddnotbook.files.wordpress.com/2023/12/image-15.png?w=541](https://ddnotbook.files.wordpress.com/2023/12/image-15.png?w=541)

![https://ddnotbook.files.wordpress.com/2023/12/image-17.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-17.png?w=1024)

其中

![https://ddnotbook.files.wordpress.com/2023/12/image-18.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-18.png?w=1024)

將上述兩個角度差異乘上Kg就再加上原本預估角度*angle*與速度偏移量*Q_bias*成為下一次step 1中疊代用的輸入

![https://ddnotbook.files.wordpress.com/2023/12/image-21.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-21.png?w=1024)

其中這個*anglenew*  就是我們由卡爾曼濾波器所濾波出來的角度。這裡可以看到當Rk越小則Kg越接近1，這樣實際值yk 與預估值的y ̌k差異就影響越大，說明我們比較相信實際量測值。Rk越大則Kg越接近0，這樣實際值yk 與預估值的y ̌k差異就影響越小，也就是卡曼濾波出來的角度越相信預估值。

## Step 5 . 修正變異數矩陣

![https://ddnotbook.files.wordpress.com/2023/12/image-22.png?w=761](https://ddnotbook.files.wordpress.com/2023/12/image-22.png?w=761)

![https://ddnotbook.files.wordpress.com/2023/12/image-23.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-23.png?w=1024)

這裡可以看到當Rk(我們預先設定的數值)越小則Kg越接近1，說明我們比較相信量測值，這樣下一次預估函數中變異數矩陣就會變小，因為(1-Kg為接近1的值)，P矩陣就會越來越小

## 簡化為兩個步驟 Predict and Correction

可以將上述的五個公式/步驟，在**observer**也就是淺然色的區域簡化成右側的兩種狀態：

- Predict→計算出預估角度以及預估變異數矩陣
- Correction→修正出新的預估角度以及預估變異數矩陣

再將修正後的預估角度以及預估變異數矩陣代入到下次計算中，卡爾曼濾波器就是不斷在這兩種狀態下疊代計算濾波結果

![https://ddnotbook.files.wordpress.com/2023/12/image-24.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-24.png?w=1024)

兩種狀態的公式分別如下：

![https://ddnotbook.files.wordpress.com/2023/12/image-36.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-36.png?w=1024)

## 用C語言實作

以下左側是Kalman filter 的c語言程式，其實看起來非常簡單，用這幾行codes就可以造成完全不一樣的運算結果。以下右側是五個卡爾曼的方程式，包含了prediction 以及 correction /update 兩個步驟。(可以下載下來看比較清楚)

![https://ddnotbook.files.wordpress.com/2023/12/image-33.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-33.png?w=1024)

實際測試可以看到當濾波過後的訊號為藍色的訊號比較平順，訊號的標準差也比原來的橙色訊號小

![https://ddnotbook.files.wordpress.com/2023/12/image-34.png?w=1024](https://ddnotbook.files.wordpress.com/2023/12/image-34.png?w=1024)

## 結語：

在做倒單擺平衡車的時候，如果MPU6050沒有在程式迴圈中加入這個濾波器，會造成很大得震動，因訊號會因為控制器收到這些雜訊造成馬達依照錯誤的雜訊去作動。反而真正的角度訊號沒有辦法傳達給控制器，所以訊號處裡是一件很重要的事情。之後應該會將整個倒單擺平衡車的原理關鍵點再寫幾篇。

Reference

[MATLAB關於Kalman Filter的講解](https://www.youtube.com/playlist?list=PLn8PRpmsu08pzi6EMiYnR-076Mh-q3tWr)

[How a Kalman filter works, in pictures](https://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/)：有詳細解說共變異數

[【演算法】卡爾曼濾波器 Kalman Filter](https://jason-chen-1992.weebly.com/home/-kalman-filter)：python的游標小程式

[Linear Kalman Filter 筆記](https://blog.techbridge.cc/2021/04/11/kalman-filter-%E7%AD%86%E8%A8%98/)

[卡尔曼滤波算法思想理解 Kalman filter 第一篇](https://zhuanlan.zhihu.com/p/129370341)：知呼上的大大解釋

[卡尔曼滤波算法思想理解 Kalman filter 第二篇](https://zhuanlan.zhihu.com/p/129694693)：知呼上的大大解釋

[卡爾曼濾波 配合程式 講解](https://github.com/jasonblog/note/blob/master/osvr/qia_er_man_lv_bo_pei_he_cheng_shi_jiang_jie.md)

[CSDN卡曼濾波器實測紀錄](https://blog.csdn.net/m0_51220742/article/details/124385513)

[CSDN卡曼濾波器推導](https://blog.csdn.net/qqliuzhitong/article/details/118337049)

[kalman filter in arduino](https://www.cntofu.com/book/46/osvr/kalman_filter.md#google_vignette)

[MPU6050加速度、角速度的解算以及互补滤波使用](https://blog.csdn.net/qq_49979053/article/details/117395838)

[Arduino讀取MPU6050陀螺儀數據——卡爾曼濾波算法實踐](https://www.twblogs.net/a/5c9c07eebd9eee7523882add)
