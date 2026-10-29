
## Implicit Representations of Geometry
- 满足一定关系的顶点集合

$$
f(x,y,z) = (2 - \sqrt{x^2 + y^2})^2 + z^2 -1
$$
## Explicit Surface - Sampling Is Easy
$$f(u,v) = ((2+cosu)cosv,(2+cosu)sinv,sinu)$$

### Distance Functions(Implicit)

### Fractals 自相似(递归)

# Explicit Representations in Graphics
1. triangle meshes
2. Bezier surfaces
3. subdivision surfaces
4. point clouds

### Point Clouds
- 无数点的集合
### Polygon Mesh
- 三角形面片构成

## Curves

### Bezier Curve
- quadratic Bezier(二次贝塞尔曲线)
![[Pasted image 20261020165419.png|465]]

### Cubic Bezier Curve - de Casteljau

![[Pasted image 20261020165720.png|479]]


### Piecewise Bezier Curves(逐段)
- 每四个控制点 定义一个贝塞尔线段


### Splines(样条)
### B-splines

### Mesh Operations: Geometry Processing
- Mesh subdivision
- Mesh simplification
- Mesh regularization

### Loop Subdivision
1. 增加三角形面数
2. 调整原本共点的坐标

![[Pasted image 20261020172820.png|483]]


### Catmull-Clark Subdivision(General Mesh)
- 奇异点（degree != 4）相邻链接数 != 4
- Quad Face
- Non-quad Face

### Mesh Simplification

![[Pasted image 20261020180142.png|495]]


### Edge collapsing（边坍缩）
- Quadric Error Metrics (二次误差度量)

## Shadow Mapping

- 如果点不在阴影里，但可以观测到这个点 说明 摄像机可以看到这个点，光源也可以看到这个点

### Pass1 Render from Light (从光源处观测)
从光源处 放置一个 虚拟相机 渲染一张深度图
![[Pasted image 20261020181556.png|345]]

### Pass 2A: Render from Eye(从观察者角度)
将相机（眼睛）能看到的点在投影回到 光源角度生成的深度贴图上，如果深度一致，说明这个点可见
![[Pasted image 20261020181708.png|365]]


### Pass 2B: Project to light

相机（眼睛）看到的深度和光源看到的深度不一致时候，说明这个点在阴影中
![[Pasted image 20261020182039.png|375]]



## Light Rays(光线追踪)

1. 光线沿直线传播
2. 光线之间不会发生碰撞
3. 光线从光源发出 reciprocity（可逆性）

## Ray Casting - Generating Eye Rays（从眼、摄像机发出光线）

1. 从摄像机发射出一条光线，相交的第一个点。再从这个点看向光源，如果这个点可以看到光源那就说明这个点被光源照射，反之这个点在阴影里
![[Pasted image 20261022120432.png|558]]


### Recursive(Whitted-Style) Ray Tacing

![[Pasted image 20261022121115.png|573]]


## Ray-Surface Intersection

1. 光线 = 起点 + 方向 r(t) = o + td o(起点) d(方向) t(时间)
2. 和球相交 $$(p - c)^2 - R^2 =0$$
![[Pasted image 20261022121639.png|555]]

任何在球上的点p 符合光线在 t 时间上一个点
$$
(o + td - c)^2 - R^2 = 0
$$

### Ray Intersection With Triangle Mesh
1. 可见性，阴影，光线
2. 判断内外

![[Pasted image 20261022122743.png|423]]

