---
title: "Continuous Parameter Stochastic Process Theory"
description: Spatial stochastic process의 기본 개념부터 covariance function, kriging predictor까지 정리한 노트입니다.
author: starrypark
date: 2024-03-07 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [spatial statistics, stochastic process, covariance function, kriging, geostatistics]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

Spatial statistics를 공부하다 보면 결국 이 질문으로 돌아와요.

> "공간상의 두 점이 얼마나 비슷한가?"

이걸 수학적으로 다루는 게 이 챕터의 핵심이에요. 단순히 "가까우면 비슷하다"는 직관을 넘어서, 그 구조를 정밀하게 정의하고 다루는 방법을 소개합니다.

PhD 과정에서 이 내용이 중요한 이유는 단 하나예요. 나중에 kriging이나 Gaussian process regression을 쓸 때, 모든 게 여기서 나오거든요.

---

## 2.1 Spatial Stochastic Process란?

**Spatial stochastic process**는 간단히 말하면 공간 위에 정의된 확률 변수들의 모음이에요.

$$Y(s) = Y(s, \omega), \quad s \in D \subseteq \mathbb{R}^d$$

여기서 $s$는 공간 위치, $\omega$는 확률 공간의 event예요.

이걸 두 가지 방향으로 볼 수 있어요.

- **위치 $s$ 고정**: 특정 위치들 $\{s_1, \ldots, s_n\}$에서의 값 $(Y(s_1), \ldots, Y(s_n))^T$는 하나의 random vector가 됩니다.
- **event $\omega$ 고정**: 모든 위치에서의 값 $Y(s, \omega)$는 하나의 실현값(realization)이 됩니다.

이 과정 전체의 분포는 **finite-dimensional joint distribution**의 모임으로 나타낼 수 있어요.

$$F(y_1, \ldots, y_n;\, s_1, \ldots, s_n) = P(Y(s_1) \leq y_1, \ldots, Y(s_n) \leq y_n)$$

---

## 2.2 Stationarity

### Strict Stationarity

**Strictly stationary**란 모든 위치를 $h$만큼 이동해도 분포가 바뀌지 않는 성질이에요.

$$F(y_1, \ldots, y_n;\, s_1+h, \ldots, s_n+h) = F(y_1, \ldots, y_n;\, s_1, \ldots, s_n)$$

직관적으로, "공간 어디서 봐도 통계적 성질이 같다"는 거예요. 하지만 이 조건은 너무 강해서 실용적으로 쓰기 어렵습니다.

### Weak Stationarity (Second-Order Stationarity)

그래서 실제로는 **weakly stationary** (또는 second-order stationary) 조건을 주로 씁니다.

두 가지 조건이에요.

1. 평균이 위치에 무관하게 일정: $E(Y(s)) = \mu$
2. Covariance가 위치 차이에만 의존: $\text{Cov}(Y(s), Y(s+h)) = C(h)$

여기서 $C(h)$를 **covariance function**이라고 해요. 두 점 사이의 공간적 관계를 오직 거리 벡터 $h$로만 표현한다는 게 핵심입니다.

### Intrinsic Stationarity

**Intrinsically stationary**는 한 단계 더 느슨한 조건이에요.

$$E[Y(s+h) - Y(s)]^2 = 2\gamma(h), \quad E[Y(s+h) - Y(s)] = 0$$

$2\gamma(h)$를 **variogram**, $\gamma(h)$를 **semi-variogram**이라고 합니다.

Weakly stationary이면 intrinsically stationary이기도 해요. 이 경우엔 다음 관계가 성립해요.

$$\gamma(h) = C(0) - C(h)$$

이게 왜 유용하냐면, variogram은 covariance function보다 추정하기가 훨씬 쉽거든요.

**예시: Brownian motion**

Brownian motion $W(t)$는 intrinsically stationary이지만 stationary는 아닌 대표적인 예입니다. 증분이 독립이고, $\text{Var}[W(t_{i+1}) - W(t_i)] = t_{i+1} - t_i$이에요. 즉, 분산이 시간 간격에 비례하기 때문에 covariance가 위치 차이에만 의존하지 않아요.

---

## 2.3 Nugget Effect

고전적인 geostatistical model은 과정을 세 부분으로 분해해요.

