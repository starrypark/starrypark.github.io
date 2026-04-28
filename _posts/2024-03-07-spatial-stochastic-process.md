---
title: "[Ch.2] 공간 확률 과정 이론"
description: Spatial stochastic process의 기본 개념부터 covariance function, kriging predictor까지 정리한 노트이다.
date: 2026-04-28 12:35:00 +0900
categories: [Statistics, Spatial Statistics]
tags: [spatial statistics, stochastic process, covariance function, kriging, geostatistics]
math: true
---

## 개요

Spatial statistics를 공부하다 보면 결국 근본적인 질문 하나로 돌아오게 됩니다.

> "공간상의 두 점이 대체 얼마나 비슷한가?"

이 챕터의 핵심은 바로 이 질문을 수학적으로 다루는 데 있습니다. 단순히 "거리가 가까우면 비슷하겠지"라는 직관을 넘어서, 그 구조를 정밀하게 정의하고 다루는 방법들을 소개하려고 합니다.

석사 과정을 밟으며 이 내용이 왜 그렇게 중요한지 뼈저리게 느꼈는데, 이유는 단순합니다. 나중에 Kriging이나 Gaussian process regression을 다룰 때 쓰이는 모든 밑바탕이 바로 여기서 출발하기 때문입니다.

---

## 2.1 Spatial Stochastic Process란?

**Spatial stochastic process**를 쉽게 말하면, 공간 위에 정의된 확률 변수(Random variable)들의 모음이라고 볼 수 있습니다.

$$Y(s) = Y(s, \omega), \quad s \in D \subseteq \mathbb{R}^d$$

여기서 $s$는 공간상의 위치를, $\omega$는 확률 공간의 Event를 의미합니다.

처음 이 수식을 보면 헷갈릴 수 있는데, 크게 두 가지 관점으로 쪼개서 보면 이해가 쉽습니다.

- **위치 $s$를 고정했을 때**: 특정한 위치들 $\\{s\\_1, \ldots, s\\_n\\}$에서의 값 $(Y(s\\_1), \ldots, Y(s\\_n))^T$는 하나의 Random vector가 됩니다.
- **Event $\omega$를 고정했을 때**: 모든 위치에서의 값 $Y(s, \omega)$는 하나의 실현값(Realization)을 그려냅니다.

그리고 이 과정 전체의 분포는 Finite-dimensional joint distribution의 모임으로 깔끔하게 나타낼 수 있습니다.

$$F(y_1, \ldots, y_n;\, s_1, \ldots, s_n) = P(Y(s_1) \leq y_1, \ldots, Y(s_n) \leq y_n)$$

---

## 2.2 Stationarity

공간 데이터를 다룰 때 어떤 형태로든 Stationarity(정상성) 가정이 필요합니다. 조건의 강도에 따라 세 가지로 나뉩니다.

### Strict Stationarity

**Strictly stationary**란 모든 위치를 $h$만큼 이동시켜도 그 분포가 전혀 바뀌지 않는 성질입니다.

$$F(y_1, \ldots, y_n;\, s_1+h, \ldots, s_n+h) = F(y_1, \ldots, y_n;\, s_1, \ldots, s_n)$$

직관적으로 "공간 어디서 쳐다보든 통계적 성질이 완벽히 똑같다"는 뜻입니다. 하지만 현실 세계의 데이터에 이 조건을 들이밀기엔 너무 강해서, 실용적으로 쓰이는 경우는 드뭅니다.

### Weak Stationarity (Second-Order Stationarity)

그래서 실제 분석에서는 이보다 완화된 **Weakly stationary** (또는 Second-order stationary) 조건을 주로 활용합니다. 다음 두 가지가 성립해야 합니다.

1. 평균이 위치에 무관하게 일정: $E(Y(s)) = \mu$
2. Covariance가 오직 위치의 차이에만 의존: $\text{Cov}(Y(s), Y(s+h)) = C(h)$

