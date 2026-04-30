---
title: "[Ch.6] 공간 과정의 점근 이론"
description: Spatial process의 점근 이론, probability measure의 equivalence와 orthogonality, kriging의 점근적 최적성까지 정리한 노트입니다.
author: starrypark
date: 2024-05-02 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [asymptotics, kriging, gaussian process, infill asymptotics, equivalence]
pin: false
math: true
mermaid: true
---

## 개요

Chapter 4에서 increasing domain vs. infill asymptotics를 짧게 언급했는데, Chapter 6은 그 내용을 훨씬 깊게 파고들습니다.

핵심 질문은 이것입니다.

> "Covariance function을 잘못 지정해도 kriging이 잘 작동할 수 있을까?"

직관적으로는 "잘못된 모델 쓰면 결과도 나쁠 것 같다"고 생각하기 쉬운데, 놀랍게도 특정 조건 하에서는 **잘못된 covariance function을 써도 kriging predictor가 점근적으로 최적**임이 밝혀졌습니다.

이걸 이해하려면 probability measure의 equivalence 개념이 먼저 필요합니다.

---

## 6.1 점근적 접근법

### Asymptotic Framework 세 가지

Chapter 4에서 나왔던 내용이지만 여기서 명확하게 정의합니다.

**1. Increasing Domain Asymptotics**

데이터 위치 사이의 최소 거리가 0보다 bounded away from zero인 상태에서 관측 영역이 무한히 커지는 경우입니다.

**2. Infill (Fixed-Domain) Asymptotics**

관측 영역은 고정된 채로, 그 안에서 관측이 점점 더 촘촘해지는 경우입니다.

**3. Mixed Approach**

데이터 수가 증가하면서 관측이 $\mathbb{R}^d$ 전체에 걸쳐 dense해지는 경우입니다.

세 framework 중 infill asymptotics가 이 챕터의 주인공입니다. 실제 응용에서는 관측 영역을 고정하고 더 많은 데이터를 모으는 경우가 많거든.

---

## 6.2 Probability Measure의 관계

### 핵심 개념들

Infill asymptotics에서는 **두 probability measure가 얼마나 구별 가능한가**가 핵심 문제입니다. 이를 위한 개념을 정의합니다.

가측 공간 $(X, \mathcal{A})$ 위의 두 measure $\mu$, $\nu$에 대해:

- $\mathcal{N}\_\mu = \\{A \in \mathcal{A} \mid \mu(A) = 0\\}$: $\mu$-null set들의 집합
- $\mathcal{N}\_\nu = \\{A \in \mathcal{A} \mid \nu(A) = 0\\}$: $\nu$-null set들의 집합

**Definition (Absolute Continuity):** $\mathcal{N}\_\nu \supseteq \mathcal{N}\_\mu$이면 $\nu \ll \mu$ ($\nu$는 $\mu$에 대해 absolutely continuous).

**Definition (Equivalence):** $\mu \ll \nu$이고 $\nu \ll \mu$이면 $\mu \sim \nu$ (두 measure가 equivalent).

즉, $\mathcal{N}\_\mu = \mathcal{N}\_\nu$인 경우입니다.

**Definition (Orthogonality):** $E \cap F = \emptyset$, $E \cup F = X$인 $E, F \in \mathcal{M}$이 존재해서 $E$가 $\mu$-null이고 $F$가 $\nu$-null이면, $\mu$와 $\nu$는 **mutually singular (orthogonal)**입니다.

### 직관적 해석

두 probability measure $P\_0$, $P\_1$에 대해:

- **Equivalent**: 어떤 관측값을 보더라도 $P\_0$와 $P\_1$ 중 어느 게 맞는지 확실히 알 수 없습니다. 무한히 관측해도 두 모델을 완전히 구별할 수 없습니다.
- **Orthogonal**: 어떤 관측값을 보더라도 $P\_0$와 $P\_1$ 중 어느 게 맞는지 확실히 알 수 있습니다. 유한한 관측으로도 두 모델을 완벽히 구별할 수 있습니다.

---

## 6.3 Gaussian Process에서의 Equivalence

### Exponential Covariance 예시

$Y(t)$가 $[0,1]$ 위의 zero-mean Gaussian process이고 autocovariance function이 $K(t) = \theta e^{-\phi\mid t\mid }$라고 해집니다. $(θ, \phi) = (\theta\_j, \phi\_j)$인 경우의 process law를 $P\_j$라 하면:

$$P_0 \sim P_1 \iff \theta_0\phi_0 = \theta_1\phi_1$$

