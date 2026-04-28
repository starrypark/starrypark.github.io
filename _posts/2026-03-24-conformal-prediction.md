---
title: "Conformal Prediction: 유한 샘플에서 보장되는 예측 구간"
description: Conformal prediction의 핵심 아이디어와 수학적 유도를 처음부터 차근차근 정리한 노트.
author: starrypark
date: 2026-03-24 00:00:00 +0810
categories: [Statistics, Conformal Prediction]
tags: [conformal prediction, prediction interval, quantile regression, split conformal, coverage]
pin: true
math: true
mermaid: true
---

## 개요

예측 구간(prediction interval)을 만드는 건 쉬워 보이지만, 사실 보장이 있는 예측 구간을 만드는 건 생각보다 어렵습니다.

선형 회귀 같은 모델은 정규성 가정을 깔고 들어가야 합니다. 딥러닝이나 복잡한 ML 모델은 그런 가정조차 없죠.

**Conformal prediction**은 이 문제를 완전히 다른 방식으로 풉니다. 모델 구조에 아무 가정도 안 합니다. 그냥 데이터가 **교환 가능(exchangeable)**하다는 조건 하나만 씁니다. 그 결과 유한 샘플에서도 커버리지를 보장합니다.

PhD 과정에서 이 방법론을 공부하면서 느낀 건, 아이디어 자체는 정말 우아한데 처음 보면 "이게 왜 되는 거지?"라는 의문이 자연스럽게 든다는 겁니다. 이 포스트는 그 흐름을 최대한 명확하게 짚어보려 합니다.

---

## Conformal Prediction의 목표

**설정부터 잡겠습니다.**

$(X_1, Y_1), \ldots, (X_n, Y_n)$은 어떤 분포에서 i.i.d.로 뽑힌 샘플입니다. 여기서 $X_i \in \mathcal{X} \subset \mathbb{R}^p$, $Y_i \in \mathcal{Y} \subset \mathbb{R}$입니다.

우리의 목표는 새로운 입력 $X_{n+1}$이 주어졌을 때, $Y_{n+1}$을 담는 집합 $\hat{C}_n(X_{n+1})$을 만드는 것입니다. 단, 다음 조건을 만족해야 합니다:

$$P\left(Y_{n+1} \in \hat{C}_n(X_{n+1})\right) \geq 1 - \alpha$$

그리고 가능하면 $\hat{C}_n(x)$의 크기가 $x$의 난이도에 따라 달라지면 좋겠죠. 예측하기 쉬운 $x$에서는 좁게, 어려운 $x$에서는 넓게.

---

## 가장 단순한 시도: 왜 그냥 분위수 쓰면 안 될까?

단순하게 생각하면, 그냥 $Y_1, \ldots, Y_n$의 $(1-\alpha)$th 분위수를 쓰면 어떨까요?

$$\hat{q}_n = \text{Quantile}\!\left(1 - \alpha,\; \frac{1}{n}\sum_{i=1}^n \delta_{Y_i}\right)$$

그래서 $\hat{C}_n(X_{n+1}) = (-\infty, \hat{q}_n]$로 정의하면?

**문제가 있습니다.** 이 방법으로는 $P(Y_{n+1} \leq \hat{q}_n) \approx 1 - \alpha$가 $n \to \infty$일 때나 성립합니다. 유한 샘플에서는 보장이 없어요.

그렇다면 어떻게 해야 할까요?

---

## 핵심 아이디어: 순열 대칭성(Exchangeability)

여기가 conformal prediction의 핵심입니다.

$Y_{n+1}$도 같은 분포에서 뽑혔으니, $Y_1, \ldots, Y_n, Y_{n+1}$ 총 $n+1$개의 값에서 어떤 순열이든 똑같이 가능합니다.

**결론:** $Y_{n+1}$은 정렬된 값 $Y_{(1)} \leq \cdots \leq Y_{(n+1)}$ 중 어느 위치에도 $\frac{1}{n+1}$의 확률로 놓입니다.

