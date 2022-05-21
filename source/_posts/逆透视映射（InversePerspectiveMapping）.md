---
title: 逆透视映射（InversePerspectiveMapping）
categories:
  - - 工作技能
    - 无人驾驶
date: 2021-09-23 21:38:19
math: true
tags:
  - 车道线检测
  - Inverse Perspective Mapping
  - 单应性矩阵
  - homography
---
## 前言
众所周知，真实世界的三维坐标在向相机的成像平面上投影时，会损失深度信息，使得整个过程不可逆。也就是说，已知图像中一个点的坐标$(u,v)$是无法求得该点在真实世界的三维坐标$(X\_w, Y\_w, Z\_w)$的。

那么，如果额外多给一项信息，即现在关心的所有三维世界的点都是同在一个平面(比如路面上的车道线)上呢？此时图像像素坐标系和世界坐标系之间的坐标转换就变成了可逆关系。

这种情形下，从像素坐标系坐标得到世界坐标系坐标的做法称作**逆透视映射**。

## 应用一：同一物理平面在两幅图像之间的转换
现有：
- 一个物理世界的平面，称作$P$
- 两个不同位置的相机，称作$A$和$B$
- $P$在$A$的成像平面上的投影，称作$P_A$，其上一点在$A$的像素坐标系上的坐标记作$(u\_A,v\_A)$
- $P$在$B$的成像平面上的投影，称作$P_B$，其上一点在$B$的像素坐标系上的坐标记作$(u\_B,v\_B)$

<div align=center><img title="" src="/img/article/homo1.jpg" width="129%" height="129%" align=center></div><br>

那么，$P\_A$和$P\_B$之间互为投影变换（单应性变换），这意味着存在一个3x3的可逆矩阵$\boldsymbol{H}$，使得：
$$
\left[
\begin{array}{c} 
 u_A      \\
 v_A      \\
 1      \\
\end{array}
\right]
 =
\boldsymbol{H}
\left[
\begin{array}{c} 
 u_B      \\
 v_B      \\
 1      \\
\end{array}
\right]
$$

矩阵$\boldsymbol{H}$中的9个元素的取值可以通过在两幅图像中取4组不共线的对应点解方程得到，也可以通过两个相机内外参矩阵之间的关系得到。

## 应用二：车道线物理坐标的求解
道路平面上的车道线上的点在世界坐标系中的坐标，与对应点在前视相机像素坐标系中的坐标之间，同样存在上述单应性变换关系。这一点可以从下面的公式推导过程中得到：

<div align=center><img title="" src="/img/article/homo2.png" width="80%" height="80%" align=center></div><br>

首先，图像像素坐标系和世界坐标系之间的坐标转换关系如下：
$$
\left[
\begin{array}{c} 
 u      \\
 v      \\
 1      \\
\end{array}
\right]
=
\frac{1}{Z_c}
\boldsymbol{K}
\boldsymbol{E}
\left[
\begin{array}{c} 
 X_w      \\
 Y_w      \\
 Z_w      \\
 1      \\
\end{array}
\right]
=
\frac{1}{Z_c}
\left[
\begin{array}{ccc} 
 \frac{f}{d_x} & 0 &fu_0   \\
 0 & \frac{f}{d_y} &fv_0   \\
 0 & 0 &1   \\
\end{array}
\right]
\left[
\begin{array}{cccc} 
 r_{11} & r_{12} &r_{12} &t_{1}      \\
 r_{21} & r_{22} &r_{23} &t_{2}      \\
 r_{31} & r_{32} &r_{33} &t_{3}      \\
\end{array}
\right]
\left[
\begin{array}{c} 
 X_w      \\
 Y_w      \\
 Z_w      \\
 1      \\
\end{array}
\right]
$$
在这个应用场景下有，$Z\_w=0$，因此有：
$$
\left[
\begin{array}{c} 
 u      \\
 v      \\
 1      \\
\end{array}
\right]
=
\frac{1}{r_{31}X_w+r_{32}Y_w+t_3}
\left[
\begin{array}{ccc} 
 \frac{f}{d_x} & 0 &fu_0   \\
 0 & \frac{f}{d_y} &fv_0   \\
 0 & 0 &1   \\
\end{array}
\right]
\left[
\begin{array}{cccc} 
 r_{11} & r_{12} &r_{12} &t_{1}      \\
 r_{21} & r_{22} &r_{23} &t_{2}      \\
 r_{31} & r_{32} &r_{33} &t_{3}      \\
\end{array}
\right]
\left[
\begin{array}{c} 
 X_w      \\
 Y_w      \\
 0      \\
 1      \\
\end{array}
\right]
$$
合并内外参矩阵得：
$$
\left[
\begin{array}{c} 
 u      \\
 v      \\
 1      \\
\end{array}
\right]
=
\frac{1}{r_{31}X_w+r_{32}Y_w+t_3}
\left[
\begin{array}{cccc} 
 h_{11} & h_{12} &h_{12} &h_{13}      \\
 h_{21} & h_{22} &h_{23} &h_{23}      \\
 r_{31} & r_{32} &r_{33} &t_{3}      \\
\end{array}
\right]
\left[
\begin{array}{c} 
 X_w      \\
 Y_w      \\
 0      \\
 1      \\
\end{array}
\right]
$$
注意到，$\boldsymbol{H}$矩阵中的第三列元素其实是无效的，因此有：
$$
\left[
\begin{array}{c} 
 u      \\
 v      \\
 1      \\
\end{array}
\right]
=
\frac{1}{r_{31}X_w+r_{32}Y_w+t_3}
\left[
\begin{array}{ccc} 
 h_{11} & h_{12}  &h_{13}      \\
 h_{21} & h_{22}  &h_{23}      \\
 r_{31} & r_{32}  &t_{3}      \\
\end{array}
\right]
\left[
\begin{array}{c} 
 X_w      \\
 Y_w      \\
 1      \\
\end{array}
\right]
$$
上式中，可以看到$\boldsymbol{H}$矩阵的前两列由内参矩阵和旋转矩阵相乘得到，第三列由内参矩阵和平移向量相乘得到。这也是apollo代码中，用conceptual方式求取透视投影矩阵所基于的原理。

这样，用$\boldsymbol{H}$的逆矩阵与像素坐标点相乘，然后归一化Z坐标，即可得到像素坐标点在世界坐标系中的XY坐标。

## 参考
1. https://towardsdatascience.com/a-hands-on-application-of-homography-ipm-18d9e47c152f
2. https://zhuanlan.zhihu.com/p/74597564
3. https://leijiezhang001.github.io/lane-det-from-BEV/
