---
title: Dynamic Linear Model
description: Dynamic Linear model의 기본적인 개념과 분포 개념 설명명
author: starrypark
date: 2024-06-11 10:00:20 +0800
categories: [Statistics, Survival Analysis]
tags: [Conformal Prediction, Statistics]
pin: true
math: true
mermaid: true
use_math: true
---

Review : West, M., & Harrison, J. (2006). *Bayesian forecasting and dynamic models*. Springer Science & Business Media. - Chapter 4

## Purpose

이 포스팅은 West & Harrison의 Bayesian forecasting and dynamic models을 요약한 것이다. 여기에 있는 책의 내용의 순서를 잘 따라가면, Dynamic Linear Model에 대한 흐름을 어느 정도 파악할 수 있을 것이다.


# 4.2 Definition & Notations

observation vector $\boldsymbol{Y}_t$가 $(r \times 1)$의 벡터로 time series를 이룬다고 가정하자.

### Definition 4.1.

일반적인 normal DLM은 각 시각 $t$마다 다음과 같은 quadruples들의 set들로 표현된다.

$$
\left\{\boldsymbol{F}, \boldsymbol{G}, \boldsymbol{V}, \boldsymbol{W} \right\}_t =\left\{\boldsymbol{F}_t, \boldsymbol{G}_t, \boldsymbol{V}_t, \boldsymbol{W}_t \right\}
$$

여기서 $\boldsymbol{F}_t$는 $(n\times r)$, $\boldsymbol{G}_t$는 $(n\times n)$, $\boldsymbol{V}_t$는 $(r\times r)$, $\boldsymbol{F}_t$는 $(n\times n)$ 행렬이다. 뒤의 두 개는 분산행렬이다. 그리고 이것은 

$$
\begin{equation}\begin{aligned}&\left(\boldsymbol{Y}_t \mid \boldsymbol{\theta}_t\right) \sim \mathrm{N}\left[\boldsymbol{F}_t^{\prime} \boldsymbol{\theta}_t, \boldsymbol{V}_t\right]\\&\left(\boldsymbol{\theta}_t \mid \boldsymbol{\theta}_{t-1}\right) \sim \mathrm{N}\left[\boldsymbol{G}_t \boldsymbol{\theta}_{t-1}, \boldsymbol{W}_t\right]\end{aligned}\end{equation}
$$

로 나타낼 수 있다.  혹은,

$$
\begin{equation}\begin{aligned}\boldsymbol{Y}_t=\boldsymbol{F}_t^{\prime} \boldsymbol{\theta}_t+\boldsymbol{\nu}_t, & \boldsymbol{\nu}_t \sim \mathrm{N}\left[\boldsymbol{0}, \boldsymbol{V}_t\right], \\\boldsymbol{\theta}_t=\boldsymbol{G}_t \boldsymbol{\theta}_{t-1}+\boldsymbol{\omega}_t, & \boldsymbol{\omega}_t \sim \mathrm{N}\left[\boldsymbol{0}, \boldsymbol{W}_t\right]\end{aligned}\end{equation}
$$

으로 나타낼 수 있다. 여기서 $\boldsymbol{\nu}_t$와 $\boldsymbol{\omega}_t$는 서로 독립이라고 가정한다.

# 4.3. Updating Equations

### Definition 4.3.

각 시각  $t$마다, univariate DLM은 다음과 같이 정의된다.

$$
\begin{equation}\begin{array}{lcl}\text { Observation equation: } & Y_t=\boldsymbol{F}_t^{\prime} \boldsymbol{\theta}_t+\nu_t, & \nu_t \sim \mathrm{N}\left[0, V_t\right], \\\text { System equation: } & \boldsymbol{\theta}_t=\boldsymbol{G}_t \boldsymbol{\theta}_{t-1}+\boldsymbol{\omega}_t, & \boldsymbol{\omega}_t \sim \mathrm{N}\left[\boldsymbol{0}, \boldsymbol{W}_t\right], \\\text { Initial information: } & \left(\boldsymbol{\theta}_0 \mid D_0\right) \sim \mathrm{N}\left[\boldsymbol{m}_0, \boldsymbol{C}_0\right], &\end{array}\end{equation}
$$

여기서 $ \boldsymbol{m}_0, \boldsymbol{C}_0 $ 은 prior moment들이다. 또한, Information set $D_t =\left\{Y_t, D_{t-1} \right\}$으로 정의된다.

그러면, 다음 정리를 통해 equation을 Update할 수 있다.

### Theorem 4.1

Definition 4.3의 univariate DLM에서, 1step 사후분포 예측은 다음과 같이 진행할 수 있다.

