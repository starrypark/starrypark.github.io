---
title: "[Ch.3] 고전 지구통계학 방법론"
description: Geostatistical model의 구조부터 semivariogram 추정, kriging predictor까지 classical geostatistics의 전체 흐름을 정리한 노트이다.
author: starrypark
date: 2024-03-21 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [geostatistics, semivariogram, kriging, spatial prediction, variogram]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

Chapter 2에서 spatial stochastic process의 이론적 토대를 쌓았다면, Chapter 3는 그걸 실제로 어떻게 쓰는지에 대한 이야기이다.

데이터가 있고, 그 데이터를 모델링하고, 관측하지 않은 위치를 예측하는 것. 이게 classical geostatistics의 전체 흐름이다.

크게 네 단계로 나뉘는다.

1. Mean function을 임시로 추정 (OLS)
2. Semivariogram을 nonparametrically 추정
3. Semivariogram에 parametric model을 fit
4. Mean function을 다시 추정하고, kriging으로 예측

이 순서대로 따라가 볼게다.

---

## 3.1 Geostatistical Model

기본 모델은 이거다.

$$Y(s) = \mu(s) + e(s)$$

- $\mu(s) \equiv E[Y(s)]$: 결정론적이고 연속인 mean function
- $e(\cdot)$: 평균이 0인 random error process

Error process $e(\cdot)$에 stationarity assumption을 부과한다.

**Second-order stationarity**를 가정하면:

$$\text{Cov}[e(s), e(t)] = C(s - t)$$

두 점 사이의 covariance가 위치 자체가 아니라 위치 차이에만 의존한다는 거다.

**Intrinsic stationarity**를 가정하면:

$$\frac{1}{2}\text{Var}[e(s) - e(t)] = \gamma(s - t)$$

$\gamma(\cdot)$을 **semivariogram**이라고 하고, $2\gamma(\cdot)$을 **variogram**이라고 해진다.

이 모델의 아이디어는 이래다. $\mu(s)$가 large-scale variation (trend)을 담당하고, $e(s)$가 small-scale variation (spatial dependence)을 담당한다.

근데 현실에서는 이 두 성분을 명확하게 분리하기가 어렵다. 항상 어느 정도 ambiguity가 남는다는 걸 감수해야 해진다.

### Error process의 추가 분해

$e(s)$를 더 세분화할 수도 있다.

$$e(s) = \eta(s) + \epsilon(s)$$

- $\eta(\cdot)$: 공간적으로 dependent한 성분
- $\epsilon(\cdot)$: measurement error (공간적으로 uncorrelated)

### Isotropy와 Anisotropy

Intrinsically stationary process에서 semivariogram이 방향이 아니라 거리에만 의존할 때:

$$\gamma(h) = \gamma(\|h\|)$$

이를 **(intrinsically) isotropic**이라고 해진다.

반대는 **anisotropic**. 가장 다루기 쉬운 anisotropy 형태는 **geometric anisotropy**로, positive definite matrix $A$를 써서:

$$\gamma(h) = \gamma\left((h^T A h)^{1/2}\right)$$

로 표현한다. 방향마다 거리 스케일이 달라지는 거다.

---

## 3.2 Mean Function의 임시 추정

분석의 첫 번째 단계는 mean function을 추정하는 거다. 이때는 second-order dependence structure를 몰라도 되는 방법을 써야 해진다.

### Linear Mean Model

가장 흔한 선택은 linear model이다.

$$\mu(s; \beta) = X(s)^T \beta$$

$X(s)$는 위치 $s$에서 관측된 covariate vector, $\beta$는 파라미터이다.

Covariate로 쓸 수 있는 것들:

- Intercept (항상 포함)
- 위도, 경도 같은 geographic coordinates
- Coordinates의 polynomial 함수
- 기타 attribute variable

Attribute variable 데이터가 없을 때는 geographic coordinates의 polynomial만 쓴다. 이걸 **trend surface model**이라고 한다.

2차원 process ($s = (s_1, s_2)$)에서 예시를 보면:

- **1차 (planar)**: $\mu(s;\beta) = \beta_0 + \beta_1 s_1 + \beta_2 s_2$
- **2차 (quadratic)**: $\mu(s;\beta) = \beta_0 + \beta_1 s_1 + \beta_2 s_2 + \beta_{11}s_1^2 + \beta_{12}s_1 s_2 + \beta_{22}s_2^2$

$q$차 polynomial을 쓸 때는 degree $\leq q$인 모든 monomial을 포함하는 "full" polynomial을 쓰는 게 좋는다. 좌표계의 origin이나 방향 선택에 무관하게 결과가 일정하게 유지되거든.

### OLS Estimator

Linear mean function을 fit하는 표준 방법은 **Ordinary Least Squares (OLS)**예다.

$$\hat{\beta}_{\text{OLS}} = \underset{\beta}{\arg\min} \sum_{i=1}^n \left[Y(s_i) - X(s_i)^T \beta\right]^2$$

$X$가 full column rank이면 closed form solution이 있다.

$$\hat{\beta}_{\text{OLS}} = (X^T X)^{-1} X^T Y$$

단, 이 OLS는 exploratory 목적으로만 써야 해진다. OLS가 기반으로 하는 independent errors 가정이 spatial data에서는 사실 성립하지 않거든.

### OLS의 실용적 한계

1. **Multicollinearity**: Polynomial trend surface model에서 covariate들이 강하게 상관되어 OLS estimator의 분산이 커질 수 있다. Regression coefficient 자체에 관심 없다면 covariate를 centering하거나 orthogonalize해서 해결할 수 있다.

2. **Boundary distortion**: 관측값이 없는 영역에서 fitted surface가 왜곡될 수 있다. 관측 설계가 공간적으로 균일하게 분포돼 있으면 이 문제를 피할 수 있다.

3. **Outlier sensitivity**: OLS는 outlier에 민감한다. Robust regression을 쓸 수 있지만, spatial data에서 "outlier"의 정의 자체가 까다로우니 주의가 필요한다.

---

## 3.3 Semivariogram의 Nonparametric 추정

두 번째 단계는 OLS residual로부터 dependence structure를 추정하는 거다.

### Regular Grid에서의 Empirical Semivariogram

데이터가 regular rectangular grid에 있을 때, distinct lag들을 $h_1, \ldots, h_k$라고 하면:

**Definition (Empirical semivariogram):**

$$\hat{\gamma}(h_u) = \frac{1}{2N(h_u)} \sum_{s_i - s_j = h_u} \{\hat{e}(s_i) - \hat{e}(s_j)\}^2, \quad u = 1, \ldots, k$$

$\hat{e}(s_i)$는 $i$번째 위치에서의 OLS residual이다. Mean이 constant인 경우 이 estimator는 unbiased이다.

### Irregular Grid에서의 Empirical Semivariogram

데이터 위치가 irregular하면 lag들이 거의 exact하게 반복되지 않는다. 그래서 lag space $H$를 bin $H_1, \ldots, H_k$로 분할한다.

$$\hat{\gamma}(h_u) = \frac{1}{2N(H_u)} \sum_{s_i - s_j \in H_u} \{\hat{e}(s_i) - \hat{e}(s_j)\}^2, \quad u = 1, \ldots, k \tag{3.5}$$

가장 흔한 분할 방법은 **polar partition**이다. Lag space를 angle과 distance class로 나누는 방식이다.

Polar partition은 두 가지를 동시에 지원한다.

- **Directional empirical semivariogram**: 같은 angle class, 다른 distance class
- **Omnidirectional empirical semivariogram**: 모든 angle class를 합쳐서

### Bin 수의 Trade-off

Bin 수와 bin 크기는 서로 trade-off가 있다.

- Bin을 많이 쓸수록 → 각 bin의 lag 대표값 $h_u$ 정확도 ↑, 하지만 bin 당 pair 수 ↓ → 추정 분산 ↑
- Bin을 적게 쓸수록 → 반대