$$Y(s) = \mu(s) + \eta(s) + \epsilon(s)$$

- $\mu(s)$: 결정론적이고 부드러운 mean function
- $\eta(s)$: 평균 0, 연속적인 공간 과정
- $\epsilon(s)$: 공간적으로 uncorrelated한 오차항

오차항 $\epsilon(s)$의 covariance는 이렇게 정의돼요.

$$\text{Cov}(\epsilon(s), \epsilon(s+h)) = \begin{cases} \sigma^2 \geq 0, & h = 0 \\ 0, & h \neq 0 \end{cases}$$

$\sigma^2 > 0$이면 이를 **nugget effect**라고 해요. Semi-variogram으로는 $\lim_{h \to 0} \gamma(h) = c_0 > 0$으로 정의합니다.

Nugget effect의 원인은 두 가지예요.

- **Measurement error**: 같은 위치에서도 측정 오차가 생길 수 있어요.
- **Microscale variability**: 측정 스케일보다 작은 곳에서 변동이 있을 수 있어요.

단, $L_2$-continuous process에서는 microscale variability가 원칙적으로 존재하지 않아요. 따라서 그 경우엔 nugget effect의 유일한 원인은 measurement error입니다.

---

## 2.4 Bochner's Theorem

Weakly stationary process의 covariance function $C$는 아무 함수나 될 수 없어요. Covariance matrix가 nonnegative definite이어야 하거든요.

이 조건을 만족하는 함수를 **positive definite**이라고 해요. 그리고 어떤 함수가 valid한 covariance function이 되는 조건을 **Bochner's theorem**이 정확히 알려줍니다.

**Theorem (Bochner):** 실수값 연속 함수 $C$가 positive definite인 것은, $\mathbb{R}^d$ 위의 symmetric nonnegative measure $F$의 Fourier transform인 것과 동치입니다.

$$C(h) = \int_{\mathbb{R}^d} \exp(ih^T x)\, dF(x) = \int_{\mathbb{R}^d} \cos(h^T x)\, dF(x)$$

이 표현을 covariance function의 **spectral representation**이라고 해요.

직관적으로 말하면, characteristic function의 실수 부분은 항상 valid한 correlation function이 될 수 있다는 거예요.

Measure $F$가 Lebesgue density $f$를 가지면, $f$를 **spectral density**라고 부릅니다. $C$가 $\mathbb{R}^d$에서 integrable하면 Fourier inversion formula로 역방향 관계도 쓸 수 있어요.

$$f(x) = \frac{1}{(2\pi)^d} \int_{\mathbb{R}^d} \cos(h^T x)\, C(h)\, dh$$

---

## 2.5 Isotropic Covariance Function

### 정의

**Isotropic** process란, covariance function이 거리 벡터 $h$의 방향이 아니라 크기 $\|h\|$에만 의존하는 경우예요.

$$C(h) = \phi(\|h\|), \quad h \in \mathbb{R}^d$$

방향에 무관하게 같은 거리면 같은 correlation을 갖는다는 거예요.

### Class $\Phi_d$

$\Phi_d$를 $\mathbb{R}^d$에서 valid한 isotropic covariance function을 생성하는 함수 $\phi$들의 class라고 하면,

$$\Phi_1 \supseteq \Phi_2 \supseteq \cdots \quad \text{and} \quad \Phi_d \downarrow \Phi_\infty = \bigcap_{d \geq 1} \Phi_d$$

차원이 높아질수록 조건이 강해져요. 즉 고차원에서 valid하면 저차원에서도 valid하지만, 역은 성립하지 않아요.

**Theorem:** $\phi \in \Phi_d$인 것은 다음 형태로 표현 가능한 것과 동치입니다.

$$\phi(t) = \int_{[0,\infty)} \Omega_d(rt)\, dF_0(r)$$

여기서 $F_0$는 positive half-axis 위의 확률 측도이고, generator $\Omega_d$는 아래와 같아요.

$$\Omega_d(t) = \Gamma(d/2)\left(\frac{2}{t}\right)^{(d-2)/2} J_{(d-2)/2}(t)$$

$J_\alpha$는 first kind Bessel function이에요.

몇 가지 특수한 경우를 보면:

| Dimension $d$ | $\Omega_d(t)$ | Lower bound |
|---|---|---|
| 1 | $\cos t$ | -1 |
| 2 | $J_0(t)$ | -0.403 |
| 3 | $t^{-1} \sin t$ | -0.218 |
| $\infty$ | $\exp(-t^2)$ | 0 |

차원이 높아질수록 negative correlation의 허용 폭이 줄어드는 게 보이죠?

### Class $\Phi_\infty$: Schoenberg's Theorem

$\Phi_\infty$에 속하는 함수는 훨씬 깔끔하게 특성화할 수 있어요.

**Theorem (Schoenberg):** $\phi \in \Phi_\infty$인 것은 다음 형태로 표현 가능한 것과 동치입니다.

$$\phi(t) = \int_{[0,\infty)} \exp(-r^2 t^2)\, dF(r)$$

즉, $\Phi_\infty$의 원소들은 squared exponential generator $\Omega_\infty(t) = \exp(-t^2)$의 scale mixture예요. 이 함수들은 strictly positive, strictly decreasing이고, origin 외부에서 무한히 미분 가능합니다.

---

## 2.6 Smoothness Properties

Covariance function의 origin 근방 거동이 sample path의 smoothness를 결정해요.

### Mean Square Continuity

**Mean square continuous**란:

$$E(Y(s) - Y(s+h))^2 \to 0 \quad \text{as } \|h\| \to 0$$

Second-order stationary process에서 이는 covariance function이 origin에서 연속인 것과 동치예요.

이 부분이 처음엔 헷갈릴 수 있어요. Mean square continuous하다고 해서 sample path가 연속인 건 아니에요. 반대도 마찬가지예요. 두 개념은 별개입니다.

### Fractal Dimension과 Smoothness

Isotropic process에서 $\phi(t)$의 origin 근방 거동이 sample path의 smoothness를 결정해요. 구체적으로, 어떤 $\alpha \in (0, 2]$에 대해

$$1 - \phi(t) \sim t^\alpha \quad \text{as } t \downarrow 0$$

이면, associated Gaussian process의 realization은 fractal dimension

$$D = d + 1 - \frac{\alpha}{2}$$

을 가집니다. $\alpha$가 클수록 $D$가 작아지고, sample path가 더 부드럽습니다.

**Theorem:** $\phi(u)$가 위 조건을 만족하고 $\alpha = 2$이면, 대칭적으로 연장한 $c(u) = \phi(|u|)$가 origin에서 $2m$번 미분 가능한 것은 associated Gaussian process의 sample path가 $m$번 미분 가능한 것과 동치입니다.

---

## 2.7 Isotropic Covariance Function의 예시들

### Matérn Class

실용적으로 가장 많이 쓰이는 family예요.

$$\phi(t) = \frac{2^{1-\nu}}{\Gamma(\nu)} \left(\frac{t}{\theta}\right)^\nu K_\nu\left(\frac{t}{\theta}\right), \quad \nu > 0,\ \theta > 0$$

$K_\nu$는 modified Bessel function이에요. 파라미터 해석은 이래요.

- $\nu$: **smoothness parameter** — sample path가 $m$번 미분 가능 iff $m < \nu$
- $\theta$: **scale parameter**

Matérn class는 $\Phi_\infty$에 속하고, 모든 차원 $d \geq 1$에서 valid합니다.

특수 케이스 몇 가지를 보면:

| $\nu$ | Correlation function |
|---|---|
| $1/2$ | $\exp(-t)$ |
| $1$ | $t K_1(t)$ |
| $3/2$ | $(1+t)\exp(-t)$ |
| $5/2$ | $\left(1 + t + \frac{t^2}{3}\right)\exp(-t)$ |

$\nu$가 커질수록 더 부드러운 process를 모델링해요.

### Powered Exponential Family

$$\phi(t) = \exp\left(-\left(\frac{t}{\theta}\right)^\alpha\right), \quad 0 < \alpha \leq 2,\ \theta > 0$$

$\Phi_\infty$에 속하고, 단순하다는 장점이 있어요. 하지만 smoothness 측면에서 한계가 있어요.

- $\alpha = 2$: infinitely differentiable (squared exponential)
- $\alpha < 2$: 전혀 미분 불가

