---
title: "Dynamic Linear Model (DLM) 기초: 정의부터 Filtering까지"
date: 2024-06-11 12:26:00 +0900
categories: [Time-Series Analysis, Bayesian Analysis]
tags: [dynamic linear model, dlm, bayesian forecasting, time series, statistics]
math: true
---

이 포스팅은 West & Harrison(2006)의 저서 *Bayesian forecasting and dynamic models* 중 4장의 내용을 바탕으로, Dynamic Linear Model (DLM)의 기초적인 개념부터 Filtering까지의 흐름을 정리한 글입니다. 실제로 이 모델이 어떤 직관을 바탕으로 움직이는지에 초점을 맞춰 풀어보겠습니다.

## 4.2 Definition & Notations

먼저 관측 벡터(observation vector) $Y\_t$가 $(r \times 1)$ 차원으로 이루어진 시계열(time series) 데이터라고 가정해 보겠습니다.

### Definition 4.1.

일반적인 정규분포를 따르는 normal DLM은 각 시점 $t$마다 다음 네 개의 행렬로 구성된 쿼드러플(quadruple) 셋으로 표현됩니다.

$$
\left\{\mathbf{F}, \mathbf{G}, \mathbf{V}, \mathbf{W} \right\}_t = \left\{\mathbf{F}_t, \mathbf{G}_t, \mathbf{V}_t, \mathbf{W}_t \right\}
$$

여기서 $\mathbf{F}\_t$는 $(n \times r)$, $\mathbf{G}\_t$는 $(n \times n)$ 크기의 행렬입니다. 뒤의 두 개 $\mathbf{V}\_t$와 $\mathbf{W}\_t$는 각각 $(r \times r)$, $(n \times n)$ 크기의 분산 행렬(variance matrix)을 의미합니다. 

이를 상태 공간 모델의 형태로 풀어서 쓰면 두 개의 방정식으로 나눌 수 있습니다.

$$
\begin{aligned}
\mathbf{Y}_t &= \mathbf{F}_t^{\prime} \mathbf{\theta}_t + \mathbf{\nu}_t, \quad \mathbf{\nu}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{V}_t\right] \\
\mathbf{\theta}_t &= \mathbf{G}_t \mathbf{\theta}_{t-1} + \mathbf{\omega}_t, \quad \mathbf{\omega}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{W}_t\right]
\end{aligned}
$$

위쪽 식이 우리가 실제로 보는 데이터를 설명하는 관측 방정식(observation equation)이고, 아래쪽 식이 숨겨진 상태(state)가 시간에 따라 어떻게 변하는지 설명하는 시스템 방정식(system equation)입니다. 이때 관측 오차인 $\mathbf{\nu}\_t$와 시스템 오차인 $\mathbf{\omega}\_t$는 서로 독립이라고 가정합니다.

## 4.3. Updating Equations

새로운 데이터가 들어왔을 때, 모델의 상태를 어떻게 업데이트할까요? 이 과정이 DLM의 가장 핵심적인 메커니즘입니다.

### Definition 4.3.

각 시점 $t$마다 univariate DLM은 관측 방정식과 시스템 방정식, 그리고 초기 정보(initial information)로 정의됩니다. 모델 세팅은 다음과 같이 주어집니다.

$$
\begin{array}{lcl}
\text{Observation equation:} & Y_t = \mathbf{F}_t^{\prime} \mathbf{\theta}_t + \nu_t, & \nu_t \sim \mathrm{N}\left[0, V_t\right] \\
\text{System equation:} & \mathbf{\theta}_t = \mathbf{G}_t \mathbf{\theta}_{t-1} + \mathbf{\omega}_t, & \mathbf{\omega}_t \sim \mathrm{N}\left[\mathbf{0}, \mathbf{W}_t\right] \\
\text{Initial information:} & \left(\mathbf{\theta}_0 \mid D_0\right) \sim \mathrm{N}\left[\mathbf{m}_0, \mathbf{C}_0\right] &
\end{array}
$$

