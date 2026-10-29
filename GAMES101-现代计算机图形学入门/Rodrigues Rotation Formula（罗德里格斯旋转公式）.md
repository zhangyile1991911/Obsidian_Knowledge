 
## 1. 基本定义

Rodrigues 旋转公式用于将三维向量围绕任意单位轴旋转指定角度。

- $\mathbf{v}$：原始向量
- $\mathbf{k}$：单位旋转轴，$\|\mathbf{k}\|=1$
- $\theta$：旋转角度（弧度）
- $\mathbf{v}'$：旋转后的向量

## 2. 核心公式

$$
\boxed{
\mathbf{v}' =
\mathbf{v}\cos\theta
+(\mathbf{k}\times\mathbf{v})\sin\theta
+\mathbf{k}(\mathbf{k}\cdot\mathbf{v})(1-\cos\theta)
}
$$

## 3. 原理推导

### 3.1 平行于旋转轴的分量

$$
\mathbf{v}_{\parallel}=(\mathbf{k}\cdot\mathbf{v})\mathbf{k}
$$

### 3.2 垂直于旋转轴的分量

$$
\mathbf{v}_{\perp}=\mathbf{v}-\mathbf{v}_{\parallel}
$$

### 3.3 旋转垂直分量

$$
\mathbf{v}_{\perp}'=\mathbf{v}_{\perp}\cos\theta+(\mathbf{k}\times\mathbf{v})\sin\theta
$$

### 3.4 合并两个分量

$$
\mathbf{v}'=\mathbf{v}_{\parallel}+\mathbf{v}_{\perp}'
$$

展开并整理：

$$
\mathbf{v}'=
\mathbf{v}\cos\theta
+(\mathbf{k}\times\mathbf{v})\sin\theta
+\mathbf{k}(\mathbf{k}\cdot\mathbf{v})(1-\cos\theta)
$$

## 4. 旋转矩阵形式

令 $\mathbf{k}=(k_x,k_y,k_z)^T$，定义叉积矩阵：

$$
[\mathbf{k}]_\times=
\begin{bmatrix}
0 & -k_z & k_y \\
k_z & 0 & -k_x \\
-k_y & k_x & 0
\end{bmatrix}
$$

则旋转矩阵为：

$$
\boxed{R=I\cos\theta+(1-\cos\theta)\mathbf{k}\mathbf{k}^T+[\mathbf{k}]_\times\sin\theta}
$$

也可以写成：

$$
R=I+[\mathbf{k}]_\times\sin\theta+[\mathbf{k}]_\times^2(1-\cos\theta)
$$

最终：

$$
\mathbf{v}'=R\mathbf{v}
$$

## 5. HLSL 实现

```hlsl
float3 RotateAroundAxis(float3 v, float3 axis, float angle)
{
    axis = normalize(axis);
    float c = cos(angle);
    float s = sin(angle);

    return v * c
         + cross(axis, v) * s
         + axis * dot(axis, v) * (1.0 - c);
}
```

## 6. 注意事项

- 旋转轴必须是非零单位向量。
- 旋转角度在 Shader 中通常采用弧度。
- 正角度遵循右手法则。
- 该公式只处理旋转，不包含平移或缩放。
