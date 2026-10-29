
Z-Buffer
- 存储最小值 （越靠近相机越深 Z值越小，接近0）
- 需要额外buffer存储深度信息
	- frame buffer 存储颜色
	- depth buffer 存储深度

![[Pasted image 20261019161433.png]]


## Shading

### Blinn-Phong Reflectance Model
- Specular highlights 高光
- Diffuse reflection 漫反射
- Ambient lighting 环境光(间接光照)
![[Pasted image 20261019173545.png]]


v 点到观察者的方向
l 点到光的方向
![[Pasted image 20261019173822.png|276]]

### Diffuse Reflection
表面接收到的光的量
$$cos\theta = l * n$$
#### Lambertian(Diffuse) 经验模型
$$L_d = K_d(\frac{I}{r^2})max(0,n * l)$$

## Specular Term(Blinn-Phong)

- v（观察方向）和r（反射方向）接近时候出现高光

$$
\begin{eqnarray}
h = bisector(v,l) = \frac{v + l}{\|v + l\|} \\
L_s = K_s(\frac{I}{r^2})max(0,cos\alpha)^p \\
=K_s(\frac{I}{r^2})max(0,n * h)^p
\end{eqnarray}
$$
## Ambient Term (环境光)

$$
\begin{eqnarray}
L_a = K_a * I_a \\
L_a = reflected ambient light 环境光 \\
K_a = ambient coefficient 
\end{eqnarray}
$$
![[Pasted image 20261019180731.png]]
$$
\begin{eqnarray}
L = L_a + L_d + L_s \\
= K_aI_a + K_d(\frac{I}{r^2})max(0,nl) + K_s(\frac{I}{r^2})max(0,nh)^p
\end{eqnarray}
$$

## Shading Frequencies

1. Flat Shading 逐三角形面片
	1. 面片够多的情况下也能达到平滑
2. Gouraud Shading  逐顶点
	1. 顶点够多也能达到平滑
3. Phong Shading 逐像素 

![[Pasted image 20261019181016.png]]


## Graphics (Real-time Rendering) Pipeline

![[Pasted image 20261019181726.png|634]]


## Barycentric Coordinates（重心坐标）

$$
\begin{eqnarray}
(x,y) = \alpha A+ \beta B + \gamma C \\
\alpha + \beta + \gamma = 1
\end{eqnarray}
$$
基于重心，三角形内部任意一个点做线性插值
重心在投影过程中会发生变化


### 纹理放大

Pixel 生成画面上的一个像素
Texel 纹理上的一个像素
当纹理小于实际需要表现的尺寸时候进行以下处理
- Nearest 四舍五入
- Bilinear 双向线性插值
- Bicubic

![[Pasted image 20261020124002.png|578]]

Bilinear Interpolation
查找邻近的四个像素，进行插值处理

![[Pasted image 20261020125210.png|637]]

$$lerp(x,v_0,v_1) = v_0 + x(v_1 - v_0)$$
1. 水平插值
2. 垂直插值
3. 水平和垂直中间插值

## 纹理过大

### Point Sampling Texture - Problem

![[Pasted image 20261020125608.png|406]]

在靠近摄像机的地方，一个pixel覆盖一小部分texel。
随着距离越来越远，一个pixel覆盖一大片texel。

![[Pasted image 20261020125635.png|542]]

### Antialiasing - Supersampling

#### Mipmap
- 范围查询（快，近似，<mark style="background: #FF5582A6;">正方形</mark>）
- 存储量是原图片的三分之一
![[Pasted image 20261020130606.png|602]]


### Trilinear Interpolation（三线性插值）
1. 在D层做一次插值（Bilinear Result）
2. 在D+1层做一次插值（Bilinear Result）
3. 基于前两次插值在进行一次插值

## Anisotropic Filtering （各向异性过滤）

1. 不同的长宽比做预计算
2. 横向和纵向分别做压缩
3. 对<mark style="background: #FF5582A6;">矩形区域</mark>做快速查询
4. 额外开销是原图的三倍
5. 不会影响性能，对显存要求高
![[Pasted image 20261020132340.png|417]]


## Environment Map

### Spherical Environment Map
- 扭曲问题，展开后维度更高的地方会显示的更小
### Cube Map
### Bump Mapping
- 存储高度变化，改变法线方向(并没有真正移动原本的顶点)
- 在边缘处明显瑕疵
 $$
 \begin{eqnarray}
 \text{使用凹凸贴图定义切线，通过切线计算出法线} \\
 dp = c * [h(p+1) - h(p)] \\
 \text{法线和切线垂直，逆时针旋转90°得到法线} \\
 n(p) = (-dp,1)
 \end{eqnarray}
 $$
$$
\begin{eqnarray}
\frac{dp}{du} = c1[h(u+1) - h(u)] \\
\frac{dp}{dv} = c2[h(v+1) - h(v)] \\
n = (-dp/du,-dp/dv,1)
\end{eqnarray}
$$

### Displacement mapping
- 实际会对原本顶点位置发生改变

![[Pasted image 20261020153222.png|619]]


### 3D Procedural Noise + Solid Modeling

### 3D Textures and Volume Rendering

