---
title: "[Ch.3] 고전 Geostatistical 방법론"
description: Geostatistical model의 구조부터 semivariogram 추정, kriging predictor까지 classical geostatistics의 전체 흐름을 정리한 노트입니다.
date: 2024-03-21 12:37:00 +0900
categories: [Statistics, Spatial Statistics]
tags: [geostatistics, semivariogram, kriging, spatial prediction, variogram]
math: true
---

## 개요

Chapter 2에서 Spatial stochastic process의 이론적 토대를 단단하게 다졌다면, 이번 Chapter 3에서는 그 이론을 현실의 데이터에 어떻게 써먹을 것인지 실전적인 이야기를 다룹니다.

관측된 데이터가 주어졌을 때, 공간적인 구조를 모델링하고, 결국 내가 아직 확인하지 못한 위치의 값을 예측해 내는 것. 이것이 Classical geostatistics가 해결하고자 하는 전체적인 흐름입니다.

작업은 크게 네 가지 단계로 쪼개볼 수 있습니다.

1. Mean function을 임시로 쓱 추정해보기 (OLS 활용)
2. 잔차를 바탕으로 Semivariogram을 Nonparametric하게 추정하기
3. 그 거친 Semivariogram 위에 뼈대가 있는 Parametric model 입히기
4. 공간적 상관구조를 반영하여 Mean function을 다시 정밀하게 추정하고, 최종적으로 Kriging 예측하기

이 흐름을 머릿속에 담아두고 각 단계를 하나씩 따라가 보겠습니다.

---

## 3.1 Geostatistical Model

공간 데이터를 다룰 때 가장 기본이 되는 모델의 뼈대는 아래와 같습니다.

$$Y(s) = \mu(s) + e(s)$$

- $\mu(s) \equiv E[Y(s)]$: 공간의 큰 흐름(Trend)을 설명해 주는 결정론적이고 연속적인 Mean function
- $e(\cdot)$: 평균이 0이고, 공간적인 상관성을 머금고 있는 Random error process

여기서 핵심은 Error process $e(\cdot)$에 어떤 형태의 Stationarity(정상성) 가정을 씌울 것인가입니다.

**Second-order stationarity**를 가정한다면:

$$\text{Cov}[e(s), e(t)] = C(s - t)$$

공간상 두 점 사이의 Covariance가 점들의 절대적인 위치가 아니라, 오직 두 점이 얼마나 떨어져 있는지(거리와 방향 차이)에만 의존한다는 뜻입니다.

한 단계 느슨한 **Intrinsic stationarity**를 가정한다면:

$$\frac{1}{2}\text{Var}[e(s) - e(t)] = \gamma(s - t)$$

여기서 $\gamma(\cdot)$을 **Semivariogram**이라고 부르고, 2를 곱한 $2\gamma(\cdot)$을 **Variogram**이라고 부릅니다. 

이 모델이 품고 있는 철학은 명확합니다. $\mu(s)$가 데이터의 굵직한 Large-scale variation을 잡아내고, $e(s)$가 그 주변에서 자글자글하게 얽혀있는 Small-scale variation(공간적 의존성)을 설명하겠다는 것이죠. 
하지만 현실 분석을 해보면 이 '큰 흐름'과 '작은 변동'을 칼같이 분리해 내는 것은 불가능에 가깝습니다. 어느 선까지 트렌드로 보고 어디부터 공간적 오차로 볼 것인지에 대한 Ambiguity는 연구자가 감수하고 가야 할 몫입니다.

### Error process의 추가 분해

상황에 따라서는 $e(s)$를 한 겹 더 벗겨내기도 합니다.

$$e(s) = \eta(s) + \epsilon(s)$$

- $\eta(\cdot)$: 실제로 공간적인 끈끈함을 가진(Dependent) 본질적인 오차 성분
- $\epsilon(\cdot)$: 순수하게 기계나 사람의 실수로 발생하는 공간적 상관성이 없는(Uncorrelated) Measurement error

