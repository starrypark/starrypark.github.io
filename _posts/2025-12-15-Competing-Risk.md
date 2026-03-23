---
title: "경쟁 위험 모델 (Competing Risk)"
description: CSH와 SH(Fine-Gray)의 차이, CIF 해석, 그리고 실무에서 두 모델을 어떻게 써야 하는지 정리
author: starrypark
date: 2025-12-15 13:22:34 +0810
categories: [Survival Analysis, Model Evaluation]
tags: [statistics, survival analysis, competing risk, Fine-Gray, biostatistics]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

생존분석에서 competing risk는 생각보다 자주 등장한다.

예를 들어 위암 환자의 "암 사망"을 분석한다고 하자. 관찰 중에 "다른 원인 사망"도 발생한다. 이때 다른 원인 사망은 단순 censoring이 아니다. **경쟁 사건(competing event)**이다. 다른 원인으로 사망한 사람은 더 이상 암 사망을 관찰할 수 없고, 암 사망의 risk set 해석 자체가 달라진다.

이 문제를 무시하면 위험비(HR)의 해석이 틀어지고, 누적발생확률(CIF)을 계산해도 의미가 꼬인다.

이 노트는 competing risk에서 핵심 개념인 **CSH**와 **SH(Fine–Gray)**의 차이를 정리하고, 실무에서 어떻게 선택하고 보고해야 하는지를 다룬다.

---

## 1. Competing Risk를 왜 별도로 다루는가

경쟁 사건이 발생하면 그 사람은 더 이상 관심 사건을 겪을 수 없다. 경쟁 사건을 단순 censoring으로 처리하면 다음 가정이 무너진다.

- 관심 사건이 관찰될 기회가 남아 있다는 가정
- censoring 독립성 가정의 의미

그 결과 Kaplan-Meier로 계산한 "암 사망 확률"은 일반적으로 **과대추정**이 된다.

---

## 2. 핵심 개념: CIF

Competing risk에서 가장 중요한 확률 객체는 **누적발생확률(Cumulative Incidence Function, CIF)**이다.

사건 종류 $k \in \{1, \ldots, K\}$에서 관심 사건을 $k=1$이라 하면:

$$F_k(t) = P(T \leq t,\; J = k)$$

여기서 $J$는 사건 타입이다. "$t$까지 사건 $k$가 발생할 확률"은 CIF로 말해야 올바르다. Kaplan-Meier로 계산한 값은 competing risk를 무시하기 때문에 과대추정이 발생한다.

---

## 3. CSH: Cause-Specific Hazard

### 정의

사건 타입 $k$에 대한 **Cause-Specific Hazard (CSH)**는 다음과 같다.

$$\lambda_k(t) = \lim_{\Delta t \to 0} \frac{P(t \leq T < t+\Delta t,\; J=k \mid T \geq t)}{\Delta t}$$

**해석**: "$t$까지 아무 사건도 발생하지 않은 사람들 중에서, 바로 그 다음 순간에 사건 $k$가 발생할 순간위험"이다.

Risk set은 **아직 어떤 사건도 경험하지 않은 사람들**이다. 경쟁 사건이 발생한 사람은 risk set에서 제외된다.

### 추정

실무에서는 다음과 같이 접근한다.

- 사건 $k$만 event = 1로 처리
- 나머지(경쟁 사건 포함)는 censoring 처리
- Cox model로 적합

$$\lambda_k(t \mid X) = \lambda_{k0}(t)\exp(\beta_k^\top X)$$

### 용도

CSH는 **원인별 위험요인(etiology) 분석**에 적합하다. "이 변수가 암 사망 자체의 위험을 올리는가?"라는 질문에 답할 때 쓴다.

---

## 4. SH: Subdistribution Hazard (Fine–Gray)

### 정의

**Subdistribution Hazard (SH)**는 CIF를 직접 모델링하기 위해 Fine과 Gray (1999)가 제안한 hazard다.

$$\tilde{\lambda}_k(t) = \lim_{\Delta t \to 0} \frac{P(t \leq T < t+\Delta t,\; J=k \mid T \geq t \text{ or } (T \leq t,\; J \neq k))}{\Delta t}$$

