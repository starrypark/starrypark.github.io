---
title: Dynamic Linear Model
description: Dynamic Linear model의 기본적인 개념과 분포 개념 설명
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

observation vector $\mathbf{Y}_t$가 $(r \times 1)$의 벡터로 time series를 이룬다고 가정하자.

### Definition 4.1.

일반적인 normal DLM은 각 시각 $t$마다 다음과 같은 quadruples들의 set들로 표현된다.

$$
\left\{\mathbf{F}, \mathbf{G}, \mathbf{V}, \mathbf{W} \right\}_t =\left\{\mathbf{F}_t, \mathbf{G}_t, \mathbf{V}_t, \mathbf{W}_t \right\}
$$

여기서 $\mathbf{F}_t$는 $(n\times r)$, $\mathbf{G}_t$는 $(n\times n)$, $\mathbf{V}_t$는 $(r\times r)$, $\mathbf{F}_t$는 $(n\times n)$ 행렬이다. 뒤의 두 개는 분산행렬이다. 그리고 이것은 

$$
\begin{equation}\begin{aligned}&\left(\mathbf{Y}_t \mid \mathbf{\theta}_t\right) \sim \mathrm{N}\left[\mathbf{F}_t^{\prime} \mathbf{\theta}_t, \mathbf{V}_t\right]\\&\left(\mathbf{\theta}_t \mid \mathbf{\theta}_{t-1}\right) \sim \mathrm{N}\left[\mathbf{G}_t \mathbf{\theta}_{t-1}, \mathbf{W}_t\right]\end{aligned}\end{equation}
$$

로 나타낼 수 있다.  혹은,

$$
\begin{equation}\begin{aligned}\mathbf{Y}_t=\mathbf{F}_t^{\prime} \mathbf{\theta}_t+\mathbf{\nu}_t, & \mathbf{\nu}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{V}_t\right], \\\mathbf{\theta}_t=\mathbf{G}_t \mathbf{\theta}_{t-1}+\mathbf{\omega}_t, & \mathbf{\omega}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{W}_t\right]\end{aligned}\end{equation}
$$

으로 나타낼 수 있다. 여기서 $\mathbf{\nu}_t$와 $\mathbf{\omega}_t$는 서로 독립이라고 가정한다.

# 4.3. Updating Equations

### Definition 4.3.

각 시각  $t$마다, univariate DLM은 다음과 같이 정의된다.

$$
\begin{equation}\begin{array}{lcl}\text { Observation equation: } & Y_t=\mathbf{F}_t^{\prime} \mathbf{\theta}_t+\nu_t, & \nu_t \sim \mathrm{N}\left[0, V_t\right], \\\text { System equation: } & \mathbf{\theta}_t=\mathbf{G}_t \mathbf{\theta}_{t-1}+\mathbf{\omega}_t, & \mathbf{\omega}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{W}_t\right], \\\text { Initial information: } & \left(\mathbf{\theta}_0 \mid D_0\right) \sim \mathrm{N}\left[\mathbf{m}_0, \mathbf{C}_0\right], &\end{array}\end{equation}
$$

여기서 $ \mathbf{m}_0, \mathbf{C}_0 $ 은 prior moment들이다. 또한, Information set $D_t =\left\{Y_t, D_{t-1} \right\}$으로 정의된다.

그러면, 다음 정리를 통해 equation을 Update할 수 있다.

### Theorem 4.1

Definition 4.3의 univariate DLM에서, 1step 사후분포 예측은 다음과 같이 진행할 수 있다.

1. Posterior at $t-1$
    
    $$
    \begin{equation}\left(\mathbf{\theta}_{t-1} \mid D_{t-1}\right) \sim \mathrm{N}\left[\mathbf{m}_{t-1}, \mathbf{C}_{t-1}\right]\end{equation}
    $$
    
