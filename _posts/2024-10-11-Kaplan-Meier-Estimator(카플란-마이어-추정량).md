---
title: Kaplan-Meier Estimator(카플란-마이어 추정량)
description: Survival Analysis에서, 가장 널리 쓰이며 중요한 추정량인 Kaplan-Meier 추정량에 대해 Counting Process의 관점에서 유도해 본다.
author: starrypark
date: 2024-10-1 11:00:20 +0800
categories: [Statistics, Survival Analysis]
tags: [Survival Analysis, Statistics, 생존분석]
pin: true
math: true
mermaid: true
use_math: true
---


Review : 신예은 교수님 강의노트 Lecture 4

Review : Aalen, O., Borgan, O., & Gjessing, H. (2008). *Survival and event history analysis: a process point of view*. Springer Science & Business Media. Chapter 3

## Survival Function

TBD (지각)

## Product Integral

목표 : Nelson-Aalen Estimator와, Kaplan-Meier Estimatior의 관계를 알아본다.

(1) $\alpha(t)=A^{\prime}(t)=-\frac{S^{\prime}(t)}{S(t)}$
(2) $S(t)=\exp \{-A(t)\}$

위에서 식 (1)은, 다음과 같이 일반화할수 있다.

$$
A(t)=-\int_0^t \frac{d S(u)}{S(u-)}
$$

강의노트 47page의 근사과정을 통해, 결국

$$
\begin{equation} S(t)=\prod_{k=1}^K S\left(t_k \mid t_{k-1}\right) \end{equation}
$$

을 얻을 수 있음.

식 (2)는, product Integral로 일반화 가능

$$
S(t)=\lim _{M \rightarrow 0} \prod\left(1-\left\{A\left(t_k\right)-A\left(t_{k-1})\right\}\right)=: \prod_{0 \leq u \leq t}(1-d A(u))\right.
$$

## Survival Function의 Closedness

증명의 Sketch만 고려해본다. $d S(t)=-S(t-) d A(t)$를 사용하면, 다음을 생각해 볼 수 있다.

$$
d\left(S_1 / S_2\right)=\left\{\left(d S_1\right) S_2-S_1 d S_2\right\} / S_2^2=S_1 d\left(A_2-A_1\right) / S_2
$$

따라서 다음 식을 얻을 수 있다.

$$
\frac{S_1(t)}{S_2(t)}=1+\int_0^t \frac{S_1(s-)}{S_2(s)} d\left(A_2-A_1\right)(s)
$$

이를 Duhamel의 방정식이라고 한다.

## Kaplan-Meier Estimator

Nelson-Aalen Estimator $\hat{A}(t)=\sum_{T_j \leq t} \frac{1}{Y\left(T_j\right)}$에서, 이것을 앞의 product Integral에 대입하면,

$$
\hat{S}(t)=\prod_{0 \leq u \leq t}(1-d \hat{A}(t))=\prod_{u \leq t}(1-\Delta \hat{A}(t))=\prod_{T_j \leq t}\left(1-\frac{1}{Y\left(T_j\right)}\right)
$$

이 유도되며, 이것이 Kaplan-Meier Estimator이다.

직관적으로는, conditional probability들의 곱이다. (식 (1) 참고.)

만약, Censoring이 없으면, KM estimator는, 다음과 같이, empirical cumulative distribution이 된다.

$$
1-\hat{S}(t)=\frac{1}{n} \sum_{j=1}^n I\left(T_j \leq t\right)\left(=F_n\right)
$$

## Properties of KM Estimator

KM 추정량의 성질을 보기 위해, 다음을 다시 기억해보자.

$$
J(t)=I\{Y(t)>0\} \text{ 이고 } A^*(t)=\int_0^t J(s) \alpha(s) ds \approx A(t) \text{ 일 때,}
$$

$$
S^*(t)=\prod_{0 \leq s \leq t}\left(1-dA^*(s)\right)=\exp \left\{-A^*(t)\right\} \approx S(t)
$$

여기서 '≈'는 시간 u ≤ t에서 위험 집합이 없을 확률이 매우 작을 때 성립한다.

KM 추정량의 성질의 핵심은 다음과 같은 Duhamel 방정식이다.

$$
\frac{\hat{S}(t)}{S^*(t)}-1=-\int_0^t \frac{\hat{S}(s-)}{S^*(s)} d\left(\hat{A}-A^*\right)(s) \approx-\left(\hat{A}(t)-A^*(t)\right)
$$

이는 다음을 의미하고,

$$
\hat{S}(t) / S(t)-1 \approx-(\hat{A}(t)-A(t))
$$

따라서 다음과 같은 근사식을 얻을 수 있다.

$$
\hat{S}(t)-S(t) \approx-S(t)(\hat{A}(t)-A(t))
$$

Nelson-Aalen의 Asymptotic에서 KM과의 관계 (특히, Delta Method를 이용한)를 이용하여 Asymptotic Normality도 보일 수 있다. (TBD)

## Median Survival Time

Note. Survival은 Mean을 구하기가, 보통은 쉽지 않다.

survival distribution $F(t)=1-S(t)$의 p번째 fractile $\xi_p$는, $F(\xi_p)=p$ 혹은 $S(\xi_p) =1-p$가 되도록 정의된다.

다만 KM estimator가 discrete한 성질을 지니기 때문에 이러한 $\xi_p$가 반드시 존재하리란 보장은 없다. 따라서,

$$
\hat{\xi}_p=\inf \{t \mid \hat{S}(t) \leq 1-p\}
$$

로 대신 정의한다.