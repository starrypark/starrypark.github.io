---
title: "[Ch.4] Likelihood 기반 추정법"
description: Geostatistical model에서 ML/REML 추정, 점근 이론, 모델 비교, 계산 효율화 방법까지 likelihood 기반 접근법을 정리한 노트이다.
author: starrypark
date: 2024-04-04 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [maximum likelihood, REML, kriging, geostatistics, likelihood]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

Chapter 3에서 WLS로 semivariogram을 추정했는데, 그 방법의 한계가 있었다. WLS는 확률 모델을 직접 기반으로 하지 않아서 이론적 보장이 약하고, mean function 추정도 EGLS라는 두 단계 방식이었는다.

Chapter 4는 다른 길을 택한다. Gaussian 가정을 하고, **likelihood를 직접 최대화**하는 거다.

이 접근법의 장점은 $\beta$와 $\theta$를 한 번에 추정할 수 있고, 불확실성 정량화와 모델 비교도 자연스럽게 따라온다는 거다. 대신 계산 비용이 문제이다. 그래서 챕터 후반부는 그 계산 문제를 어떻게 해결할지에 집중한다.

---

## 4.1 Maximum Likelihood Estimation

### 모델 설정

Mean function이 linear하고 error process가 Gaussian인 경우를 가정한다.

$$Y(s) = X(s)^T \beta + e(s)$$

- $X(s)$: 위치 $s$에서의 $p$차원 covariate vector
- $\beta$: unknown parameter vector
- $e(\cdot)$: 평균 0인 Gaussian process

Error process의 covariance function:

$$\text{Cov}[e(s), e(t)] \equiv C(s, t; \theta)$$

$\theta$는 $m$차원 unknown parameter이다. 데이터를 모아 쓰면:

- $Y = [Y(s_1), \ldots, Y(s_n)]^T$: $n$차원 관측 벡터
- $X = [X(s_1), \ldots, X(s_n)]^T$: $n \times p$ covariate matrix
- $\Sigma(\theta)$: $(i,j)$ 원소가 $C(s_i, s_j; \theta)$인 $n \times n$ covariance matrix

### Likelihood Function

$Y$의 likelihood는:

$$l(\beta, \theta; Y) = (2\pi)^{-n/2} |\Sigma(\theta)|^{-1/2} \exp\left(-\frac{(Y - X\beta)^T \Sigma^{-1}(\theta)(Y - X\beta)}{2}\right)$$

### MLE

**Theorem:** $\hat{\beta}$와 $\hat{\theta}$의 MLE는 다음과 같다.

$$\hat{\beta} = \left(X^T \Sigma^{-1}(\theta_0) X\right)^{-1} X^T \Sigma^{-1}(\theta_0) Y$$

$\hat{\theta}$는 다음 profile likelihood를 최대화하는 값이다.

$$L(\theta; Y) = -\frac{1}{2} \log |\Sigma(\theta)| - \frac{1}{2} Y^T P(\theta) Y$$

여기서

$$P(\theta) = \Sigma^{-1}(\theta) - \Sigma^{-1}(\theta) X \left(X^T \Sigma^{-1}(\theta) X\right)^{-1} X^T \Sigma^{-1}(\theta)$$

$\hat{\beta}$가 $\theta$의 함수라는 점에서, 결국 $\theta$에 대한 최적화 문제로 귀결된다.

### 수치 최적화 방법

$L(\theta; Y)$를 닫힌 형태로 최대화하는 건 특수한 경우 외엔 불가능한다. 수치 방법이 필요한다.

**1. Grid Search**

$\theta$의 차원이 낮을 때 유효한다. 가장 단순하지만 고차원에서는 비실용적이다.

**2. Gradient Algorithm**

$$\theta^{(k+1)} = \theta^{(k)} + \rho^{(k)} M^{(k)} g^{(k)}$$

$g^{(k)} = \partial L(\theta; Y)/\partial\theta \big|_{\theta = \theta^{(k)}}$는 gradient, $\rho^{(k)}$는 step size, $M^{(k)}$는 방향을 결정하는 matrix이다.

**3. Newton-Raphson**

$M^{(k)}$를 negative Hessian의 역행렬로 쓰는 gradient algorithm이다.

$$M^{(k)} = \left[-\frac{\partial^2 L}{\partial\theta_i \partial\theta_j}\bigg|_{\theta^{(k)}}\right]^{-1}$$

**4. Fisher Scoring**

Newton-Raphson에서 Hessian 대신 Fisher information matrix를 쓴다.

$$M^{(k)} = \left[B^{(k)}\right]^{-1}, \quad B^{(k)}_{ij} = E\left[-\frac{\partial^2 L}{\partial\theta_i \partial\theta_j}\bigg|_{\theta^{(k)}}\right]$$

Hessian 계산이 복잡할 때 Fisher scoring이 더 안정적이다.

