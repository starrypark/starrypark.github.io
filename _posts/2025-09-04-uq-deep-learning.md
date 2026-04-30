---
title: "딥러닝에서의 불확실성 정량화: Softmax에서 Variational Inference까지"
description: Softmax의 한계, 신경망 calibration, Bayesian Neural Network, Variational Inference, Laplace Approximation을 정리한 세미나 노트.
author: starrypark
date: 2025-09-04 00:00:00 +0810
categories: [Statistics, Uncertainty Quantification]
tags: [bayesian neural network, variational inference, laplace approximation, calibration, deep learning, UQ]
pin: false
math: true
mermaid: true
---

## 개요

딥러닝 모델은 예측값과 함께 confidence score를 출력합니다. 그런데 이 숫자를 그대로 믿을 수 있을까?

결론부터 말하면, 대부분의 경우 믿기 어렵습니다. Softmax 출력은 종종 과도하게 높은 confidence를 보이고, 모델이 실제로 무엇을 모르는지를 반영하지 못합니다.

이 세미나는 딥러닝의 UQ 문제를 소개하고, 이를 해결하려는 Bayesian 접근법들을 다룹니다.

---

## 1. Softmax와 그 한계

Softmax는 logit 벡터 $z$를 클래스 확률로 변환하는 함수입니다.

$$\text{softmax}(z)_j = \frac{\exp(z_j)}{\sum_{k=1}^K \exp(z_k)}$$

출력이 $[0,1]$ 범위에 있고 합이 1이 되기 때문에, 종종 prediction confidence로 직접 사용됩니다.

그러나 두 가지 근본적인 한계가 있습니다.

**Limitation 1: 과도한 자신감 (Over-confidence)**. 신경망은 학습 데이터에서 loss를 줄이기 위해 softmax 출력을 극단적으로 0 또는 1에 가깝게 만들도록 학습됩니다. 결과적으로 실제 정확도보다 confidence가 훨씬 높게 나오는 경향이 있습니다.

**Limitation 2: Model uncertainty 미반영**. Softmax는 주어진 파라미터 하에서 입력 $x$에 대한 예측만 나타냅니다. 파라미터 자체에 대한 불확실성, 즉 epistemic uncertainty는 포함하지 않습니다. Out-of-distribution 입력이 들어와도 높은 confidence를 출력하는 문제가 여기서 발생합니다.

---

## 2. 신경망의 Calibration

**Calibration**은 모델의 predicted confidence가 실제 정확도를 얼마나 잘 반영하는지를 나타냅니다. 잘 calibrated된 모델에서는 70% confidence의 예측이 실제로 70% 정도의 확률로 맞아야 합니다.

**Reliability diagram**은 calibration을 시각화하는 표준적인 도구다. x축은 confidence bin, y축은 해당 bin에서의 실제 accuracy다. 완벽하게 calibrated된 모델은 대각선을 따릅니다.

세 가지 경우가 있습니다.

- **Overconfident**: accuracy보다 confidence가 높습니다. 대각선 아래에 위치합니다.
- **Underconfident**: accuracy보다 confidence가 낮습니다. 대각선 위에 위치합니다.
- **Calibrated**: 대각선을 따릅니다.

딥러닝 모델은 일반적으로 overconfident한 경향이 있고, 네트워크가 깊을수록 이 경향이 강해집니다.

Calibration error를 정량화하는 지표로는 **Expected Calibration Error (ECE)**가 널리 쓰입니다. Confidence bin들에 걸쳐 |accuracy - confidence|의 가중 평균입니다.

---

## 3. Classification에서의 UQ

단일 softmax 출력은 poorly calibrated되어 있고 model uncertainty를 포착하지 못합니다. 더 나은 불확실성 추정을 위해 파라미터에 대한 (근사적) posterior distribution $p(\theta \mid D)$를 활용할 수 있습니다.

주요 uncertainty 측도는 다음과 같습니다.

- **Mutual Information (MI)**: 예측 $y$와 파라미터 $\theta$ 사이의 mutual information. Epistemic uncertainty를 포착합니다.
- **Expected KL Divergence (EKL)**: stochastic softmax 출력과 expected softmax 출력 사이의 기대 KL divergence.
- **Predictive variance**: 예측 분포의 분산.

이 측도들은 모두 stochastic softmax 출력과 expected softmax 출력 사이의 기대 divergence를 계산합니다.

$$\hat{p} = \mathbb{E}_{\theta \sim p(\theta|D)}[p(y \mid x, \theta)]$$

---

## 4. Regression에서의 UQ

### 분포 파라미터 예측

가장 흔한 접근은 네트워크가 확률 분포의 파라미터를 직접 예측하도록 하는 것입니다. 정규 분포를 가정하면 평균 $\hat{y}$와 표준편차 $\sigma$를 출력하고, 예측 구간은 다음과 같습니다.

