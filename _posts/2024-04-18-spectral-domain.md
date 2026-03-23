---
title: "[Ch.5] 공간 통계의 Spectral Domain 분석"
description: Spatial process를 주파수 영역에서 분석하는 방법, spectral representation, periodogram, Whittle likelihood까지 정리한 노트이다.
author: starrypark
date: 2024-04-18 00:00:00 +0810
categories: [Statistics, Spatial Statistics]
tags: [spectral analysis, periodogram, fourier transform, matern, geostatistics]
pin: true
math: true
mermaid: true
use_math: true
---

## 개요

지금까지는 spatial process를 공간 영역(space domain)에서 다뤘는다. Covariance function, semivariogram, kriging 모두 "두 점 사이의 거리"를 기반으로 했다.

Chapter 5는 같은 과정을 **주파수 영역(frequency domain)**에서 보는 거다.

왜 이게 필요하냐면, 어떤 연산은 공간 영역에서 복잡하지만 주파수 영역에서는 단순해져다. 대표적인 예가 convolution이다. 공간 영역에서의 convolution은 주파수 영역에서 단순한 곱셈이 된다. 이 도메인 전환 덕분에 계산을 크게 줄일 수 있다.

실용적으로 가장 중요한 결과는 챕터 마지막의 **Whittle likelihood**예다. Exact likelihood 계산이 $O(n^3)$인 것과 달리, $O(N \log^2 N)$으로 줄여줍니다.

---

## 5.1 Spectral Representation

### Continuous Fourier Transform

$\mathbb{R}^d$ 위에서 integrable한 함수 $g$에 대해 Fourier transform을 정의한다.

$$G(\omega) = \int_{\mathbb{R}^d} g(s) \exp\!\left(i\omega^t s\right) ds \tag{5.1}$$

$G$가 integrable하면 역방향도 성립한다.

$$g(s) = \frac{1}{(2\pi)^d} \int_{\mathbb{R}^d} G(\omega) \exp\!\left(-i\omega^t s\right) d\omega \tag{5.2}$$

$G(\omega)$는 주파수 $\omega$에 해당하는 복소 지수 성분의 amplitude이다.

공간 영역과 주파수 영역은 서로 대응 관계를 갖고 있다. 공간 영역에서의 convolution은 주파수 영역에서 곱셈이고, 그 반대도 성립한다. 이런 성질 덕분에 계산이 편한 쪽으로 이동해서 작업할 수 있다.

### Aliasing

연속 과정 $Z(s)$를 regular grid $\Delta\mathbb{Z}^d$에서만 관측할 때 문제가 생긴다.

Grid 간격이 $\Delta = (\delta_1, \ldots, \delta_d)$라면, 주파수 $\omega$와 $\omega + 2\pi z_2/\Delta$ ($z_2 \in \mathbb{Z}^d$)는 이 grid 관측값에서 구별이 불가능한다. 이를 **aliasing effect**라고 한다.

결과적으로, grid 관측의 spectrum은 주파수 구간 $[-\pi/\Delta, \pi/\Delta]$ 안에 집중된다. 이 구간을 넘어서는 주파수는 모두 이 구간 안의 값으로 접혀 들어와 (alias), 구간 내부의 power에 겹쳐집니다.

정확한 spectral 분석을 원한다면 **Nyquist frequency** $\pi/\Delta$가 충분히 높아야 해진다. 즉, 더 높은 주파수 성분이 총 power에 무시할 수 있을 만큼만 기여해야 한다는 뜻이다. 실용적으로는 $\Delta$를 충분히 작게, 즉 촘촘하게 관측해야 한다.

### Spectral Representation of a Stationary Process

유한한 harmonic 성분들의 합으로 stationary process를 만들 수 있다. 복소 확률 변수 $Y_1, \ldots, Y_m$이 서로 uncorrelated하고 $E|Y_j|^2 = f_j$이면:

$$Z(s) = \sum_{j=1}^m \exp\!\left(i\omega_j^t s\right) Y_j$$

는 covariance $C(s) = \sum_{j=1}^m \exp(i\omega_j^t s) f_j$를 가지는 weakly stationary process이다.

이걸 연속 버전으로 확장하면 spectral representation theorem이 나온다.

**Theorem (Spectral Representation):** 평균이 0인 mean square continuous weakly stationary process $Z(s)$에 대해, orthogonal increment를 가지는 spectral process $Y(\omega)$가 존재해서 다음이 성립한다.

$$Z(s) = \int_{\mathbb{R}^d} \exp\!\left(i\omega^t s\right) dY(\omega)$$

Spectral process $Y(\omega)$는 이런 성질을 가져다.