따라서:

$$P\left(Y_{n+1} \leq Y_{\left(\left\lceil(1+n)(1-\alpha)\right\rceil\right)} \text{ among } Y_1, \ldots, Y_n, Y_{n+1}\right) \geq 1 - \alpha$$

### 왜 이게 성립하나요?

$n+1$개의 값이 동등한 확률로 각 순서 통계량 위치를 차지합니다. $Y_{n+1}$이 상위 $\alpha$ 비율에 들어갈 확률은 최대 $\alpha$이므로, 하위 $1-\alpha$ 비율에 들어갈 확률은 최소 $1-\alpha$입니다.

---

## $\hat{q}_n$ 구체적으로 구하기

### 이벤트 동치 관계 확인

다음 두 이벤트가 동치임을 보입니다.

$$A = \left\{Y_{n+1} \leq Y_{\left(\left\lceil(1+n)(1-\alpha)\right\rceil\right)} \text{ among } Y_1,\ldots,Y_n,Y_{n+1}\right\}$$

$$B = \left\{Y_{n+1} \leq Y_{\left(\left\lceil(1+n)(1-\alpha)\right\rceil\right)} \text{ among } Y_1,\ldots,Y_n\right\}$$

**증명:**

$A^c$: $Y_{n+1}$이 $Y_1,\ldots,Y_n,Y_{n+1}$ 중 $\lceil(1+n)(1-\alpha)\rceil$번째 값보다 크다.

그런데 $Y_{n+1}$이 자기 자신보다 엄격히 클 수는 없습니다. 따라서 $Y_{n+1}$을 제외한 $Y_1,\ldots,Y_n$ 중의 $\lceil(1+n)(1-\alpha)\rceil$번째 값보다 크다는 것과 같습니다. 즉:

$$A^c \iff Y_{n+1} > Y_{\left(\left\lceil(1+n)(1-\alpha)\right\rceil\right)} \text{ among } Y_1,\ldots,Y_n$$

$$\iff B^c$$

따라서 $A \iff B$.

### $\hat{q}_n$ 정의

따라서 다음처럼 $\hat{q}_n$을 정의하면 됩니다:

$$\hat{q}_n = \left\lceil(n+1)(1-\alpha)\right\rceil\text{-th smallest value of } Y_1, \ldots, Y_n$$

동치 표현으로는:

$$\hat{q}_n = \text{Quantile}\!\left(\frac{(n+1)(1-\alpha)}{n},\; \frac{1}{n}\sum_{i=1}^n \delta_{Y_i}\right)$$

그러면 최종적으로:

$$P\left(Y_{n+1} \in \hat{C}_n(X_{n+1})\right) = P(Y_{n+1} \leq \hat{q}_n) \geq 1 - \alpha$$

여기서 $\hat{C}_n(X_{n+1}) = (-\infty, \hat{q}_n]$입니다.

**포인트:** 분위수 레벨을 $1-\alpha$가 아니라 $\frac{(n+1)(1-\alpha)}{n}$으로 올린 것이 핵심입니다. 이 작은 조정 하나가 유한 샘플 보장을 만들어냅니다.

---

## Tie가 없을 때 커버리지 범위 (상한도 있다!)

Tie가 거의 확실하게(almost surely) 없는 경우, 커버리지가 더 정밀하게 characterize됩니다:

$$P(Y_{n+1} \leq \hat{q}_n) \in \left[1-\alpha,\; 1-\alpha+\frac{1}{n+1}\right]$$

하한은 항상 보장됩니다. 상한은 tie가 없을 때만 성립합니다. 즉 과도하게 보수적이지 않다는 보장도 있는 셈입니다.

---

## 회귀에 적용하기: Residual Conformal Prediction

지금까지는 $X_i$를 전혀 안 썼습니다. 이제 써봅시다.

