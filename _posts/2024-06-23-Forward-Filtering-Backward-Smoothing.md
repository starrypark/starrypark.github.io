---
title: "Dynamic Factor Model에서의 FFBS(Forward-Filtering, Backward Smoothing) 알고리즘 이해하기"
categories: [Data Science, Time Series]
tags: [Statistics, Dynamic Factor Model, FFBS, MCMC, Bayesian, R]
math: true
date: 2024-06-23 12:43:00 +0900
---

거시경제 지표나 금융 데이터를 다루다 보면, 수십에서 수백 개에 달하는 시계열 변수들이 같이 움직이는 현상을 자주 보게 됩니다. 이 거대한 데이터 속에 숨겨진 몇 개의 핵심 동인(Driver)을 뽑아내는 방법론이 바로 Dynamic Factor Model (DFM)입니다.

그런데 이 숨겨진 요인, 즉 Latent Factor를 어떻게 추정할 수 있을까요? 베이지안 프레임워크나 EM 알고리즘에서 이 Factor들의 사후분포를 효과적으로 뽑아내기 위해 거의 필수적으로 쓰이는 기법이 하나 있습니다. 바로 오늘 다룰 FFBS (Forward-Filtering, Backward Smoothing) 알고리즘입니다. 처음 보면 수식의 바다에 빠지기 쉬운데, 직관적으로 접근하면 꽤 명쾌한 아이디어입니다.

## DFM과 State-Space Model

FFBS를 이해하려면 먼저 DFM을 State-Space Model(상태공간모형) 형태로 바라봐야 합니다. 시간 인덱스 집합 $\\\\{1, \dots, T\\\\}$ 에 대해, 관측된 $N$ 차원의 시계열 데이터를 $y\_t$, 그리고 우리가 찾고자 하는 $K$ 차원의 잠재 요인을 $f\_t$ 라고 해봅시다. 

$$
\begin{align*}
y_t &= \Lambda f_t + e_t \quad \text{(Observation Equation)} \\
f_t &= \Phi f_{t-1} + u_t \quad \text{(State Equation)}
\end{align*}
$$

여기서 $\Lambda$는 Factor Loading 행렬입니다. $e\_t$와 $u\_t$는 각각 관측 오차와 상태 오차를 의미하죠.
우리의 목표는 주어진 전체 관측치 $y\_{1:T}$ 를 바탕으로 숨겨진 요인 $f\_{1:T}$ 의 궤적을 추론하는 것입니다. 즉, 결합 사후분포 $p(f\_{1:T} \mid y\_{1:T})$ 에서 데이터를 샘플링하거나 최적값을 찾아야 합니다.

## 왜 한 번에 추정하지 않고 FFBS를 쓸까?

단순하게 각 시점 $t$ 마다 독립적으로 Factor를 추정하면 안 될까요? State Equation을 보면 $f\_t$는 이전 시점의 $f\_{t-1}$에 강하게 의존하고 있기 때문에 불가능합니다. 시계열 특유의 자기상관성(Autocorrelation)을 유지하면서 전체 궤적(Trajectory)의 조인트 분포를 구해야 하죠.

여기서 아주 멋진 수학적 트릭이 들어갑니다. 바로 마르코프 속성(Markov Property)을 이용해 결합 분포를 조건부 분포들의 곱으로 잘게 쪼개는 것입니다.

$$
p(f_{1:T} | y_{1:T}) = p(f_T | y_{1:T}) \prod_{t=1}^{T-1} p(f_t | f_{t+1}, y_{1:t})
$$

이 수식이 FFBS 알고리즘의 심장입니다. $T$ 개의 Factor 궤적을 한 번에 통째로 샘플링하는 건 차원이 너무 커서 비효율적이지만, 맨 끝 시점 $T$ 에서 출발해서 과거로 하나씩 되짚어가며 샘플링하는 것은 아주 쉽거든요. 

## 알고리즘의 두 단계: 필터링과 스무딩

이제 수식이 의미하는 바를 실제 작동 순서에 맞춰 풀어보겠습니다.

### 1. Forward-Filtering (Kalman Filter)
과거부터 현재 $t$ 까지의 데이터 $y\_{1:t}$ 만을 이용해서 $f\_t$ 의 분포를 계속 업데이트하며 미래로 나아갑니다. 이 과정은 흔히 아는 Kalman Filter와 완전히 동일합니다. 매 시점 $t$ 에 대해 다음 두 가지 파라미터를 계산해서 메모리에 저장해 둡니다.
* 사후 평균: $m\_t = E(f\_t \mid y\_{1:t})$
* 사후 분산: $C\_t = Var(f\_t \mid y\_{1:t})$