여기서 등장하는 $C(h)$를 **Covariance function**이라고 부릅니다. 두 점 사이의 공간적 관계를 오직 거리 벡터 $h$ 하나로만 표현해낸다는 점이 핵심입니다.

### Intrinsic Stationarity

**Intrinsically stationary**는 여기서 한 단계 더 느슨하게 푼 조건입니다.

$$E[Y(s+h) - Y(s)]^2 = 2\gamma(h), \quad E[Y(s+h) - Y(s)] = 0$$

여기서 $2\gamma(h)$를 **Variogram**, 반으로 나눈 $\gamma(h)$를 **Semi-variogram**이라고 합니다.

Weakly stationary를 만족하면 자연스럽게 Intrinsically stationary도 만족하게 되는데, 이 경우 둘 사이에 아주 예쁜 관계식이 성립합니다.

$$\gamma(h) = C(0) - C(h)$$

이게 실무에서 유용한 이유가 있습니다. Covariance function보다 Variogram을 추정하는 것이 현실적으로 훨씬 수월하기 때문입니다.

**예시: Brownian motion**
Brownian motion $W(t)$는 Intrinsically stationary이지만 Stationary는 아닌 대표적인 예입니다. 증분(Increment)이 독립이고, $\text{Var}[W(t\\_{i+1}) - W(t\\_i)] = t\\_{i+1} - t\\_i$가 됩니다. 즉, 분산이 시간 간격에 비례해서 계속 커지기 때문에 Covariance가 단순히 위치 차이에만 의존하지 못하는 것이죠.

---

## 2.3 Nugget Effect

고전적인 Geostatistical model에서는 공간 과정을 보통 세 가지 요소로 분해해서 생각합니다.

$$Y(s) = \mu(s) + \eta(s) + \epsilon(s)$$

- $\mu(s)$: 결정론적이고 부드러운 Mean function
- $\eta(s)$: 평균이 0인 연속적인 공간 과정
- $\epsilon(s)$: 공간적으로 Uncorrelated한 오차항

여기서 오차항 $\epsilon(s)$의 Covariance는 다음과 같이 뚝 떨어지는 형태로 정의됩니다.

$$\text{Cov}(\epsilon(s), \epsilon(s+h)) = \begin{cases} \sigma^2 \geq 0, & h = 0 \\ 0, & h \neq 0 \end{cases}$$

만약 $\sigma^2 > 0$이라면, 우리는 이를 **Nugget effect**라고 부릅니다. Semi-variogram의 극한으로 표현하면 $\lim\_{h \to 0} \gamma(h) = c\\_0 > 0$이 되는 현상입니다. 거리가 0에 한없이 가까워지는데도 차이가 0으로 수렴하지 않고 점프하는 느낌이죠.

Nugget effect가 발생하는 원인은 크게 두 가지를 꼽을 수 있습니다.
- **Measurement error**: 아예 같은 위치에서 측정하더라도 기기나 사람의 실수로 오차가 생기는 경우
- **Microscale variability**: 우리가 데이터를 수집하는 단위나 스케일보다 훨씬 더 미세한 공간에서 이미 변동이 일어나고 있는 경우

다만, 모델이 $L\\_2$-continuous process라면 원칙적으로 Microscale variability는 존재할 수 없습니다. 이 경우 Nugget effect를 만드는 유일한 범인은 Measurement error뿐입니다.

---

## 2.4 Bochner's Theorem

Weakly stationary process를 쓴다고 해서 $C(h)$를 내 마음대로 아무 함수나 잡을 수는 없습니다. Covariance matrix는 항상 Nonnegative definite이어야 한다는 수학적 제약이 있기 때문입니다.

이 까다로운 조건을 만족하는 함수를 **Positive definite**이라고 부르는데, 어떤 함수가 Valid한 Covariance function이 될 수 있는지 그 명확한 자격 요건을 알려주는 것이 바로 **Bochner's theorem**입니다.