여기서 $\mathbf{m}\_0$와 $\mathbf{C}\_0$는 초기의 prior moment를 뜻합니다. 또한 누적된 정보 셋(information set)은 $D\_t = \\\{Y\_t, D\_\{t-1\}\\\}$ 처럼 과거부터 현재까지의 데이터를 재귀적으로 묶어 정의합니다.

이제 Theorem 4.1을 통해 1-step 사후분포가 어떻게 순차적으로 업데이트되는지 따라가 보겠습니다.

### Theorem 4.1

Univariate DLM에서 1-step 사후분포 예측은 직관적으로 이해하기 쉬운 4단계를 거칩니다.

1. **Posterior at $t-1$**: 직전 시점의 정보를 바탕으로 파악한 상태입니다.
$$
\left(\mathbf{\theta}_{t-1} \mid D_{t-1}\right) \sim \mathrm{N}\left[\mathbf{m}_{t-1}, \mathbf{C}_{t-1}\right]
$$

2. **Prior at $t$**: 과거 정보를 바탕으로 현재 상태가 어떨지 예측해 봅니다.
$$
\left(\mathbf{\theta}_t \mid D_{t-1}\right) \sim \mathrm{N}\left[\mathbf{a}_t, \mathbf{R}_t\right]
$$
이때 평균과 분산은 시스템 방정식을 따라 기댓값을 취한 결과인 $\mathbf{a}\_t = \mathbf{G}\_t \mathbf{m}\_\{t-1\}$, $\mathbf{R}\_t = \mathbf{G}\_t \mathbf{C}\_\{t-1\} \mathbf{G}\_t^{\prime} + \mathbf{W}\_t$ 로 계산됩니다.

3. **One-step forecast**: 앞서 예측한 상태를 이용해, 우리가 실제로 보게 될 관측치를 미리 예측합니다.
$$
\left(Y_t \mid D_{t-1}\right) \sim \mathrm{N}\left[f_t, Q_t\right]
$$
여기서 관측 예측치 $f\_t = \mathbf{F}\_t^{\prime} \mathbf{a}\_t$ 가 되며, 예측 분산은 $Q\_t = \mathbf{F}\_t^{\prime} \mathbf{R}\_t \mathbf{F}\_t + V\_t$ 가 됩니다.

4. **Posterior at $t$**: 마침내 실제 데이터 $Y\_t$가 들어오면, 앞선 예측과의 오차를 이용해 현재 상태를 보정(update)합니다.
$$
\left(\mathbf{\theta}_t \mid D_t\right) \sim \mathrm{N}\left[\mathbf{m}_t, \mathbf{C}_t\right]
$$
보정 과정에서는 예측 오차 $e\_t = Y\_t - f\_t$ 를 사용하여, $\mathbf{m}\_t = \mathbf{a}\_t + \mathbf{A}\_t e\_t$, $\mathbf{C}\_t = \mathbf{R}\_t - \mathbf{A}\_t Q\_t \mathbf{A}\_t^{\prime}$ 로 최종 계산합니다. 이때 보정 폭을 결정하는 칼만 게인(Kalman Gain) 역할의 보정 계수는 $\mathbf{A}\_t = \mathbf{R}\_t \mathbf{F}\_t Q\_t^{-1}$ 입니다.

**증명의 흐름 (Proof Sketch)**
이 과정의 유도는 매우 간단합니다. 1~3번 과정은 정규분포의 선형 결합 성질만 이용하면 쉽게 도출되며, 4번 역시 두 변수의 결합분포(joint distribution)를 정의한 후 Conditional Normal 분포의 성질을 적용하면 바로 얻을 수 있습니다.

## 4.4. Forecast Distributions

당장 한 스텝 앞이 아니라, 먼 미래의 값은 어떻게 예측할 수 있을까요?

### Definition 4.4

Forecast function $f\_t(k)$는 임의의 현재 시점 $t$와 예측하려는 미래의 스텝 수 $k \ge 0$에 대해 기댓값으로 정의됩니다.

