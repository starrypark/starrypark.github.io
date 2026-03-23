---
title: Docker에 대한 설명
description: 협업 시 필요한 Docker에 대한 소개
author: starrypark
date: 2025-12-26 11:26:31 +0810
categories: [Computer Science]
tags: [Docker, Image, Container]
pin: true
math: true
mermaid: true
use_math: true
---

# Docker로 “가상 환경 + 버전 일치”를 잡는 법: image / container 개념부터 실전 삽질까지 (Windows+WSL 포함)

> **필수 키워드:** Docker

저는 “로컬에서는 되는데 서버에 올리면 깨지는” 문제를 정말 많이 겪었습니다. 특히 Python API(FastAPI) + R Shiny 조합처럼 언어/의존성이 섞이는 순간, 버전 차이와 시스템 라이브러리 문제는 거의 확정적으로 터집니다.  
그래서 결국 저는 **Docker로 실행 환경 자체를 고정**하는 방향으로 정리했습니다.

이 글은 “Docker가 뭔지”만 설명하지 않습니다. **왜 Docker가 필요했는지, 어떻게 적용했는지, 어디서 터졌고 어떤 방식으로 복구했는지**까지, 실전에서 겪는 흐름 그대로 정리합니다.

---

## 1. 이 기술을 왜 다루게 되었나

제가 Docker를 본격적으로 다루게 된 이유는 단순합니다.

- 로컬에서는 패키지가 잘 깔리는데, 컨테이너 빌드에서는 `EOF`, `ANTICONF` 같은 에러로 죽음
- R 패키지 설치에서 리눅스 시스템 라이브러리(헤더)가 없어서 컴파일 실패
- Windows + Docker Desktop + WSL2 조합에서 빌드가 중간에 뻗으면서 WSL distro가 꼬임

즉, **“버전 불일치 + 실행 환경 차이”**가 원인이었습니다.  
이 문제를 한 번에 해결하려면, 결국 실행 환경 자체를 “동일한 박스”로 고정해야 했고, 그게 Docker였습니다.

---

## 2. 결론부터 말하면: 내가 선택한 방식

제가 최종적으로 선택한 방식은 아래처럼 정리됩니다.

1) **Docker image로 실행 환경을 고정**하고  
2) **container로 실행을 분리**하며  
3) Python API와 R Shiny는 `docker-compose`로 묶되  
4) Shiny에서 API 호출은 `http://api:8000`처럼 **서비스 이름 기반**으로 연결합니다.

그리고 Windows 환경에서는 **WSL 리소스 설정(.wslconfig)**이 사실상 필수였습니다. (그 전에는 `rpc error: EOF`가 반복적으로 발생했습니다.) [R1]

---

## 3. 배경지식: 이것만 알면 글을 따라올 수 있다

여기서는 핵심만 잡습니다. Docker를 처음 보는 분들도 이 파트만 이해하면 뒤가 따라옵니다.

### 3.1 Docker란 무엇인가 (가상환경 vs Docker)

많은 분들이 “Docker = 가상머신(VM)” 정도로 생각하지만, 실제 느낌은 조금 다릅니다.

- **가상머신(VM)**: OS까지 통째로 가상화
- **Docker**: 호스트 OS 커널을 공유하고, 파일시스템/프로세스를 격리해서 실행

그래서 Docker는 대체로 VM보다 가볍고, 실행과 배포가 빠릅니다.

### 3.2 image / container 차이

이걸 정확히 구분하면 이후 모든 것이 쉬워집니다.

- **Docker image**: “레시피/스냅샷” (정해진 환경이 담긴 템플릿)
- **Docker container**: image를 실제로 실행한 “프로세스”

비유로는 이런 느낌입니다.

- image = **게임 설치 파일**
- container = **게임 실행 중인 프로세스**

**버전 일치**를 위해서 중요한 건 image입니다.  
컨테이너가 몇 번 죽어도, image만 안정적이면 다시 띄우면 그만입니다.

### 3.3 왜 버전 일치에서 Docker가 강력한가

버전이 깨지는 이유는 늘 같습니다.

- OS 레벨 라이브러리 다름
- python/R 패키지 버전 다름
- 컴파일러/헤더 유무 다름
- locale/timezone 차이

Docker image에는 이런 것들이 “굳어버린 상태”로 들어갑니다.  
그래서 “로컬만 되는 코드”를 “어디서나 되는 코드”로 바꾸기 좋습니다.