**Theorem (Bochner):** 실수값을 갖는 연속 함수 $C$가 Positive definite인 것은, $\mathbb{R}^d$ 위에 정의된 Symmetric nonnegative measure $F$의 Fourier transform으로 표현되는 것과 동치이다.

$$C(h) = \int_{\mathbb{R}^d} \exp(ih^T x)\, dF(x) = \int_{\mathbb{R}^d} \cos(h^T x)\, dF(x)$$

이 우아한 적분 표현을 Covariance function의 **Spectral representation**이라고 부릅니다. 직관적으로 해석하자면, 어떤 Characteristic function의 실수 부분은 조건만 잘 맞추면 항상 훌륭한 Correlation function으로 써먹을 수 있다는 이야기입니다.

만약 Measure $F$가 Lebesgue density $f$를 가진다면, 이 $f$를 **Spectral density**라고 부릅니다. $C$가 적분 가능(Integrable)하다면 Fourier inversion formula를 써서 역방향으로도 식을 끄집어낼 수 있습니다.

$$f(x) = \frac{1}{(2\pi)^d} \int_{\mathbb{R}^d} \cos(h^T x)\, C(h)\, dh$$

---

## 2.5 Isotropic Covariance Function

### 정의

**Isotropic** process란, Covariance function이 거리 벡터 $h$의 '방향'은 깡그리 무시하고 오직 '크기(거리)' $\\|h\\|$에만 의존하는 과정을 말합니다.

$$C(h) = \phi(\|h\|), \quad h \in \mathbb{R}^d$$

북쪽으로 1km 떨어져 있든, 동쪽으로 1km 떨어져 있든 방향 상관없이 같은 거리면 같은 Correlation을 갖는다는 뜻입니다. 모델링이 훨씬 단순해지죠.

### Class $\Phi\_d$

$\Phi\\_d$를 $d$차원 공간 $\mathbb{R}^d$에서 Valid한 Isotropic covariance function을 만들어내는 함수 $\phi$들의 집합(Class)이라고 해봅시다. 흥미롭게도 차원이 커질수록 조건은 팍팍해집니다.

$$\Phi_1 \supseteq \Phi_2 \supseteq \cdots \quad \text{and} \quad \Phi_d \downarrow \Phi_\infty = \bigcap_{d \geq 1} \Phi_d$$

즉, 고차원에서 깐깐한 검증을 통과해 Valid 판정을 받은 함수는 저차원으로 내려와도 Valid하지만, 그 역은 성립하지 않습니다. 2차원에선 멀쩡하던 함수가 3차원 공간에서는 Covariance 행렬을 망가뜨릴 수 있다는 뜻입니다.

**Theorem:** $\phi \in \Phi\\_d$인 것은 다음 적분 형태로 표현 가능한 것과 동치이다.

$$\phi(t) = \int_{[0,\infty)} \Omega_d(rt)\, dF_0(r)$$

여기서 $F\\_0$는 양수 축 위의 확률 측도이고, 핵심이 되는 Generator $\Omega\\_d$는 꽤 무섭게 생겼습니다.

$$\Omega_d(t) = \Gamma(d/2)\left(\frac{2}{t}\right)^{(d-2)/2} J_{(d-2)/2}(t)$$

($J\\_\alpha$는 First kind Bessel function입니다.)

차원별로 특수한 케이스를 살펴보면 이렇습니다.

| Dimension $d$ | $\Omega_d(t)$ | Lower bound |
|---|---|---|
| 1 | $\cos t$ | -1 |
| 2 | $J_0(t)$ | -0.403 |
| 3 | $t^{-1} \sin t$ | -0.218 |
| $\infty$ | $\exp(-t^2)$ | 0 |

표를 보면 차원이 높아질수록 하한값(Lower bound)이 0에 가까워집니다. 공간의 차원이 늘어날수록 변수들끼리 강한 Negative correlation을 갖는 것이 구조적으로 불가능해진다는 것을 알 수 있습니다.

### Class $\Phi\_\infty$: Schoenberg's Theorem

