---
title: "Bayesian UQ 심화: MCMC, Credible Interval, 그리고 LLM에서의 한계"
description: Gibbs sampling, MH algorithm, HPD interval을 정리하고 전통적 UQ 방법이 LLM에 적용되기 어려운 이유를 분석한 세미나 노트.
author: starrypark
date: 2025-08-26 00:00:00 +0810
categories: [Statistics, Uncertainty Quantification]
tags: [MCMC, gibbs sampling, metropolis-hastings, credible interval, LLM, bayesian]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

지난 세미나에서 MCMC의 개념을 소개했다. 이번 세미나에서는 MCMC의 두 가지 대표 알고리즘인 Gibbs sampling과 Metropolis-Hastings를 구체적으로 다루고, Bayesian UQ의 핵심 도구인 Credible Interval과 HPD Interval을 정리한다.

그리고 마지막에 한 가지 중요한 질문을 다룬다. 전통적인 UQ 방법들이 LLM과 Deep Learning에는 왜 그대로 적용하기 어려운가?

---

## 1. MCMC 복습

**Markov Chain Monte Carlo (MCMC)**는 복잡한 확률 분포에서 sample을 생성하는 일반적인 방법이다.

핵심 아이디어는 세 가지다.

- Target distribution을 stationary distribution으로 갖는 Markov chain을 설계한다.
- 충분한 iteration 후 생성된 sample들이 target distribution을 근사한다.
- 이 sample들로 평균, 분산, credible interval 등을 추정한다.

Bayesian statistics에서 posterior distribution이 analytically intractable할 때 가장 널리 쓰인다. Posterior 위의 적분을 Monte Carlo approximation으로 대체하는 셈이다.

---

## 2. Gibbs Sampling

**Gibbs sampling**은 full conditional distribution $p(\theta_j \mid \theta_{-j}, Y)$에서 순차적으로 sampling하는 방법이다. 여기서 $\theta_{-j}$는 $\theta_j$를 제외한 나머지 파라미터 벡터다.

### Algorithm

1. $\theta^{(0)} = (\theta_1^{(0)}, \ldots, \theta_p^{(0)})$을 초기화한다.
2. Iteration $t$에서 순차적으로 업데이트한다.

$$\theta_1^{(t+1)} \sim p\!\left(\theta_1 \mid \theta_2^{(t)}, \ldots, \theta_p^{(t)}, Y\right)$$

$$\theta_2^{(t+1)} \sim p\!\left(\theta_2 \mid \theta_1^{(t+1)}, \theta_3^{(t)}, \ldots, \theta_p^{(t)}, Y\right)$$

$$\vdots$$

$$\theta_p^{(t+1)} \sim p\!\left(\theta_p \mid \theta_1^{(t+1)}, \ldots, \theta_{p-1}^{(t+1)}, Y\right)$$

3. 수렴할 때까지 반복한다. 수렴 후 $\{\theta^{(t)}\}$는 $p(\theta_1, \ldots, \theta_p \mid Y)$에서의 draw를 근사한다.

### 특징

Gibbs sampling의 장점은 각 step에서 full conditional만 sampling하면 되기 때문에, 이 조건부 분포가 알려진 closed form이면 구현이 단순하다는 것이다. 예를 들어 conjugate prior를 쓰면 full conditional이 표준 분포 형태로 나온다. 대신 파라미터 간 상관이 강하면 수렴이 느릴 수 있다.

---

## 3. Metropolis-Hastings Algorithm

Full conditional이 알려진 형태가 아닐 때는 **Metropolis-Hastings (MH)** 알고리즘을 쓴다.

### Algorithm

1. $\theta^{(0)}$을 초기화한다.
2. Iteration $t$에서:
   1. Proposal distribution에서 candidate를 생성한다: $\theta' \sim q(\theta' \mid \theta^{(t)})$
   2. Acceptance probability를 계산한다.