1. Posterior at $t-1$
    
    $$
    \begin{equation}\left(\boldsymbol{\theta}_{t-1} \mid D_{t-1}\right) \sim \mathrm{N}\left[\boldsymbol{m}_{t-1}, \boldsymbol{C}_{t-1}\right]\end{equation}
    $$
    
2. Prior at $t$
    
    $$
    \left(\boldsymbol{\theta}_t \mid D_{t-1}\right) \sim \mathrm{N}\left[\boldsymbol{a}_t, \boldsymbol{R}_t\right]
    $$
    
    여기서, $\boldsymbol{a}_t=\boldsymbol{G}_t \boldsymbol{m}_{t-1} , \ \boldsymbol{R}_t=\boldsymbol{G}_t \boldsymbol{C}_{t-1} \boldsymbol{G}_t^{\prime}+\boldsymbol{W}_t$이다.
    
3. One-step forecast
    
    $$
    \left(Y_{t} \mid D_{t-1}\right) \sim \mathrm{N}\left[f_{t}, Q_{t}\right]
    $$
    
    여기서, $f_{t}=\boldsymbol{F}_{t}^{\prime} \boldsymbol{a}_{t} , \  Q_{t}=\boldsymbol{F}_{t}^{\prime} \boldsymbol{R}_{t} \boldsymbol{F}_{t}+V_{t}$이다.
    
4. Posterior at $t$
    
    $$
    \left(\boldsymbol{\theta}_{t} \mid D_{t}\right) \sim \mathrm{N}\left[\boldsymbol{m}_{t}, \boldsymbol{C}_{t}\right]
    $$
    
    여기서, $\boldsymbol{m}_{t}=\boldsymbol{a}_{t}+\boldsymbol{A}_{t} e_{t} ,\ \boldsymbol{C}_{t}=\boldsymbol{R}_{t}-\boldsymbol{A}_{t} Q_{t} \boldsymbol{A}_{t}^{\prime}\boldsymbol{A}_{t}=\boldsymbol{R}_{t} \boldsymbol{F}_{t} Q_{t}^{-1} ,\  e_{t}=Y_{t}-f_{t}$이다.
    

(증명) 

1~3까진 Easy. 4는 Conditional Normal 분포를 생각해보자.

# 4.4. Forecast Distributions

### Definition 4.4

Forecast function $f_t(k)$는 모든 시간 $t$, 모든 음이 아닌 정수 $k$에 대해 다음과 같이 정의된다.

$$
\begin{equation}f_t(k)=\mathrm{E}\left[\mu_{t+k} \mid D_t\right]=\mathrm{E}\left[\boldsymbol{F}_{t+k}^{\prime} \boldsymbol{\theta}_{t+k} \mid D_t\right]\end{equation}
$$

여기서 $\mu_v=\boldsymbol{F}_v^{\top} \boldsymbol{\theta}_v$는 mean response function이라고 부른다.

### Theorem 4.2

$0 \le j <k$인 모든 $j$에 대해, 각 time point $t$에서 다음이 성립한다.

- (a) State distribution : $\left(\boldsymbol{\theta}_{t+k} \mid D_t\right) \sim \mathrm{N}\left[\boldsymbol{a}_t(k), \boldsymbol{R}_t(k)\right]$
- (b) Forecast distribution :  $\left(Y_{t+k} \mid D_t\right) \sim \mathrm{N}\left[f_t(k), Q_t(k)\right]$
- (c) State Covariance : $\mathrm{C}\left[\boldsymbol{\theta}_{t+k}, \boldsymbol{\theta}_{t+j} \mid D_t\right]=\boldsymbol{C}_t(k, j)$
- (d) Obsn. Covariance : $\mathrm{C}\left[Y_{t+k}, Y_{t+j} \mid D_t\right]=\boldsymbol{F}_{t+k}^{\prime} \boldsymbol{C}_t(k, j) \boldsymbol{F}_{t+j}$
- (e) Other Covariance : $\begin{aligned}& \mathrm{C}\left[\boldsymbol{\theta}_{t+k}, Y_{t+j} \mid D_t\right]=\boldsymbol{C}_t(k, j) \boldsymbol{F}_{t+j} \\& \mathrm{C}\left[Y_{t+k}, \boldsymbol{\theta}_{t+j} \mid D_t\right]=\boldsymbol{F}_{t+k}^{\prime} \boldsymbol{C}_t(k, j)\end{aligned}$

여기서, $f_t(k)=\boldsymbol{F}_{t+k}^{\top} \boldsymbol{a}_t(k) \quad \text { and } \quad Q_t(k)=\boldsymbol{F}_{t+k}^{\top} \boldsymbol{R}_t(k) \boldsymbol{F}_{t+k}+V_{t+k}$이고, 이는 다음과 같이 순차적으로 계산이 가능하다.