무한 차원에서도 살아남는 $\Phi\\_\infty$ 클래스의 함수들은 조건이 빡센 만큼 형태가 훨씬 깔끔합니다.

**Theorem (Schoenberg):** $\phi \in \Phi\\_\infty$인 것은 다음 형태로 표현 가능한 것과 동치이다.

$$\phi(t) = \int_{[0,\infty)} \exp(-r^2 t^2)\, dF(r)$$

결국 $\Phi\\_\infty$의 원소들은 $\Omega\\_\infty(t) = \exp(-t^2)$라는 Squared exponential 함수를 스케일별로 잘 섞어놓은(Scale mixture) 형태에 불과합니다. 이 함수들은 항상 양수이고, 거리가 멀어질수록 부드럽게 감소하며, 0이 아닌 곳에서는 무한 번 미분이 가능하다는 아주 예쁜 성질을 가집니다.

---

## 2.6 Smoothness Properties

Covariance function이 원점($h=0$) 근처에서 어떻게 행동하는지를 보면, 그 과정에서 튀어나오는 Sample path가 얼마나 매끄러울지(Smoothness) 알 수 있습니다.

### Mean Square Continuity

먼저 **Mean square continuous**의 정의를 보겠습니다.

$$E(Y(s) - Y(s+h))^2 \to 0 \quad \text{as } \|h\| \to 0$$

Second-order stationary process를 가정할 때, 이 식이 성립한다는 것은 곧 Covariance function이 원점에서 연속이라는 말과 완벽히 동치입니다.

처음 공부할 때 여기서 헷갈리기 쉬운데, "Mean square continuous하니까 함수 그래프(Sample path)도 끊어지지 않고 이어지겠지?"라고 생각하면 안 됩니다. Mean square 연속성과 Sample path의 연속성은 아예 다른 차원의 이야기입니다. 

### Fractal Dimension과 Smoothness

Isotropic process의 경우 $\phi(t)$가 원점 근처에서 어떻게 떨어지는지가 Smoothness를 결정짓습니다. 구체적으로 어떤 $\alpha \in (0, 2]$에 대해서 아래의 감소율을 보인다고 해봅시다.

$$1 - \phi(t) \sim t^\alpha \quad \text{as } t \downarrow 0$$

그러면 이와 연관된 Gaussian process의 실현값은 다음과 같은 Fractal dimension을 가집니다.

$$D = d + 1 - \frac{\alpha}{2}$$

즉, 원점 근처에서 함수가 천천히 떨어질수록($\alpha$가 클수록) 차원 $D$의 값이 작아지고, 덜 구불거리는 매끄러운 Sample path가 그려집니다.

**Theorem:** $\phi(u)$가 위 조건을 만족하고 $\alpha = 2$일 때, 대칭으로 연장한 $c(u) = \phi(\\|u\\|)$가 원점에서 $2m$번 미분 가능하다면, 이는 Associated Gaussian process의 Sample path가 $m$번 미분 가능하다는 것과 동치이다.

---

## 2.7 Isotropic Covariance Function의 예시들

이론은 여기까지 하고, 실제로 코드를 짜거나 논문을 쓸 때 자주 만나는 유명한 함수들을 소개합니다.

### Matérn Class

실무와 연구를 통틀어 공간 통계에서 가장 사랑받는 가문(Family)입니다.

$$\phi(t) = \frac{2^{1-\nu}}{\Gamma(\nu)} \left(\frac{t}{\theta}\right)^\nu K_\nu\left(\frac{t}{\theta}\right), \quad \nu > 0,\ \theta > 0$$

수식에 있는 $K\\_\nu$는 Modified Bessel function이라 좀 복잡해 보이지만, 파라미터의 역할은 직관적입니다.
- $\nu$: **Smoothness parameter**. Sample path가 $m$번 미분 가능하려면 $m < \nu$를 만족해야 합니다.
- $\theta$: **Scale parameter**. 거리에 따라 Correlation이 얼마나 빨리 식는지 결정합니다.