$$
f_t(k) = \mathrm{E}\left[\mu_{t+k} \mid D_t\right] = \mathrm{E}\left[\mathbf{F}_{t+k}^{\prime} \mathbf{\theta}_{t+k} \mid D_t\right]
$$

이때 $\mu\_v = \mathbf{F}\_v^{\top} \mathbf{\theta}\_v$ 를 mean response function이라 부릅니다.

### Theorem 4.2

$0 \le j < k$를 만족하는 모든 $j$에 대하여, 현재 시점 $t$에서 미래를 내다보는 $k$-step 예측 분포와 공분산 구조는 다음과 같이 정리할 수 있습니다.

- **(a) State distribution**: 미래 상태에 대한 예측입니다. $(\mathbf{\theta}\_\{t+k\} \mid D\_t) \sim \mathrm{N}[\mathbf{a}\_t(k), \mathbf{R}\_t(k)]$
- **(b) Forecast distribution**: 미래 관측치에 대한 예측입니다. $(Y\_\{t+k\} \mid D\_t) \sim \mathrm{N}[f\_t(k), Q\_t(k)]$
- **(c) State Covariance**: $\mathrm{C}[\mathbf{\theta}\_\{t+k\}, \mathbf{\theta}\_\{t+j\} \mid D\_t] = \mathbf{C}\_t(k, j)$
- **(d) Obsn. Covariance**: $\mathrm{C}[Y\_\{t+k\}, Y\_\{t+j\} \mid D\_t] = \mathbf{F}\_\{t+k\}^{\prime} \mathbf{C}\_t(k, j) \mathbf{F}\_\{t+j\}$
- **(e) Other Covariance**: 관측치와 상태 사이의 교차 공분산입니다.
$$
\begin{aligned}
\mathrm{C}\left[\mathbf{\theta}_{t+k}, Y_{t+j} \mid D_t\right] &= \mathbf{C}_t(k, j) \mathbf{F}_{t+j} \\
\mathrm{C}\left[Y_{t+k}, \mathbf{\theta}_{t+j} \mid D_t\right] &= \mathbf{F}_{t+k}^{\prime} \mathbf{C}_t(k, j)
\end{aligned}
$$

이 값들은 아래의 수식을 통해 순차적(recursive)으로 계산이 가능합니다.
$$
\begin{aligned}
\mathbf{a}_t(k) &= \mathbf{G}_{t+k} \mathbf{a}_t(k-1) \\
\mathbf{R}_t(k) &= \mathbf{G}_{t+k} \mathbf{R}_t(k-1) \mathbf{G}_{t+k}^{\top} + \mathbf{W}_{t+k} \\
\mathbf{C}_t(k, j) &= \mathbf{G}_{t+k} \mathbf{C}_t(k-1, j), \quad k=j+1, \ldots
\end{aligned}
$$

**증명의 흐름 (Proof Sketch)**
행렬곱을 누적해서 계산하는 보조 도구 $\mathbf{H}\_\{t+k\}(r) = \mathbf{G}\_\{t+k\} \mathbf{G}\_\{t+k-1\} \ldots \mathbf{G}\_\{t+k-r+1\}$ 를 정의해 두면 증명이 편해집니다. 초기값을 $\mathbf{H}\_\{t+k\}(0) = \mathbf{I}$ 로 두고 시스템 방정식을 전개(unrolling)하면 다음과 같은 꼴을 얻습니다.

$$
\mathbf{\theta}_{t+k} = \mathbf{H}_{t+k}(k) \mathbf{\theta}_t + \sum_{r=1}^k \mathbf{H}_{t+k}(k-r) \mathbf{\omega}_{t+r}
$$

이 식 양변에 기대치와 분산을 취하면 $\mathbf{a}\_t(k)$와 $\mathbf{R}\_t(k)$에 대한 관계식이 자연스럽게 떨어집니다. 관측치에 대한 (b) 역시 관측 방정식을 동일하게 적용하여 간단한 행렬 연산을 수행하면 유도할 수 있습니다.

