---
layout: single
comments: true
author_profile: true
excerpt: "INTRODUCTION-to-KALMAN-FILTER"
header:
  overlay_image: https://ddnotbook.files.wordpress.com/2023/11/image-7.png?w=1024
  teaser: https://ddnotbook.files.wordpress.com/2023/11/image-7.png?w=1024
classes:
  - landing
  - dark-theme
title:  "INTRODUCTION-to-KALMAN-FILTER"
date:   2023-11-30 14:00:00 +0800
categories: KALMANFILTER
---

# 2023-11-30-INTRODUCTION-to-KALMAN-FILTER.md

## 卡爾曼濾波器

[卡爾曼濾波器](https://zh.wikipedia.org/zh-tw/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2)是由魯道夫·卡爾曼([匈牙利語](https://zh.wikipedia.org/wiki/%E5%8C%88%E7%89%99%E5%88%A9%E8%AA%9E)：Rudolf Emil Kálmán)所提出，被廣泛運用在控制系統、導航系統、航空電子系統、太空飛行器的電腦中。將帶有雜訊的訊號中提取出正確有用的訊號。事實上首次實踐這種濾波器的為[斯坦利·施密特](https://zh.wikipedia.org/w/index.php?title=%E6%96%AF%E5%9D%A6%E5%88%A9%C2%B7%E6%96%BD%E5%AF%86%E7%89%B9&action=edit&redlink=1)（Stanley Schmidt）。卡爾曼在訪問NASA研究中心的時候提出了這種濾波器可以解決阿波羅計畫中的軌道預測問題。

## 如準確判斷車子會到哪?

如果我們開車以一個固定速度，要到一個地方，如何知道我們開多久後會到哪個地點呢?

![https://ddnotbook.files.wordpress.com/2023/11/image-9.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-9.png?w=1024)

1. 在車子中可以運用輪軸計算速度，這樣就可以知道多久後可以到哪了?
2. 也可以用先進的GPS知道我們移動的距離依照這個速度我們多久後會抵達?

但車子內部量測的車速會準嗎?還是GPS的量測比較準。到底要相信哪個感測器量測的訊號?

卡爾曼在1960年訪問美國NASA艾姆斯研究中心時，提出如果可以使用一種濾波器(Kalman Filter)就可以解決阿波羅計畫中的軌道預測問題：如何準確的預測阿波羅飛船在太空中的位置。

![https://ddnotbook.files.wordpress.com/2023/11/image-7.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-7.png?w=1024)

[NASA60年前的太空計畫](https://www.nasa.gov/history/60-years-ago-nasa-decides-on-lunar-orbit-rendezvous-for-moon-landing/)

要知道訊號中有需多雜訊，例如電磁干擾、機電震動、傳輸訊號雜訊等等非我們想要知道的準確訊號。如何在這些帶有雜訊的訊號中取出更貼近真實的訊號，這就是卡爾曼濾波器厲害的地方。用兩個不是很準確的訊號去預估出一個更準確的訊號，聽起來真的很不可思議。

![https://ddnotbook.files.wordpress.com/2023/11/image-11.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-11.png?w=1024)

回到車子的問題，假設衛星訊號中帶有真實訊號x以及雜訊noise_x

![https://ddnotbook.files.wordpress.com/2023/11/image-17.png?w=250](https://ddnotbook.files.wordpress.com/2023/11/image-17.png?w=250)

而車子中的感測器也有一可以量測車速的感測器，可以讀取到真實訊號加上速度雜訊的訊號

![https://ddnotbook.files.wordpress.com/2023/11/image-18.png?w=252](https://ddnotbook.files.wordpress.com/2023/11/image-18.png?w=252)

因為雜訊以及各種因素的影響，例如空氣阻力造成預估的不準確，所以以速度量測的訊號所預估的車子會到藍色X的位置，而衛星訊號會預估到橙色X位置。但是實際的狀況應該為紅色位置。如何使用卡爾曼濾波器預測到車子會抵達紅色位置呢?

![https://ddnotbook.files.wordpress.com/2023/11/image-12.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-12.png?w=1024)

我們先假設一台車子的實際模型

![https://ddnotbook.files.wordpress.com/2023/11/image-20.png?w=1023](https://ddnotbook.files.wordpress.com/2023/11/image-20.png?w=1023)

我們給他速度u，他會產出一個位置給我們y，如果我們持續給予u速度k次他就會抵達yk的地方。例如以每秒1m的速度u1前進，經過dt =1 sec過後就會抵達1m的距離

![https://ddnotbook.files.wordpress.com/2023/11/image-21.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-21.png?w=1024)

但是實際模型中可能會有很多干擾或是雜訊vk、wk在yk的訊號中。所以我們在下方虛擬一個模型(可以是推測的數學模型**Estimate model**，或是另外的感測器訊號)作為一個觀察器**observer**與實際的訊號做比較。並且把比較的結果再帶回到**Estimate model**。這個不斷把實際訊號y_k與推測出的訊號y ̂_k融合計算後疊代，到下一次計算的**Estimate model**模型的過程就是卡爾曼濾波器的基本原理。

![https://ddnotbook.files.wordpress.com/2023/11/image-22.png?w=1024](https://ddnotbook.files.wordpress.com/2023/11/image-22.png?w=1024)

註：上圖為一個控制理論的方塊圖(block diagram )，其實可以用數學推導的方式證明他的誤差收斂。

## 後記

為何會去深入瞭解這個濾波器，其實也是一個很久遠的原因。當時在讀書的時候有一個倒單擺系統的計畫。我們花了很久的時間卻也解決不了。無法完成的可能因素如電機響應、控制器設計、系統模型假設、訊號處裡等等東西。當時學得還不夠多，直到最後還無法順利成功。也許是想要挑戰當時的問題，或是彌補當時的遺憾，在離開學校不知道多久的現在，再次研究這些問題。而卡曼濾波器卻是我在研究這個倒單擺系統的關鍵之一。也可能是這樣才會寫這篇，之後可能還有幾篇文章會以這個濾波器探討。