2. Prior at $t$
    
    $$
    \left(\mathbf{\theta}_t \mid D_{t-1}\right) \sim \mathrm{N}\left[\mathbf{a}_t, \mathbf{R}_t\right]
    $$
    
    여기서, $\mathbf{a}_t=\mathbf{G}_t \mathbf{m}_{t-1} , \ \mathbf{R}_t=\mathbf{G}_t \mathbf{C}_{t-1} \mathbf{G}_t^{\prime}+\mathbf{W}_t$이다.
    
3. One-step forecast
    
    $$
    \left(Y_{t} \mid D_{t-1}\right) \sim \mathrm{N}\left[f_{t}, Q_{t}\right]
    $$
    
    여기서, $f_{t}=\mathbf{F}_{t}^{\prime} \mathbf{a}_{t} , \  Q_{t}=\mathbf{F}_{t}^{\prime} \mathbf{R}_{t} \mathbf{F}_{t}+V_{t}$이다.
    
4. Posterior at $t$
    
    $$
    \left(\mathbf{\theta}_{t} \mid D_{t}\right) \sim \mathrm{N}\left[\mathbf{m}_{t}, \mathbf{C}_{t}\right]
    $$
    
    여기서, $\mathbf{m}_{t}=\mathbf{a}_{t}+\mathbf{A}_{t} e_{t} ,\ \mathbf{C}_{t}=\mathbf{R}_{t}-\mathbf{A}_{t} Q_{t} \mathbf{A}_{t}^{\prime}\mathbf{A}_{t}=\mathbf{R}_{t} \mathbf{F}_{t} Q_{t}^{-1} ,\  e_{t}=Y_{t}-f_{t}$이다.
    

(증명) 

1~3까진 Easy. 4는 Conditional Normal 분포를 생각해보자.

# 4.4. Forecast Distributions

### Definition 4.4

Forecast function $f_t(k)$는 모든 시간 $t$, 모든 음이 아닌 정수 $k$에 대해 다음과 같이 정의된다.

$$
\begin{equation}f_t(k)=\mathrm{E}\left[\mu_{t+k} \mid D_t\right]=\mathrm{E}\left[\mathbf{F}_{t+k}^{\prime} \mathbf{\theta}_{t+k} \mid D_t\right]\end{equation}
$$

여기서 $\mu_v=\mathbf{F}_v^{\top} \mathbf{\theta}_v$는 mean response function이라고 부른다.

### Theorem 4.2

$0 \le j <k$인 모든 $j$에 대해, 각 time point $t$에서 다음이 성립한다.

- (a) State distribution : $\left(\mathbf{\theta}_{t+k} \mid D_t\right) \sim \mathrm{N}\left[\mathbf{a}_t(k), \mathbf{R}_t(k)\right]$
- (b) Forecast distribution :  $\left(Y_{t+k} \mid D_t\right) \sim \mathrm{N}\left[f_t(k), Q_t(k)\right]$
- (c) State Covariance : $\mathrm{C}\left[\mathbf{\theta}_{t+k}, \mathbf{\theta}_{t+j} \mid D_t\right]=\mathbf{C}_t(k, j)$
- (d) Obsn. Covariance : $\mathrm{C}\left[Y_{t+k}, Y_{t+j} \mid D_t\right]=\mathbf{F}_{t+k}^{\prime} \mathbf{C}_t(k, j) \mathbf{F}_{t+j}$
- (e) Other Covariance : $\begin{aligned}& \mathrm{C}\left[\mathbf{\theta}_{t+k}, Y_{t+j} \mid D_t\right]=\mathbf{C}_t(k, j) \mathbf{F}_{t+j} \\& \mathrm{C}\left[Y_{t+k}, \mathbf{\theta}_{t+j} \mid D_t\right]=\mathbf{F}_{t+k}^{\prime} \mathbf{C}_t(k, j)\end{aligned}$

여기서, $f_t(k)=\mathbf{F}_{t+k}^{\top} \mathbf{a}_t(k) \quad \text { and } \quad Q_t(k)=\mathbf{F}_{t+k}^{\top} \mathbf{R}_t(k) \mathbf{F}_{t+k}+V_{t+k}$이고, 이는 다음과 같이 순차적으로 계산이 가능하다.