Matérn class는 그 까다롭다는 $\Phi\\_\infty$에 속하기 때문에 어떤 차원($d \geq 1$)에서 쓰든 에러가 나지 않습니다. $\nu$값에 따라 형태가 어떻게 변하는지 볼까요?

| $\nu$ | Correlation function |
|---|---|
| $1/2$ | $\exp(-t)$ |
| $1$ | $t K_1(t)$ |
| $3/2$ | $(1+t)\exp(-t)$ |
| $5/2$ | $\left(1 + t + \frac{t^2}{3}\right)\exp(-t)$ |

$\nu = 1/2$이면 뾰족한 Exponential 형태가 되고, $\nu$가 커질수록 점점 부드러워져서 $\nu \to \infty$가 되면 Squared exponential로 수렴합니다. 데이터의 Smoothness를 유연하게 맞춰줄 수 있다는 게 가장 큰 장점입니다.

### Powered Exponential Family

$$\phi(t) = \exp\left(-\left(\frac{t}{\theta}\right)^\alpha\right), \quad 0 < \alpha \leq 2,\ \theta > 0$$

이 녀석도 $\Phi\\_\infty$에 속하고 수식이 직관적이라 쓰기 편합니다. 하지만 치명적인 단점이 있는데, 매끄러움의 정도(Smoothness)를 미세하게 조절할 수가 없습니다.
$\alpha = 2$이면 아예 무한 번 미분 가능할 정도로 밋밋해지고, $\alpha < 2$로 조금만 내려가도 아예 한 번도 미분할 수 없게 되어버립니다. Matérn class에 밀리는 이유가 이것이죠.

### Cauchy Family

$$\phi(t) = \left(1 + \left(\frac{t}{\theta}\right)^\alpha\right)^{-\beta/\alpha}, \quad 0 < \alpha \leq 2,\ \beta > 0,\ \theta > 0$$

여기서는 $\beta$라는 **Long-memory parameter**가 핵심입니다. 거리가 무한히 멀어질 때 $\phi(t) \sim t^{-\beta}$의 속도로 다소 느리게(Power law) 감소하는 특징을 보입니다. $\beta$가 작을수록 멀리 있는 점들끼리도 미련을 못 버리고 서로 영향을 주고받는 Long-range dependence를 모델링하기 좋습니다.

### Hole Effect

가끔 데이터 중에 멀리 떨어져 있는데도 유의미한 음의 상관관계(Negative correlation)가 나타나는 경우가 있습니다. 이 현상을 **Hole effect**라고 부르는데, 이를 모델에 우겨넣고 싶을 땐 파동이 감쇠하는 Exponentially damped cosine 모델을 씁니다.

$$\phi(t) = \exp\left(-\tau \frac{t}{\theta}\right) \cos\left(\frac{t}{\theta}\right), \quad \tau \geq \frac{1}{\tan\frac{\pi}{2d}},\ \theta > 0$$

주의할 점은 아무렇게나 쓰면 안 되고, 감쇠 파라미터 $\tau$가 저 오른쪽의 조건식을 만족해야만 $\Phi\\_d$에 속하는 Valid한 모델이 된다는 것입니다.

### Spherical Covariance Function

거리가 일정 수준 이상 멀어지면 미련 없이 Correlation을 딱 '0'으로 끊어버리는 모델입니다. (**Compact support**를 가집니다.)

$$\phi(t) = \begin{cases} 1 - \dfrac{3}{2}\dfrac{t}{\theta} + \dfrac{1}{2}\left(\dfrac{t}{\theta}\right)^3 & \text{if } t < \theta \\ 0 & \text{otherwise} \end{cases}$$

영향력을 계산할 때 거리가 먼 데이터는 아예 행렬에서 지워버리면 되기 때문에 계산 속도가 비약적으로 빨라집니다. 하지만 3차원 이하($d \leq 3$)에서만 Valid하기 때문에 고차원 데이터에는 함부로 들이밀면 안 됩니다.

---

## 2.8 Prediction Theory: Kriging