- $E(Y(\omega)) = 0$
- **Orthogonal increments**: $\omega_3 < \omega_2 < \omega_1 < \omega_0$이면, $E[(Y(\omega_3) - Y(\omega_2))(Y(\omega_1) - Y(\omega_0))] = 0$

$E|dY(\omega)|^2 = F(d\omega)$로 정의하면, $F$는 positive finite measure이다. $F$를 **spectral measure**라고 해진다.

물리적 직관: spectral representation은 process의 총 fluctuation을 각 주파수 성분으로 분해하는 거다. $F(A)$는 주파수 구간 $A$에 할당된 평균 power이다. 전체 power는:

$$E|Z(s)|^2 = C(0) = F(\mathbb{R}^d)$$

### Bochner's Theorem (재등장)

Chapter 2에서 나왔던 Bochner's theorem이 spectral 언어로 다시 표현된다.

**Theorem:** 실수값 연속 함수 $C$가 positive definite인 것은 다음과 동치이다.

$$C(s) = \int_{\mathbb{R}^d} \exp\!\left(is^t\omega\right) F(d\omega) \tag{5.3}$$

$F$가 Lebesgue density $f$를 가지면, $f$를 **spectral density**라고 해진다. $C$가 연속이면 Fourier inversion formula가 성립한다.

$$f(\omega) = \frac{1}{(2\pi)^d} \int_{\mathbb{R}^d} \exp\!\left(-i\omega^t x\right) C(x)\, dx$$

즉, **covariance function과 spectral density는 Fourier transform 쌍**이다. Spatial dependence 분석은 covariance function을 추정하는 방식과 spectral density를 추정하는 방식이 등가이다.

### Isotropic Covariance의 Spectral Representation

Isotropic process $Z$에서 $C(h) = C_0(\|h\|)$이면, spectral representation이 단순화된다.

**Theorem:** Isotropic covariance $C_0(h)$는:

$$C_0(h) = \int_0^\infty Y_d(\omega h)\, \Phi(d\omega)$$

로 쓸 수 있다. 여기서

$$Y_d(h) = \left(\frac{2}{h}\right)^{(d-2)/2} \Gamma\!\left(\frac{d}{2}\right) J_{(d-2)/2}(h)$$

$J_v(\cdot)$는 first kind Bessel function이다.

반대로, spectral density가 주어졌을 때 covariance function은 다음으로 구할 수 있다 (isotropy 가정 하에):

$$f(\omega) = \frac{1}{(2\pi)^{d/2}} \int_0^\infty \frac{J_{(d-2)/2}(\omega h)}{(\omega h)^{(d-2)/2}} h^{d-1} C_0(h)\, dh \tag{5.4}$$

$d = 2$이면:

$$f(\omega) = \frac{1}{2\pi} \int_0^\infty J_0(\omega h)\, h C_0(h)\, dh$$

$d = 3$이면:

$$f(\omega) = \frac{1}{2\pi^2} \int_0^\infty \frac{\sin(\omega h)}{\omega h}\, h^2 C_0(h)\, dh$$

### Principal Irregular Term

**Definition:** Isotropic covariance function $C$의 **principal irregular term (PIT)**는 $h = 0$ 근방에서 $C$를 series expansion할 때 $h$의 짝수 거듭제곱에 비례하지 않는 첫 번째 항이다.

Stein (1999)에 따르면, 대부분의 common covariance model에서 PIT는 $\alpha h^\beta$ ($\beta > 0$, even integer가 아닌) 또는 $\alpha \log(h) h^\beta$ ($\beta$가 even integer) 형태이다.

PIT는 sample path의 smoothness와 직결된다. Chapter 2에서 봤던 $1 - \phi(t) \sim t^\alpha$와 같은 아이디어이다.

---

## 5.2 주요 Spectral Densities

### Triangular Model

Triangular isotropic covariance:

$$C(h) = C_0(h) = \sigma(a - h)^+, \quad \sigma, a > 0$$

$(a)^+ = a$이면 $a > 0$, 아니면 0이다. 이 모델은 $d = 1$에서만 valid한다.

$d = 1$에서 spectral density:

$$f(\omega) = \sigma\pi^{-1}\{1 - \cos(\alpha\omega)\}/\omega^2, \quad \omega > 0$$

$f(0) = \sigma\pi^{-1}\alpha^2/2$이다.

$\alpha$가 커질수록 spectral density가 낮은 주파수 쪽에 더 집중된다. 공간적으로 범위가 넓어지는 것과 대응된다.

### Squared Exponential Model

Isotropic squared exponential covariance:

$$C_0(h) = \sigma e^{-\alpha h^2}$$

Spectral density:

$$f_0(\omega) = \frac{1}{2}\sigma(\pi\alpha)^{-1/2} e^{-\omega^2/(4\alpha)}$$

흥미로운 점은 $C_0$와 $f_0$가 모두 Gaussian 형태라는 거다. Fourier transform이 Gaussian 함수를 Gaussian으로 보내기 때문이다. $\alpha$가 커지면 공간에서 더 빠르게 감소하고, 주파수 영역에서는 더 넓게 퍼져다.

### Matérn Class

Matérn class의 spectral density:

$$f(\omega) = f_0(\omega) = \phi\left(\alpha^2 + \omega^2\right)^{-\nu - d/2}$$

파라미터는 $\nu > 0$, $\alpha > 0$, $\phi > 0$이다. $\phi \propto \sigma\alpha^{2\nu}$예다.

대응하는 covariance:

$$C(h) = C_0(h) = \frac{\pi^{d/2}\phi}{2^{\nu-1}\Gamma(\nu + d/2)\alpha^{2\nu}} (\alpha h)^\nu K_\nu(\alpha h)$$

$K_\nu$는 modified Bessel function of the third kind이다.

Spectral density가 $(\alpha^2 + \omega^2)^{-\nu - d/2}$ 형태라는 게 중요한다. 높은 주파수 $\omega$에서 power law로 감소하고, $\nu$가 클수록 더 빠르게 감소한다. 즉, 고주파 성분이 줄어들면서 process가 더 smooth해지는 거다.

**Handcock & Wallis (1994) reparametrization**

차원 $d$에 의존하지 않는 covariance 표현:

$$C_0(h) = \frac{\sigma}{2^{\nu-1}\Gamma(\nu)} \left(2\nu^{1/2}h/\rho\right)^\nu K_\nu\!\left(2\nu^{1/2}h/\rho\right)$$

대응하는 spectral density (단, 이건 $d$에 의존):

$$f_0(\omega) = \frac{\sigma\, g(\nu, \rho)}{(4\nu/\rho^2 + \omega^2)^{\nu+d/2}}, \quad g(\nu, \rho) = \frac{\Gamma(\nu + d/2)(4\nu)^\nu}{\pi^{d/2}\rho^{2\nu}\Gamma(\nu)}$$

Covariance를 $d$-free하게 표현하면 파라미터 해석이 쉬워지지만, spectral density에서 $d$가 다시 등장한다.

---

## 5.3 Spectral Density 추정

### Periodogram

$n_1 \times n_2$ regular grid 위에서 $N = n_1 n_2$개의 등간격 관측값이 있다고 해진다. Grid 간격은 $\Delta = (\delta_1, \delta_2)$예다.

**Definition (Periodogram):**

$$I_N(\omega_0) = \delta_1\delta_2(2\pi)^{-2}(n_1 n_2)^{-1} \left|\sum_{s_1=1}^{n_1}\sum_{s_2=1}^{n_2} Z(\Delta s) \exp\!\left(-i\Delta s^t\omega\right)\right|^2 \tag{5.5}$$

Periodogram은 spectral measure $F$의 추정량으로 생각할 수 있다. $J(\omega)$를 discrete spectral process라고 정의하면 $I_N(\omega) = |J(\omega)|^2$이다.

Periodogram은 다음과 같이도 쓸 수 있다.

$$I_N(\omega_0) = \delta_1\delta_2(2\pi)^{-2} \sum_{h_1}\sum_{h_2} c_N(\Delta h)\exp\!\left(-i\Delta h^t\omega\right) \tag{5.6}$$

여기서 $c_N(\Delta h) = N^{-1}\sum_{s_1}\sum_{s_2} Z(\Delta s)Z(\Delta(s+h))$이다. 즉, periodogram은 sample autocovariance의 Fourier transform이다.

### Periodogram의 이론적 성질

Periodogram의 기댓값은:

$$E(I_N(\omega_0)) = (2\pi)^{-2}(n_1 n_2)^{-1} \int_{\Pi^2_\Delta} f_\Delta(\omega) W(\omega - \omega_0)\, d\omega$$

$W_N(\omega)$는 Fejér kernel 형태이고, $f_\Delta(\omega) = \sum_{Q \in \mathbb{Z}^2} f(\omega + 2\pi Q/\Delta)$는 aliasing이 반영된 spectral density이다.

### Periodogram의 점근 성질

$n_1, n_2 \to \infty$, $n_1/n_2 \to \lambda$일 때:

1. $E[I_N(\omega)] \to f_\Delta(\omega)$ (점근적으로 unbiased)
2. $\text{Var}[I_N(\omega)] \to f_\Delta^2(\omega)$ (분산이 0으로 가지 않음!)
3. 서로 다른 주파수 $\omega \neq \omega'$에서 $I_N(\omega)$와 $I_N(\omega')$는 점근적으로 독립
4. Process $Z$가 stationary이고 cumulant 조건을 만족하면, Fourier frequency $\omega_j$에서:

$$I_N(\omega_j) \xrightarrow{d} f(\omega_j) \cdot \chi^2_2/2$$

2번이 중요한다. $n$이 아무리 커도 periodogram의 분산이 0으로 가지 않는다. Periodogram은 **inconsistent** estimator이다. 그래서 smoothing이 필요한다.

### Missing Value가 있는 격자 데이터

관측값 중 일부가 빠져 있을 때, **zero-filling taper**를 쓴다. Weight function $g(s/n)$이 관측이 없는 위치에서 0, 있는 위치에서 1이다.

$$\tilde{I}_N(\omega) = \frac{1}{H_2(0)} \left|\sum_{k=1}^N \left[Y(s_k) - g(s_k/n)\tilde{Z}\right] \exp(-i\omega^t s_k)\right|^2 \tag{5.7}$$

여기서 $H_j(\lambda) = (2\pi)^2 \sum_{k=1}^N g^j(s_k/n)\exp(i\lambda^t s_k)$이고, $\tilde{Z}$는 가중 평균이다.

$g$가 bounded이고 bounded variation이면:

1. $E[\tilde{I}_N(\omega)] = f(\omega)^2 + O(N^{-1})$
2. $\text{Var}\{\tilde{I}_N(\omega)\} = |H_2(0)|^{-2}\left[H_2(0)^2 + H_2(2\omega)^2\right]f(\omega)^2 + O(N^{-1})$

### Spectral Domain에서의 Least Squares

Periodogram 값에 parametric spectral density model $f$를 fit하는 데 **WNLS**를 쓸 수 있다. 분산이 작은 주파수에 더 많은 weight를 줍니다. $f(\omega)^{-1}$을 weight로 쓰는 방식은 Chapter 3에서 semivariogram에 WLS를 적용하는 것과 유사한다.

---

## 5.4 Spectral Domain에서의 Likelihood 추정

### Whittle Likelihood

Spectral domain의 가장 강력한 도구이다. **Whittle's approximation**은 Gaussian negative log likelihood를 다음으로 근사한다.

$$\frac{N}{(2\pi)^2} \int_{\mathbb{R}^2} \left[\log f(\omega) + I_N(\omega) f(\omega)^{-1}\right] d\omega$$

적분을 discrete Fourier frequency에서의 합으로 근사하고, $I_N$을 Fast Fourier Transform (FFT)로 계산하면 전체 계산량이 $O(N \log^2 N)$으로 줄는다. $n_1$, $n_2$가 highly composite number (small prime factors의 곱)이면 FFT가 가장 효율적으로 동작한다.

Chapter 4의 exact likelihood는 $\Sigma(\theta)^{-1}$과 $|\Sigma(\theta)|$ 계산 때문에 $O(n^3)$이었는다. Whittle approximation은 이걸 $O(N \log^2 N)$으로 대폭 줄여줍니다. 대규모 데이터셋에서 큰 차이이다.

### 점근 공분산 행렬

MLE 추정량 $\theta_1, \ldots, \theta_r$의 asymptotic covariance matrix:

$$2N \left\{\frac{1}{4\pi^2} \int_{[-\pi,\pi]^2} \frac{\partial \log f(\omega_1)}{\partial\theta_j} \frac{\partial \log f(\omega_2)}{\partial\theta_k} d\omega_1 d\omega_2\right\}^{-1} \tag{5.8}$$

이 표현은 Fisher information matrix의 역행렬보다 훨씬 계산이 쉽다. Spectral density의 log derivative만 계산하면 되거든.

---

## 정리

이 챕터의 핵심 흐름을 한 줄로 요약하면 이래다.

> Covariance function $\leftrightarrow$ Spectral density 는 Fourier transform 쌍이고, spectral 쪽에서 작업하면 계산이 훨씬 효율적이다.

구체적인 이득은 두 가지이다.

- **Whittle likelihood**: $O(n^3) \to O(N\log^2 N)$
- **점근 분산**: Fisher information matrix 역행렬 대신 spectral density log derivative로 계산

단, 이 효율성은 regular grid 데이터에서 극대화된다. Irregularly spaced data에서는 FFT를 직접 쓸 수 없어 별도의 방법이 필요한다.

---

## References

Gelfand, A. E., Diggle, P., Fuentes, M., & Guttorp, P. (Eds.). (2010). *Handbook of Spatial Statistics*. CRC Press. Chapter 5.