### Isotropy와 Anisotropy

Intrinsically stationary process 중에서, Semivariogram이 두 점 사이의 '방향'은 깡그리 무시하고 오직 직선거리 $\\|h\\|$에만 의존할 때가 있습니다.

$$\gamma(h) = \gamma(\|h\|)$$

이런 예쁜 케이스를 **(Intrinsically) isotropic** 하다고 부릅니다. 방향 상관없이 물결이 동심원으로 퍼져나가는 모습을 상상하면 됩니다.

반대로 방향에 따라 퍼지는 정도가 다른 경우를 **Anisotropic**이라고 합니다. 이 중에서 수학적으로 가장 다루기 편한 것이 **Geometric anisotropy**인데, Positive definite matrix $A$를 가져와서 공간을 적절히 찌그러뜨리거나 늘려주는 방식을 씁니다.

$$\gamma(h) = \gamma\left((h^T A h)^{1/2}\right)$$

이렇게 하면 특정 방향으로는 거리가 더 빨리 멀어지게, 혹은 더 천천히 멀어지게 스케일을 조정할 수 있습니다.

---

## 3.2 Mean Function의 임시 추정

본격적인 공간 분석에 앞서, 데이터에 껴있는 큰 덩어리(Trend)를 먼저 발라내야 합니다. 이때는 아직 두 점 사이의 Covariance 구조를 모르는 상태이므로, 일단 공간 구조를 무시하고 덤비는 방법을 씁니다.

### Linear Mean Model

가장 만만하고 널리 쓰이는 게 바로 Linear model입니다.

$$\mu(s; \beta) = X(s)^T \beta$$

$X(s)$는 위치 $s$에서 수집된 Covariate vector이고, $\beta$는 우리가 찾아내야 할 파라미터입니다.

여기에 들어갈 수 있는 Covariate들은 대략 이렇습니다.
- Intercept (절편, 필수)
- 위도, 경도 같은 Geographic coordinates
- 그 좌표들을 지지고 볶은 Polynomial 함수
- 고도나 온도 같은 기타 Attribute variable

만약 써먹을 만한 추가 변수가 없고 오직 위치 좌표계밖에 없다면, 좌표의 다항식(Polynomial)으로 떡칠을 해서 모델을 만드는데, 이를 전문 용어로 **Trend surface model**이라고 부릅니다.

2차원 공간($s = (s\\_1, s\\_2)$)에서 예를 들어보면:
- **1차 (Planar)**: 평면 모양의 트렌드 $\mu(s;\beta) = \beta\\_0 + \beta\\_1 s\\_1 + \beta\\_2 s\\_2$
- **2차 (Quadratic)**: 밥그릇 모양의 트렌드 $\mu(s;\beta) = \beta\\_0 + \beta\\_1 s\\_1 + \beta\\_2 s\\_2 + \beta\\_{11}s\\_1^2 + \beta\\_{12}s\\_1 s\\_2 + \beta\\_{22}s\\_2^2$

다항식을 쓸 때 소소한 팁이 있다면, $q$차 모델을 짠다면 차수가 $q$ 이하인 모든 Monomial 항을 싹 다 집어넣는 "Full" polynomial을 구성하는 게 낫습니다. 그래야 나중에 누군가 좌표계의 원점이나 각도를 휙 돌려버려도 예측 결과가 뒤틀리지 않거든요.

### OLS Estimator

이 Linear mean function을 피팅할 때 가장 먼저 손이 가는 도구는 단연 **Ordinary Least Squares (OLS)**입니다.

$$\hat{\beta}_{\text{OLS}} = \underset{\beta}{\arg\min} \sum_{i=1}^n \left[Y(s_i) - X(s_i)^T \beta\right]^2$$