$$P_0 \perp P_1 \iff \theta_0\phi_0 \neq \theta_1\phi_1$$

놀라운 결과입니다. $K\_0(t) = e^{-\mid t\mid }$이고 $K\_1(t) = 2e^{-\mid t\mid /2}$이면, 두 모델은 완전히 달라 보이지만 $P\_0$와 $P\_1$은 **equivalent**해집니다. $[0,1]$ 위의 모든 관측값을 봐도 어느 모델이 맞는지 알 수 없습니다.

반면, $K\_0(t) = e^{-\mid t\mid }$이고 $K\_1(t) = e^{-\mid t\mid /2}$이면 $P\_0 \perp P\_1$입니다. 이 경우엔 확률 1로 어느 모델이 맞는지 판별할 수 있습니다.

### Spectral Density로 보는 Equivalence

Autocovariance function에서 equivalence 조건을 보는 것보다 **spectral density의 고주파 거동**을 보는 게 더 직접적입니다.

$K(t) = \theta e^{-\phi\mid t\mid }$의 spectral density는:

$$f(\omega) = \pi^{-1}\phi\theta / (\phi^2 + \omega^2)$$

$\omega \to \infty$일 때 $f(\omega) = \pi^{-1}\phi\theta\omega^{-2} + O(\omega^{-4})$입니다. $P\_0$와 $P\_1$이 equivalent한 것은 두 spectral density가 고주파에서 같은 점근 거동을 가지는 것과 정확히 대응합니다.

왜 고주파냐면, infill asymptotics에서 관측이 점점 촘촘해질수록 정보가 고주파 영역에 집중되기 때문입니다. Bounded domain에서 어떤 유한한 주파수 구간의 spectral behavior는 equivalence에 영향을 주지 않습니다.

### Equivalence의 일반 조건

Stationary Gaussian process의 spectral density $f\_0$가 다음을 만족한다고 해집니다.

$$f_0(\omega)|\omega|^\alpha \text{ 가 } |\omega| \to \infty \text{ 에서 0과 } \infty \text{ 에서 bounded away} \tag{6.1}$$

이 조건 하에서, 만약

$$\int_{|\omega|>C} \left(\frac{f_1(\omega) - f_0(\omega)}{f_0(\omega)}\right)^2 d\omega < \infty \tag{6.2}$$

이면 bounded domain $D$ 위에서 $G\_D(0, K\_0) \sim G\_D(0, K\_1)$입니다.

1차원에서는 (6.1)이 성립할 때 (6.2)는 equivalence의 거의 필요조건이기도 해집니다.

---

## 6.4 Kriging의 점근적 최적성

### 문제 제기

Yakowitz and Szidarovszky (1985)는 kriging에 근본적인 도전을 제기했습니다.

> Bounded domain에서는 variogram을 consistent하게 추정하는 게 일반적으로 불가능합니다. 그렇다면 추정된 variogram에 기반한 kriging predictor가 얼마나 좋을 수 있을까?

즉, covariance function을 잘못 써도 괜찮냐는 것입니다.

### Stein (1988)의 답변

결론은 "equivalent한 covariance function을 쓰면 점근적으로 문제없다"입니다.

**설정:**

- $Y(s\_0)$를 예측하고 싶습니다.
- 관측 위치 $s\_1, s\_2, \ldots$의 sequence가 있습니다.
- 올바른 covariance: $K\_0$, 잘못 지정된 covariance: $K\_1$
- $K\_j$ 하에서의 BLUP(= universal kriging predictor)의 오차: $e\_j(s\_0, n)$
- $K\_1$ 하에서의 BLUP을 **pseudo-BLUP**이라고 부릅니다.
- $E\_j$는 $K\_j$ 하에서의 기댓값입니다.

주목해야 할 두 비율입니다.

- $E\_0 e\_1(s\_0, n)^2 / E\_0 e\_0(s\_0, n)^2$: pseudo-BLUP의 실제 MSE vs. 최적 BLUP의 MSE
- $E\_0 e\_1(s\_0, n)^2 / E\_1 e\_1(s\_0, n)^2$: pseudo-BLUP의 실제 MSE vs. 잘못된 모델이 생각하는 MSE

**Theorem 6.2.1:** $E\_0 e\_0(s\_0, n)^2 \to 0$이고 $G\_D(0, K\_0) \sim G\_D(0, K\_1)$이면:

$$\lim_{n\to\infty} \frac{E_0 e_1(s_0, n)^2}{E_0 e_0(s_0, n)^2} = 1 \tag{6.3}$$

