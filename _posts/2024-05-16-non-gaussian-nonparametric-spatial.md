---
title: "[Ch.11] Non-Gaussian 및 비모수 공간 모델"
description: Gaussian 가정을 넘어서 GLG mixture model, Dirichlet process, spatial stick-breaking prior까지 non-Gaussian spatial modeling을 정리한 노트이다.
author: starrypark
date: 2024-05-16 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [non-gaussian, dirichlet process, spatial statistics, bayesian nonparametrics, kriging]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

지금까지는 spatial process가 Gaussian이라는 가정 위에서 모든 걸 전개했다. Covariance function, kriging, likelihood — 전부 Gaussian에 기대고 있었다.

근데 현실 데이터는 종종 Gaussian을 벗어나다. Heavy tail이 있거나, 공간에 따라 분산이 달라지거나, 분포 자체의 형태가 복잡한 경우다.

Chapter 11은 이 문제를 두 방향으로 접근한다.

- **Parametric하게**: Gaussian process에 scale mixing을 섞어서 GLG (Gaussian-Log-Gaussian) 모델을 만드는 방법
- **Nonparametric하게**: Dirichlet process 기반의 prior를 공간에 확장하는 방법

---

## 11.1 Non-Gaussian Parametric Modeling

### 기존 Gaussian 모델 복습

출발점은 익숙한 모델이다.

$$Y(s) = x(s)^T\beta + \eta(s) + \epsilon(s)$$

- $\eta(s)$: 평균 0, 분산 $\sigma^2$, correlation function $C_\theta(\|s_i - s_j\|)$인 second-order stationary process
- $\epsilon(s)$: 평균 0, 분산 $\tau^2$인 nugget effect (iid Gaussian)
- $\omega^2 = \tau^2/\sigma^2$: nugget effect의 상대적 크기

$\eta(s)$가 Gaussian이면 관측 벡터 $y$는 multivariate Normal을 따라다.

$$E[y] = X^T\beta, \quad \text{var}[y] = \sigma^2 C_\theta + \tau^2 I_n$$

### Gaussian-Log-Gaussian (GLG) Mixture Model

**Scale mixing**이라는 아이디어가 핵심이다. 각 관측 $i$마다 mixing variable $\lambda_i \in \mathbb{R}^+$를 도입하고, 모델을 이렇게 바꿔다.

$$y_i = x(s_i)^T\beta + \frac{\eta_i}{\sqrt{\lambda_i}} + \epsilon_i \tag{11.1}$$

$\lambda_i$가 1이면 원래 Gaussian 모델과 같다. $\lambda_i$가 작으면 그 위치에서 분산이 커진다.

조건부 분포는:

$$p(y \mid \beta, \sigma^2, \tau^2, \theta, \Lambda) = f^n_N\!\left(y \mid X\beta,\; \sigma^2\Lambda^{-1/2}C_\theta\Lambda^{-1/2} + \tau^2 I_n\right)$$

여기서 $\Lambda = \text{diag}(\lambda_1, \ldots, \lambda_n)$이다.

### Mean Square Continuity 문제

Scale mixing을 도입할 때 조심해야 할 점이 있다. 각 위치에 독립적인 $\lambda_i$를 쓰면 process가 mean square continuous하지 않다.

수식으로 보면, $\lambda_i$와 $\lambda_j$가 독립일 때:

$$\lim_{\|s_i - s_j\| \to 0} E\!\left[\lambda_i^{-1/2}\lambda_j^{-1/2}\right] = \left(E\!\left[\lambda_i^{-1/2}\right]\right)^2 \leq E\!\left[\lambda_i^{-1}\right]$$

Jensen's inequality에 의해 부등호가 성립하고, 따라서 $\eta(s)/\sqrt{\lambda(s)}$는 mean square continuous하지 않다.

해결책은 $\lambda$를 공간적으로 correlate하는 거다. Palacios and Steel (2006)은 다음을 제안한다.

$$\ln(\lambda) \sim N_n\!\left(-\frac{\nu}{2}\mathbf{1},\; \nu C_\theta\right) \tag{11.2}$$

$\eta$와 **같은 correlation matrix** $C_\theta$를 쓰는 게 핵심이다. 가까운 위치는 비슷한 $\lambda$ 값을 가지게 되고, 그러면 process가 mean square continuous해집니다.

파라미터 $\nu \in \mathbb{R}^+$는 $\lambda_i$의 분포를 결정한다. $E[\lambda_i] = 1$이고 $\text{var}[\lambda_i] = \exp(\nu) - 1$이다. $\nu$가 작을수록 $\lambda_i \approx 1$ (Gaussian에 가까움), $\nu$가 클수록 더 spread out되고 right-skewed해진다.