Matérn class처럼 중간 단계 smoothness를 조절하기 어렵습니다.

### Cauchy Family

$$\phi(t) = \left(1 + \left(\frac{t}{\theta}\right)^\alpha\right)^{-\beta/\alpha}, \quad 0 < \alpha \leq 2,\ \beta > 0,\ \theta > 0$$

여기엔 **long-memory parameter** $\beta$가 추가돼요. $t \to \infty$일 때 $\phi(t) \sim t^{-\beta}$로 power law 감소를 보입니다. $\beta$가 작을수록 long-range dependence가 강해요.

Spatial interpolation보다는 estimation/inference 문제에서 중요하게 작용합니다.

### Hole Effect

Moderate negative correlation이 관측되는 경우를 **hole effect**라고 해요. 이를 반영하는 대표적인 함수는 exponentially damped cosine입니다.

$$\phi(t) = \exp\left(-\tau \frac{t}{\theta}\right) \cos\left(\frac{t}{\theta}\right), \quad \tau \geq \frac{1}{\tan\frac{\pi}{2d}},\ \theta > 0$$

Decay parameter $\tau$에 대한 조건이 $\Phi_d$에 속하기 위한 필요충분조건이에요.

### Spherical Covariance Function

**Compact support**를 가지는 대표적인 예로, 계산 효율이 좋아요.

$$\phi(t) = \begin{cases} 1 - \dfrac{3}{2}\dfrac{t}{\theta} + \dfrac{1}{2}\left(\dfrac{t}{\theta}\right)^3 & \text{if } t < \theta \\ 0 & \text{otherwise} \end{cases}$$

단, $d \leq 3$에서만 valid해요. 고차원에서는 사용할 수 없어요.

---

## 2.8 Prediction Theory: Kriging

Spatial statistics에서 가장 핵심적인 문제는 **prediction**이에요. 관측하지 않은 위치 $s$에서 $Y(s)$를 어떻게 예측할까요?

Least squares 관점에서 optimal predictor는 조건부 기댓값이에요.

$$\hat{Y}(s) = E(Y(s) \mid Y(s_1) = y_1, \ldots, Y(s_n) = y_n)$$

일반적으로 closed form이 없어요. 하지만 multivariate normal을 가정하면 이야기가 달라집니다.

### Gaussian Conditional Distribution

$(U, V)^T$가 multivariate normal을 따를 때:

$$\begin{pmatrix} U \\ V \end{pmatrix} \sim \mathcal{N}\left(\begin{pmatrix} \mu_U \\ \mu_V \end{pmatrix},\ \begin{pmatrix} \Sigma_{UU} & \Sigma_{UV} \\ \Sigma_{VU} & \Sigma_{VV} \end{pmatrix}\right)$$

조건부 분포는:

$$(V \mid U) \sim \mathcal{N}\left(\mu_V + \Sigma_{VU}\Sigma_{UU}^{-1}(U - \mu_U),\ \Sigma_{VV} - \Sigma_{VU}\Sigma_{UU}^{-1}\Sigma_{UV}\right)$$

### Ordinary Kriging Predictor

$V = Y(s)$, $U = (Y(s_1), \ldots, Y(s_n))^T$로 놓고, mean $\mu$, covariance function $C$를 가진 second-order stationary process를 가정하면:

$$\hat{Y}(s) = \mu + \bigl(C(s-s_1), \ldots, C(s-s_n)\bigr) \bigl[C(s_i - s_j)\bigr]^{-1} \begin{pmatrix} Y(s_1) - \mu \\ \vdots \\ Y(s_n) - \mu \end{pmatrix}$$

이게 바로 **ordinary kriging predictor**예요.

중요한 포인트가 있어요. 이 predictor는 Gaussian 가정이 없어도 least squares 의미에서 **best linear predictor**입니다.

예측 surface는 두 가지 성질을 가져요.

- Nugget effect가 없으면 관측값을 정확히 통과해요 (interpolation).
- 관측 위치 근방에서 예측의 거동이 covariance function $C$의 거동을 그대로 이어받아요.

이 마지막 성질이 왜 covariance function의 smoothness가 중요한지 설명해 줘요. 결국 예측 결과의 부드러움은 선택한 covariance function에 달려 있거든요.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 2.
