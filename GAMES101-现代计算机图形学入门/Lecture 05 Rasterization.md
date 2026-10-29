
将 [-1,1]^2
映射到[0,width] [0,height] 屏幕空间

$$
\begin{eqnarray}
M_{viewport}
=
\begin{bmatrix}
\frac{width}{2} & 0 & 0 & \frac{width}{2} \\
0 & \frac{height}{2} & 0 & \frac{height}{2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\end{eqnarray}
$$
1. 缩放x和y
2. 定义屏幕左下角为原点(0,0)，所以还要进行一次平移


 采样的问题
 1. Jaggies锯齿 空间采样
 2. Moire摩尔纹 空间采样
 3. Wagon Wheel effect 车轮效应 时间采样


## Pre-Filter 采样前先模糊来解决锯齿问题

![[Pasted image 20261019121447.png]]


![[Pasted image 20261019121514.png]]


## 在相通的采样频率下，对于最后一个频率无法通过采样点来还原原始的信号。

![[Pasted image 20261019122157.png]]


## 傅里叶变化(fourier) 将时域变换到频域

右图 中心是低频率区域，外部是高频率区域
![[Pasted image 20261019122537.png]]

屏蔽低频信号，只保留高频信号，然后在逆傅里叶变换回去，剩下左图的轮廓 【高通滤波器】

![[Pasted image 20261019122920.png]]


 屏蔽高频信息，只保留低频信息，在通过逆傅里叶变换回去，达到模糊效果 【低通滤波器】
![[Pasted image 20261019123209.png]]

只留下中间 中间频率信息
![[Pasted image 20261019123340.png]]


## 卷积

![[Pasted image 20261019123535.png]]


 ## MSAA Multiply Sample Antialiasing
 将一个像素继续细分成更小的单位，然后在判断每个单位是否在三角形内部，在取平均值（覆盖率）
 代价 计算量指数级增加
 ![[Pasted image 20261019125452.png]]


![[Pasted image 20261019125648.png]]

FXAA (Fast Approximate AA) 得到一幅有锯齿的图片，找到锯齿点做模糊
TAA (Temporal AA) 复用上一帧的数据