### GLG 모델의 성질

GLG 모델의 correlation structure는:

$$\text{corr}[y_i, y_j] = C_\theta(\|s_i - s_j\|) \cdot \frac{\exp\!\left(v\left\{1 + \frac{1}{4}[C_\theta(\|s_i - s_j\|) - 1]\right\}\right)}{\exp(v) + \omega^2}$$

Nugget effect가 없을 때 ($\omega^2 = 0$), 거리가 0으로 가면 correlation은 1로 수렴한다. 즉, mixing이 discontinuity를 유발하지 않는다.

Kurtosis는 $\text{kurt}[y_i] = 3\exp(\nu)$예다. $\nu \to 0$이면 Gaussian kurtosis인 3으로 돌아오고, $\nu$가 클수록 heavy tail을 가집니다.

Smoothness도 mixing에 영향받지 않는다. Nugget effect가 없으면 $Y(s)$의 smoothness는 $\eta(s)$와 정확히 같다.

### Spatial Heteroscedasticity 탐지

$\lambda(s)$가 smooth process이기 때문에, $\lambda_i$가 작은 관측들은 공간적으로 몰려 있는 경향이 있다. 이건 개별 outlier가 아니라 **공간의 특정 영역에서 분산이 크다**는 걸 의미한다. 이를 **spatial heteroscedasticity**라고 해진다.

이걸 정량화하려면 Savage-Dickey density ratio를 쓴다.

$$R_i = \frac{p(\lambda_i \mid y)}{p(\lambda_i)}\bigg|_{\lambda_i = 1}$$

$R_i$가 작으면 "이 위치의 $\lambda_i$가 1이 아닐 가능성이 높다", 즉 해당 관측이 variance-inflated 영역에 있다는 거다.

### Prediction

관측값 $y_o$와 예측 대상 $y_p$로 분할하면, posterior predictive distribution은:

$$p(y_p \mid y_o) = \int p(y_p \mid y_o, \lambda, \zeta)\, p(\lambda_p \mid \lambda_o, \zeta, y_o)\, p(\lambda_o, \zeta \mid y_o)\, d\lambda\, d\zeta \tag{11.3}$$

여기서 $\zeta = (\beta, \sigma, \tau, \theta, \nu)$예다.

MCMC로 $(\lambda_o, \zeta)$를 sampling한 뒤, 각 sample에서 $\lambda_p$를 다음으로 생성한다.

$$\ln\lambda_p \mid \lambda_o, \nu \sim N_f\!\left(-\frac{\nu}{2}\mathbf{1}_f + C_{po}C_{oo}^{-1}\ln\lambda_o,\; \nu(C_{pp} - C_{po}C_{oo}^{-1}C_{op})\right) \tag{11.4}$$

그런 다음 조건부 predictive density를 계산한다.

$$p(y_p \mid y_o, \lambda, \zeta) = f^f_N\!\left(y_p \mid (X_p - AX_o)\beta + Ay_o,\; \sigma^2(\Lambda_p^{-1/2}C_{pp}\Lambda_p^{-1/2} + \omega^2 I_f - A\Lambda_o^{-1/2}C_{op}\Lambda_p^{-1/2})\right) \tag{11.5}$$

이 density들을 평균내면 posterior predictive density가 된다.

### 사전 분포 설정

**$\beta$**: $\beta \sim N_k(0, c_1 I_k)$, benchmark $c_1 = 10^4$

**$\sigma^{-2}$**: $\sigma^{-2} \sim \text{Ga}(c_2, c_3)$, benchmark $c_2 = c_3 = 10^{-6}$ (rescaling에 근사적으로 불변)

**$\omega^2$**: Nugget이 작을 거라는 기대 하에 GIG prior. $\omega^2 \sim \text{GIG}(0, c_4, c_5)$, benchmark $c_4 = 0.66$, $c_5 = 1$ (prior mode 0.2, prior mean 1.07)

**$\nu$**: 작은 값이 기대될 때, $\nu \sim \text{GIG}(0, c_6, c_7)$, benchmark $c_6 = 0.5$, $c_7 = 2$ (prior mode 0.1, prior mean 0.36)

**$\theta$**: Matérn 파라미터. Range parameter $\rho = 2\theta_1\sqrt{\theta_2}$와 smoothness $\theta_2$에 각각 exponential prior를 쓴다.

### 스페인 기온 데이터 사례

2001년 5월 스페인 바스크 지방 63개 관측소의 최고 기온 데이터에 적용한 결과이다.

Gaussian 모델 vs. GLG 모델을 비교하면:

| 파라미터 | Gaussian | GLG |
|---|---|---|
| $\sigma$ | 0.32 (0.11) | 0.09 (0.03) |
| $\tau$ | 0.31 (0.06) | 0.08 (0.02) |
| $\nu$ | 0 | 2.51 (0.76) |
| $\sigma^2\exp(\nu)$ | 0.11 (0.09) | 0.12 (0.15) |

GLG 모델에서 $\sigma$와 $\tau$ 모두 훨씬 작아져다. Scale mixing이 분산을 흡수했기 때문이다. $\nu = 2.51$은 상당한 heavy tail을 시사한다.

4개 관측 (#20, #36, #40, #41)에서 Bayes factor가 0에 가까워, 두 개의 variance-inflated 영역이 존재함을 확인할 수 있었다.

Predictive density를 비교하면 GLG 모델의 꼬리가 더 heavy하지만, 중심부는 오히려 더 집중돼 있다. Scale mixing으로 인한 불확실성 증가가 다른 파라미터 추정 개선으로 상쇄된 거다.

---

## 11.2 Bayesian Nonparametric Approaches

### Dirichlet Process

Dirichlet process는 probability distribution function에 대한 Bayesian inference의 conjugate prior이다.

**Definition:** $\alpha > 0$이고 $P_0$가 $X$ 위의 probability measure일 때, random probability measure $P$가 $P \sim \text{DP}(\alpha \cdot P_0)$이면 $X$의 임의의 measurable finite partition $B_1, \ldots, B_k$에 대해:

$$(P(B_1), \ldots, P(B_k)) \sim \text{Dirichlet}(\alpha P_0(B_1), \ldots, \alpha P_0(B_k))$$

$\alpha$는 concentration parameter이다. $\alpha$가 클수록 $P$가 $P_0$에 가까워지고, 작을수록 더 다양한 분포가 가능한다.

### Stick-Breaking Prior

Dirichlet process의 sample은 거의 확실히 discrete해진다. 이를 생성하는 직관적인 방법이 **stick-breaking construction**이다.

길이 1짜리 막대를 무한히 쪼개는 과정을 상상하면 된다.

**Definition:** $\pi = (\pi_1, \pi_2, \ldots)$가 stick-breaking process인 것은:

$$\pi_j = \gamma_j \prod_{k=1}^{j-1}(1 - \gamma_k)$$

여기서 $\gamma_1, \gamma_2, \ldots$는 $[0,1]$ 위의 iid 확률 변수이다.

**Definition:** Random probability distribution $F$가 **stick-breaking prior**를 가지면:

$$F \overset{d}{=} \sum_{i=1}^N p_i \delta_{\theta_i} \tag{11.6}$$

여기서 $p_i = V_i \prod_{j < i}(1 - V_j)$, $V_i \sim \text{Beta}(a_i, b_i)$, $\theta_i \sim H$예다.

Stick-breaking prior는 discrete distribution을 만들기 때문에, 연속 과정의 모델링에 직접 쓰기 어렵다. 그래서 실제로는 **mixture of Dirichlet process** 방식을 쓴다. 관측값에는 continuous model을 쓰고, 그 파라미터에 stick-breaking prior를 붙이는 거다.

이 framework를 공간에 적용하려면 index를 위치 $s$에 따라 변화시켜야 해진다.

### Generalized Spatial Dirichlet Process

Gelfand et al. (2005)의 아이디어는 stick-breaking의 location atoms $\theta$를 위치 $s$에 의존하게 하는 거다. $\theta(s)$를 stationary Gaussian process의 realization으로 만들면, covariance function 선택에 따라 연속성이 확보된다.

간단한 모델 $Y(s) = \eta(s) + \epsilon(s)$에서 $\eta(s)$에 이 spatial Dirichlet prior를 쓰면, 관측 벡터의 joint density는 거의 확실히 다음 형태의 location mixture of normals가 된다.

$$\sum_{i=1}^N p_i f^n_N(y \mid \eta_i, \tau^2 I_n)$$

하지만 이 모델에서는 모든 위치가 **같은 weight** $\{p_i\}$를 공유한다. 위치에 따라 surface 선택이 달라지지 않는다는 한계가 있다.

이를 극복하기 위한 **generalized spatial Dirichlet process**에서는:

$$F^{(n)} \overset{d}{=} \sum_{i_1=1}^\infty \cdots \sum_{i_n=1}^\infty p_{i_1,\ldots,i_n} \delta_{\theta_{i_1}} \cdots \delta_{\theta_{i_n}} \tag{11.7}$$

위치별로 다른 surface를 선택할 수 있고, weight가 continuous하게 변한다. 단, 이 방법은 replications이 필요한다.

### Hybrid Dirichlet Mixture Models

Petrone et al. (2009)은 functional data analysis 맥락에서 이 아이디어를 발전시켰다. $n$개의 curve $y_i = [Y_i(x_1), \ldots, Y_i(x_m)]^T$를 더 작은 canonical curve 집합으로 표현하는 거다.

모델은:

$$y_i \mid \theta_i \overset{\text{ind}}{\sim} N_m(\theta_i, \sigma^2 I_m), \quad \theta_i \mid F_{x_1,\ldots,x_m} \overset{\text{iid}}{\sim} F_{x_1,\ldots,x_m}$$

$F_{x_1,\ldots,x_m}$는 위치 조합별로 species 비율을 정의하는 finite mixture이다. Label process의 functional dependence는 auxiliary Gaussian copula로 모델링한다.

### Order-Based Dependent Dirichlet Process (πDDP)

Griffin and Steel (2006)의 접근법은 위치 $s$마다 stick-breaking의 순서 $\pi(s)$를 다르게 정의하는 거다.

아이디어: stick-breaking에서 앞에 나올수록 weight가 크기 때문에, 비슷한 순서 $\pi(s_1) \approx \pi(s_2)$를 가진 위치들은 비슷한 distribution $F_{s_1} \approx F_{s_2}$를 갖게 된다.

구체적으로, point process $\Phi$와 집합 $U(s)$를 이용해 ordering을 정의한다.

$$\|s - z_{\pi_1(s)}\| < \|s - z_{\pi_2(s)}\| < \|s - z_{\pi_3(s)}\| < \cdots$$

즉, $s$에서 가까운 점들이 먼저 순서에 오는 거다.

1차원에서 stationary Poisson process (intensity $\lambda$)를 쓰면 autocorrelation이 analytic form으로 표현된다.

$$\text{corr}(F_{s_1}, F_{s_2}) = 1 + \frac{2\lambda h}{M+2}\exp\!\left(-\frac{2\lambda h}{M+1}\right)$$

여기서 $h = |s_1 - s_2|$, $M$은 marginal Dirichlet process의 mass parameter이다. 이 방법은 각 위치의 marginal distribution이 여전히 usual Dirichlet process를 따르면서, 관측의 영향이 거리에 따라 감소하는 local updating을 구현한다.

### Spatial Kernel Stick-Breaking (SSB) Prior

Reich and Fuentes (2007)의 SSB prior는 stick-breaking weight $V_i(s)$를 위치 $s$에 의존하게 만든다.

모델:

$$Y(s) = \eta(s) + x(s)^T\beta + \epsilon(s), \quad \epsilon(s) \overset{\text{iid}}{\sim} N(0, \tau^2)$$

$\eta(s)$의 prior:

$$F_s(\eta) \overset{d}{=} \sum_{i=1}^N p_i(s)\delta_{\theta_i} \tag{11.8}$$

$$p_i(s) = V_i(s)\prod_{j=1}^{i-1}(1 - V_j(s)), \quad V_i(s) = w_i(s)V_i$$

여기서 $V_i \sim \text{Beta}(a, b)$, $\theta_i \sim H$는 위치 무관하게 sampling되고, $w_i(s) \in [0,1]$은 kernel function으로 모델링된다. Kernel은 knot $\psi_i$를 중심으로 하고 bandwidth $\kappa_i$로 spread를 조절한다.

Kernel function의 예시는 다양한다 (uniform, exponential, squared exponential 등). Bandwidth를 fixed로 할지 random으로 할지에 따라 induced covariance 형태가 달라집니다.

### SSB의 Covariance 구조

$p_j(s)$를 conditioning하면 두 관측의 covariance는:

$$\text{Cov}(Y(s), Y(s')) = \sigma^2 P(\eta(s) = \eta(s')) = \sigma^2 \sum_{j=1}^N p_j(s)p_j(s')$$

$(V_i, \psi_i, \kappa_i)$를 적분해서 $N \to \infty$로 보내면:

$$\text{Cov}(Y(s), Y(s')) = \sigma^2\gamma(s, s')\!\left(\frac{2}{a+b+1} \cdot \frac{a+1}{a+1} - \gamma(s,s')^{-1}\right)$$

여기서 $\gamma(s, s') = \frac{\iint w_i(s)w_i(s')p(\psi_i, \kappa_i)\,d\psi_i\,d\kappa_i}{\iint w_i(s)p(\psi_i, \kappa_i)\,d\psi_i\,d\kappa_i} \in [0,1]$이다.

Prior covariance는 stationary하지만, **posterior predictive distribution은 nonstationarity를 수용**할 수 있다. 이게 SSB 모델의 가장 큰 장점이다. 전통적인 stationary kriging보다 nonstationarity에 더 robust한다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 11.