만약 $X$ 행렬이 꽉 찬(Full column rank) 상태라면 아주 예쁜 수식으로 한 번에 답이 나옵니다.

$$\hat{\beta}_{\text{OLS}} = (X^T X)^{-1} X^T Y$$

하지만 여기서 주의할 점이 있습니다. 공간 데이터는 자기들끼리 끈끈하게 엮여있기 때문에, OLS가 태생적으로 깔고 있는 '오차항끼리 독립이다(Independent errors)'라는 가정이 와장창 깨져버립니다. 따라서 이 단계에서 구한 OLS 결과는 어디까지나 뼈대만 잡아보는 **Exploratory** 목적으로만 써야 합니다.

### OLS의 실용적 한계

1. **Multicollinearity (다중공선성)**: 다항식 모델을 쓰다 보면 변수들끼리 상관관계가 너무 높아져서 $\beta$의 분산이 폭발하는 경우가 생깁니다. 추정된 계수 자체에 큰 의미를 두지 않을 거라면, 좌표를 Centering 하거나 직교화(Orthogonalize) 시켜서 열을 내릴 수 있습니다.
2. **Boundary distortion**: 관측소가 없는 구석진 영역에서 모형이 혼자 미쳐 날뛰며 Surface가 기괴하게 꺾이는 현상입니다. 데이터가 공간에 고르게 뿌려져 있어야 이 현상을 피할 수 있습니다.
3. **Outlier sensitivity**: OLS는 태생적으로 튀는 값에 매우 취약합니다. Robust regression으로 방어할 수 있긴 한데, 애초에 공간 데이터에서 이게 진짜 에러인지, 아니면 그 동네 특유의 기질인지(진짜 Outlier의 정의)를 판단하는 것 자체가 꽤 철학적인 문제가 됩니다.

---

## 3.3 Semivariogram의 Nonparametric 추정

일단 큰 덩어리(Mean)를 대충 덜어냈으니, 이제 남은 잔차(Residual)를 들여다보며 공간의 끈적끈적한 구조(Dependence)를 파악할 차례입니다.

### Regular Grid에서의 Empirical Semivariogram

데이터가 바둑판처럼 예쁜 격자(Regular grid) 위에 놓여있다고 해봅시다. 점들 사이의 고유한 거리 벡터(Lag)들을 $h\\_1, \ldots, h\\_k$라고 할 때, 계산식은 꽤 직관적입니다.

**Definition (Empirical semivariogram):**

$$\hat{\gamma}(h_u) = \frac{1}{2N(h_u)} \sum_{s_i - s_j = h_u} \{\hat{e}(s_i) - \hat{e}(s_j)\}^2, \quad u = 1, \ldots, k$$

$\hat{e}(s\\_i)$는 아까 OLS 피팅하고 남은 찌꺼기(Residual)이고, $N(h\\_u)$는 정확히 그 간격만큼 떨어져 있는 점 쌍(Pair)의 개수입니다. 만약 원래 데이터의 Mean이 상수였다면, 이 추정량은 통계적으로 완벽하게 Unbiased 하다는 좋은 성질을 가집니다.

### Irregular Grid에서의 Empirical Semivariogram

문제는 현실 데이터가 바둑판처럼 예쁘게 놓여있을 리가 없다는 겁니다. 관측소 위치가 삐뚤빼뚤(Irregular)하면 점들 사이의 거리 벡터가 정확히 일치하는 경우가 거의 안 나옵니다. 

그래서 이때는 Lag 공간 전체를 뭉텅뭉텅 바구니(Bin)에 쓸어 담는 분할 전략을 씁니다. $H\\_1, \ldots, H\\_k$라는 Bin을 만들고,

$$\hat{\gamma}(h_u) = \frac{1}{2N(H_u)} \sum_{s_i - s_j \in H_u} \{\hat{e}(s_i) - \hat{e}(s_j)\}^2, \quad u = 1, \ldots, k \tag{3.5}$$

