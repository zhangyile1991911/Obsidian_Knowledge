
# Vectors

## 终点 - 起始
$$
\vec{a} = B - A
$$

## 单位向量
$$\|\vec{v}\| = \sqrt{x^2+y^2}$$
## 向量求和
$$
\vec{a} + \vec{b}
$$
$$
\begin{eqnarray}
\vec{a} * \vec{b} = \vec{b} * \vec{a} \\
\vec{a} * (\vec{b} + \vec{c}) = \vec{a} * \vec{b} + \vec{a} * \vec{c} \\
(k\vec{a}) * \vec{b} = \vec{a} * (k\vec{b}) = k(\vec{a} * \vec{b})
\end{eqnarray}
$$
## 点乘
$$
\begin{eqnarray}
\vec{a} * \vec{b} = \|\vec{a}\|\|\vec{b}\|cos\theta \\
cos\theta = \frac{\vec{a}\vec{b}}{\|\vec{a}\|\|\vec{b}\|} \\
cos\theta = \frac{\vec{a}\vec{b}}{\|\vec{a}\|\|\vec{b}\|} \\
cos\theta = \vec{a} \vec{b}
\end{eqnarray}
$$

$$
\vec{a} \vec{b} = \begin{pmatrix} x_a \\ y_a \end{pmatrix} \begin{pmatrix} x_b \\ y_b \end{pmatrix} = x_ax_b + y_ay_b
$$
$$
\vec{a} \vec{b} = \begin{pmatrix} x_a \\ y_a \\ z_a \end{pmatrix} \begin{pmatrix} x_b \\ y_b \\ z_b \end{pmatrix} = x_ax_b + y_ay_b + z_az_b
$$

### 将b向量投影到a向量
![[Pasted image 20261014180644.png|467]]

$$
\begin{eqnarray}
\vec{b}在\vec{a}上的投影一定可以表示为以下公式 \\
\vec{b} = k\vec{a} \\
k = \|\vec{b}\| = \|\vec{b}\|cos\theta \\
\end{eqnarray}
$$
### 计算出b在a上的投影后，在用b - b投影 = b和a垂直的向量
![[Pasted image 20261015122602.png|481]]

## 叉乘

1. 判断左右
2. 判断内外
$$
\begin{eqnarray}
\vec{a} * \vec{b} = -\vec{b} * \vec{a} \\
\|\vec{a}*\vec{b}\| = \|\vec{a}\|\|\vec{b}\|sin\theta
\end{eqnarray}
$$

# 矩阵
### 加法
$$
\begin{eqnarray}
\begin{bmatrix} 
a_{11} & a_{12} \\ 
a_{21} & a_{22} 
\end{bmatrix} 
+ 
\begin{bmatrix} 
b_{11} & b_{12} \\ 
b_{21} & b_{22} 
\end{bmatrix} 
= 
\begin{bmatrix} 
a_{11} + b_{11} & a_{12} + b_{12} \\ 
a_{21} + b_{21} & a_{22} + b_{22} 
\end{bmatrix}
\end{eqnarray}
$$
### 乘法
$$
\begin{bmatrix}
1 & 2 \\
3 & 4 \\
0 & -1
\end{bmatrix}_{3 \times 2}
\begin{bmatrix}
1 & 0 & 2 & -1 \\
3 & 1 & 0 & 2
\end{bmatrix}_{2 \times 4}
=
\begin{bmatrix}
(1 \cdot 1 + 2 \cdot 3) & (1 \cdot 0 + 2 \cdot 1) & (1 \cdot 2 + 2 \cdot 0) & (1 \cdot (-1) + 2 \cdot 2) \\
(3 \cdot 1 + 4 \cdot 3) & (3 \cdot 0 + 4 \cdot 1) & (3 \cdot 2 + 4 \cdot 0) & (3 \cdot (-1) + 4 \cdot 2) \\
(0 \cdot 1 - 1 \cdot 3) & (0 \cdot 0 - 1 \cdot 1) & (0 \cdot 2 - 1 \cdot 0) & (0 \cdot (-1) - 1 \cdot 2)
\end{bmatrix}
$$

$$
=
\begin{bmatrix}
7 & 2 & 2 & 3 \\
15 & 4 & 6 & 5 \\
-3 & -1 & 0 & -2
\end{bmatrix}_{3 \times 4}
$$
$$(AB)^T = B^T A^T$$
$$A^{-1}A = AA^{-1} = I$$