1. 光线和平面是否相交
2. 交点是否在三角形内
平面 = 一根法线 + 某个点
$$
\begin{eqnarray}
{(p - p')}{N} = 0 \\
ax + by + cz + d = 0
\end{eqnarray}
$$

###  Möller–Trumbore Algorithm（射线与三角形相交）

## 1. 算法原理

Möller–Trumbore 算法用于判断射线是否与三角形相交，并计算交点。
**核心思想：通过射线方程和三角形的重心坐标方程求解三个未知数 $t,u,v$。**
#### 射线方程

$$
R(t)=O+tD,\quad t\geq0
$$

- $O$：射线起点
- $D$：射线方向
- $t$：射线参数。如果 $D$ 是单位向量，$t$ 就是到交点的距离

#### 三角形方程

假设三角形顶点为 $V_0,V_1,V_2$。

定义两条边：

$$
E_1=V_1-V_0
$$

$$
E_2=V_2-V_0
$$

三角形所在平面上的点可以表示为：

$$
P(u,v)=V_0+uE_1+vE_2
$$

其中 $u,v$ 是重心坐标参数。

### 相交条件

当射线与三角形相交时：

$$
\boxed{O+tD=V_0+uE_1+vE_2}
$$

求解三个未知数 $t,u,v$。

满足以下条件，说明交点位于射线正方向和三角形内部（含边界）：

$$
\boxed{t\geq0,\quad u\geq0,\quad v\geq0,\quad u+v\leq1}
$$

#### 2. 算法计算过程

##### Step 1：计算三角形边向量

$$
E_1=V_1-V_0,\quad E_2=V_2-V_0
$$

##### Step 2：判断射线是否与三角形平面平行

$$
H=D\times E_2
$$

$$
a=E_1\cdot H
$$

如果 $|a|<\varepsilon$，通常判定为不相交（射线与平面平行或三角形退化）。

##### Step 3：计算重心坐标 u

$$
f=\frac{1}{a},\quad S=O-V_0
$$

$$
\boxed{u=f(S\cdot H)}
$$

如果 $u<0$ 或 $u>1$，交点在三角形外。

##### Step 4：计算重心坐标 v

$$
Q=S\times E_1
$$

$$
\boxed{v=f(D\cdot Q)}
$$

如果 $v<0$ 或 $u+v>1$，交点在三角形外。

##### Step 5：计算射线参数 t

$$
\boxed{t=f(E_2\cdot Q)}
$$

如果 $t<0$，交点位于射线起点后方。

否则交点为：

$$
\boxed{P=O+tD}
$$

#### 3. 算法总结

- **输入**：射线起点 $O$、方向 $D$、三角形顶点 $V_0,V_1,V_2$
- **输出**：是否相交、交点位置、射线参数 $t$、重心坐标 $u,v$
- **核心运算**：向量叉积与点积
- **时间复杂度**：$O(1)$
- **特点**：不需要预先计算平面方程或三角形法线

> 核心理解：射线方程与三角形的重心坐标方程表示同一个交点，通过求解 $t,u,v$，判断射线是否命中三角形。



## Uniform Spatial Partitions(Grids)
使用AABB盒加速光线求交点计算
1 找到包围盒
2 创建网格
3 存储每个盒子在Grid网格中位置

### Spatial Partitions
1. Oct-Tree 八叉树
2. KD-Tree 
3. BSP-Tree

### Bounding Volume Hierarchy(BVH)


### Basic radiometry(辐射度量学)
- Radiant flux,intensity,irradiance,radiance

#### Radiant flux
Radiant energy 测量每个单位内能量
Q[J = Joule]
单位时间的能量
dQ/dt [W = watt] [lm = lumen]
#### Radiant Intensity
每一个单位角上的能量
power per unit solid angle
$$
[\frac{W}{sr}][\frac{lm}{sr}=cd=candela]
$$
在2D中 角度定义 弧长/半径
$$
\theta = \frac{l}{r}
$$
在3D中 立体角定义 球表面的面积/半径平方
$$\Omega = \frac{A}{r^2}$$

Differential Solid Angles微分立体角
$$
dA = (rd\theta)(rsin\theta d\phi) = r^2sin\theta d\theta d\phi
$$
$$
d\omega = \frac{dA}{r^2}=sin\theta d\theta d\phi
$$
#### irradiance
每个单位面积上对应的能量 power per unit area
Falloff 

#### radiance
光的传播方向
per unit solid angle,per projected unit area
$$
L(p,\omega) = \frac{d^2\phi(p,\omega)}{d\omega dAcos\theta}
$$
### Bidirectional Reflectance Distribution Function

![[Pasted image 20261023123650.png]]

### Light Transport & Global Illumination

### Monte Carlo Integration
在一个函数上 随机取点相加，然后求平均数
 $$
 \int f(x) dx = \frac{1}{N}\sum_{i=1}^{n} \frac{f(X_i)}{p(X_i)}
 $$
 1. 更多的样本,更少的误差
 2. 只能在x上采样

### Path Tracing


![[Pasted image 20261023151122.png]]

1. 从概念上 像素点出发 现实情况是 从发光源出发

$$
L_o(p,\omega_o) = \frac{1}{N}\sum_{i=1}^{n}\frac{L_i(p,\omega_i)f_r({p,\omega_i,\omega_o})(n,\omega_i)}{p(\omega_i)}
$$
1. 在任何一个点 往半球上发射N个光线
2. 如果命中一个光源，可以获取L_i,根据入射 出射方向获取BRDF,cosine也可以得到
3. 如果命中不是光源，直接将命中物体的光照计算结果当做 L_i计算
4. 累积结果/采样次数

