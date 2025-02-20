---
title: Conformal Prediction
description: Conformal Prediction에 관한 기본적인 설명 (증명은 상당부분 생략)
author: starrypark
date: 2025-02-20 10:00:20 +0800
categories: [Statistics, Uncertainty Quantification]
tags: [Conformal Prediction, Statistics]
pin: true
math: true
mermaid: true
use_math: true
---

Review : Won Chang 교수님 강의노트 - Chap7

## Setting

1.  $(X_i, Y_i)$ : iid Sample
2.  $X_i \subset \mathscr{X} \subset R^p$
3.  $Y_i \subset \mathscr{Y} \subset R^p$

## Purpose

우리는 유한 표본을 기반으로 prediction band $\hat{C}_n \mid \mathcal{X} \rightarrow\{$ subsets of $\mathcal{Y}\}$를 찾고자 하며, 이는 다음을 만족한다.

$$
P\left(Y_{n+1} \in \hat{C}_n\left(X_{n+1}\right)\right) \geq 1-\alpha
$$

## Permutation

$Y$가 iid이기 때문에, permutation들은 비슷한 정도의 가능성을 지닌다(equally likely).

따라서, ordered value $Y_{(1)}, \ldots, Y_{(n)}, Y_{(n+1)}$에 대해, $P(Y_{n+1} = Y_{(i)} ) = \frac{1}{n+1}$이다. 따라서,

$$
P\left(Y_{n+1} \text { is among the }\lceil(1+n)(1-\alpha)\rceil \text { smallest of } Y_{1}, \ldots, Y_{n}, Y_{n+1}\right) \geq 1-\alpha
$$

임을 알 수 있다. 즉 $Y_(i)$들의 순서를 통해 예측 구간을 구한다는 원리이다.

하지만, 이렇게 추측하는 것은 $Y_{n+1}$ 추측을 위해 $Y_{n+1}$를 사용하는 이상한 상황이 연출되어,

어떻게 저것을 지울까 고민할 필요가 있음.

### But

$$
A=\left\{Y_{n+1} \text { is among the }\lceil(1+n)(1-\alpha)\rceil \text { smallest of } Y_1, \ldots, Y_n, Y_{n+1}\right\}
$$

와,

$$
B=\left\{Y_{n+1} \text { is among the }[(1+n)(1-\alpha)\rceil \text { smallest of } Y_1, \ldots, Y_n\right\}
$$

가, equivalent한 event임에 주목하자. 그 이유는, $Y_{n+1} > Y_{n+1}$일 수 없기 때문에,

$$
\begin{aligned}A^c= & \left\{Y_{n+1}>\text { the }[(1+n)(1-\alpha)\rceil \text { th smallest of } Y_1, \ldots, Y_n, Y_{n+1}\right\} \\& \Leftrightarrow\left\{Y_{n+1}>\text { the }[(1+n)(1-\alpha)] \text { th smallest of } Y_1, \ldots, Y_n\right\} = B^c\end{aligned} 
$$

가 성립하기 때문이다. 
