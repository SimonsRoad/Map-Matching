# Map Matching using Particle Filter

### 简介
An example of indoor map matching using particle filter to calibrate the preliminaryly estimated PDR trajectories based on the indoor map information

利用粒子滤波实现地图匹配的样例，用于结合地图信息（检测是否穿墙）来校正PDR初始估算轨迹


![](http://ww1.sinaimg.cn/mw690/7b4b737bly1fprm2q3626j218g0lqq4l.jpg)

### 操作说明

+ 程序主函数为 testMain.m，可直接运行
+ 输入参数包括：初始PDR轨迹，默认为”trajBefore.mat“（m*2矩阵）；室内地图图片（.jpg，.png格式）及其文件名，默认为"F8配色1.jpg"。若轨迹进入房间，建议房门用扇形表示，即建议采用建筑CAD的截图图片
+ 有关参数调整：PDR轨迹的初始点位置（对应所需的坐标系），初始航向（通过初始轨迹旋转实现），轨迹与地图的比例换算。

``` 

......

%轨迹与旋转
%本案例中初始旋转角度为30度 (逆时针旋转)

RMtheta = 30*pi/180; 
x = x0.*cos(RMtheta) - y0.*sin(RMtheta);
y = x0.*sin(RMtheta) + y0.*cos(RMtheta);

.......

%给定PDR初始点位置
%按指定距离距离平移到最佳位置
%本案例为向右平移8.09，向下平移-27.22

x = x + 8.09;
y = y - 27.22;


......

% 计算真实轨迹与地图之间的比例尺，并将地图与墙壁向量进行比例转换
% 本案例中进行了若干手动调整的比例参数（如4、6、20），建议根据实际地图手动调整，待改进


MaxX=max(X');
MinY=min(X');

scaleX=(t1/round(MaxX(1))+6);
scaleY=-1*(t2/(floor(abs(MinY(2)))+20));
wall_new(2,:)=-wall(1,:)/(t2/(floor(abs(MinY(2)))+20));
wall_new(1,:)=wall(2,:)/(t1/round(MaxX(1))+6)+4;

Xmin=min(wall_new(1,:));  %地图界限
Xmax=max(wall_new(1,:));   
Ymin=min(wall_new(2,:));
Ymax=max(wall_new(2,:));

M2(:,1)=M2(:,1)/scaleX+4;
M2(:,3)=M2(:,3)/scaleX+4;
M2(:,2)=M2(:,2)/scaleY;
M2(:,4)=M2(:,4)/scaleY;

lines=wall_vectors';
lines(1,:)=lines(1,:)/scaleX+4;
lines(2,:)=lines(2,:)/scaleX+4;
lines(3,:)=lines(3,:)/scaleY;
lines(4,:)=lines(4,:)/scaleY;

``` 




### 基本原理

+ 将不同时刻的轨迹点表示为一堆粒子，利用粒子模拟轨迹走向，并结合地图信息（是否穿墙）来校正初始估算轨迹
+ 使用[LSD-OpenCV-Matlab](https://github.com/primetang/LSD-OpenCV-MATLAB) 提供的lsd函数提取地图图片内部的线段。lsd函数输入为地图图片文件名，输出结果为m*5矩阵，前四列分别表示线段两个端点的坐标，第五列表示线段的宽度（不常用）
![经LSD提取的直线如红色线段所示](http://ww1.sinaimg.cn/mw690/7b4b737bly1fprmqnvf6zj218g0lqjso.jpg)

+ 使用基于[Fast Line Segment Intersection](https://cn.mathworks.com/matlabcentral/fileexchange/27205-fast-line-segment-intersection)修改的lineSegmentIntersect函数判断轨迹是否穿墙，并计算穿墙点坐标。lineSegmentIntersect函数输入为两组待检测相交线段组的坐标，输出结果为结构体，包含交叉检验矩阵、交叉点坐标矩阵等。

![穿墙点检测](http://ww1.sinaimg.cn/mw690/7b4b737bly1fprmrlm0fyj218g0lqabn.jpg)

+ 使用最基本的粒子滤波算法实现地图匹配，粒子权重由穿墙检测函数确定，若粒子穿墙，置其权重为0，主函数内部提供了两种重采样方法，可选择其一。

![粒子滤波过程](http://ww1.sinaimg.cn/mw690/7b4b737bly1fprmu2p9daj218g0lq40s.jpg)

### 声明

+ 由于LSD直线检测工具存在误差，部分线段可能无法被检测出，因而会造成少量粒子仍然“穿墙”的现象
+ 本例中，粒子滤波仅考虑了航向，可进一步考虑步长等其他因素。最终的定位误差取决于多种因素：初始位置、初始航向、粒子数目及粒子噪声
+ 本工具基于[@Gefu Tang](https://github.com/primetang) 的[LSD-OpenCV-Matlab](https://github.com/primetang/LSD-OpenCV-MATLAB)和[U. Murat Erdem](https://cn.mathworks.com/matlabcentral/profile/authors/1752910-u-murat-erdem)的[Fast Line Segment Intersection](https://cn.mathworks.com/matlabcentral/fileexchange/27205-fast-line-segment-intersection)开发，其余还受到了实验室师兄师姐的帮助，特此感谢。
