---
title: 통계학에서의 Uncertainty Quantification
description: 불확실성 정량화(Uncertainty Quantification, UQ)의 이해와 통계적 방법
author: starrypark
date: 2025-08-13 10:00:20 +0800
categories: [Statistics, Uncertainty Quantification]
tags: [Uncertainty Quantification, Statistics, Bayesian]
pin: true
math: true
mermaid: true
use_math: true
---


# 불확실성 정량화(Uncertainty Quantification, UQ)의 이해와 통계적 방법

## 개요

불확실성(Uncertainty)은 실험, 측정, 또는 모델 예측의 결과가 어느 정도 알려지지 않았는지를 의미한다. 데이터 자체의 무작위성, 제한된 지식이나 모델 불완전성, 또는 불완전한 정보 등 다양한 원인에서 발생할 수 있다. 여기서 핵심 질문은 “이 결과에 대해 얼마나 확신할 수 있는가?”이다. 앞으로는 Uncertainty Quantification 탭에 불확실성을 정량적으로 계산하는 방법을 알아볼 것이다.

## 예측 정확도와 예측 불확실성

예측 정확도(prediction accuracy)는 실제 값에 얼마나 근접한지를 나타내며, 일반적으로 MSE, RMSE, MAE, R²와 같은 단일 점 추정(point estimate) 지표로 평가된다.
반면 예측 불확실성(prediction uncertainty)은 특정 예측값에 대한 신뢰도를 구간 또는 확률 분포 형태로 나타낸다. 이때 coverage, calibration, interval width 등이 주요 평가 지표이다. 중요한 점은 높은 정확도가 반드시 낮은 불확실성을 보장하지는 않는다는 것이다.

## 불확실성의 유형

* **Aleatoric Uncertainty**: 데이터의 내재적 변동성에서 기인한다. 표본 추출 변동성, 측정 오차 등이 포함되며, 동일한 데이터의 반복 수집으로는 줄일 수 없다.
* **Epistemic Uncertainty**: 제한된 지식이나 모델의 불완전성에서 발생한다. 작은 표본 크기, 누락된 변수 등이 원인으로, 더 다양한 데이터 수집이나 모델 개선을 통해 줄일 수 있다.

## 빈도주의적 접근

1. **Confidence Interval**: 반복 표본 추출에서 \$(1-\alpha)\times100%\$의 비율로 모수를 포함하는 구간을 설정한다. 전형적인 형태는
   $$\hat{\theta} \pm z_{\alpha/2} \cdot SE(\hat{\theta})$$
으로 나타낸다.

2. **Delta Method**: 함수형 추정량의 분산을 근사하는 방법으로, 복잡한 함수의 불확실성을 추정하는 데 사용된다.
3. **Bootstrap**: 표본 재추출을 통해 추정량의 분포를 경험적으로 근사하여 표준오차, 편향, 신뢰구간을 추정한다.

## 신뢰구간 평가

신뢰구간은 coverage와 width 두 가지 기준으로 평가된다. Coverage는 구간이 참값을 포함하는 비율을 의미하며, width는 구간의 길이를 의미한다. 좁은 구간은 정밀성을 높이지만 coverage를 떨어뜨릴 수 있으며, 반대로 지나치게 넓은 구간은 보수적일 수 있다.

## 베이지안 접근

베이지안 방법에서는 모든 미지수를 확률변수로 취급한다. 사전분포(prior) \$p(\theta)\$와 데이터로부터 얻는 likelihood \$p(y|\theta)\$를 결합하여 사후분포(posterior) \$p(\theta|y)\$를 도출한다.
이때 사후분포로부터 얻은 **Credible Interval**은 “데이터와 사전 정보가 주어졌을 때, 모수가 해당 구간에 존재할 확률”을 직접적으로 제시한다는 점에서 빈도주의적 신뢰구간과 다르다.

## MCMC와 불확실성 정량화

베이지안 추론에서는 사후분포가 해석적으로 닫힌 형태를 갖지 않는 경우가 많다. 이때 Markov Chain Monte Carlo(MCMC) 방법을 사용하여 표본을 생성함으로써 사후분포를 근사한다.
MCMC 결과를 통해 신뢰할 수 있는 credible interval을 구성할 수 있으며, 이는 비대칭적 분포나 복잡한 모형에서도 유연하게 적용 가능하다.

## 결론

결론적으로 불확실성 정량화는 단순히 추정치의 정확성을 넘어, 결과에 대한 신뢰 수준을 체계적으로 제시하는 데 핵심적인 역할을 한다. 빈도주의적 방법과 베이지안적 방법은 서로 다른 철학적 기반을 가지지만, 실제 연구에서는 상호 보완적으로 사용되며 불확실성 해석의 폭을 넓힌다.