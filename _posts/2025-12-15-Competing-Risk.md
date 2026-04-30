---
title: "생존분석에서 Competing Risk 다루기: CSH와 Fine-Gray 모델의 진짜 차이"
description: Cause-Specific Hazard와 Fine-Gray(Subdistribution Hazard) 모델의 차이, CIF의 개념, 그리고 실무에서 어떤 모델을 선택해야 하는지 정리한 노트.
categories: [Data Science, Survival Analysis]
tags: [Statistics, Competing Risk, Fine-Gray, Cause-Specific Hazard, CIF, Biostatistics]
author: starrypark
pin: false
math: true
mermaid: true
date: 2025-12-15 12:47:00 +0900
---

생존분석(Survival Analysis)을 실제 임상 데이터나 비즈니스 데이터에 적용하다 보면 '경쟁 위험(Competing Risk)'이라는 까다로운 장벽에 부딪히게 됩니다.

위암 환자의 생존 데이터를 분석한다고 상상해 볼게요. 우리의 최대 관심사는 '위암으로 인한 사망'입니다. 그런데 추적 관찰 기간 동안 교통사고나 심장마비 등 '다른 원인으로 인한 사망'이 발생할 수 있죠. 이 분들은 더 이상 위암으로 사망할 수 없는 상태가 됩니다. 이런 상황을 관찰이 끝난 단순 중도절단(Censoring)으로 처리해도 될까요? 전혀 그렇지 않습니다. 이게 바로 분석의 판도를 바꾸는 경쟁 사건(Competing Event)이거든요.

이걸 무시하고 평소처럼 모델링을 돌려버리면 위험비(Hazard Ratio, HR)의 해석이 완전히 엉뚱한 방향으로 흘러갑니다. 누적발생확률을 계산해도 숫자가 꼬여버리죠. 오늘 포스팅에서는 실무 현장에서 정말 자주 혼동하는 Cause-Specific Hazard(CSH)와 Subdistribution Hazard(SH, Fine-Gray 모델)의 차이를 명확히 짚어보고, 언제 어떤 모델을 꺼내 들어야 하는지 제 경험을 녹여 정리해 보겠습니다.

## 1. 왜 Competing Risk를 따로 고민해야 할까?

경쟁 사건이 발생했다는 건, 그 관찰 대상자가 앞으로 우리가 관심 있는 사건을 겪을 가능성이 영원히 소멸되었다는 뜻입니다. 이걸 단순히 "관찰이 중단되었다(Censored)"라고 퉁치면 생존분석의 아주 중요한 가정이 무너집니다. 일반적인 분석에서는 Censoring된 사람도 언젠가는 관심 사건을 겪을 잠재적인 위험이 남아있고, 그 중단 시점이 사건 발생과 독립적이라고 가정하거든요.

결과적으로 경쟁 사건을 무시하고 흔히 쓰는 Kaplan-Meier 추정법으로 '암 사망 확률'을 구하게 되면, 실제 일어날 확률보다 수치가 뻥튀기되는 과대추정(Overestimation) 문제가 발생합니다. 

## 2. 진짜 발생 확률, CIF (Cumulative Incidence Function)

그래서 Competing risk가 존재하는 환경에서 특정 사건이 발생할 진짜 확률을 알고 싶다면 누적발생확률, 즉 CIF를 봐야 합니다. 
사건의 종류를 $k \in \\{1, \ldots, K\\}$ 라고 하고, 우리가 궁금한 타겟 사건을 $k=1$이라고 해볼게요.

$$F_k(t) = P(T \leq t,\; J = k)$$

수식을 풀어서 읽어보면 직관적입니다. "$t$ 시점까지 타겟 사건 $k$(여기서는 $J=k$)가 발생할 확률"이죠. KM 곡선은 경쟁 사건을 무시하고 사망 확률을 누적해 버리기 때문에, 반드시 이 CIF 수식과 개념으로 접근해야만 왜곡 없는 진짜 누적 발생 확률을 얻을 수 있습니다.

## 3. 원인 분석에 딱 맞는 CSH (Cause-Specific Hazard)

### 정의와 해석
먼저 특정 원인에 대한 위험 자체를 파고드는 CSH(Cause-Specific Hazard)부터 보겠습니다.

$$\lambda_k(t) = \lim_{\Delta t \to 0} \frac{P(t \leq T < t+\Delta t,\; J=k \mid T \geq t)}{\Delta t}$$

이 수식의 의미는 이렇습니다. "$t$ 시점까지 그 어떤 사건도 겪지 않고 무사히 살아남은 사람들 중에서, 아주 찰나의 순간($\Delta t$)에 사건 $k$가 발생할 순간 위험"을 뜻합니다. 
여기서 조건부를 잘 보면 아직 아무 일도 일어나지 않은 사람들이 Risk set(위험군)을 형성합니다. 경쟁 사건을 이미 겪은 사람은 이 Risk set에서 자연스럽게 제외되죠.

### 추정 방법
실무에서 코드를 돌릴 때는 생각보다 세팅이 단순합니다.
- 관심 사건 $k$만 이벤트를 1로 둡니다.
- 나머지 경쟁 사건들은 전부 Censoring(이벤트 0)으로 덮어버립니다.
- 그 상태로 우리가 잘 아는 Cox Proportional Hazard 모델을 적합시킵니다.

