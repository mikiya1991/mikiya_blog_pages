---
layout: markdown
title: "余弦距离与欧拉公式"
comments: true
tags: 余弦距离 欧拉公式 三角函数 和差公式
---

## 1、余弦距离

衡量两个向量的相似度。可通过[余弦距离][1]， $$\theta$$角为向量A、B的夹角:

$$
\begin{align}
\cos{\theta}&=\frac{\vec{A}\cdot\vec{B}}{\left|\vec{A}\right|\left|\vec{B}\right|} \tag{1} \\
\cos{\theta}&=\frac{\vec{A}}{\left|\vec{A}\right|} \cdot \frac{\vec{B}}{\left|\vec{B}\right|} \tag{2}
\end{align}
$$

我们可以将两个单位向量表示为：

$$
\begin{align}
\vec{A} = (\cos{\alpha}, \sin{\alpha}) \tag{3} \\
\vec{B} = (\cos{\beta}, \sin{\beta}) \tag{4} \\
\end{align}
$$

所以余弦距离等于

$$
\begin{align}
\cos(\theta) = \vec{A} \cdot \vec{B} = \cos{\alpha}\cdot\cos{\beta} + \sin{\alpha} \cdot \sin{\beta} \tag{5}
\end{align}
$$

#### 1.1 这个结论怎么来的呢？为什么单位向量的点积，可以用来衡量其相似度？

其实余弦距离$$\cos{\theta}$$就是两向量间的夹角， 通过夹角的$$\cos$$值来衡量单位向量的相似度。


根据公式（2）公式（5）我们得到

$$
\cos{\theta} = \cos(\alpha - \beta) ?= \cos{\alpha}\cdot\cos{\beta} + \sin{\alpha} \cdot \sin{\beta} \tag{6}
$$

如果余弦距离公式成立，我们需要公式（6）的等式成立。

明显的我们知道公式（6）是三角函数中的 [**和差公式**][2]。 只要和差公式成立，则余弦距离成立。

## 2、三角函数和差公式的证明

三角函数和差公式为：

$$
\begin{align}
\cos{(a + b)} &= \cos{a}\cdot\cos{b} - \sin{a}\cdot\sin{b} \tag{7} \\
\sin(a+b) &= \cos{b}\cdot\sin{a} + \cos{a}\cdot\sin{b} \tag{8}
\end{align}
$$

怎么得到呢？还是从定义去证明？

为此我上网查阅资料，得到两种方式的证明。

### 2.1 几何方法

![fig1](/assets/trisub.png)

如上图所示。

做两个锐角分别为a和b的直角三角形。
- 将a角和b角靠在一起，上面三角形的底边长等于下面三角的斜边长。
- 下面三角形底边长为1
- 然后补全一个长方形。

红三角斜边长`l`为

$$
\begin{align}
l = \frac{1/\cos(\beta)}{\cos(\beta)} = \frac{1}{\cos(\alpha)\cos(\beta)}\\
\end{align}
$$

上图左上三角形可得，详细证明省略

$$
\begin{align}
\tan(\alpha+\beta) &= \frac{\tan{\alpha}\tan{\beta}}{1 - \tan{\alpha}\tan{\beta}} \\
\cos(\alpha+\beta) &= \frac{1-\tan(\alpha)\tan(\beta)}{l} = \frac{1-\tan(\alpha)\tan(\beta)}{1/\cos(\alpha)\cos(\beta)} = \cos(\alpha)\cos(\beta) - \sin(\alpha)\sin(\beta) \\
\sin(\alpha+\beta) &= \frac{\tan(\alpha)+\tan(\beta)}{l} = \frac{\tan(\alpha)+\tan(\beta)}{1/\cos(\alpha)\cos(\beta)} = \sin(\alpha)\cos(\beta) + \cos(\alpha)\sin(\beta)
\end{align}
$$

虽然上图可以证明和差公式，我还是不明白为什么？！感觉上面推导像是文字游戏一样，只是碰巧发现了规律。

为什么三角形会有这种关系。上面的证明 **不能从根本上解释三角形的本质关系** 。

### 2.2 更加本质？

于是我想到了一个公式，即[欧拉公式][3]

$$
e^{i\theta} = \cos{\theta} + i\sin{\theta} \tag{7}
$$

根据其可以把 三角函数的相加关系转化为相乘的关系（通过指数的方式，*指数就是这么神奇，将乘法关系转化为加法关系*）。

于是我们有

$$
\begin{align}
e^{i(a + b)} &= e^{ia} \cdot e^{ib} \\
\cos(a + b) + i\sin(a+b) &= (\cos a + i\sin a)\cdot(\cos b + i\sin b) \\
& = (\cos a\cos b - \sin a \sin b) + i(\cos a\sin b + \cos b\sin a)
\end{align}
$$

#### 根据实部与实部相等，虚步与虚步相等，可以自然得到三角的和差公式

而且欧拉公式不仅包含了和差公式的关系，其更内涵了三角更多的内在关系。

到了这里我们就有一个问题了，欧拉公式是什么意思？指数上的复数代表什么？为什么复指数有这种关系？

## 3、欧拉公式

欧拉公式很神奇：
- 为什么存在指数和三角函数的对应关系，复指数怎么就能表示为复三角函数
- $$e^{i\pi} = -1$$， 一个底是正数的指数函数居然小于0了

其实我们可以通过泰勒展开的方式，将 $$e^{x}$$展开，然后将 $$x=i\theta$$带入；同理右侧泰勒展开带入，来验证欧拉公式正确性。

欧拉公式内涵了，直角坐标系的相乘， 等于极坐标系的旋转。

但是欧拉公式什么意思，究竟表达了什么关系？欧拉公式怎么推导？这个我们以后再说吧



[1]: https://zh.wikipedia.org/wiki/%E4%BD%99%E5%BC%A6%E7%9B%B8%E4%BC%BC%E6%80%A7
[2]: https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E5%87%BD%E6%95%B0
[3]: https://zh.wikipedia.org/wiki/%E6%AC%A7%E6%8B%89%E5%85%AC%E5%BC%8F