$$\alpha = \min\!\left(1,\; \frac{\pi(\theta')\, q(\theta^{(t)} \mid \theta')}{\pi(\theta^{(t)})\, q(\theta' \mid \theta^{(t)})}\right)$$

   3. $\theta^{(t+1)}$을 결정한다.

$$\theta^{(t+1)} = \begin{cases} \theta' & \text{with probability } \alpha \\ \theta^{(t)} & \text{otherwise} \end{cases}$$

3. 수렴할 때까지 반복한다.

### 직관

Acceptance ratio의 분자는 "candidate $\theta'$가 target에서 얼마나 그럴듯한가"를, 분모는 "현재 $\theta^{(t)}$가 얼마나 그럴듯한가"를 나타낸다. Candidate가 더 그럴듯하면 항상 accept하고, 덜 그럴듯해도 일정 확률로 accept한다. 이 randomness 덕분에 chain이 한 곳에 갇히지 않고 parameter space 전체를 탐색할 수 있다.

Proposal distribution $q$의 선택이 수렴 속도에 큰 영향을 준다. 너무 넓으면 rejection rate이 높아지고, 너무 좁으면 탐색이 느려진다. 경험적으로 acceptance rate이 약 23~44% 사이가 되도록 proposal을 조정하는 것이 권장된다.

---

## 4. Credible Interval

### Frequentist와의 차이

Frequentist는 confidence interval을 "반복 표본 추출에서 참값을 포함하는 구간의 비율"로 정의한다. 실제로는 한 번만 구간을 계산하고, CLT에 기반한 이론으로 그 비율을 보장한다.

Bayesian은 prior 정보를 이용해 파라미터에 대한 "믿음"을 모델링한다. 이것이 **credible interval**이다.

### Equal-Tail Credible Interval

가장 흔한 선택은 equal-tail credible interval이다. Posterior quantile로 정의한다.

$$\Pr\!\left(\theta \in [q_{\alpha/2},\, q_{1-\alpha/2}] \mid \text{data}\right) = 1 - \alpha$$

여기서 $q_p$는 posterior distribution의 $p$-th quantile이다.

이 구간은 직접적인 확률 진술을 가능하게 한다. "데이터가 주어졌을 때 파라미터가 이 범위에 있을 확률이 $1-\alpha$다."

---

## 5. Highest Posterior Density (HPD) Interval

**HPD interval**은 credible interval의 또 다른 형태로, 구간 내 모든 점이 구간 밖의 어떤 점보다 높은 posterior density를 갖도록 정의한다.

Posterior density $p(\theta \mid y)$에서 level $1-\alpha$의 HPD region은 다음으로 정의된다.

$$\text{HPD}_{1-\alpha} = \{\theta : p(\theta \mid y) \geq c_{1-\alpha}\}$$

여기서 $c_{1-\alpha}$는 다음 조건을 만족하는 상수다.

$$\int_{\text{HPD}_{1-\alpha}} p(\theta \mid y)\, d\theta = 1 - \alpha$$

### Equal-Tail vs. HPD

HPD interval의 핵심 성질은 두 가지다. 구간 내부의 모든 점이 외부의 어떤 점보다 높은 posterior density를 갖는다. 그리고 동일한 $1-\alpha$에서 HPD interval이 가장 짧은 credible interval이다.

Posterior가 symmetric하고 unimodal이면 equal-tail interval과 HPD interval이 일치한다. 하지만 posterior가 skewed하거나 bimodal이면 둘이 달라질 수 있다. 그 경우 HPD interval이 더 정보적이다.

---

## 6. 왜 전통적 UQ를 LLM에 바로 쓸 수 없는가?

### Bootstrap의 한계

LLM과 Deep Learning에서 bootstrap은 두 가지 이유로 적용하기 어렵다.

첫째, 계산 비용이 극단적으로 크다. 데이터를 여러 번 resampling하고 매번 모델을 재학습하는 것은 LLM 규모에서 현실적이지 않다.

둘째, LLM의 text 데이터는 bootstrap이 가정하는 i.i.d. 조건을 위반할 가능성이 높다. 문장들 간에는 문맥, 주제, 스타일의 의존성이 있다.

대안으로 Deep Ensemble, Bayesian Dropout, Conformal Prediction 같은 방법들이 쓰인다.

### Delta Method의 한계

Delta method는 추정량 $\hat{\theta}$와 smooth function $g(\cdot)$에 대해 다음의 근사 confidence interval을 준다.

$$g(\hat{\theta}) \pm z_{\alpha/2} \sqrt{\nabla g(\theta)^\top \widehat{\text{Var}}(\hat{\theta})\, \nabla g(\theta)}$$

Deep model에서 이 방법이 비실용적인 이유는 두 가지다. 수백만 개의 파라미터를 가진 모델에서 $\widehat{\text{Var}}(\hat{\theta})$에 해당하는 공분산 행렬을 계산하고 저장하는 것이 계산적으로 불가능하다. 또한 discrete token decoding 같은 LLM의 많은 출력이 불연속이거나 미분 불가능해서, delta method의 미분가능성 가정을 위반한다.

### MCMC의 한계

MCMC는 posterior를 근사하는 강력한 방법이지만 LLM에는 여러 문제가 있다.

LLM의 파라미터 수가 방대하기 때문에 수렴 속도가 매우 느릴 가능성이 높다. 속도가 느리고 계산 비용이 높다는 점이 LLM에서 MCMC를 사용하기 어렵게 만드는 가장 큰 단점이다. 또한 MCMC는 sequential chain 구조 때문에 병렬 처리가 불가능하다. 현대 GPU/TPU 기반 학습에서 병렬화는 필수적인데, 이 부분에서 근본적인 한계가 있다.