조건부의 구조가 CSH와 다르다. Risk set이 "아직 사건이 없는 사람"뿐 아니라, **이미 경쟁 사건을 겪은 사람(가중 방식으로)**까지 포함한다.

Fine–Gray 모델은 다음과 같다.

$$\tilde{\lambda}_k(t \mid X) = \tilde{\lambda}_{k0}(t)\exp(\tilde{\beta}_k^\top X)$$

### 해석 주의

SH의 계수는 "순간위험의 증가"라기보다 **CIF를 키우는 방향**으로 이해하는 것이 안전하다.

### 용도

SH는 **절대위험(CIF) 예측**에 적합하다. "이 변수가 있을 때 3년 내 암 사망 확률이 얼마나 달라지는가?"라는 질문에 답할 때 쓴다.

---

## 5. CSH vs SH: 언제 무엇을 쓰는가

| 목적 | 적합한 모델 |
|---|---|
| 원인별 위험요인 해석 (etiology) | CSH |
| 절대위험(CIF) 예측 및 비교 | SH (Fine–Gray) |
| 보고서에 HR을 쓸 때 | 반드시 CSH/SH 중 어느 것인지 명시 |

둘 중 어느 것이 "맞는" 모델이냐의 문제가 아니다. **목적이 다른 것이다.**

---

## 6. 자주 발생하는 혼동

### CSH HR과 SH HR이 반대로 나오는 경우

어떤 covariate에서 CSH HR > 1 (암 사망 위험 증가)인데 SH HR < 1 (CIF 감소 방향)이 나올 수 있다. 이는 오류가 아니라 competing risk의 본질적인 현상이다.

CIF는 다음 관계를 갖는다.

$$F_k(t) = \int_0^t S(u-)\, \lambda_k(u)\, du$$

여기서 $S(u-)$는 "아무 사건도 안 난 상태로 버티는 확률"이고, 이 안에 경쟁 사건의 영향이 들어간다.

어떤 변수가 "암 사망 위험도 올리지만 다른 원인 사망을 더 많이 올리는" 경우, $S(u-)$가 작아지면서 결과적으로 CIF는 크게 올라가지 않을 수 있다.

### "HR이 크면 확률도 높아야 하지 않나?"

이 오해가 실무 보고에서 가장 자주 발생한다. CSH HR이 2.0이라도 competing risk 때문에 CIF는 크게 증가하지 않을 수 있다. HR과 절대위험(CIF)은 다른 것이다.

---

## 7. 보고서 구성 권장 형태

Competing risk가 있는 분석의 보고서는 다음 구조가 혼동을 줄인다.

1. Competing risk 배경 설명 (1–2 문단)
2. CIF 그래프 (관심 사건, 경쟁 사건 각각)
3. CSH 결과표 (위험요인 해석용, HR의 의미 명시)
4. SH(Fine–Gray) 결과표 (CIF 변화 방향 / 예측용, HR의 의미 명시)
5. 결론 문장: "CSH 기준으로는 위험요인이지만, competing risk 때문에 CIF 효과는 제한적/상반됨"

이 구조로 가면 "HR이 큰데 왜 확률이 안 올라가냐"는 질문에 대한 답이 이미 보고서 안에 있다.

---

## 8. 정리

- Competing risk에서 "확률"을 말하려면 CIF를 써야 한다. KM 곡선으로 관심 사건의 절대 확률을 말하지 않는다.
- **CSH**는 원인별 위험 해석에 적합하고, **SH(Fine–Gray)**는 CIF 예측에 적합하다.
- 둘의 HR은 의미가 다르다. 보고서에 HR을 쓸 때는 반드시 어느 것인지 명시해야 한다.
- 가능하면 CIF 그래프를 함께 제시한다. 숫자보다 그림이 오해를 줄인다.

---

## 참고문헌

Fine, J. P., & Gray, R. J. (1999). A proportional hazards model for the subdistribution of a competing risk. *Journal of the American Statistical Association*, 94(446), 496–509.