수렴에 대한 이론적 보장은 일반적으로 없다. 하지만 Matérn class 같은 실용적인 covariance function에서 multiple mode가 나오는 경우는 극히 드물는다. 그래도 안전을 위해 **여러 시작점에서 반복 실행**해서 global maximum인지 확인하는 게 좋는다.

---

## 4.2 REML Estimation

ML estimator에는 잘 알려진 단점이 있다. $\beta$를 추정하면서 degrees of freedom을 잃기 때문에 **$\theta$의 ML 추정량이 편향(biased)**되는 거다.

일반 선형 모델에서 $\sigma^2$의 MLE가 $n$ 대신 $n-p$로 나누지 않아서 편향되는 것과 같은 이유이다.

**REML (Restricted Maximum Likelihood)**은 이 문제를 완화한다.

**Definition (REML estimator):**

$$(\hat{\beta}, \hat{\theta}) = \underset{(\beta,\theta)}{\arg\max}\; L_R(\theta; Y)$$

$$L_R(\theta; Y) = -\frac{1}{2}\log|\Sigma(\theta)| - \frac{1}{2}\log\left|X^T \Sigma^{-1}(\theta) X\right| - \frac{1}{2} Y^T P(\theta) Y$$

ML과 비교하면 중간에 $-\frac{1}{2}\log|X^T \Sigma^{-1}(\theta) X|$ 항이 추가됐다. 이 항이 $\beta$ 추정에서 생기는 편향을 보정해 줍니다.

REML은 편향을 효과적으로 줄이지만, 분포의 꼬리가 두꺼워질 수 있다. 즉, 분산이 커질 수 있다는 trade-off가 있다.

---

## 4.3 Asymptotic Results

Spatial statistics에서 점근 이론은 두 가지 framework가 있다. 이 부분이 처음엔 헷갈릴 수 있다.

**1. Increasing Domain Asymptotics**

데이터 위치들 사이의 최소 거리가 0에서 bounded away from zero인 상태에서, 관측 영역 자체가 무한히 커지는 경우이다. 쉽게 말하면 관측 영역이 넓어지면서 데이터가 많아지는 상황이다.

이 framework에서 ML, REML estimator는 일정한 regularity conditions 하에서 **consistent하고 asymptotically normal**해진다.

**2. Infill (Fixed-Domain) Asymptotics**

관측 영역은 고정된 채로, 데이터 위치가 점점 밀집되는 경우이다. 같은 영역 안에서 더 촘촘하게 관측하는 상황이다.

이 framework에서의 결과는 훨씬 제한적이다. 일부 파라미터는 infill 하에서 consistent하게 추정할 수 없다.

**두 framework의 관계**

두 framework 모두에서 consistent한 파라미터의 경우, 두 framework가 제공하는 finite-sample 근사의 성능은 비슷한다. 하지만 infill 하에서 consistent하지 않은 파라미터의 경우, infill asymptotic framework의 finite-sample 근사가 더 잘 작동한다.

---

## 4.4 Hypothesis Testing과 Model 비교

파라미터 $\beta$와 $\theta$에 대한 가설 검정은 결국 "full model vs. reduced model" 비교이다.

두 모델이 **nested**이면 **likelihood ratio test (LRT)**를 쓸 수 있다.

두 모델이 **non-nested**이면 **penalized likelihood criteria**를 쓴다. 주요 기준들이다.