$$
\begin{aligned}\mathbf{a}_t(k) & =\mathbf{G}_{t+k} \mathbf{a}_t(k-1) \\\mathbf{R}_t(k) & =\mathbf{G}_{t+k} \mathbf{R}_t(k-1) \mathbf{G}_{t+k}^{\top}+\mathbf{W}_{t+k} \\\mathbf{C}_t(k, j) & =\mathbf{G}_{t+k} \mathbf{C}_t(k-1, j), k=j+1, \ldots\end{aligned}
$$

- Proof of Theorem 4.2
    
    (증명)
    
    먼저 $n \times n$ 행렬 $\mathbf{H}_{t+k}(r)=\mathbf{G}_{t+k} \mathbf{G}_{t+k-1} \ldots \mathbf{G}_{t+k-r+1}$를 모든 $t$와 $r \le k$에 대해 정의하자. 또한, $\mathbf{H}_{t+k}(0)=\mathbf{I}$라고 가정하자.
    
    여기서 $\mathbf{\theta}_t=\mathbf{G}_t \mathbf{\theta}_{t-1}+\mathbf{\omega}_t$를 반복적으로 적용하여 계산할 시,
    
    $$
    \mathbf{\theta}_{t+k}=\mathbf{H}_{t+k}(k) \mathbf{\theta}_t+\sum_{r=1}^k \mathbf{H}_{t+k}(k-r) \mathbf{\omega}_{t+r}
    $$
    
    를 얻을 수 있다. 이 식을 통해, 
    
    $$
    \left(\mathbf{\theta}_{t+k} \mid D_t\right) \sim \mathrm{N}\left[\mathbf{a}_t(k), \mathbf{R}_t(k)\right]
    $$
    
    를 얻을 수 있다. 여기서, $\mathbf{a}_t(k)=\mathbf{H}_{t+k}(k) \mathbf{m}_t=\mathbf{G}_{t+k} \mathbf{a}_t(k-1)$이며 
    
    $$
    \begin{aligned}\mathbf{R}_t(k) & =\mathbf{H}_{t+k}(k) \mathbf{C}_t \mathbf{H}_{t+k}(k)^{\top}+\sum_{r=1}^k \mathbf{H}_{t+k}(k-r) \mathbf{W}_{t+r} \mathbf{H}_{t+k}(k-r)^{\top} \\& =\mathbf{G}_{t+k} \mathbf{R}_t(k-1) \mathbf{G}_{t+k}^{\top}+\mathbf{W}_{t+k}\end{aligned}
    $$
    
    이다. 또한 Initial Value는 $\mathbf{a}_t(0)=\mathbf{m}_t, \text { and } \mathbf{R}_t(0)=\mathbf{C}_t$이다. 여기서 (a) 증명이 끝난다.
    
    (b) 역시, Observational Equation을 통해서 얻을 수 있다. 어차피 간단한 Matrix 계산이기에 나머지도 생략.
    

# 4.7 Filtering Recurrences

최근 데이터를 사용하여 상태 벡터의 이전 값에 대한 추론을 수정하는 것을 **Filtering**이라고 한다.  $k \ge 1$$k \ge1$$\left(\mathbf{\theta}_{t-k} \mid D_t\right)$를 **$k$-step filtered distribution**이라고 한다.

이 filtered distribution을 다음 정리를 이용해서 유도할 것이다.

### Theorem 4.4.

Univariate DLM $\left\{\mathbf{F}_t, \mathbf{G}_t, V_t, \mathbf{W}_t\right\}$에서, 모든 $t$에 대해,

$$
\mathbf{B}_t=\mathbf{C}_t \mathbf{G}_{t+1}^{\top} \mathbf{R}_{t+1}^{-1}
$$

로 정의하자. 그러면 모든 $0 \le k < t$인 $k$에 대해, filtered marginal distribution은 다음과 같이 주어진다.

$$
\left(\mathbf{\theta}_{t-k} \mid D_t\right) \sim \mathrm{N}\left[\mathbf{a}_t(-k), \mathbf{R}_t(-k)\right]
$$

