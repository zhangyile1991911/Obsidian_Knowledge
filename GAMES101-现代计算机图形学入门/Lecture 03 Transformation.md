
# 二维变化
## 缩放（Scaling）
$$
\begin{eqnarray}
\begin{bmatrix} 
x' \\ 
y'
\end{bmatrix}
= 
\begin{bmatrix} 
s & 0 \\ 
0 & s 
\end{bmatrix} 
\begin{bmatrix} 
x \\ 
y 
\end{bmatrix} 
\end{eqnarray}
$$
![[Pasted image 20261015153810.png]]

$$
\begin{eqnarray}
\begin{bmatrix} 
x' \\ 
y'
\end{bmatrix}
=
\begin{bmatrix} 
s_x & 0\\ 
0 & s_y 
\end{bmatrix}
\begin{bmatrix} 
x \\ 
y
\end{bmatrix}
\end{eqnarray}
$$

![[Pasted image 20261015153857.png]]

$$
\begin{eqnarray}
\begin{bmatrix} 
x' \\ 
y'
\end{bmatrix}
=
\begin{bmatrix} 
-1 & 0\\ 
0 & 1 
\end{bmatrix}
\begin{bmatrix} 
x \\ 
y
\end{bmatrix}
\end{eqnarray}
$$


![[Pasted image 20261015154136.png]]


## 切变（Shear）
$$
\begin{eqnarray}
\begin{bmatrix} 
x' \\ 
y'
\end{bmatrix}
=
\begin{bmatrix} 
1 & a\\ 
0 & 1 
\end{bmatrix}
\begin{bmatrix} 
x \\ 
y
\end{bmatrix}
\end{eqnarray}
$$
![[Pasted image 20261015155407.png]]


## 旋转（Rotate）

$$
\begin{eqnarray}
R
=
\begin{bmatrix} 
cos\theta & -sin\theta\\ 
sin\theta & cos\theta 
\end{bmatrix}
\end{eqnarray}
$$
## 平移（Translation）

$$
\begin{eqnarray}

\begin{bmatrix} 
x' \\ 
y' \\ 
w' \\ 
\end{bmatrix}
=
\begin{bmatrix} 
1 & 0 & t_x \\ 
0 & 1 & t_y \\ 
0 & 0 & 1 \\ 
\end{bmatrix}
\begin{bmatrix} 
x \\ 
y \\ 
w \\ 
\end{bmatrix}
=
\begin{bmatrix} 
x + t_x \\ 
y + t_y\\ 
1 \\ 
\end{bmatrix}
\end{eqnarray}
$$
### 先平移后旋转
$$
\begin{eqnarray}
T_{1,0} R_{45}
\begin{bmatrix} 
x \\ 
y \\ 
1 \\ 
\end{bmatrix}
=
\begin{bmatrix} 
1 & 0 & 0 \\ 
0 & 1 & 0 \\ 
0 & 0 & 1 \\ 
\end{bmatrix}
\begin{bmatrix} 
cos45 & -sin45 & 0 \\ 
sin45 & cos45 & 0 \\ 
0 & 0 & 1 \\ 
\end{bmatrix}
=
\begin{bmatrix} 
x \\ 
y \\ 
1 \\ 
\end{bmatrix}
\end{eqnarray}
$$

# 三维空间变换

$$
\begin{eqnarray}
\begin{bmatrix} 
x' \\ 
y' \\
z' \\
1 \\ 
\end{bmatrix}
=
\begin{bmatrix} 
a & b & c & t_x \\ 
d & e & f & t_y \\ 
g & h & i & t_z \\ 
0 & 0 & 0 & 1 \\ 
\end{bmatrix}
\begin{bmatrix} 
x \\ 
y \\ 
z \\ 
1 \\ 
\end{bmatrix}
\end{eqnarray}
$$