비슷한 거리와 방향을 가진 쌍들을 한 Bin 안에 모아서 평균을 내버리는 겁니다. 

보통 공간을 피자 조각처럼 나누는 **Polar partition** 방식을 가장 많이 씁니다. 방향(Angle)과 거리(Distance) 기준을 동시에 둬서 조각을 내면 두 가지 분석을 할 수 있습니다.
- **Directional empirical semivariogram**: 피자 한 조각 안에서 거리만 바꿔가며 방향별 특성을 보는 것
- **Omnidirectional empirical semivariogram**: 굳이 방향 안 가리고, 동심원 형태로 띠를 묶어서 전체적인 거리 특성만 보는 것

### Bin 수의 Trade-off

여기도 영원한 숙제인 Trade-off가 존재합니다.
- Bin을 너무 잘게 쪼개면: "이 거리는 대충 5km야!"라는 대푯값 $h\\_u$의 신뢰도는 올라가지만, 그 바구니 안에 들어가는 점의 쌍 $N(H\\_u)$가 너무 적어져서 통계적 분산이 널뛰게 됩니다.
- Bin을 너무 듬성듬성 크게 묶으면: 쌍은 많아져서 안정적이긴 한데, 3km와 7km가 같은 취급을 받으니 공간 해상도가 뭉개집니다.

실무에서 자주 쓰이는 룰(Rule of thumb)은 "최소한 바구니당 30쌍($N(h\\_u) \geq 30$)은 확보해야 하고, 분석하는 최대 Lag 길이는 전체 영역 크기의 절반을 넘기지 말아라"입니다. 하지만 30쌍을 훌쩍 넘겨도 여전히 추정값이 널뛸 수 있는데, 그건 점들 사이의 변동성이 독립적이지 않고 서로 얽혀있기 때문입니다.

### Robust Estimator (Cressie & Hawkins, 1980)

수식을 보면 제곱의 합(Sum of squares) 구조라서, 잔차 하나만 크게 튀어도 전체 Variogram 값이 우주로 날아갑니다. 이걸 방어하기 위해 Cressie와 Hawkins가 제안한 **Robust estimator**가 있습니다.

$$\bar{\gamma}(h_u) = \frac{\left\{\frac{1}{N(H_u)} \sum_{s_i - s_j \in H_u} |\hat{e}(s_i) - \hat{e}(s_j)|^{1/2}\right\}^4}{.914 + [.988/N(H_u)]}$$

절댓값에 루트를 씌워서 튀는 값의 영향을 확 죽인 다음, 밖에서 다시 4제곱을 때려주는 꽤 신박한 수학적 트릭을 씁니다.

---

## 3.4 Semivariogram Modeling

### 왜 Parametric Model이 필요한가?

기껏 Empirical semivariogram을 찍어봤더니 점들이 위아래로 미친 듯이 널뛰는 모습을 보게 될 겁니다. 이 거친 점들 위에 수학적으로 반듯한 Parametric model을 씌워야 하는 이유는 크게 세 가지입니다.

1. 점들로만 보면 너무 거칠고 불규칙(Bumpy)해서, 이를 매끄럽게(Smooth) 다듬어야 이후 예측이 안정됩니다.
2. 그냥 찍어낸 점들은 나중에 Covariance 행렬로 만들 때 Conditional negative definiteness라는 필수 조건을 통과하지 못할 확률이 큽니다. 이 조건이 깨지면 나중에 분산이 음수(-)로 계산되는 대형 사고가 터집니다.
3. 점과 점 사이의 애매한 거리(우리가 관측하지 못한 Lag)에서의 상관관계 값도 필요한데, 수식이 있어야 이걸 계산해 낼 수 있습니다.

### Valid Semivariogram의 조건

내가 짠 Semivariogram 모델 $\gamma(h;\theta)$가 수학적으로 합격 판정을 받으려면 다음 세 관문을 무사히 통과해야 합니다.

