---
title: 중요도 추출(Importance Sampling)
description: 중요도 추출의 기본적인 개념 정리
author: starrypark
date: 2024-10-01 10:00:20 +0800
categories: [Statistics, Bayesian Analysis]
tags: [Statistics, Importance Sampling]
pin: true
math: true
mermaid: true
use_math: true
---

## Purpose

이 포스팅은 중요도 추출에 대한 내용이다. 베이지안 샘플링이나, 딥러닝 일부 알고리즘에서도 정말 중요하게 쓰이는 알고리즘이니 꼭 알아두자.

Review : 이재용 & 이기재 (2022) 베이즈 데이터 분석 - Chapter 5

Review : 오승상 교수님 강화학습의 수학 강의노트 

## 1. Importance Sampling 개요

- Importance Sampling은 $\mathbb{E}[f]$를 $p(x)$를 잘 모르는 상태에서 계산하기 위해 사용한다.
- 대신, 표본 $\left\{x_n\right\}$은 더 간단한 분포 $q(x)$에서 추출한다. 이 때 $q(x)>0$ if $p(x)>0$이고, corresponding term이 $\frac{p\left(x_n\right)}{q\left(x_n\right)}$의 가중치를 가지며 importance weight라고 한다.
    
    $$
    \begin{aligned}
    \mathbb{E}[f] &= \int f(x) p(x) dx = \int f(x) \frac{p(x)}{q(x)} q(x) dx \\
    &\approx \frac{1}{N} \sum_{n=1}^N \frac{p(x_n)}{q(x_n)} f(x_n), \quad \text{where } x_n \sim q(x)
    \end{aligned}
    $$
    
    이거나
    
    $$
    \begin{aligned}
    \mathbb{E}_{X \sim P}[f(X)] &= \sum P(X) f(X) = \sum Q(X) \frac{P(X)}{Q(X)} f(X) \\
    &= \mathbb{E}_{X \sim Q}\left[\frac{P(X)}{Q(X)} f(X)\right]
    \end{aligned}
    $$
    
    로 계산이 가능하다.
    

## 2. 알고리즘

1. $\theta_1 , \cdots,\theta_m \sim q(\theta)$를 추출한다.
2. 가중치 $w_i = \frac{p(\theta_i)}{q(\theta_i)}$를 추출한다.
3. $\mathbb{E}[f]$를 다음 두 값중 하나로 추정한다.