## 4.7 Filtering Recurrences

새롭게 얻은 데이터를 바탕으로 모델을 앞만 향해 진행시키는 것이 아니라, 반대로 과거의 상태에 대한 추론을 다듬는 과정을 **Filtering(필터링)**이라고 합니다. 구체적으로 현재 시점 $t$의 정보를 바탕으로 한 과거 $t-k$ 시점의 분포 $(\mathbf{\theta}\_\{t-k\} \mid D\_t)$ ($k \ge 1$)를 **$k$-step filtered distribution**이라 부릅니다.

### Theorem 4.4.

모든 시점 $t$에 대해 역방향 보정에 사용할 행렬 $\mathbf{B}\_t = \mathbf{C}\_t \mathbf{G}\_\{t+1\}^{\top} \mathbf{R}\_\{t+1\}^{-1}$ 를 정의해 보겠습니다. 그러면 $0 \le k < t$를 만족하는 모든 $k$에 대해 filtered marginal distribution은 다음과 같은 정규분포를 따릅니다.

$$
\left(\mathbf{\theta}_{t-k} \mid D_t\right) \sim \mathrm{N}\left[\mathbf{a}_t(-k), \mathbf{R}_t(-k)\right]
$$

이때 평균과 분산은 역방향(backward)으로 다음과 같이 업데이트됩니다.
$\mathbf{a}\_t(-k) = \mathbf{m}\_\{t-k\} + \mathbf{B}\_\{t-k\}[\mathbf{a}\_t(-k+1) - \mathbf{a}\_\{t-k+1\}]$
$\mathbf{R}\_t(-k) = \mathbf{C}\_\{t-k\} + \mathbf{B}\_\{t-k\}[\mathbf{R}\_t(-k+1) - \mathbf{R}\_\{t-k+1\}] \mathbf{B}\_\{t-k\}^{\top}$

당연히 출발점이 되는 초기값은 $\mathbf{a}\_t(0) = \mathbf{m}\_t$, $\mathbf{R}\_t(0) = \mathbf{C}\_t$ 입니다.

**증명의 흐름 (Proof Sketch)**
이 필터링 밀도함수는 귀납법(Induction)을 통해 순차적으로 증명할 수 있습니다. 먼저 적분을 활용해 결합분포를 쪼개어 보겠습니다.

$$
\begin{aligned}
p\left(\mathbf{\theta}_{t-k} \mid D_t\right) &= \int p\left(\mathbf{\theta}_{t-k}, \mathbf{\theta}_{t-k+1} \mid D_t\right) d \mathbf{\theta}_{t-k+1} \\
&= \int p\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_t\right) p\left(\mathbf{\theta}_{t-k+1} \mid D_t\right) d\mathbf{\theta}_{t-k+1}
\end{aligned}
$$

여기서 $k-1$ 스텝에서 식이 성립한다고 가정하면 뒤쪽의 확률은 이미 아는 값이 됩니다. 앞쪽 확률은 베이즈 정리를 적용해 풉니다. 미래의 관측치들 $\mathbf{Y} = \{Y\_\{t-k+1\}, \ldots, Y\_t\}$ 이 주어졌을 때 조건부 독립성 $\mathbf{Y} \perp \mathbf{\theta}\_\{t-k\} \mid \mathbf{\theta}\_\{t-k+1\}$ 을 이용해 복잡한 항들을 소거하면 식이 극적으로 단순해집니다.

$$
p\left(\mathbf{\theta}_{t-k} \mid \mathbf{\theta}_{t-k+1}, D_{t-k}\right) \propto p\left(\mathbf{\theta}_{t-k} \mid D_{t-k}\right) p\left(\mathbf{\theta}_{t-k+1} \mid \mathbf{\theta}_{t-k}, D_{t-k}\right)
$$

결과적으로 이렇게 쪼개진 개별 정규분포들을 결합분포로 재조립한 뒤, 조건부 평균과 분산 공식을 적용하면 목표하던 필터링 수식을 깔끔하게 얻어낼 수 있습니다.