1. $\gamma(0;\theta) = 0$ (거리가 0이면 차이도 당연히 0)
2. $\gamma(-h;\theta) = \gamma(h;\theta)$ (방향만 반대일 뿐 차이는 같아야 하는 Even function)
3. **Conditional negative definiteness**: 가중치 합이 0($\sum\\_i a\\_i = 0$)일 때, $\sum\\_i \sum\\_j a\\_i a\\_j \gamma(s\\_i - s\\_j;\theta) \leq 0$를 항상 만족. (이게 제일 빡센 조건입니다.)

### 주요 Isotropic Semivariogram 모델들

실무에서 가장 많이 꺼내 쓰는 대표적인 Isotropic 모델 다섯 가지입니다. 

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

(여기서 $\theta\\_1$은 모두 양수입니다.) 
Matérn 모델이 재밌는 게, 파라미터 $\nu$를 0.5로 두면 완벽하게 Exponential 모델이 되고, 무한대로 쏴버리면 Gaussian 모델로 부드럽게 변신합니다.

### 모델 속성들

Semivariogram의 생김새를 결정짓는 핵심 세 가지 지표를 짚고 넘어갑시다.

**Sill**

$$\text{sill} = \lim_{h \to \infty} \gamma(h;\theta)$$

거리가 멀어지면 멀어질수록 Semivariogram 값이 어느 한계선에 부딪혀 더 이상 안 올라가고 평평해질 때가 있는데, 이 천장 값을 **Sill**이라고 합니다. 모델에 Sill이 존재한다는 건 곧 그 데이터가 Second-order stationary를 만족한다는 뜻이고, 이때 이론적인 Covariance $C(0;\theta)$값은 Sill과 완벽히 일치하게 됩니다.

**Range**

모델이 꺾여서 저 Sill에 도달하기까지 걸리는 거리 $h$를 **Range**라고 합니다. 딱 떨어지지 않고 영원히 수렴만 하는 모델(Exponential, Gaussian 등)의 경우, Sill의 95% 턱밑까지 차오르는 거리를 **Effective range**로 삼습니다. (Exponential은 대략 $3\theta\\_2$, Gaussian은 대략 $\sqrt{3}\theta\\_2$ 쯤 됩니다.)

문제는 이 Range 파라미터를 추정하는 게 여간 까다로운 게 아닙니다. 특히 영향력이 미치는 거리가 우리가 관측한 전체 영역 크기랑 비슷해져 버리면 거의 답이 없습니다. 그래서 아예 "수렴 안 하고 계속 무한대로 뻗어나갈 거야!"라며 Power class 모델을 던져버려서 이 골치 아픈 문제를 피해버리기도 합니다.

**Nugget effect**

$$\text{nugget} = \lim_{h \to 0} \gamma(h;\theta)$$

이론적으로는 거리가 0으로 가면 값도 0으로 꽂혀야 하지만, 현실의 측정 오차 때문에 원점에서 붕 떠서 출발하는 높이를 **Nugget**이라고 합니다. 기본 모델들은 이걸 0으로 깔고 가지만, 필요하면 상수 $\theta\\_3$를 톡 떨어뜨려서 추가할 수 있습니다. 

예를 들어 Nugget을 얹은 Exponential 모델은 이렇게 생겼습니다.

$$\gamma(h;\theta) = \begin{cases} 0 & h = 0 \\ \theta_3 + \theta_1\{1 - \exp(-h/\theta_2)\} & h > 0 \end{cases}$$

**모델 선택 시 소소한 팁**
- Gaussian 모델은 수학적으로 너무 과하게 매끄러워서(Too smooth), 거친 자연 현상을 반영하기엔 안 맞을 때가 많습니다.
- 매끄러움의 정도를 미세하게 조절할 수 있는 Matérn 모델(단, $\nu$가 적당히 1보다 큰 수준)이 실무자들에게 인기가 좋습니다.
- Spherical 모델은 지구통계학 동네에서 전통적으로 사랑받지만, 모델이 꺾이는 지점($\theta\\_2 = h$)에서 Likelihood 함수가 뾰족해져서 수학적으로 골치를 썩일 때가 있습니다.