$$
\begin{aligned}\boldsymbol{a}_t(k) & =\boldsymbol{G}_{t+k} \boldsymbol{a}_t(k-1) \\\boldsymbol{R}_t(k) & =\boldsymbol{G}_{t+k} \boldsymbol{R}_t(k-1) \boldsymbol{G}_{t+k}^{\top}+\boldsymbol{W}_{t+k} \\\boldsymbol{C}_t(k, j) & =\boldsymbol{G}_{t+k} \boldsymbol{C}_t(k-1, j), k=j+1, \ldots\end{aligned}
$$

- Proof of Theorem 4.2
    
    (증명)
    
    먼저 $n \times n$ 행렬 $\boldsymbol{H}_{t+k}(r)=\boldsymbol{G}_{t+k} \boldsymbol{G}_{t+k-1} \ldots \boldsymbol{G}_{t+k-r+1}$를 모든 $t$와 $r \le k$에 대해 정의하자. 또한, $\boldsymbol{H}_{t+k}(0)=\boldsymbol{I}$라고 가정하자.
    
    여기서 $\boldsymbol{\theta}_t=\boldsymbol{G}_t \boldsymbol{\theta}_{t-1}+\boldsymbol{\omega}_t$를 반복적으로 적용하여 계산할 시,
    
    $$
    \boldsymbol{\theta}_{t+k}=\boldsymbol{H}_{t+k}(k) \boldsymbol{\theta}_t+\sum_{r=1}^k \boldsymbol{H}_{t+k}(k-r) \boldsymbol{\omega}_{t+r}
    $$
    
    를 얻을 수 있다. 이 식을 통해, 
    
    $$
    \left(\boldsymbol{\theta}_{t+k} \mid D_t\right) \sim \mathrm{N}\left[\boldsymbol{a}_t(k), \boldsymbol{R}_t(k)\right]
    $$
    
    를 얻을 수 있다. 여기서, $\boldsymbol{a}_t(k)=\boldsymbol{H}_{t+k}(k) \boldsymbol{m}_t=\boldsymbol{G}_{t+k} \boldsymbol{a}_t(k-1)$이며 
    
    $$
    \begin{aligned}\boldsymbol{R}_t(k) & =\boldsymbol{H}_{t+k}(k) \boldsymbol{C}_t \boldsymbol{H}_{t+k}(k)^{\top}+\sum_{r=1}^k \boldsymbol{H}_{t+k}(k-r) \boldsymbol{W}_{t+r} \boldsymbol{H}_{t+k}(k-r)^{\top} \\& =\boldsymbol{G}_{t+k} \boldsymbol{R}_t(k-1) \boldsymbol{G}_{t+k}^{\top}+\boldsymbol{W}_{t+k}\end{aligned}
    $$
    
    이다. 또한 Initial Value는 $\boldsymbol{a}_t(0)=\boldsymbol{m}_t, \text { and } \boldsymbol{R}_t(0)=\boldsymbol{C}_t$이다. 여기서 (a) 증명이 끝난다.
    
    (b) 역시, Observational Equation을 통해서 얻을 수 있다. 어차피 간단한 Matrix 계산이기에 나머지도 생략.
    

# 4.7 Filtering Recurrences

최근 데이터를 사용하여 상태 벡터의 이전 값에 대한 추론을 수정하는 것을 **Filtering**이라고 한다.  $k \ge 1$$k \ge1$$\left(\boldsymbol{\theta}_{t-k} \mid D_t\right)$를 **$k$-step filtered distribution**이라고 한다.

이 filtered distribution을 다음 정리를 이용해서 유도할 것이다.

### Theorem 4.4.

Univariate DLM $\left\{\boldsymbol{F}_t, \boldsymbol{G}_t, V_t, \boldsymbol{W}_t\right\}$에서, 모든 $t$에 대해,

$$
\boldsymbol{B}_t=\boldsymbol{C}_t \boldsymbol{G}_{t+1}^{\top} \boldsymbol{R}_{t+1}^{-1}
$$

로 정의하자. 그러면 모든 $0 \le k < t$인 $k$에 대해, filtered marginal distribution은 다음과 같이 주어진다.

$$
\left(\boldsymbol{\theta}_{t-k} \mid D_t\right) \sim \mathrm{N}\left[\boldsymbol{a}_t(-k), \boldsymbol{R}_t(-k)\right]
$$