결국 이 복잡한 이론들을 배우는 이유는 공간 통계의 알파이자 오메가인 **Prediction(예측)**을 하기 위해서입니다. "내가 아직 안 가본 $s$라는 위치의 값 $Y(s)$를 어떻게 기가 막히게 때려 맞출 것인가?"

가장 오차를 줄이는(Least squares) 관점에서 생각해보면, 완벽한 정답(Optimal predictor)은 우리가 아는 모든 데이터를 쥐어짜서 구한 조건부 기댓값입니다.

$$\hat{Y}(s) = E(Y(s) \mid Y(s_1) = y_1, \ldots, Y(s_n) = y_n)$$

문제는 현실의 데이터로 저 조건부 기댓값을 딱 떨어지는 수식(Closed form)으로 구하는 건 불가능에 가깝다는 겁니다. 하지만 우리가 데이터를 Multivariate normal로 깔고 들어가면 마법처럼 문제가 풀립니다.

### Gaussian Conditional Distribution

어떤 두 확률 벡터 $(U, V)^T$가 Multivariate normal 분포를 따른다고 쳐봅시다.

$$\begin{pmatrix} U \\ V \end{pmatrix} \sim \mathcal{N}\left(\begin{pmatrix} \mu_U \\ \mu_V \end{pmatrix},\ \begin{pmatrix} \Sigma_{UU} & \Sigma_{UV} \\ \Sigma_{VU} & \Sigma_{VV} \end{pmatrix}\right)$$

그러면 $U$가 주어졌을 때 $V$의 조건부 분포도 예쁜 정규분포가 됩니다.

$$(V \mid U) \sim \mathcal{N}\left(\mu_V + \Sigma_{VU}\Sigma_{UU}^{-1}(U - \mu_U),\ \Sigma_{VV} - \Sigma_{VU}\Sigma_{UU}^{-1}\Sigma_{UV}\right)$$

### Ordinary Kriging Predictor

위의 성질을 그대로 가져와서 $V = Y(s)$(우리가 알고 싶은 값)로 두고, $U = (Y(s\\_1), \ldots, Y(s\\_n))^T$(우리가 이미 아는 데이터들)로 세팅해 봅시다. 그리고 데이터가 평균 $\mu$와 공분산 $C$를 가지는 Second-order stationary process라고 가정하면 다음 식이 도출됩니다.

$$\hat{Y}(s) = \mu + \bigl(C(s-s_1), \ldots, C(s-s_n)\bigr) \bigl[C(s_i - s_j)\bigr]^{-1} \begin{pmatrix} Y(s_1) - \mu \\ \vdots \\ Y(s_n) - \mu \end{pmatrix}$$

이 웅장한 수식이 바로 공간 통계의 꽃, **Ordinary kriging predictor**입니다.

이 예측 모델에는 놀라운 비밀이 하나 숨어있는데, 애초에 데이터가 정규분포(Gaussian)를 따르지 않더라도 1차 결합으로 만들 수 있는 예측기 중에서는 무조건 이 녀석이 가장 오차가 작다는 것(Best linear predictor)이 수학적으로 증명되어 있습니다.

이렇게 그려낸 예측 곡면(Surface)은 보통 두 가지 특징을 보여줍니다.
- Nugget effect(측정 오차 등)가 없다고 가정하면, 내가 이미 알고 있는 관측점에서는 100% 그 값을 정확히 통과합니다 (Interpolation).
- 관측점이 없는 빈 공간으로 넘어갈 때 곡면이 얼마나 출렁일지, 얼마나 부드럽게 이어질지는 전적으로 우리가 아까 고른 Covariance function $C$의 생김새를 그대로 물려받습니다.

결국 "예측을 얼마나 매끄럽게 할 것인가"는 전적으로 우리가 어떤 Covariance function을 채택하느냐에 달려있다는 뜻이죠. 앞에서 왜 그렇게 빡세게 Smoothness니, Matérn class니 배웠는지 이제 퍼즐이 맞춰집니다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 2.