$\hat{f}_n(x)$를 $(X_1,Y_1),\ldots,(X_n,Y_n)$으로 훈련한 점 예측 모델이라 하겠습니다.

잔차(absolute residual)를 정의합니다:

$$R_i = |Y_i - \hat{f}_n(X_i)|, \quad i = 1, \ldots, n$$

그리고 $\hat{q}_n$을 $R_1,\ldots,R_n$의 $\lceil(1-\alpha)(n+1)\rceil$번째 작은 값으로 놓으면, 예측 구간은:

$$\hat{C}_n(x) = \bigl[\hat{f}_n(x) - \hat{q}_n,\; \hat{f}_n(x) + \hat{q}_n\bigr]$$

### 그런데 이게 정말 작동할까요?

$Y_{n+1} \in \hat{C}_n(X_{n+1})$이 성립하려면 $R_{n+1} \leq \hat{q}_n$이어야 하고, 이는 $R_{n+1}$이 $R_1,\ldots,R_n$ 중 $\lceil(1-\alpha)(n+1)\rceil$번째 이하여야 한다는 뜻입니다.

이 논리가 성립하려면 $R_1,\ldots,R_{n+1}$이 **교환 가능(exchangeable)**해야 합니다.

**그런데 실제로는 그렇지 않습니다.** 이 부분이 처음엔 헷갈릴 수 있어요.

---

## 왜 Residual이 Exchangeable하지 않은가?

$R_1,\ldots,R_n$은 $\hat{f}_n(x)$를 학습할 때 이미 사용된 데이터로 계산됩니다. 반면 $R_{n+1}$은 훈련에 전혀 참여하지 않은 새 관측치입니다.

따라서:

- $\mathbb{E}[R_i]$ for $i = 1,\ldots,n$: **작은 편** (훈련 데이터니까 모델이 이미 이 값들에 맞춰져 있음)
- $\mathbb{E}[R_{n+1}]$: **큰 편** (새 데이터이므로 모델이 과적합된 만큼 잔차가 더 큼)

$R_{n+1}$이 체계적으로 더 크기 때문에, 이걸 $R_1,\ldots,R_n$과 함께 교환 가능하다고 보면 안 됩니다.

---

## 해결책: Split Conformal Prediction

**아이디어:** 훈련 데이터를 두 부분으로 나눕니다.

- **Proper training set** $\mathcal{D}_1$ (크기 $n_1$): 모델 $\hat{f}_{n_1}$을 훈련
- **Calibration set** $\mathcal{D}_2$ (크기 $n_2$): 잔차 계산 및 분위수 보정

### 절차

**Step 1.** $\mathcal{D}_1$으로 $\hat{f}_{n_1}$을 학습합니다.

**Step 2.** $\mathcal{D}_2$에 대해 잔차를 계산합니다:

$$R_i = |Y_i - \hat{f}_{n_1}(X_i)|, \quad i \in \mathcal{D}_2$$

**Step 3.** Conformal quantile을 구합니다:

$$\hat{q}_{n_2} = \left\lceil(1-\alpha)(n_2+1)\right\rceil\text{-th smallest of } R_i,\; i \in \mathcal{D}_2$$

**Step 4.** 예측 구간을 구성합니다:

$$\hat{C}_n(x) = \bigl[\hat{f}_{n_1}(x) - \hat{q}_{n_2},\; \hat{f}_{n_1}(x) + \hat{q}_{n_2}\bigr]$$

### 왜 이게 작동하나요?

이제 $R_i$ ($i \in \mathcal{D}_2$)와 $R_{n+1}$은 **모두 $\mathcal{D}_1$으로만 학습된 $\hat{f}_{n_1}$에 기반**합니다. 어느 쪽도 상대방의 데이터로 훈련하지 않았습니다.

$\mathcal{D}_1$이 고정되었다는 조건 하에, $R_i$ ($i \in \mathcal{D}_2$)와 $R_{n+1}$은 교환 가능합니다.

따라서:

$$P\left(Y_{n+1} \in \hat{C}_n(X_{n+1}) \mid (X_i,Y_i),\; i \in \mathcal{D}_1\right) \in \left[1-\alpha,\; 1-\alpha+\frac{1}{n_2+1}\right]$$

---

## 일반화: Score 함수

잔차에만 국한하지 않겠습니다. 임의의 **점수 함수(conformity score)** $V(x,y)$를 씁니다.

- **Negatively-oriented score**: $R_i$가 작을수록 예측이 좋음. 예: 잔차

$$\hat{C}_n(x) = \left\{y : V(x,y) \leq \left\lceil(1-\alpha)(n+1)\right\rceil\text{-th smallest } R_i\right\}$$

- **Positively-oriented score**: $R_i$가 클수록 예측이 좋음. 예: 분류 확률

$$\hat{C}_n(x) = \left\{y : V(x,y) \geq \left\lceil\alpha(n+1)\right\rceil\text{-th smallest } R_i\right\}$$

---

## 지역 적응성 문제 (Local Adaptability)

Split conformal의 한계: 예측 구간의 너비가 항상 $2\hat{q}_{n_2}$로 **고정**됩니다. $x$가 어떤 위치든 상관없이요.

직관적으로 이상합니다. 데이터가 드문 영역과 밀집한 영역에서 불확실성이 같을 리 없으니까요.

### 해결책 1: Studentized Residuals

$\mathcal{D}_1$에서 점 예측기 $\hat{f}_{n_1}(x)$ 외에 **분산 예측기** $\hat{\sigma}_{n_1}(x)$도 함께 학습합니다.

보정 집합에서의 점수는:

$$R_i = \frac{|Y_i - \hat{f}_{n_1}(X_i)|}{\hat{\sigma}_{n_1}(X_i)}, \quad i \in \mathcal{D}_2$$

Conformal quantile $\hat{q}_{n_2}$를 기존과 같이 구한 뒤, 예측 구간을:

$$\hat{C}_n(x) = \bigl[\hat{f}_{n_1}(x) - \hat{\sigma}_{n_1}(x)\hat{q}_{n_2},\; \hat{f}_{n_1}(x) + \hat{\sigma}_{n_1}(x)\hat{q}_{n_2}\bigr]$$

로 정의합니다. 이제 폭이 $x$마다 달라집니다.

**단점:**
- 예측 분포가 대칭이라고 암묵적으로 가정합니다.
- $\hat{\sigma}_{n_1}(x)$ 추정에 query point $x$ 근방의 충분한 데이터가 필요합니다.

### 해결책 2: Quantile Regression

**Check(tilted) loss function**을 사용합니다:

$$\rho_\tau(u) = u \cdot \left(\tau - \mathbf{1}_{u < 0}\right) = \begin{cases} \tau |u| & u \geq 0 \\ (1-\tau)|u| & u < 0 \end{cases}$$

$\tau = 0.5$이면 MAE와 같고, $\tau \neq 0.5$이면 비대칭적으로 오차를 페널티합니다.

코딩할 때는 다음과 같이 쓸 수 있습니다:

$$\rho_\tau(u) = \max\bigl(\tau u,\; (\tau - 1) u\bigr)$$

이 손실을 최소화하면 $\mathbb{E}[Y_i | x_i]$가 아니라 **$Y_i$의 $\tau$번째 분위수**를 예측합니다.

**95% 예측 구간을 만들려면:**

- $\tau = 0.025$로 학습한 $\hat{f}_{n_1}^{0.025}$: 하한 예측기
- $\tau = 0.975$로 학습한 $\hat{f}_{n_1}^{0.975}$: 상한 예측기

그러면 $[\hat{f}_{n_1}^{0.025}(x),\; \hat{f}_{n_1}^{0.975}(x)]$가 예측 구간이 됩니다.

**문제:** 모델이 유한 샘플에서는 완벽하지 않고, 정확한 분위수 커버리지가 보장되지 않습니다.