### Anisotropic으로 일반화

지금까지 본 둥글둥글한(Isotropic) 모델들을 찌그러뜨리고 싶다면, 거리 $h$ 자리에 아까 배운 늘리기 행렬 수식 $(h^T A h)^{1/2}$을 쏙 밀어 넣으면 됩니다. 

예를 들어 2차원 공간에서 각도를 튼 Exponential 모델은 이런 무시무시한 꼴이 됩니다.

$$\gamma(h;\theta) = \theta_1 \left\{1 - \exp\left(-\frac{(h_1^2 + 2\theta_3 h_1 h_2 + \theta_4 h_2^2)^{1/2}}{\theta_2}\right)\right\}$$

참고로 이 방식을 안 따르고 각 파트별로 아예 따로 노는 Zonally anisotropic 모델은 이론과 계산 모두 엉망진창이라 실무에선 거의 버려진 카드입니다.

### WLS Estimator

모델의 파라미터 $\theta$를 추정하는 대표적인 방법에는 **WLS**와 **ML/REML**이 있는데, 여기서는 고전적인 WLS(Weighted Least Squares)만 짧게 보겠습니다.

**Definition (WLS estimator):**

$$\hat{\theta} = \underset{\theta}{\arg\min} \sum_{u \in U} \frac{N(h_u)}{[\gamma(h_u;\theta)]^2} [\hat{\gamma}(h_u) - \gamma(h_u;\theta)]^2$$

수식을 뜯어보면 분모에 이론적 Variogram 값 $\gamma(h\\_u;\theta)$의 제곱이 깔려 있습니다. 이게 무슨 뜻이냐면, $h$가 작아서 이론값이 작을 때 가중치가 미친 듯이 커진다는 겁니다. 즉, 저 멀리 있는 데이터들의 오차보다 **서로 가까이 붙어있는 원점 근처 데이터들의 핏(Fit)을 훨씬 중요하게 본다**는 아주 훌륭한 성질을 가지고 있습니다.

물론 베이지안이나 우도(Likelihood) 기반의 깐깐한 이론 모델은 아니지만, 데이터가 심하게 꺾여있는(Nondifferentiable) 환경에서는 생각보다 ML 방식과 비등비등한 괜찮은 성능을 보여줍니다.

---

## 3.5 Mean Function 재추정

처음에 OLS로 대충 발라냈던 Mean function을 기억하시나요? 이제 우리가 번듯한 Semivariogram(공간 상관구조)을 손에 넣었으니, 이걸 활용해서 Mean function을 다시 정밀하게 깎아낼 차례입니다.

이때 쓰는 칼이 바로 **Estimated Generalized Least Squares (EGLS)**입니다. 원래 이론적인 GLS 공식에 들어갈 Variance-covariance 행렬 자리에, 방금 우리가 기껏 추정한 녀석들을 대입해 넣는 겁니다.

행렬을 조립하는 과정은 이렇습니다.
1. 각 위치 $Y(s\\_i)$들의 공통된 분산을 아까 피팅한 Semivariogram 모델의 최고점(**Sill**)으로 박아 넣습니다. এটাকে $\hat{C}(0)$라고 부릅니다.
2. 두 점 $Y(s\\_i)$와 $Y(s\\_j)$ 사이의 공분산은 아래 공식으로 우회해서 추정합니다.

$$\hat{C}(s_i - s_j) = \hat{C}(0) - \gamma(s_i - s_j;\hat{\theta})$$

이렇게 긁어모은 값들로 꽉 찬 거대한 추정 공분산 행렬 $\hat{\Sigma} = [\hat{C}(s\\_i - s\\_j)]$를 만듭니다.