여기서, $\boldsymbol{a}_t(-k)=\boldsymbol{m}_{t-k}+\boldsymbol{B}_{t-k}\left[\boldsymbol{a}_t(-k+1)-\boldsymbol{a}_{t-k+1}\right]$이고,  $\boldsymbol{R}_t(-k)=\boldsymbol{C}_{t-k}+\boldsymbol{B}_{t-k}\left[\boldsymbol{R}_t(-k+1)-\boldsymbol{R}_{t-k+1}\right] \boldsymbol{B}_{t-k}^{\top}$이다.

Initial value는 $\boldsymbol{a}_t(0)=\boldsymbol{m}_t \quad \text { and } \quad \boldsymbol{R}_t(0)=\boldsymbol{C}_t$라고 가정한다.

- Proof of Theorem 4.4 (미완)
    
    Filtered density는 다음은 식에서 순차적으로 계산할 수 있다.
    
    $$
    \begin{equation}\begin{aligned}p\left(\theta_{t-k} \mid D_k\right) & =\int p\left(\theta_{t-k}, \theta_{t-k+1} \mid D_k\right) d \theta_{t-k+1} \\& =\int p\left(\theta_{t-k} \mid \theta_{t-k+1}, D_k\right) p\left(\theta_{t-k+1} \mid D_k\right) d\theta_{t-k+1}\end{aligned}\end{equation}
    $$
    
    이를 통해, Induction을 이용해 정리를 증명해 보자.
    
    먼저 $k-1$에서 식이 성립한다고 가정하자. 그러면 
    
    $$
    \left(\boldsymbol{\theta}_{t-k+1} \mid D_t\right) \sim \mathrm{N}\left[\boldsymbol{a}_t(-k+1), \boldsymbol{R}_t(-k+1)\right]
    $$
    
    가 성립하므로 식 (6)에서 두 번째 확률은 구할 수 있다.
    
    첫 번째 확률의 경우, 베이즈 정리를 이용하여,
    
    $$
    p\left(\boldsymbol{\theta}_{t-k} \mid \boldsymbol{\theta}_{t-k+1}, D_t\right)=\frac{p\left(\boldsymbol{\theta}_{t-k} \mid \boldsymbol{\theta}_{t-k+1}, D_{t-k}\right) p\left(\boldsymbol{Y} \mid \boldsymbol{\theta}_{t-k}, \boldsymbol{\theta}_{t-k+1}, D_{t-k}\right)}{p\left(\boldsymbol{Y} \mid \boldsymbol{\theta}_{t-k+1}, D_{t-k}\right)}
    $$
    
    다음과 같은 식을 얻을 수 있다. 여기서 $\boldsymbol{Y}=\left\{Y_{t-k+1}, \ldots, Y_t\right\}$이다. 여기서 $\boldsymbol{\theta}_{t-k+1}$가 주어져 있는 경우, $\boldsymbol{Y} \perp \boldsymbol{\theta}_{t-k}$의 조건부 독립을 이용하여, 뒤의 항은 cancel-out이 가능하고,
    
    $$
    \begin{equation}p\left(\boldsymbol{\theta}_{t-k} \mid \boldsymbol{\theta}_{t-k+1}, D_{t-k}\right) \propto p\left(\boldsymbol{\theta}_{t-k} \mid D_{t-k}\right) p\left(\boldsymbol{\theta}_{t-k+1} \mid \boldsymbol{\theta}_{t-k}, D_{t-k}\right)\end{equation}
    $$
    
    가 베이즈 정리에 의해 성립한다. 여기서,
    
    $$
    \begin{equation}\left(\boldsymbol{\theta}_{t-k+1} \mid \boldsymbol{\theta}_{t-k}, D_{t-k}\right) \sim \mathrm{N}\left[\boldsymbol{G}_{t-k+1} \boldsymbol{\theta}_{t-k}, \boldsymbol{W}_{t-k+1}\right]\end{equation}
    $$
    
    이고,
    
    $$
    \begin{equation}\left(\boldsymbol{\theta}_{t-k} \mid D_{t-k}\right) \sim \mathrm{N}\left[\boldsymbol{m}_{t-k}, \boldsymbol{C}_{t-k}\right]\end{equation}
    $$
    
    이다. 여기서, Joint distribution을 정의하고 그것의 conditional mean과 variance를 구하는 방법으로, 식 (7)에서,
    
    $$
    \begin{equation}\left(\boldsymbol{\theta}_{t-k} \mid \boldsymbol{\theta}_{t-k+1}, D_{t-k}\right) \sim \mathrm{N}\left[\boldsymbol{h}_t(k), \boldsymbol{H}_t(k)\right]\end{equation}
    $$
    
    을 얻을 수 있다. 여기서, /math