---

## Conformalized Quantile Regression (CQR)

분위수 회귀의 지역 적응성 + Conformal prediction의 보장 = **CQR**입니다.

### 절차

**Step 1.** $\mathcal{D}_1$으로 두 분위수 모델을 학습합니다:
- $\hat{f}_{n_1}^{\alpha/2}$: $\alpha/2$ 분위수 예측기
- $\hat{f}_{n_1}^{1-\alpha/2}$: $1-\alpha/2$ 분위수 예측기

**Step 2.** 보정 집합에서 점수를 계산합니다:

$$R_i = \max\!\left\{\hat{f}_{n_1}^{\alpha/2}(X_i) - Y_i,\; Y_i - \hat{f}_{n_1}^{1-\alpha/2}(X_i)\right\}, \quad i \in \mathcal{D}_2$$

이 점수는 $Y_i$가 예측 구간 **밖에 있을수록** 큽니다. 음수면 구간 안에 있다는 뜻이죠.

**Step 3.** $\hat{q}_{n_2} = \lceil(1-\alpha)(n_2+1)\rceil$번째 작은 값 of $R_i$

**Step 4.** CQR 예측 구간:

$$\hat{C}_n(x) = \bigl[\hat{f}_{n_1}^{\alpha/2}(x) - \hat{q}_{n_2},\; \hat{f}_{n_1}^{1-\alpha/2}(x) + \hat{q}_{n_2}\bigr]$$

분위수 모델이 이미 $x$에 따라 가변적인 구간을 만들고, $\hat{q}_{n_2}$로 일괄 조정해서 커버리지를 보정합니다. 지역 적응성 + 보장.

---

## 분류 문제로 확장: Conformal Classification

분류 문제에서는 $Y \in \{1, \ldots, K\}$입니다. 분류기 $\hat{f}_{n_1}(x; k)$는 $P(Y = k | x)$를 추정합니다.

### 단순 접근 (Naive)

보정 집합의 점수:

$$R_i = \hat{f}_{n_1}(X_i;\; Y_i), \quad i \in \mathcal{D}_2$$

"정답 클래스일 확률" — 클수록 좋은 **positively-oriented score**입니다.

$$\hat{q}_{n_2} = \left\lceil\alpha(n+1)\right\rceil\text{-th smallest of } R_i$$

$$\hat{C}_n(x) = \left\{k : \hat{f}_{n_1}(x;\;k) \geq \hat{q}_{n_2}\right\}$$

### 더 나은 접근: 누적 확률 (RAPS)

각 관측치 $i$에 대해, 클래스들을 예측 확률 내림차순으로 정렬합니다:

$$\hat{f}_{n_1}(X_i;\;\pi_i(1)) \geq \hat{f}_{n_1}(X_i;\;\pi_i(2)) \geq \cdots \geq \hat{f}_{n_1}(X_i;\;\pi_i(K))$$

점수를 정답 클래스까지의 **누적 확률**로 정의합니다:

$$R_i = \sum_{j=1}^{k_i} \hat{f}_{n_1}(X_i,\;\pi_i(j)), \quad \text{where } \pi_i(k_i) = Y_i$$

이 점수는 정답 클래스가 확률 상위에 있을수록 작고, 하위에 있을수록 큽니다. **Negatively-oriented score**입니다.

예측 집합:

$$\hat{C}_n(x) = \left\{\pi_x(1),\ldots,\pi_x(k_x)\right\}$$

여기서 $k_x = \sup\left\{k : \sum_{j=1}^k \hat{f}_{n_1}(x, \pi_i(j)) < \hat{q}_{n_2} + 1\right\}$

### 정규화 버전 (RAPS 변형)

집합 크기를 줄이기 위해 페널티를 추가합니다:

$$R_i = \sum_{j=1}^{k_i} \hat{f}_{n_1}(X_i, \pi_i(j)) + \lambda (k_i - k_{\text{reg}})^+$$

