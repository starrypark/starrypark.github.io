---
title: "FastAPI (2): async/await와 파일 업로드 처리"
author: starrypark
date: 2025-10-12 00:00:00 +0900
categories: [Computer Science, Backend, FastAPI]
tags: [FastAPI, Python, async, await, UploadFile, HTTPException, Backend]
pin: false
math: true
mermaid: true
---

지난 포스팅에서 FastAPI의 기본 라우팅을 다뤘습니다. 이번엔 FastAPI가 "빠르다"고 불리는 진짜 이유 — **비동기 처리**를 먼저 이해하고, 실제 서비스에서 자주 쓰이는 **파일 업로드**와 **에러 핸들링**까지 살펴봅니다.

***

## 동기 vs 비동기

카페를 예로 들어보겠습니다.

**동기 방식**: 손님 한 명의 커피가 완성될 때까지 다른 손님을 아예 받지 않습니다. 한 명씩 처리하니 대기줄이 쌓입니다.

**비동기 방식**: 손님에게 커피를 주문받고, 커피가 추출되는 동안 다음 손님을 받습니다. 완성되면 해당 손님을 호출합니다.

웹 서버에서도 마찬가지입니다. 파일을 읽거나 데이터베이스를 조회하는 동안 다른 요청을 처리할 수 있다면, 같은 서버로 훨씬 많은 요청을 동시에 처리할 수 있습니다. 이게 비동기의 핵심입니다.

***

## async / await 문법

Python에서 비동기 함수는 `async def`로 선언하고, 비동기 작업을 기다릴 때 `await`를 붙입니다.

```python
# 동기 함수
def read_file():
    data = file.read()  # 읽는 동안 멈춤
    return data

# 비동기 함수
async def read_file():
    data = await file.read()  # 읽는 동안 다른 일 가능
    return data
```

FastAPI에서는 `async def`와 일반 `def` 모두 사용할 수 있습니다. 다만 파일 읽기, 외부 API 호출처럼 **기다리는 시간이 있는 작업**은 `async def` + `await`를 써야 비동기의 이점을 제대로 누릴 수 있습니다.

간단한 계산만 하는 함수라면 그냥 `def`를 써도 무방합니다.

***

## 파일 업로드: UploadFile

FastAPI에서 파일을 받을 때는 `UploadFile` 타입을 씁니다.

```python
from fastapi import FastAPI, UploadFile, File, HTTPException

app = FastAPI()

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    # 파일 내용 읽기
    data = await file.read()

    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(data),
    }
```

`UploadFile`이 가진 주요 속성은 이렇습니다:

| 속성 | 설명 | 예시 |
|---|---|---|
| `file.filename` | 업로드된 파일명 | `"report.pdf"` |
| `file.content_type` | 파일 종류 (MIME Type) | `"application/pdf"` |
| `await file.read()` | 파일 내용을 bytes로 읽기 | `b"%PDF-1.4..."` |

***

## MIME Type

**MIME Type**은 파일의 종류를 문자열로 표현하는 방식입니다. 브라우저와 서버가 "이 파일이 뭔지"를 서로 알아볼 수 있게 해줍니다.

자주 보게 되는 MIME Type은 이렇습니다:

| 파일 종류 | MIME Type |
|---|---|
| PDF | `application/pdf` |
| CSV | `text/csv` |
| PNG 이미지 | `image/png` |
| JPEG 이미지 | `image/jpeg` |
| JSON | `application/json` |

업로드 API에서는 허용할 파일 종류를 MIME Type으로 검사하는 패턴을 자주 씁니다:

```python
allowed = {"application/pdf", "text/csv", "image/png", "image/jpeg"}

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    if file.content_type not in allowed:
        raise HTTPException(
            status_code=400,
            detail=f"허용되지 않는 파일 형식: {file.content_type}"
        )
    data = await file.read()
    return {"filename": file.filename, "size": len(data)}
```

***

## 에러 핸들링: HTTPException

서버에서 뭔가 잘못됐을 때, 클라이언트에게 적절한 에러 메시지를 돌려줘야 합니다. FastAPI에서는 `HTTPException`을 씁니다.

```python
from fastapi import HTTPException

@app.get("/user/{user_id}")
def get_user(user_id: int):
    if user_id <= 0:
        raise HTTPException(
            status_code=400,
            detail="user_id는 양수여야 합니다."
        )
    return {"user_id": user_id}
```

`raise HTTPException(...)`을 호출하면 FastAPI가 자동으로 적절한 HTTP 에러 응답을 만들어 돌려줍니다. `status_code`에는 HTTP 상태 코드를, `detail`에는 에러 메시지를 넣으면 됩니다.

***

## 파일 파싱 실전 예시

PDF와 CSV를 받아서 내용을 추출하는 패턴입니다. 실제 프로젝트에서 굉장히 자주 쓰입니다.

```python
import io
import pandas as pd
from pypdf import PdfReader

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    data = await file.read()

    # PDF 파싱
    if file.content_type == "application/pdf":
        try:
            reader = PdfReader(io.BytesIO(data))
            text = "\n".join(p.extract_text() or "" for p in reader.pages)
        except Exception as e:
            raise HTTPException(status_code=400, detail=f"PDF 파싱 실패: {e}")
        return {"text": text[:500]}  # 앞 500자만 미리보기

    # CSV 파싱
    elif file.content_type == "text/csv":
        try:
            df = pd.read_csv(io.BytesIO(data))
        except Exception as e:
            raise HTTPException(status_code=400, detail=f"CSV 파싱 실패: {e}")
        return {"columns": df.columns.tolist(), "rows": len(df)}

    else:
        raise HTTPException(status_code=400, detail="지원하지 않는 형식입니다.")
```

`io.BytesIO(data)`는 bytes를 파일 객체처럼 다룰 수 있게 해주는 Python 내장 도구입니다. 실제 파일을 디스크에 저장하지 않고 메모리에서 바로 처리할 수 있어서 효율적입니다.

***

## 정리

이번 포스팅의 핵심만 추리면 이렇습니다.

- **async def + await** — 기다리는 작업(파일 읽기, DB 조회)을 비동기로 처리해 서버 성능을 높인다
- **UploadFile** — FastAPI에서 파일을 받을 때 쓰는 타입; `file.filename`, `file.content_type`, `await file.read()` 세 가지가 핵심
- **MIME Type** — 파일 종류를 문자열로 표현; 허용 파일 검사에 씀
- **HTTPException** — 에러 발생 시 적절한 HTTP 응답을 돌려주는 방법

다음 포스팅에서는 **CORS**, **Middleware**, 그리고 실제 서버 구조를 갖추는 방법을 다룹니다.

***

## References

- FastAPI 공식 문서 — File Uploads: <https://fastapi.tiangolo.com/tutorial/request-files/>
- FastAPI 공식 문서 — Handling Errors: <https://fastapi.tiangolo.com/tutorial/handling-errors/>
- Python asyncio 공식 문서: <https://docs.python.org/3/library/asyncio.html>
