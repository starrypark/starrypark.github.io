---
title: "Conformal Prediction 기초"
description: 분포 가정 없이 유한 표본에서 coverage를 보장하는 prediction band 구성 원리 정리
author: starrypark
date: 2025-02-20 10:00:20 +0800
categories: [Statistics, Uncertainty Quantification]
tags: [conformal prediction, statistics, uncertainty quantification]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

Prediction interval을 만드는 방법은 많다. 하지만 대부분은 분포 가정이 필요하거나, 점근적으로만 coverage를 보장한다.

**Conformal Prediction**은 다르다. 분포 가정 없이, 유한 표본에서도 정확한 coverage를 보장하는 prediction band를 만든다. 핵심은 iid 조건과 permutation argument다.

Won Chang 교수님 강의노트 Chapter 7을 바탕으로 정리했다.

---

## 1. 세팅

- $(X_i, Y_i)$: iid sample
- $X_i \subset \mathscr{X} \subset \mathbb{R}^p$
- $Y_i \subset \mathscr{Y} \subset \mathbb{R}^p$

---

## 2. 목표

유한 표본 $(X_1, Y_1), \ldots, (X_n, Y_n)$을 기반으로, 새 입력 $X_{n+1}$에 대해 다음을 만족하는 prediction band를 구성한다.

$$\hat{C}_n : \mathscr{X} \to \{\text{subsets of } \mathscr{Y}\}$$

$$P\!\left(Y_{n+1} \in \hat{C}_n(X_{n+1})\right) \geq 1 - \alpha$$

Gaussian 가정도 없고, 점근 조건도 없다. 이것이 Conformal Prediction의 핵심 강점이다.

---

## 3. Permutation 논증

### 아이디어

$Y_1, \ldots, Y_n, Y_{n+1}$이 iid이므로, 이 $n+1$개 값의 모든 순열은 equally likely하다.

따라서 $Y_{n+1}$이 순서통계량 $Y_{(1)} \leq \cdots \leq Y_{(n+1)}$ 중 $i$번째일 확률은 다음과 같다.

$$P\!\left(Y_{n+1} = Y_{(i)}\right) = \frac{1}{n+1}$$

이로부터 다음이 성립한다.

$$P\!\left(Y_{n+1} \text{ is among the } \lceil(1+n)(1-\alpha)\rceil \text{ smallest of } Y_1, \ldots, Y_n, Y_{n+1}\right) \geq 1 - \alpha$$

$n+1$개 값 중 가장 작은 $\lceil(1-\alpha)(n+1)\rceil$개 안에 $Y_{n+1}$이 들어올 확률이 $1-\alpha$ 이상이라는 뜻이다.

### 문제: 순환 구조

위 부등식에는 한 가지 문제가 있다. $Y_{n+1}$을 예측하기 위해 $Y_{n+1}$ 자체를 사용한다. 이 순환을 없애야 한다.

### 핵심: 두 사건은 동치다

다음 두 사건을 정의한다.

$$A = \left\{Y_{n+1} \text{ is among the } \lceil(1+n)(1-\alpha)\rceil \text{ smallest of } Y_1, \ldots, Y_n, Y_{n+1}\right\}$$

$$B = \left\{Y_{n+1} \leq \text{ the } \lceil(1+n)(1-\alpha)\rceil\text{-th smallest of } Y_1, \ldots, Y_n\right\}$$

$A = B$임을 보인다. $A^c$를 쓰면:

$$A^c = \left\{Y_{n+1} > \text{ the } \lceil(1+n)(1-\alpha)\rceil\text{-th smallest of } Y_1, \ldots, Y_n, Y_{n+1}\right\}$$

$Y_{n+1}$은 자기 자신보다 클 수 없으므로, $Y_{n+1}$이 그 $\lceil(1+n)(1-\alpha)\rceil$번째 값일 수 없다. 따라서:

$$A^c \Leftrightarrow \left\{Y_{n+1} > \text{ the } \lceil(1+n)(1-\alpha)\rceil\text{-th smallest of } Y_1, \ldots, Y_n\right\} = B^c$$

즉 $A = B$다.

### 결론

$$P\!\left(Y_{n+1} \leq Y_{\left(\lceil(1-\alpha)(n+1)\rceil\right)}\right) \geq 1 - \alpha$$

여기서 $Y_{\left(\lceil(1-\alpha)(n+1)\rceil\right)}$는 $Y_1, \ldots, Y_n$만으로 계산된 quantile이다.

$Y_{n+1}$ 없이도 coverage를 보장하는 prediction band를 만들 수 있다.

---

## 4. 정리

| 구분 | 내용 |
|---|---|
| 가정 | $(X_i, Y_i)$ iid |
| 목표 | $P(Y_{n+1} \in \hat{C}_n(X_{n+1})) \geq 1-\alpha$ |
| 핵심 도구 | Permutation + 동치 사건 변환 |
| 강점 | 분포 가정 불필요, 유한 표본 보장 |

Conformal Prediction의 이후 확장(nonconformity score, split conformal, full conformal 등)은 모두 이 permutation 구조 위에서 전개된다.