**Definition (EGLS estimator):**

$$\hat{\beta}_{\text{EGLS}} = (X^T \hat{\Sigma}^{-1} X)^{-1} X^T \hat{\Sigma}^{-1} Y$$

OLS 공식 가운데에 $\hat{\Sigma}^{-1}$가 예쁘게 끼어 들어간 형태입니다. $\hat{\beta}\\_{\text{EGLS}}$는 극단적인 상황만 아니면 편향되지 않은(Unbiased) 깔끔한 값을 내어줍니다. 물론 진짜 신이 내려준 $\theta$값을 알고 계산하는 완벽한 GLS보다는 분산이 조금 클 수밖에 없습니다.

어쨌든 OLS의 독립 가정이라는 망상에서 벗어나, 데이터 간의 끈적한 공간적 관계 구조를 모델에 온전히 태울 수 있다는 점에서 어마어마한 진전입니다. 

---

## 3.6 Kriging

먼 길을 돌아왔습니다. Classical geostatistical analysis의 대미를 장식할 마지막 단계, 바로 **Prediction(예측)**입니다. 관측소도 뭣도 없는 허허벌판 $s\\_0$ 위치에 도대체 어떤 값 $Y(s\\_0)$가 뜰 것인지 기가 막히게 찍어보는 이 일련의 기법들을 통틀어 **Kriging**이라고 부릅니다.

### Universal Kriging

Kriging 중에서도 가장 보편적인 뼈대인 **Universal kriging**은, "이 두 조건만 맞춰준다면 내가 가질 수 있는 최소의 예측 오차 분산(Prediction error variance)을 뽑아내겠다!"라는 최적화 문제에서 출발합니다.

1. **Linearity**: 예측값은 결국 우리가 아는 관측값 $Y$들의 1차 결합으로 떨어져야 한다. ($\hat{Y}(s\\_0) = \lambda^T Y$)
2. **Unbiasedness**: 내가 찍은 값의 기댓값과 실제 값의 기댓값은 일치해야 한다. ($E[\hat{Y}(s\\_0)] = E[Y(s\\_0)]$, 즉 $\lambda^T X = X(s\\_0)$)

만약 우리가 Semivariogram을 완벽히 알고 있다고 치고 저 조건부 최소화 문제를 끙끙 풀다 보면, 다음과 같은 **Universal kriging predictor** 공식이 툭 튀어나옵니다.

$$\hat{Y}(s_0) = \left[\gamma + X\left(X^T \Gamma^{-1} X\right)^{-1}\left(x_0 - X^T \Gamma^{-1} \gamma\right)\right]^T \Gamma^{-1} Y \tag{3.6}$$

수식이 꽤 험악한데, 각 조각은 이렇습니다.
- $\gamma = [\gamma(s\\_1 - s\\_0), \ldots, \gamma(s\\_n - s\\_0)]^T$: (내가 예측할 점과 기존 점들 사이의 Semivariogram 벡터)
- $\Gamma$: $(i,j)$ 위치에 기존 점들끼리의 $\gamma(s\\_i - s\\_j)$를 빼곡히 채워 넣은 $n \times n$ 행렬
- $x\\_0 = X(s\\_0)$: (예측할 위치의 설명 변수 값)

그리고 이때 우리가 감수해야 할 최소 방어선, **Kriging variance**(예측 오차 분산)는 다음과 같습니다.

$$\sigma^2(s_0) = \gamma^T \Gamma^{-1} \gamma - \left(X^T \Gamma^{-1} \gamma - x_0\right)^T \left(X^T \Gamma^{-1} X\right)^{-1} \left(X^T \Gamma^{-1} \gamma - x_0\right) \tag{3.7}$$

통계학에서는 이렇게 선형이고 편향도 없으면서 오차까지 제일 작은 완벽한 예측기를 **BLUP (Best Linear Unbiased Predictor)**이라고 칭송합니다.