$$\lambda_k(t \mid X) = \lambda_{k0}(t)\exp(\beta_k^\top X)$$

### 언제 쓸까?
CSH는 "이 변수가 진짜로 암 사망 자체의 위험(Etiology)을 높이는 인자인가?"라는 인과적, 생물학적 원인 분석을 할 때 아주 적합합니다.

## 4. 확률 예측에 강한 SH (Subdistribution Hazard, Fine-Gray)

### 정의와 해석
반면에 Fine과 Gray가 1999년에 제안한 SH(Subdistribution Hazard)는 아예 CIF 자체를 직접 모델링하기 위해 뼈대를 설계한 모델입니다.

$$\tilde{\lambda}_k(t) = \lim_{\Delta t \to 0} \frac{P(t \leq T < t+\Delta t,\; J=k \mid T \geq t \text{ or } (T \leq t,\; J \neq k))}{\Delta t}$$

수식을 가만히 들여다보면 CSH와 조건부가 사뭇 다릅니다. Risk set에 "아무 일도 없었던 사람"뿐만 아니라 "이미 경쟁 사건을 겪어버린 사람"까지 가중치를 줘서 끝까지 품고 갑니다. 이렇게 억지로 Risk set을 유지시켜야 전체 누적 확률인 CIF 곡선과 수학적으로 부드럽게 연결되거든요.
회귀 모델의 형태 자체는 콕스 모델과 비슷하게 생겼습니다.

$$\tilde{\lambda}_k(t \mid X) = \tilde{\lambda}_{k0}(t)\exp(\tilde{\beta}_k^\top X)$$

### 해석할 때 주의점
여기서 튀어나오는 SH의 HR(Hazard Ratio) 계수는 "해당 사건의 순간 위험이 올라간다"라고 직역하면 위험합니다. 그보다는 **"절대적인 누적발생확률(CIF)의 크기를 키우는 방향으로 작동한다"**고 이해하시는 게 훨씬 안전하고 정확합니다.

### 언제 쓸까?
"그래서 이 요인이 있을 때 3년 안에 환자가 암으로 죽을 절대적인 확률(CIF)이 정확히 얼마나 변하는데?" 같은 예측이나 예후 평가가 필요할 때 사용합니다.

## 5. CSH vs Fine-Gray: 정답은 없다, 목적만 있을 뿐

둘 중 어떤 모델이 '맞다 틀리다'를 따지는 건 의미가 없습니다. 내가 데이터에 던지는 질문이 무엇이냐에 따라 도구를 골라 써야 합니다.

| 분석 목적 | 적합한 모델 |
| :--- | :--- |
| 원인별 위험 요인 파악 및 병인학적(Etiology) 해석 | CSH (Cause-Specific Hazard) |
| 실제 체감하는 절대 위험(CIF)의 예측 및 변화량 비교 | SH (Fine-Gray Subdistribution Hazard) |
| 보고서나 논문에 HR 수치를 적을 때 | 반드시 CSH와 SH 중 어떤 것인지 명확히 표기 |

## 6. 실무에서 제일 많이 받는 질문과 오해들

### "CSH HR은 위험하다는데, SH HR은 반대로 안전하대요. 이거 코드 에러 아닌가요?"
어떤 변수가 암 사망 위험(CSH)은 높이는데, 결과적인 CIF 효과(SH)는 오히려 떨어뜨리는 방향으로 나올 때가 종종 있습니다. 결코 에러가 아니라 Competing risk가 만들어내는 본질적인 현상입니다. CIF를 구하는 과정을 적분으로 뜯어보면 이유를 단번에 알 수 있어요.

$$F_k(t) = \int_0^t S(u-)\, \lambda_k(u)\, du$$

수식을 보면 타겟 사건 $k$의 순간 위험인 $\lambda\_k(u)$를 적분하기도 하지만, 그 앞에 $S(u-)$라는 값이 곱해져 있습니다. 이건 "이 시점까지 아무 일 없이 잘 버텨낸 확률"이에요.
만약 어떤 변수가 암 사망 위험도 높이지만, 심장마비 같은 다른 사망 위험을 어마어마하게 더 높여버린다고 가정해 볼게요. 그러면 사람들이 암에 걸리기도 전에 이미 다른 이유로 Risk set에서 사라져 버리니 $S(u-)$가 급격하게 쪼그라듭니다. 결과적으로 두 개를 곱해서 적분한 CIF 자체는 별로 안 크거나, 오히려 감소하는 기현상이 발생하는 거죠.

### "HR이 크면 당연히 발생 확률도 쑥쑥 높아야 하는 거 아니야?"
방금 설명한 이유 때문에 CSH HR이 2.0으로 꽤 높게 나오더라도, 경쟁 사건이 강하게 치고 들어오면 실제 누적 확률(CIF)은 찔끔 오르고 말 수 있습니다. "위험도(HR)"와 "절대위험(CIF)"은 완전히 별개의 개념으로 분리해서 소통해야 합니다.


### 참고문헌
Fine, J. P., & Gray, R. J. (1999). A proportional hazards model for the subdistribution of a competing risk. *Journal of the American Statistical Association*, 94(446), 496–509.