$k_i > k_{\text{reg}}$일 때 추가 비용을 부과합니다. 튜닝 파라미터 $\lambda > 0$와 $k_{\text{reg}} \in \{2,\ldots,K\}$를 씁니다.

예측 집합:

$$k_x = \sup\!\left\{k : \sum_{j=1}^k \hat{f}_{n_1}(x, \pi_i(j)) + \lambda(k - k_{\text{reg}})^+ < \hat{q}_{n_2} + 1\right\}$$

---

## Bias-Variance Trade-off: 데이터 분할의 딜레마

Split conformal의 현실적인 문제:

- **$\mathcal{D}_1$이 작으면**: 모델 $\hat{f}_{n_1}$의 품질이 낮아짐 → **bias 증가**
- **$\mathcal{D}_2$가 작으면**: $\hat{q}_{n_2}$ 추정이 불안정해짐 → **variance 증가**

### Cross-Conformal (Vovk, 2015)

데이터를 $G$개의 폴드로 나눕니다. $i$번째 관측치가 $g(i)$번째 폴드에 있을 때, $\hat{f}_{n - g(i)}$는 $\mathcal{D}_{g(i)}$를 **제외하고** 학습한 모델입니다.

각 관측치의 점수:

$$R_i = V\!\left(X_i, Y_i;\; \hat{f}_{n - g(i)}\right), \quad i \in \mathcal{D}_{g(i)}$$

전체 conformal quantile:

$$\hat{q}_n = \left\lceil(1-\alpha)(n+1)\right\rceil\text{-th smallest of } R_i,\; i = 1,\ldots,n$$

K-fold cross-validation 구조로 데이터를 더 효율적으로 씁니다.

---

## Full Conformal Prediction

Split 없이 하려면? $(x, y)$를 데이터셋에 포함해서 처음부터 모델을 다시 학습합니다.

새로운 query $(x, y)$에 대해:
1. $(X_1,Y_1),\ldots,(X_n,Y_n),(x,y)$로 $\hat{f}_{n,x,y}$를 학습
2. 각 $i$에 대해 $R_i^{(x,y)} = V(X_i, Y_i)$와 $R_{n+1}^{(x,y)} = V(x, y)$ 계산
3. Negatively-oriented의 경우:

$$\hat{C}_n(x) = \left\{y : V(x,y) \leq \left\lceil(1-\alpha)(n+1)\right\rceil\text{-th smallest } R_i^{(x,y)}\right\}$$

**문제:** 연속형 $Y$에서는 $(x, y)$ 쌍마다 모델을 다시 학습해야 하므로 **계산이 불가능에 가깝습니다.**

---

## Conformal Bayes (Fong & Holmes, 2021)

Full conformal의 계산 문제를 Bayesian importance sampling으로 우회합니다.

### Bayesian Conformity Score

Bayesian 예측 밀도를 점수로 씁니다:

$$p(Y \mid X, \mathcal{Z}_{n+1}) = \int f_\theta(Y \mid X)\; \pi(\theta \mid \mathcal{Z}_{n+1})\; d\theta$$

여기서 $\mathcal{Z}_{n+1} = \{Z_1, \ldots, Z_{n+1}\}$이고, $\pi(\theta \mid \mathcal{Z}_{n+1}) \propto \pi(\theta) \prod_{i=1}^{n+1} f_\theta(Y_i \mid X_i)$

이 밀도는 $\mathcal{Z}_{n+1}$의 순열에 **불변**합니다. 따라서 $R_i = p(Y_i \mid X_i, \mathcal{Z}_{n+1})$도 교환 가능합니다.

### Importance Sampling으로 근사

$\pi(\theta \mid \mathcal{Z}_n)$에서 얻은 MCMC 샘플 $\theta^{(1)},\ldots,\theta^{(T)}$가 있다고 합시다.

$\mathcal{Z}_{n+1}$에 대한 사후 분포로 업데이트하려면:

$$w(\theta) = \frac{\pi(\theta \mid \mathcal{Z}_{n+1})}{\pi(\theta \mid \mathcal{Z}_n)} = \frac{\pi(\theta)\prod_{i=1}^{n+1} f_\theta(Y_i \mid X_i)}{\pi(\theta)\prod_{i=1}^n f_\theta(Y_i \mid X_i)} = f_\theta(Y_{n+1} \mid X_{n+1})$$

중요도 가중치가 바로 **새 관측치의 likelihood**입니다.

### Add-One-In (AOI) 근사 밀도

$$\hat{p}(Y \mid X, \mathcal{Z}_{n+1}) = \frac{1}{T}\sum_{t=1}^T \tilde{w}^{(t)} f_{\theta^{(t)}}(Y \mid X)$$

여기서 $\tilde{w}^{(t)} = \frac{w^{(t)}}{\sum_{t'} w^{(t')}}$, $w^{(t)} = f_{\theta^{(t)}}(Y_{n+1} \mid X_{n+1})$

이제 $(x, y)$마다 모델을 재학습할 필요 없이 가중치만 재계산하면 됩니다.

### 계산 비용

- $Tn$번: $f_{\theta^{(t)}}(Y_i \mid X_i)$ 평가 ($t = 1,\ldots,T$, $i = 1,\ldots,n$)
- $Tm$번: $w^{(t)} = f_{\theta^{(t)}}(Y_{n+1} \mid X_{n+1})$ 평가 (grid $y_1,\ldots,y_m$에 대해)
- 각 grid point에서 가중 평균 계산: $O(T)$

Full conformal보다 훨씬 현실적입니다.

---

### 📝 Summary (English)

- Conformal prediction gives **finite-sample coverage guarantees** by exploiting exchangeability, not model assumptions.
- The key fix over naive quantile estimation: use the $\lceil(n+1)(1-\alpha)\rceil$-th smallest value, not the $(1-\alpha)$-th quantile. This small adjustment is everything.
- Split conformal fixes the non-exchangeability of residuals by separating training from calibration. This introduces a bias-variance trade-off on data splitting.
- Local adaptability requires either studentized residuals or quantile regression. CQR is the cleanest solution: quantile regression for shape + conformal calibration for coverage.
- Conformal classification extends the same logic with cumulative probability scores, with optional regularization to shrink prediction set size.
- Full conformal avoids data splitting but is computationally prohibitive for continuous responses.
- Conformal Bayes offers a practical importance-sampling trick to approximate full conformal without refitting the model.

---

### 🇰🇷 한국어 요약

- Conformal prediction은 모델에 아무 가정 없이, **교환 가능성(exchangeability)** 하나만으로 유한 샘플 커버리지를 보장합니다.
- 핵심 트릭: 분위수 레벨을 $1-\alpha$에서 $\frac{(n+1)(1-\alpha)}{n}$으로 살짝 올리는 것. 이 조정 하나가 유한 샘플 보장을 만들어냅니다.
- 잔차는 훈련 데이터와 새 데이터 사이에 교환 가능하지 않아요. 그래서 Split conformal이 필요합니다. 데이터를 나눠서 훈련과 보정을 분리하면 됩니다.
- Split conformal의 단점은 예측 구간 폭이 $x$와 무관하게 고정된다는 거예요. CQR이 이걸 해결합니다. 분위수 회귀로 지역 적응성을 확보하고, conformal로 커버리지를 보정합니다.
- 분류 문제에서는 누적 확률을 점수로 써서 똑같은 구조를 적용합니다. 집합 크기를 줄이려면 페널티 항을 추가할 수 있습니다.
- Full conformal은 데이터 분할이 필요 없지만 연속형 반응변수에서는 현실적으로 쓰기 어렵습니다.
- Conformal Bayes는 importance sampling으로 full conformal을 근사합니다. 모델 재학습 없이 가중치만 재계산하면 돼서 훨씬 실용적입니다.
