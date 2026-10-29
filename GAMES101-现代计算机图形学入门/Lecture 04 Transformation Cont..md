
# 观测(Viewing)

## 缩放（Scale）
$$
\begin{eqnarray}
S(s_x,s_y,s_z)
=
\begin{bmatrix} 
s_x & 0 & 0 & 0 \\
0 & s_y & 0 & 0 \\
0 & 0 & s_z & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix} 
\end{eqnarray} 
$$
## 平移(Translation)

$$
\begin{eqnarray}
S(t_x,t_y,t_z)
=
\begin{bmatrix} 
1 & 0 & 0 & t_x \\
0 & 1 & 0 & t_y \\
0 & 0 & 1 & t_z \\
0 & 0 & 0 & 1 \\
\end{bmatrix} 
\end{eqnarray} 
$$

## 旋转(Rotation)

$$
\begin{eqnarray}
R_x
=
\begin{bmatrix} 
1 & 0 & 0 & 0 \\
0 & cos\alpha & -sin\alpha & 0 \\
0 & sin\alpha & cos\alpha & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix} \\
\\
R_y
=
\begin{bmatrix} 
cos\alpha & 0 & sin\alpha & 0 \\
0 & 1 & 0 & 0 \\
-sin\alpha & 0 & cos\alpha &  0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix} \\
\\
R_z
=
\begin{bmatrix} 
cos\alpha & -sin\alpha & 0 & 0 \\
sin\alpha & cos\alpha & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix} 
\end{eqnarray} 
$$

## [[Rodrigues Rotation Formula（罗德里格斯旋转公式）]]

$$
\mathbf{v}'=
\mathbf{v}\cos\theta
+(\mathbf{k}\times\mathbf{v})\sin\theta
+\mathbf{k}(\mathbf{k}\cdot\mathbf{v})(1-\cos\theta)
$$
# View/Camera Transformation

## 定义相机
- 相机位置
- 看向方向
- 向上方向

$$M_{view} = R_{view}T_{view}$$
1. 将相机移动到原点
	$$
	\begin{eqnarray}
	T_{view}
	=
	\begin{bmatrix} 
	1 & 0 & 0 & -x_e \\
	0 & 1 & 0 & -y_e \\
	0 & 0 & 1 & -z_e \\
	0 & 0 & 0 & 1 \\
	\end{bmatrix} \\
	\end{eqnarray} 
	$$
2. 旋转轴
	$$
		\begin{eqnarray}
		R^{-1}_{view}
		=
		\begin{bmatrix} 
		x_{\vec{g}x\vec{t}} & x_t & x_g & 0 \\
		y_{\vec{g}x\vec{t}} & y_t & y_g & 0 \\
		z_{\vec{g}x\vec{t}} & z_t & z_g & 0 \\
		0 & 0 & 0 & 1 \\
		\end{bmatrix} \\
		\end{eqnarray} 
	$$

## 投影

- 正交投影
$$
\begin{eqnarray}
M_{ortho}
=
	\begin{bmatrix} 
	\frac{2}{right-left} & 0 & 0 & 0 \\
	0 & \frac{2}{top-bottom} & 0 & 0 \\
	0 & 0 & \frac{2}{near-far} & 0 \\
	0 & 0 & 0 & 1 \\
	\end{bmatrix}
	\begin{bmatrix} 
	1 & 0 & 0 & -\frac{r+l}{2} \\
	0 & 1 & 0 & -\frac{t+b}{2} \\
	0 & 0 & 1 & -\frac{n+f}{2} \\
	0 & 0 & 0 & 1 \\
	\end{bmatrix} \\
\end{eqnarray} 
$$
- 透视投影
$$
\begin{eqnarray}
M_{ortho}
=
	\begin{bmatrix} 
	\frac{2}{right-left} & 0 & 0 & 0 \\
	0 & \frac{2}{top-bottom} & 0 & 0 \\
	0 & 0 & \frac{2}{near-far} & 0 \\
	0 & 0 & 0 & 1 \\
	\end{bmatrix}
	\begin{bmatrix} 
	1 & 0 & 0 & -\frac{r+l}{2} \\
	0 & 1 & 0 & -\frac{t+b}{2} \\
	0 & 0 & 1 & -\frac{n+f}{2} \\
	0 & 0 & 0 & 1 \\
	\end{bmatrix} \\
\end{eqnarray} 
$$
1. 远平面上的点挤压到近平面上

$$y' = \frac{n}{z}y$$
$$x' = \frac{n}{z}x$$
![[Pasted image 20261016161515.png|583]]

2. 
 $$
 \begin{eqnarray}
 \begin{bmatrix}
 0 & 0 & A & B
 \end{bmatrix}
 \begin{bmatrix}
 x \\
 y \\
 n \\
 z
 \end{bmatrix}
 = 
 n^2 \\
 An + B = n^2 \\
 Af + B = f^2 \\
 A = n + f
 B = -nf
 \end{eqnarray}
 $$
 