여기서, $\mathbf{a}_t(-k)=\mathbf{m}_{t-k}+\mathbf{B}_{t-k}\left[\mathbf{a}_t(-k+1)-\mathbf{a}_{t-k+1}\right]$이고,  $\mathbf{R}_t(-k)=\mathbf{C}_{t-k}+\mathbf{B}_{t-k}\left[\mathbf{R}_t(-k+1)-\mathbf{R}_{t-k+1}\right] \mathbf{B}_{t-k}^{\top}$이다.

Initial value는 $\mathbf{a}_t(0)=\mathbf{m}_t \quad \text { and } \quad \mathbf{R}_t(0)=\mathbf{C}_t$라고 가정한다.

- Proof of Theorem 4.4 (미완)
    
    Filtered density는 다음은 식에서 순차적으로 계산할 수 있다.
    
    $$
    \begin{equation}\begin{aligned}p\left(\theta_{t-k} \mid D_k\right) & =\int p\left(\theta_{t-k}, \theta_{t-k+1} \mid D_k\right) d \theta_{t-k+1} \\& =\int p\left(\theta_{t-k} \mid \theta_{t-k+1}, D_k\right) p\left(\theta_{t-k+1} \mid D_k\right) d\theta_{t-k+1}\end{aligned}\end{equation}
    $$
    
    이를 통해, Induction을 이용해 정리를 증명해 보자.
    
    먼저 $k-1$에서 식이 성립한다고 가정하자. 그러면 
    
    $$
    \left(\mathbf{\theta}_{t-k+1} \mid D_t\right) \sim \mathrm{N}\left[\mathbf{a}_t(-k+1), \mathbf{R}_t(-k+1)\right]
    $$
    
    가 성립하므로 식 (6)에서 두 번째 확률은 구할 수 있다.
    
    첫 번째 확률의 경우, 베이즈 정리를 이용하여,
    
    $$
    p\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_t\right)=\frac{p\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_{t-k}\right) p\left(\mathbf{Y} \mid \mathbf{\theta}_{t-k}, \mathbf{\theta}_{t-k+1}, D_{t-k}\right)}{p\left(\mathbf{Y} \mid \mathbf{\theta}_{t-k+1}, D_{t-k}\right)}
    $$
    
    다음과 같은 식을 얻을 수 있다. 여기서 $\mathbf{Y}=\left\{Y_{t-k+1}, \ldots, Y_t\right\}$이다. 여기서 $\mathbf{\theta}_{t-k+1}$가 주어져 있는 경우, $\mathbf{Y} \perp \mathbf{\theta}_{t-k}$의 조건부 독립을 이용하여, 뒤의 항은 cancel-out이 가능하고,
    
    $$
    \begin{equation}p\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_{t-k}\right) \propto p\left(\mathbf{\theta}_{t-k} \mid D_{t-k}\right) p\left(\mathbf{\theta}_{t-k+1} \mid \mathbf{\theta}_{t-k}, D_{t-k}\right)\end{equation}
    $$
    
    가 베이즈 정리에 의해 성립한다. 여기서,
    
    $$
    \begin{equation}\left(\mathbf{\theta}_{t-k+1} \mid \mathbf{\theta}_{t-k}, D_{t-k}\right) \sim \mathrm{N}\left[\mathbf{G}_{t-k+1} \mathbf{\theta}_{t-k}, \mathbf{W}_{t-k+1}\right]\end{equation}
    $$
    
    이고,
    
    $$
    \begin{equation}\left(\mathbf{\theta}_{t-k} \mid D_{t-k}\right) \sim \mathrm{N}\left[\mathbf{m}_{t-k}, \mathbf{C}_{t-k}\right]\end{equation}
    $$
    
    이다. 여기서, Joint distribution을 정의하고 그것의 conditional mean과 variance를 구하는 방법으로, 식 (7)에서,
    
    $$
    \begin{equation}\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_{t-k}\right) \sim \mathrm{N}\left[\mathbf{h}_t(k), \mathbf{H}_t(k)\right]\end{equation}
    $$
    
    을 얻을 수 있다. 여기서, /math