실용적인 rule of thumb: $N(h_u) \geq 30$이고, $h_u$의 길이가 최대 lag 길이의 절반 미만. 하지만 $N(h_u)$가 30보다 훨씬 커도 분산이 클 수 있다. Lag 사이의 의존성이 있어서 sum의 항들이 독립적이지 않거든.

### Robust Estimator (Cressie & Hawkins, 1980)

Empirical semivariogram은 sum of squares 구조라 outlier에 민감한다. 더 robust한 추정량은 다음과 같다.

$$\bar{\gamma}(h_u) = \frac{\left\{\frac{1}{N(H_u)} \sum_{s_i - s_j \in H_u} |\hat{e}(s_i) - \hat{e}(s_j)|^{1/2}\right\}^4}{.914 + [.988/N(H_u)]}$$

---

## 3.4 Semivariogram Modeling

### 왜 Parametric Model이 필요한가?

Empirical semivariogram에 parametric model을 fit하는 이유가 세 가지이다.

1. Empirical semivariogram은 보통 거칠고 bumpy해진다. Smooth한 버전이 더 안정적이다.
2. Empirical semivariogram은 conditionally nonpositive definite 성질을 만족 못할 수 있다. 이 성질이 없으면 prediction error variance가 음수가 될 수 있다.
3. 데이터 위치에 없는 lag에서도 semivariogram 값이 필요한다. Parametric model이 있어야 해진다.

### Valid Semivariogram의 조건

Semivariogram model $\gamma(h;\theta)$가 valid하려면 세 조건을 만족해야 해진다.

1. $\gamma(0;\theta) = 0$ (origin에서 소멸)
2. $\gamma(-h;\theta) = \gamma(h;\theta)$ (even function)
3. **Conditional negative definiteness**: $\sum_i \sum_j a_i a_j \gamma(s_i - s_j;\theta) \leq 0$, 단 $\sum_i a_i = 0$

### 주요 Isotropic Semivariogram 모델들

**Spherical**

$$\gamma(h;\theta) = \begin{cases} \theta_1 \left(\dfrac{3h}{2\theta_2} - \dfrac{h^3}{2\theta_2^3}\right) & 0 \leq h \leq \theta_2 \\ \theta_1 & h > \theta_2 \end{cases}$$

**Exponential**

$$\gamma(h;\theta) = \theta_1 \{1 - \exp(-h/\theta_2)\}$$

**Gaussian**

$$\gamma(h;\theta) = \theta_1 \left\{1 - \exp\left(-h^2/\theta_2^2\right)\right\}$$

**Matérn**

$$\gamma(h;\theta) = \theta_1 \left\{1 - \frac{(h/\theta_2)^\nu K_\nu(h/\theta_2)}{2^{\nu-1}\Gamma(\nu)}\right\}$$

**Power**

$$\gamma(h;\theta) = \theta_1 h^{\theta_2}, \quad 0 \leq \theta_2 < 2$$

모든 모델에서 $\theta_1 > 0$이다. Matérn에서 $\nu = 0.5$이면 exponential, $\nu \to \infty$이면 Gaussian과 일치한다.

### 모델 속성들

**Sill**

$$\text{sill} = \lim_{h \to \infty} \gamma(h;\theta)$$

Sill이 존재하면 process가 second-order stationary이기도 하고, $C(0;\theta)$와 sill이 일치한다.

**Range**

Sill이 존재할 때, semivariogram이 sill에 처음 도달하는 $h$ 값이다. Sill이 없으면 sill의 95%에 해당하는 $h$를 **effective range**로 정의한다.

Exponential model의 effective range는 약 $3\theta_2$, Gaussian은 약 $\sqrt{3}\theta_2$예다.

Range parameter는 추정하기 어렵다. 특히 range가 관측 영역 크기와 비슷할 때. Power class variogram은 range를 무한대로 보내서 이 문제를 우회하는 방법이다.

**Nugget effect**