$$\left[\hat{y} - \frac{1}{2}\Phi^{-1}(\alpha) \cdot \sigma;\;\; \hat{y} + \frac{1}{2}\Phi^{-1}(\alpha) \cdot \sigma\right]$$

여기서 $\Phi^{-1}$는 표준정규분포의 역함수입니다.

### Prediction Interval 직접 예측

또 다른 접근은 prediction interval (PI)을 직접 예측하는 것입니다. 이 경우 두 가지 지표로 PI의 질을 평가합니다.

- **MPIW (Mean Prediction Interval Width)**: 평균 구간 폭. 모델의 평균적 확실성을 측정합니다.
- **PICP (Prediction Interval Coverage Probability)**: 실제값이 PI 안에 들어오는 비율. PI의 정확성을 평가합니다.

좋은 PI는 PICP가 명목 수준(예: 95%)을 만족하면서 MPIW를 최소화하는 것입니다.

---

## 5. Bayesian Neural Network

### 기본 아이디어

파라미터 $\theta$에 prior distribution $p(\theta)$를 부여하면, 새로운 출력 $y^*$에 대한 예측은 marginalizing을 통해 얻습니다.

$$p(y^* \mid x^*, X, Y) = \int p(y^* \mid x^*, \theta)\, p(\theta \mid X, Y)\, d\theta$$

이 적분이 바로 불확실성을 자연스럽게 포착하는 방법입니다. 파라미터에 대한 불확실성이 예측에 전파되기 때문입니다.

### Monte Carlo 근사

위 적분은 대부분의 경우 analytically intractable합니다. Monte Carlo approximation으로 근사합니다.

$$y^* \approx \frac{1}{N}\sum_{i=1}^N y_i^* = \frac{1}{N}\sum_{i=1}^N f_{\theta_i}(x^*)$$

여기서 $\theta\_i \sim p(\theta \mid X, Y)$는 posterior에서의 sample입니다. Posterior $p(\theta \mid X, Y)$를 어떻게 근사할 것인지가 핵심 과제입니다.

---

## 6. Variational Inference

**Variational Inference (VI)**는 posterior $p(\theta \mid X, Y)$를 다루기 쉬운 approximate distribution $q(\theta)$로 근사하는 방법입니다.

목표는 $q(\theta)$와 $p(\theta \mid X, Y)$ 사이의 KL divergence를 최소화하는 것입니다.

$$D_{KL}(q(\theta) \mid p(\theta \mid X, Y)) = \int q(\theta) \log \frac{q(\theta)}{p(\theta \mid X, Y)}\, d\theta$$

이를 전개하면:

$$D_{KL}(q(\theta) \mid p(\theta \mid X, Y)) = -D_{KL}(q(\theta) \mid p(\theta)) + E_q[\log p(Y \mid X, \theta)] + \text{const}$$

오른쪽에서 상수를 빼고 부호를 바꾼 것이 **Evidence Lower Bound (ELBO)**입니다.

$$\text{ELBO} = E_q[\log p(Y \mid X, \theta)] - D_{KL}(q(\theta) \mid p(\theta))$$

ELBO를 최대화하는 것이 KL divergence를 최소화하는 것과 동치다. ELBO의 첫 항은 데이터를 잘 설명하려는 likelihood 항이고, 두 번째 항은 $q$가 prior $p(\theta)$에서 너무 멀어지지 않도록 하는 regularization 역할을 합니다.

MCMC와 달리 VI는 최적화 문제로 변환되기 때문에 gradient descent로 풀 수 있고, 따라서 딥러닝 프레임워크와 자연스럽게 통합됩니다.

---

## 7. Laplace Approximation

**Laplace Approximation**은 posterior distribution을 MAP (Maximum A Posteriori) 추정치 주변의 Gaussian으로 근사하는 방법입니다.

Log posterior의 MAP 추정치 $\hat{\theta}$ 주변에서의 2차 Taylor expansion을 이용합니다.

$$\log p(\theta \mid x, y) \approx \log p(\hat{\theta} \mid x, y) - \frac{1}{2}(\theta - \hat{\theta})^\top (H + \tau I)(\theta - \hat{\theta})$$

여기서 $H$는 $\log p(\theta \mid X, Y)$의 negative Hessian입니다.

이 근사는 posterior를 평균 $\hat{\theta}$, 공분산 $(H + \tau I)^{-1}$인 Gaussian으로 만듭니다.

Laplace Approximation의 실용적 장점은 이미 학습된 네트워크에 사후적으로 적용할 수 있다는 것입니다. MSE나 cross entropy 같은 표준 loss function을 쓴 경우 일반적으로 적용 가능합니다. 단, Hessian 계산이 파라미터가 많은 대형 모델에서는 여전히 비용이 크기 때문에, 전체 Hessian 대신 diagonal이나 Kronecker-factored 근사를 쓰는 방법들이 연구되고 있습니다.


