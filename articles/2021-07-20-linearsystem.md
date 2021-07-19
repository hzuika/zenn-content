---
title: "線形システムの勉強ノート"
emoji: "🖋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["math"]
published: false
---
# 2次元線形システム

2次元線形システム

$$
\begin{aligned}
\dot{\vec{x}} = \mathbf{A}\vec{x}\\
\end{aligned}
$$

2次元行列

$$
\begin{aligned}
\mathbf{A} = \begin{bmatrix}
a & b\\
c & d
\end{bmatrix}
\end{aligned}
$$

固有値: $\lambda_1, \lambda_2$
固有ベクトル: $\vec{v}_1, \vec{v}_2$

一般解

$$
\begin{aligned}
\vec{x} = c_1 e^{\lambda_1 t} \vec{v}_1 + c_2 e^{\lambda_2 t} \vec{v}_2\\
\end{aligned}
$$

---

## 例

### 例1

$$
\begin{aligned}
\frac{ dx }{ dt } &= ax + by\\
\frac{ dy }{ dt } &= -bx - ay\\
\end{aligned}
$$

ただし，$a, b > 0$．

1. 行列形式で書く．

$$
\begin{aligned}
\dot{\vec{x}} = \begin{bmatrix}
\frac{ d x }{ d t }\\
\frac{ d y }{ d t }
\end{bmatrix} &= 
\mathbf{A}\vec{x}\\
&= \begin{bmatrix}
a & b\\
-b & -a
\end{bmatrix}
\begin{bmatrix}
x \\ y
\end{bmatrix}
\end{aligned}
$$

2. $\mathbf{A}$の固有値$\lambda_1, \lambda_2$を求める．

$$
\begin{aligned}
\det(\mathbf{A} - \lambda \mathbf{I}) &= 0\\
\begin{vmatrix}
a - \lambda & b\\
-b & -a - \lambda
\end{vmatrix} &= 0\\
(a - \lambda)(-a - \lambda) - b(-b) &= 0\\
(\lambda - a)(\lambda + a) + b^2 &= 0\\
\lambda^2 - a^2 + b^2 &= 0\\
\lambda^2 &= a^2 - b^2\\
\lambda &= \pm \sqrt{a^2 - b^2}\\
\lambda_1, \lambda_2 &= + \sqrt{a^2 - b^2}, -\sqrt{a^2 - b^2}\\
\end{aligned}
$$

3. 固有ベクトル$\vec{v}_1, \vec{v}_2$を求める．

$$
\begin{aligned}
(\mathbf{A} - \lambda \mathbf{I})\vec{v} &= \mathbf{0}\\
\end{aligned}
$$

$\lambda_1 = + \sqrt{a^2 - b^2}$の場合，

$$
\begin{aligned}
\begin{bmatrix}
a - \sqrt{a^2 - b^2} & b\\
-b & -a - \sqrt{a^2 - b^2}
\end{bmatrix}
\begin{bmatrix}
v_x \\ v_y
\end{bmatrix} &= 
\begin{bmatrix}
0 \\ 0
\end{bmatrix}\\
\end{aligned}
$$

$$
\begin{aligned}
\vec{v}_1 = \begin{bmatrix}
v_x \\ v_y
\end{bmatrix} &= 
\begin{bmatrix}
{ a + \sqrt{a^2 - b^2} }\\
-b\\
\end{bmatrix}\\
&= 
\begin{bmatrix}
{ \frac{ a }{ b } + \sqrt{\left(\frac{ a }{ b }\right)^2 - 1} }\\
-1\\
\end{bmatrix}\\
\end{aligned}
$$

$\lambda_2 = - \sqrt{a^2 - b^2}$の場合，

$$
\begin{aligned}
\begin{bmatrix}
a + \sqrt{a^2 - b^2} & b\\
-b & -a + \sqrt{a^2 - b^2}
\end{bmatrix}
\begin{bmatrix}
v_x \\ v_y
\end{bmatrix} &= 
\begin{bmatrix}
0 \\ 0
\end{bmatrix}\\
\end{aligned}
$$

$$
\begin{aligned}
\vec{v}_2 = \begin{bmatrix}
v_x \\ v_y
\end{bmatrix} &= 
\begin{bmatrix}
{ \frac{ a }{ b } - \sqrt{\left(\frac{ a }{ b }\right)^2 - 1} }\\
-1\\
\end{bmatrix}
\end{aligned}
$$

---

### 例2

$$
\begin{aligned}
\mathbf{A} &= 
\begin{bmatrix}
a & b\\
b & a
\end{bmatrix}\\
\end{aligned}
$$

固有値を求める．

$$
\begin{aligned}
\begin{vmatrix}
a - \lambda & b\\
b & a - \lambda
\end{vmatrix} &= 0\\
(a - \lambda)^2 - b^2 &= 0\\
a - \lambda &= \pm b\\
\lambda &= a \pm b\\
\lambda_1, \lambda_2 &= a + b, a - b\\
\end{aligned}
$$

固有ベクトルを求める．

$$
\begin{aligned}
\begin{bmatrix}
a - (a + b) & b\\
b & a - (a + b)
\end{bmatrix}\vec{v}_1 &= 0\\
\begin{bmatrix}
-b & b\\
b & -b
\end{bmatrix}\vec{v}_1 &= 0\\
\vec{v}_1 &= 
\begin{bmatrix}
1 \\ 1
\end{bmatrix}
\end{aligned}
$$

$$
\begin{aligned}
\begin{bmatrix}
a - (a - b) & b\\
b & a - (a - b)
\end{bmatrix}
\vec{v}_2 &= \vec{0}\\
\begin{bmatrix}
b & b\\
b & b
\end{bmatrix}
\vec{v}_2 &= \vec{0}\\
\vec{v}_2 &= \begin{bmatrix}
1 \\ -1
\end{bmatrix}\\
\end{aligned}
$$

グラフに描く際は，固有ベクトル方向は固有値倍になることを意識する．