$$\text{nugget} = \lim_{h \to 0} \gamma(h;\theta)$$

위의 모델들은 모두 nugget이 0이지만, 필요하면 추가할 수 있다. 예를 들어 nugget $\theta_3$을 가진 exponential model은:

$$\gamma(h;\theta) = \begin{cases} 0 & h = 0 \\ \theta_3 + \theta_1\{1 - \exp(-h/\theta_2)\} & h > 0 \end{cases}$$

**모델 선택 팁**

- Gaussian semivariogram: 너무 smooth해서 자연 현상 모델링엔 종종 부적합한다.
- Matérn with $\nu > 1$ (but not too large): differentiable process에 일반적으로 선호된다.
- Spherical: geostatistics 커뮤니티에서 인기 있지만, $\theta_2 = h$에서 likelihood function이 이상하게 생기는 단점이 있다.

### Anisotropic으로 일반화

Valid한 isotropic semivariogram model은 $h$를 $(h^T A h)^{1/2}$로 치환해서 geometrically anisotropic하게 만들 수 있다. 예를 들어 $\mathbb{R}^2$에서 geometrically anisotropic exponential은:

$$\gamma(h;\theta) = \theta_1 \left\{1 - \exp\left(-\frac{(h_1^2 + 2\theta_3 h_1 h_2 + \theta_4 h_2^2)^{1/2}}{\theta_2}\right)\right\}$$

Zonally anisotropic model (geometrically anisotropic이 아닌 경우)은 이론적으로도 실용적으로도 문제가 많아서 잘 안 쓴다.

### WLS Estimator

파라미터를 추정하는 두 가지 주요 방법은 **WLS**와 **ML/REML**이다. 여기서는 WLS만 소개한다.

**Definition (WLS estimator):**

$$\hat{\theta} = \underset{\theta}{\arg\min} \sum_{u \in U} \frac{N(h_u)}{[\gamma(h_u;\theta)]^2} [\hat{\gamma}(h_u) - \gamma(h_u;\theta)]^2$$

Weight가 이론적 variogram 값이 작을 때 커진다. 즉 $h = 0$에 가까운 lag들이 더 큰 weight를 받는다. Variogram의 origin 근방 fit이 중요하기 때문에 이건 좋은 성질이다.

WLS는 확률 모델을 기반으로 하지 않아서 likelihood-based 방법보다 이론적 기반이 약한다. 하지만 nondifferentiable process에서는 성능 차이가 크지 않는다.

---

## 3.5 Mean Function 재추정

Covariate effect 추정에 관심이 있다면, semivariogram을 추정한 다음 mean function을 다시 추정해야 해진다.

방법은 **Estimated Generalized Least Squares (EGLS)**예다. GLS에서 알려져 있어야 할 variance-covariance 정보를 추정값으로 대체한 거다.

Covariance matrix 추정 방법은 이래다.

1. $Y(s_i)$들의 공통 분산을 fitted semivariogram model의 **sill**으로 추정. 이를 $\hat{C}(0)$라고 해진다.
2. $Y(s_i)$와 $Y(s_j)$ 사이의 covariance를 다음으로 추정:

$$\hat{C}(s_i - s_j) = \hat{C}(0) - \gamma(s_i - s_j;\hat{\theta})$$

이 추정값들을 모아 estimated variance-covariance matrix $\hat{\Sigma} = [\hat{C}(s_i - s_j)]$을 만든다.

**Definition (EGLS estimator):**

$$\hat{\beta}_{\text{EGLS}} = (X^T \hat{\Sigma}^{-1} X)^{-1} X^T \hat{\Sigma}^{-1} Y$$

$\hat{\beta}_{\text{EGLS}}$는 mild한 조건 하에서 unbiased이다. 하지만 $\theta$를 알고 있는 GLS estimator보다는 분산이 크다.

EGLS를 쓰면 OLS에 비해 공간적 covariation 구조를 반영할 수 있다. 단 linear estimator라는 한계가 있고, likelihood method를 쓰면 이 한계를 어느 정도 넘어설 수 있다.

