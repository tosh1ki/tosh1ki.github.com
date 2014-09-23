---
layout: post
title: "scikit-learnのソースコードを読む:Gaussian process"
description: ""
category: 備忘録
tags: [scikit-learn,Python]
---
{% include JB/setup %}

主に`GaussianProcess.reduced_likelihood_function()`内でどういった処理をしているかを，数式で解説する．
	
### Correlation model の選択 ###

[scikit-learn/correlation_models.py at master · scikit-learn/scikit-learn](https://github.com/scikit-learn/scikit-learn/blob/master/sklearn/gaussian_process/correlation_models.py)

$$ \begin{align}
	\mathrm{absolute\_exponential}(\theta, d) &= \exp(-\sum_{i=1}^n \theta_i |d_i|) \\
	\mathrm{squared\_exponential}(\theta, d) &= \exp(-\sum_{i=1}^n \theta_i (d_i)^2) \\
	\mathrm{generalized\_exponential}((\theta,p), d) &= \exp(-\sum_{i=1}^n \theta_i |d_i|^p) \\
	\mathrm{absolute\_exponential}(\theta, d) &= \exp(-\sum_{i=1}^n \theta_i |d_i|) \\
	\mathrm{pure\_nugget}(\theta, d) &=
		\begin{cases}
			1&\mathrm{if}\, \sum_{i=1}^n |d_i| = 0 \\
			0 & \mathrm{otherwise}
		\end{cases} \\
	\left(\mathrm{cubic}(\theta, d)\right)_{i} &=
		\prod_{j=1}^n \max(0, 1 - 3(\theta_j d_{ij})^2 + 2(\theta_j d_{ij})^3),\quad i = 1,\ldots,m \\
	\left(\mathrm{linear}(\theta, d)\right)_{i} &= \prod_{j=1}^n \max(0, 1 - \theta_j d_{ij}),\quad i = 1,\ldots,m
\end{align}$$

例えば，$$k(x_i,x_j) := \mathrm{absolute\_exponential}(\theta, x_i - x_j)$$などとする．
$n_s$ はサンプル数

$$
	R :=
	\begin{pmatrix}
		k(x_1,x_1) + \mathrm{nugget} & k(x_1,x_2) & \cdots & k(x_1,x_\mathrm{n_{s}}) \\
		k(x_2,x_1) & k(x_2,x_1) + \mathrm{nugget} & \cdots & k(x_2,x_\mathrm{n_{s}}) \\
		\vdots & \vdots & \ddots & \vdots \\
		k(x_\mathrm{n_{s}},x_1) & k(x_\mathrm{n_{s}},x_2) & \cdots & k(x_\mathrm{n_{s}},x_\mathrm{n_{s}}) + \mathrm{nugget}
	\end{pmatrix}
$$

$C$を$R$のCholesky分解，すなわち $R=CC^{\mathrm{T}}$ を満たす下三角行列とする．

$$ Ft := C \setminus F $$

$Q,G$を$Ft$のQR分解$Ft = QG$とする．

$\beta = G \setminus (Q^\mathrm{T} Yt)$
	
### 参考文献 ###

- [scikit-learn/sklearn/gaussian_process at master · scikit-learn/scikit-learn](https://github.com/scikit-learn/scikit-learn/tree/master/sklearn/gaussian_process)
- [1.5. Gaussian Processes — scikit-learn 0.15.2 documentation](http://scikit-learn.org/stable/modules/gaussian_process.html)
- [Gaussian Processes for Machine Learning: Book webpage](http://www.gaussianprocess.org/gpml/)