**AIC (Akaike's Information Criterion)**

$$\text{AIC} = -2\log l(\hat{\beta}, \hat{\theta}) + 2(p + m)$$

**BIC (Schwarz's Bayesian Information Criterion)**

$$\text{BIC} = -2\log l(\hat{\beta}, \hat{\theta}) + (p + m)\log n$$

**DIC (Deviance Information Criterion)**

$$\text{DIC} = \bar{D} + p_D$$

여기서 $D(\beta,\theta) = -2\log l(\beta,\theta;Y) + c$이고, $\bar{D} = E[D(\beta,\theta)|Y]$, $p_D = \bar{D} - D(\bar{\beta},\bar{\theta})$, $(\bar{\beta},\bar{\theta}) = E[(\beta,\theta)|Y]$예다.

세 기준 모두 "fit이 좋을수록 좋고, 복잡할수록 패널티를 부과한다"는 아이디어이다. AIC보다 BIC가 더 강한 패널티를 부과해서, BIC는 더 단순한 모델을 선호하는 경향이 있다.

---

## 4.5 Computational Issues

### 왜 계산이 문제인가?

$\Sigma(\theta)$가 $n \times n$ matrix이기 때문에, $\Sigma(\theta)$의 역행렬과 행렬식 계산이 $O(n^3)$이다. 데이터가 많아질수록 계산 비용이 급격히 증가한다.

### Scale Parameter 활용

Covariance function의 파라미터 중 하나가 scale parameter이면 계산을 줄일 수 있다. Covariance matrix를:

$$\Sigma(\theta) = \theta_1 W(\theta_{-1})$$

로 쓸 수 있다고 해진다. 여기서 $\theta_{-1} = (\theta_2, \ldots, \theta_m)^T$예다.

이 경우 $L(\theta; Y)$의 최대값은 다음에서 달성된다.

$$\hat{\theta}_1 = Y^T Q(\hat{\theta}_{-1}) Y / n$$

$\hat{\theta}_{-1}$는 다음을 최대화하는 값이다.

$$L_{-1}(\theta_{-1}; Y) = -\frac{1}{2}\log|W(\theta_{-1})| - \frac{n}{2}\log\left(Y^T Q(\theta_{-1}) Y\right)$$

여기서

$$Q(\theta_{-1}) = W^{-1}(\theta_{-1}) - W^{-1}(\theta_{-1}) X \left(X^T W^{-1}(\theta_{-1}) X\right)^{-1} X^T W^{-1}(\theta_{-1})$$

$\theta_1$을 바깥으로 빼내고 $\theta_{-1}$만 최적화하면 되니까 계산 문제의 차원이 줄어들는다.

### 기타 계산 최적화 방법

**Sparse matrix 활용**

Covariance function의 range가 관측 영역에 비해 작으면 $\Sigma(\theta)$가 sparse해져다. 멀리 떨어진 위치들의 covariance가 거의 0이 되거든. Sparse matrix 알고리즘을 쓰면 계산 비용을 크게 줄일 수 있다.

**Regular grid 활용**

Covariance function이 isotropic이거나 separable하고 데이터가 regular rectangular grid에 있으면 $\Sigma(\theta)$의 구조를 이용해 계산을 대폭 줄일 수 있다.

---

## 4.6 Approximate and Composite Likelihood

$n$이 커서 exact likelihood를 쓰기 어려울 때의 대안들이다.

### Vecchia의 Likelihood 근사

Vecchia (1988)는 관측 벡터 $Y$를 subvector $Y_1, \ldots, Y_b$로 분할한다.

$Y^{(j)} = (Y_1^T, \ldots, Y_j^T)^T$로 정의하면, exact likelihood는:

$$p(Y; \beta, \theta) = p(Y_1; \beta, \theta) \prod_{j=2}^b p\left(Y_j \mid Y^{(j-1)}; \beta, \theta\right) \tag{4.1}$$

Vecchia의 핵심 아이디어는 각 conditioning vector $Y^{(j-1)}$를 그 부분집합 $S^{(j-1)}$으로 교체하는 거다. Conditioning set이 작아지면 계산해야 할 행렬의 크기가 줄어들는다. 근사이기 때문에 exact likelihood와 차이가 있지만, conditioning set을 충분히 크게 잡으면 근사 오차는 작는다.

### Composite Likelihood

Intrinsically stationary Gaussian random field에서, 모든 pair $(s_i, s_j)$의 pairwise difference에 대한 marginal density로 composite log likelihood를 만든다.

$$CL(\theta; Y) = -\frac{1}{2} \sum_{i=1}^{n-1} \sum_{j>i} \left\{\log \gamma(s_i - s_j; \theta) + \frac{(Y(s_i) - Y(s_j))^2}{2\gamma(s_i - s_j; \theta)}\right\}$$

Pair별로 독립적으로 기여를 계산해서 더하는 구조이다. 각 pair의 계산은 작은 크기의 행렬만 다루면 된다. 단, pair들 사이의 의존성을 무시하기 때문에 standard error 계산에는 주의가 필요한다.

### Covariance Tapering

공간적으로 멀리 떨어진 위치 쌍의 covariance를 강제로 0으로 만들어 sparse matrix 알고리즘을 쓸 수 있게 하는 방법이다.

$C_0(h;\theta)$를 원래 covariance function, $C_T(h;\gamma)$를 $h \geq \gamma$이면 0인 isotropic correlation function (tapering function)이라고 하면, tapered covariance function은:

$$C_1(h;\theta,\gamma) = C_0(h;\theta) \cdot C_T(h;\gamma)$$

이 $C_1$은 valid positive definite covariance function이다. 분산은 $C_0$와 동일하게 유지되고, 먼 거리에서만 0으로 눌러다.

Tapered log likelihood는:

$$L_T(\theta, \beta; Y) = -\frac{1}{2}\log|\Sigma(\theta) \circ T(\gamma)| - \frac{1}{2}(Y - X\beta)^T [\Sigma(\theta) \circ T(\gamma)]^{-1}(Y - X\beta)$$

여기서 $T(\gamma)$는 $(i,j)$ 원소가 $C_T(\|s_i - s_j\|;\gamma)$인 matrix이고, $\circ$는 elementwise multiplication이다.

$\Sigma(\theta) \circ T(\gamma)$가 sparse matrix가 되기 때문에 sparse matrix 알고리즘을 쓸 수 있다. $\gamma$가 작을수록 더 sparse해지고 계산은 빠르지만 근사 오차는 커집니다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 4.