---

## 3.6 Kriging

### Universal Kriging

Classical geostatistical analysis의 마지막 단계는 **prediction**이다. 관측하지 않은 위치 $s_0$에서 $Y(s_0)$를 예측하는 거다. 이를 위한 방법들을 통틀어 **kriging**이라고 한다.

**Universal kriging**의 목표는 다음 두 조건을 만족하는 predictor 중 prediction error variance를 최소화하는 거다.

1. **Linearity**: $\hat{Y}(s_0) = \lambda^T Y$
2. **Unbiasedness**: $E[\hat{Y}(s_0)] = E[Y(s_0)]$, 즉 $\lambda^T X = X(s_0)$

Semivariogram을 안다고 가정할 때, 이 constrained minimization의 해, **universal kriging predictor**는:

$$\hat{Y}(s_0) = \left[\gamma + X\left(X^T \Gamma^{-1} X\right)^{-1}\left(x_0 - X^T \Gamma^{-1} \gamma\right)\right]^T \Gamma^{-1} Y \tag{3.6}$$

여기서:
- $\gamma = [\gamma(s_1 - s_0), \ldots, \gamma(s_n - s_0)]^T$
- $\Gamma$는 $(i,j)$ 원소가 $\gamma(s_i - s_j)$인 $n \times n$ matrix
- $x_0 = X(s_0)$

**Kriging variance** (최소화된 prediction error variance):

$$\sigma^2(s_0) = \gamma^T \Gamma^{-1} \gamma - \left(X^T \Gamma^{-1} \gamma - x_0\right)^T \left(X^T \Gamma^{-1} X\right)^{-1} \left(X^T \Gamma^{-1} \gamma - x_0\right) \tag{3.7}$$

Universal kriging predictor는 **BLUP (Best Linear Unbiased Predictor)**의 한 예이다.

$Y(\cdot)$가 Gaussian이면 prediction interval을 만들 수 있다.

$$\hat{Y}(s_0) \pm z_{\alpha/2}\, \sigma(s_0)$$

$\gamma(\cdot)$가 알려져 있다면 이 interval의 coverage probability는 정확히 $1 - \alpha$예다.

### 실제 적용에서의 수정

**1. Local kriging**

계산량을 줄이기 위해 전체 데이터 $Y$가 아니라 $s_0$ 주변의 neighborhood 관측값만 쓴다. Neighborhood 크기 결정에는 range, nugget-to-sill ratio, 관측 위치의 공간적 배치가 중요한다. Nugget이 클수록 더 큰 neighborhood가 필요한다.

**2. Empirical kriging**

현실에서 semivariogram은 모르기 때문에 $\hat{\gamma} = \gamma(\hat{\theta})$와 $\hat{\Gamma} = \Gamma(\hat{\theta})$를 식 (3.6), (3.7)에 대입한다. 여기서 $\hat{\theta}$는 예를 들어 WLS로 얻은 추정값이다. 이 empirical universal kriging predictor는 더 이상 데이터의 linear function이 아니지만, mild한 조건 하에서 여전히 unbiased이다.

### Block Kriging

지금까지는 단일 점 $s_0$을 예측하는 **point kriging**이었는다. 때로는 블록 $B \subset D$ 위의 평균값

$$Y(B) \equiv \int_B Y(s)\,ds \Big/ |B|$$

를 예측하고 싶을 때가 있다. 이를 **block kriging**이라고 해진다.

식의 형태는 (3.6), (3.7)과 동일하지만, 각 항을 블록에 대해 적분한 값으로 교체한다.

- $\gamma_i = \gamma(B, s_i) = |B|^{-1} \int_B \gamma(u - s_i)\, du$
- $x_{0,j} = X_j(B) = |B|^{-1} \int_B X_j(u)\, du$

블록 평균을 예측하기 때문에 point kriging보다 kriging variance가 작아지는 경향이 있다. 공간적 평균화가 변동성을 줄여주거든.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 3.
