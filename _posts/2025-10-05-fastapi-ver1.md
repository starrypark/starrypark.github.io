---
title: "FastAPI (1): 라우팅과 엔드포인트 기초"
author: starrypark
date: 2025-10-05 00:00:00 +0900
categories: [Computer Science, Backend, FastAPI]
tags: [FastAPI, Python, REST API, HTTP, Backend, Web]
pin: false
math: true
mermaid: true
---

## 개요

이번 포스팅에서는 FastAPI를 쓰기 전에 알아야 할 HTTP 기초부터, 실제로 엔드포인트를 만드는 방법까지 순서대로 짚어봅니다.


## FastAPI

FastAPI는 Python으로 웹 API를 만드는 프레임워크입니다. Flask, Django와 비슷한 역할인데, 몇 가지 차별점이 있습니다.

| 항목 | Flask | FastAPI |
|---|---|---|
| **속도** | 동기 기반 | 비동기(async) 기반, 훨씬 빠름 |
| **타입 힌팅** | 선택 사항 | 필수 — Pydantic으로 자동 검증 |
| **API 문서** | 별도 작업 필요 | `/docs`에서 자동 생성 |
| **학습 난이도** | 낮음 | 약간 높지만 구조가 명확 |

FastAPI는 내부적으로 **Starlette**(웹 프레임워크)과 **Pydantic**(데이터 검증)을 기반으로 만들어져 있습니다. 이 두 라이브러리 덕분에 성능과 안전성이 모두 확보됩니다.

***

## HTTP 기초: 요청과 응답

FastAPI는 HTTP 위에서 동작하므로, HTTP의 기본 구조를 먼저 알아야 합니다.

클라이언트(브라우저, 앱)가 서버에 **요청(Request)**을 보내면, 서버가 **응답(Response)**을 돌려줍니다. 이때 요청의 종류를 **HTTP Method**로 구분합니다.

| Method | 용도 | 예시 |
|---|---|---|
| **GET** | 데이터 조회 | 게시글 목록 불러오기 |
| **POST** | 데이터 생성/전송 | 로그인, 파일 업로드 |
| **PUT** | 데이터 전체 수정 | 회원정보 수정 |
| **DELETE** | 데이터 삭제 | 게시글 삭제 |

응답에는 항상 **Status Code**가 포함됩니다. 자주 쓰이는 것만 알아두면 됩니다.

- `200 OK` — 성공
- `400 Bad Request` — 클라이언트가 잘못된 요청을 보냄
- `404 Not Found` — 요청한 리소스가 없음
- `500 Internal Server Error` — 서버 내부 오류

***

## FastAPI 설치 및 기본 구조

```bash
pip install fastapi uvicorn
```

가장 단순한 FastAPI 앱은 이렇게 생겼습니다:

```python
from fastapi import FastAPI

app = FastAPI(title="나의 첫 API")

@app.get("/")
def home():
    return {"message": "안녕하세요!"}
```

실행은 이렇게 합니다:

```bash
uvicorn app:app --reload
```

여기서 `uvicorn`은 FastAPI를 실제로 서빙하는 **ASGI 서버**입니다. `--reload`는 코드가 바뀔 때 서버를 자동으로 재시작해주는 개발 편의 옵션입니다.

***

## 라우팅과 Decorator

FastAPI에서 URL 경로와 함수를 연결하는 방식을 **라우팅(Routing)**이라고 합니다. `@app.get(...)`, `@app.post(...)` 같은 **Decorator**가 그 역할을 합니다.

```python
@app.get("/health")
def health_check():
    return {"status": "ok"}

@app.post("/chat")
def chat():
    return {"answer": "안녕하세요!"}
```

`@app.get("/health")`는 "누군가 GET 방식으로 `/health`에 접속하면 이 함수를 실행해라"는 의미입니다. 아주 직관적인 구조입니다.

***

## 파라미터 받는 3가지 방법

클라이언트로부터 데이터를 받는 방법은 크게 세 가지입니다.

### Path Parameter — URL 경로 안에 변수

```python
@app.get("/user/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

`/user/42`로 접속하면 `user_id`에 `42`가 들어옵니다. URL 자체가 리소스를 특정할 때 씁니다.

### Query Parameter — URL 뒤에 붙는 `?key=value`

```python
@app.get("/search")
def search(keyword: str, page: int = 1):
    return {"keyword": keyword, "page": page}
```

`/search?keyword=FastAPI&page=2`처럼 씁니다. `page=1`처럼 기본값을 지정하면 생략 가능한 선택 파라미터가 됩니다.

### Form Data — HTML form으로 전송

```python
from fastapi import Form

@app.post("/login")
def login(username: str = Form(...), password: str = Form(...)):
    return {"username": username}
```

`Form(...)`의 `...`은 "반드시 있어야 한다"는 의미입니다. 파일 업로드나 로그인 폼처럼 HTML form 태그로 데이터를 보낼 때 씁니다.

***

## 자동 생성되는 API 문서

FastAPI의 가장 편리한 기능 중 하나는 **Swagger UI**가 자동으로 생성된다는 점입니다. 서버를 실행한 후 `http://localhost:8000/docs`에 접속하면 모든 엔드포인트를 브라우저에서 바로 테스트할 수 있습니다.

별도의 Postman이나 curl 없이도 API 동작을 확인할 수 있어서, 개발 초기에 굉장히 편리합니다.

***

## 정리

FastAPI에서 엔드포인트를 만드는 흐름을 정리하면 이렇습니다.

1. `FastAPI()` 인스턴스를 만든다
2. `@app.get()` / `@app.post()` decorator로 경로를 연결한다
3. 함수 인자로 Path / Query / Form 파라미터를 선언한다
4. `return`으로 dict를 돌려주면 JSON 응답이 된다

다음 포스팅에서는 FastAPI가 왜 빠른지 — **async/await**와 **파일 업로드 처리**를 다룹니다.

***

## References

- FastAPI 공식 문서: <https://fastapi.tiangolo.com/>
- FastAPI GitHub: <https://github.com/fastapi/fastapi>