만약 데이터 $Y(\cdot)$가 정규분포(Gaussian)를 따른다는 축복받은 상황이라면, 이 예측값에 분산을 위아래로 붙여서 아주 깔끔한 신뢰구간(Prediction interval)도 만들 수 있습니다.

$$\hat{Y}(s_0) \pm z_{\alpha/2}\, \sigma(s_0)$$

진짜 $\gamma(\cdot)$를 알고 있다면, 이 구간 안에 실제 정답이 들어올 확률은 기가 막히게 정확히 $1 - \alpha$가 됩니다.

### 실제 적용에서의 수정

물론 현실은 항상 시궁창이라 위의 이상적인 이론을 두 가지 방향에서 꺾어서 써야 합니다.

**1. Local kriging**
수식에 있는 저 무시무시한 $\Gamma^{-1}$ 역행렬을 계산하려면 데이터 전체를 끌고 와야 하는데, 데이터가 수십만 개면 컴퓨터가 터집니다. 그래서 "어차피 멀리 있는 애들은 영향력도 별로 없잖아?"라며, 내가 예측할 $s\\_0$ 반경 안에 있는(Neighborhood) 몇 놈들만 쏙 빼와서 계산하는 방식을 씁니다. 동네 크기를 정할 때는 상관관계가 끊어지는 거리(Range)와 Nugget 비율을 꼼꼼히 따져봐야 합니다. (변동성이 심한 Nugget 환경일수록 동네를 더 넓게 잡아야 안정적입니다.)

**2. Empirical kriging**
진짜 신이 주신 Semivariogram을 아는 사람은 아무도 없습니다. 그래서 우리가 아까 WLS로 땀 뻘뻘 흘리며 구한 $\hat{\theta}$를 대입해서 만든 짝퉁 $\hat{\gamma} = \gamma(\hat{\theta})$와 $\hat{\Gamma} = \Gamma(\hat{\theta})$를 공식에 쑤셔 넣습니다. 이렇게 대입하는 순간 예측기가 더 이상 $Y$에 대한 예쁜 Linear 함수가 아니게 되어버리지만, 통계적 조건만 심하게 안 틀어지면 여전히 Unbiased 하다는 건 증명이 되어있으니 안심하고 써도 됩니다.

### Block Kriging

지금까지 한 건 바늘 끝처럼 뾰족한 단일 점 $s\\_0$을 핀셋으로 예측하는 **Point kriging**이었습니다. 하지만 현실에서는 "이 땅값 평균이 얼마야?", "이 동네 광물 매장량 평균이 얼마야?"처럼 특정 덩어리(블록) $B$ 전체의 평균값을 원할 때가 더 많습니다.

$$Y(B) \equiv \int_B Y(s)\,ds \Big/ |B|$$

이걸 구하는 걸 **Block kriging**이라고 부릅니다.

어려울 거 없이, 아까 point kriging 공식 구조는 그대로 살리되, 점과 점 사이를 구하던 행렬 항들을 '점과 블록', 혹은 '블록과 블록' 사이의 적분 평균값으로 싹 다 갈아 끼우면 끝입니다.

- $\gamma\\_i = \gamma(B, s\\_i) = \mid B\mid ^{-1} \int\\_B \gamma(u - s\\_i)\\, du$
- $x\\_{0,j} = X\\_j(B) = \mid B\mid ^{-1} \int\\_B X\\_j(u)\\, du$

흥미로운 사실은 블록 단위로 뭉쳐서 평균을 예측하다 보니 짜잘한 공간적 노이즈가 서로 상쇄되어 깎여나갑니다. 그래서 점 하나를 때려 맞추는 Point kriging보다 예측 분산(Kriging variance)이 훨씬 작고 안정적으로 떨어지는 기분 좋은 결과를 얻게 됩니다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 3.