### 2. Backward-Smoothing (Sampling)
데이터의 끝인 시점 $T$ 에 도달했다면, 이제 방향을 틀어서 과거로 돌아갑니다.
가장 먼저 맨 마지막 시점의 Factor $f\_T$ 를 필터링의 최종 결과물인 정규분포 $N(m\_T, C\_T)$ 에서 하나 샘플링합니다. 그다음, $t = T-1, T-2, \dots, 1$ 순서로 1보씩 후퇴하면서 아래의 분포에서 $f\_t$ 를 샘플링합니다.

$$
f_t | f_{t+1}, y_{1:t} \sim N(h_t, H_t)
$$

여기서 $h\_t$ 와 $H\_t$ 는 이미 구해둔 $m\_t, C\_t$ 와 State Equation의 파라미터 $\Phi$ 등을 조합해 계산할 수 있습니다. 
직관적으로 보면, "내가 방금 전에 뽑은 미래의 값 $f\_{t+1}$"와 "현재까지의 관측치 $y\_{1:t}$" 양쪽의 정보를 모두 종합해서 "현재의 값 $f\_t$"를 결정하는 과정입니다. 미래의 정보가 과거의 추정치를 부드럽게 다듬어주기 때문에 'Smoothing'이라는 이름이 붙은 것이죠.

## R을 이용한 간단한 예제 구현

실무에서 MCMC 프레임워크를 짤 때는 깁스 샘플러(Gibbs Sampler) 내부에 이 수식들을 직접 하드코딩해서 쓰기도 하지만, R의 `dlm` 패키지를 사용하면 알고리즘의 뼈대를 아주 우아하게 구현할 수 있습니다.

```R
# install.packages("dlm")
library(dlm)

# 1. 간단한 Local Level Model(Factor가 1개인 단순화된 형태) 설정
# 실제 DFM 분석에서는 dlmModReg 등에 다변량 팩터 구조를 얹어서 사용합니다.
model <- dlmModPoly(order = 1, dV = 1.2, dW = 0.5)

# 2. 가상의 시계열 데이터 생성
set.seed(42)
y <- cumsum(rnorm(100, 0, sqrt(0.5))) + rnorm(100, 0, sqrt(1.2))

# 3. Forward-Filtering (칼만 필터 진행)
filtered_result <- dlmFilter(y, model)

# 4. Backward-Smoothing (1개의 MCMC 궤적 샘플을 뽑는다고 가정)
# dlmBSample 함수가 내부적으로 Backward 과정을 완벽하게 처리해 줍니다.
smoothed_sample <- dlmBSample(filtered_result)

# 결과 확인: y_t (관측치)와 f_t (스무딩된 샘플) 궤적 비교
plot(y, type="l", col="darkgray", lwd=2, ylab="Value", main="FFBS Result")
lines(dropFirst(smoothed_sample), col="red", lwd=2)
legend("topleft", legend=c("Observed y", "Smoothed Factor f"),
       col=c("darkgray", "red"), lty=1, lwd=2)
```

코드를 보면 `dlmFilter`로 데이터를 끝까지 한 번 밀고(Forward), 그 결과를 통째로 `dlmBSample`에 넘겨서 한 번에 역방향 샘플링(Backward)을 수행하는 것을 볼 수 있습니다. 만약 베이지안 추론을 한다면, 이 과정을 MCMC 루프 안에서 수천 번 반복하면서 파라미터 $\Phi, \Lambda$ 와 Factor $f\_t$ 의 사후분포를 수렴시키게 됩니다.

처음 DFM과 MCMC 논문을 읽을 때 $ \sum $ 이나 $ \prod $ 가 난무하는 증명 과정에 압도되기 쉽습니다. 하지만 결국 "앞으로 가면서 데이터를 모아 분포를 만들고, 뒤로 돌아오면서 궤적의 퍼즐 조각을 맞춘다"는 컨셉만 잡으면 구현 자체는 단순한 행렬 연산의 연속일 뿐입니다. 시계열 데이터의 차원 축소를 고민하고 계신다면, 정적인 PCA를 넘어 DFM과 FFBS의 동적인 매력을 꼭 경험해 보시길 바랍니다.