$$\lim_{n\to\infty} \frac{E_0 e_1(s_0, n)^2}{E_1 e_1(s_0, n)^2} = 1 \tag{6.4}$$

(6.3)은 잘못된 $K\_1$을 써도 예측 오차가 점근적으로 최적임을 말합니다.

(6.4)는 잘못된 $K\_1$이 스스로 추정하는 MSE가 실제 MSE와 점근적으로 같다는 것입니다. 즉, prediction interval도 점근적으로 올바르다는 뜻입니다.

$E\_0 e\_0(s\_0, n)^2 \to 0$ 조건은 $Y$가 mean square continuous하고 $s\_0$가 $s\_1, s\_2, \ldots$의 limit point이면 만족됩니다.

### 더 약한 조건

Equivalent Gaussian measure 조건보다 훨씬 약한 조건으로도 정리가 성립합니다. $f\_0$가 (6.1)을 만족하면, 다음 조건으로 충분합니다.

$$f_1(\omega)/f_0(\omega) \to 1 \quad \text{as } |\omega| \to \infty$$

이 조건은 (6.2)보다 약하고, 심지어 두 Gaussian measure가 equivalent하지 않아도 성립할 수 있습니다. Spectral density의 고주파 꼬리만 맞으면 kriging이 asymptotically optimal하다는 것입니다.

### 수치 예시

단위 정사각형 $D = [0,1]^2$에서 25개 랜덤 위치를 관측하고, $s\_0 = (0,0)$과 $s\_0 = (0.5, 0.5)$를 예측합니다.

- 올바른 covariance: $K\_0(s) = e^{-\mid s\mid }$
- 잘못된 covariance: $K\_1(s) = 2e^{-\mid s\mid /2}$ (equivalent임을 확인 가능)

| | Error Var = 0 (Corner) | Error Var = 0 (Center) | Error Var = 0.4 (Corner) | Error Var = 0.4 (Center) |
|---|---|---|---|---|
| $E\_0 e\_0^2$ | 0.2452 | 0.1689 | 0.3920 | 0.2340 |
| $E\_0 e\_1^2 / E\_0 e\_0^2$ | 1.0046 | 1.0001 | 1.0083 | 1.0003 |
| $E\_0 e\_1^2 / E\_1 e\_1^2$ | 0.9623 | 0.9931 | 0.9383 | 0.9861 |

비율이 모두 1에 가까워다. 예측 위치가 domain 중앙에 가까울수록 비율이 1에 더 가까운 것도 보입니다. 경계 근처에서는 점근 결과가 덜 정확합니다.

---

## 6.5 점근 결과의 한계

이론적으로는 좋아 보이지만, 실제로 spatial covariance 추정의 점근 이론에는 한계가 많습니다.

**1. 격자 데이터 vs. 불규칙 데이터**

Guyon (1995) 등이 격자 데이터의 increasing-domain 결과를 많이 정리했지만, kriging에 쓰려면 불규칙 위치의 결과가 필요합니다. 그런데 fixed-domain asymptotics에서 불규칙 관측에 대한 meaningful한 결과는 거의 없습니다.

**2. Consistent 추정의 한계**

Fixed-domain asymptotics에서는 일부 파라미터를 consistent하게 추정할 수 없습니다. Exponential covariance $K(t) = \theta e^{-\phi\mid t\mid }$에서 $\theta$와 $\phi$를 각각 추정하는 건 불가능하지만, 그 곱 $\theta\phi$는 consistent하게 추정 가능할 수 있습니다.

이 부분이 처음에 이해하기 어려울 수 있습니다. 데이터가 아무리 많아도 두 파라미터를 분리해서 추정하는 건 불가능하다는 것입니다. Equivalent measures 사이의 구별이 원천적으로 불가능하기 때문입니다.

**3. ML/REML의 이론적 어려움**

ML과 REML 추정량은 explicit form이 없습니다. Fixed-domain asymptotics에서 표준 점근 이론이 적용되지 않아서 분석이 특히 어렵습니다.

**4. 실용적 결론**

Stein은 올바르게 지정된 parametric covariance model과 REML 같은 좋은 추정 방법을 쓰면 Theorem 6.2.1이 실제로 성립할 것으로 예상합니다. 하지만 이는 강한 model assumption에 의존하고 있습니다.

결국 **spatial prediction이 목표라면 covariance function 선택과 fitting에 여전히 많은 주의가 필요**하다. 점근 이론이 "어떤 covariance든 써도 된다"고 말하는 게 아니에다. Equivalent한 범위 내에서 고주파 거동을 맞춰야 한다는 구체적인 조건이 있습니다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 6.
