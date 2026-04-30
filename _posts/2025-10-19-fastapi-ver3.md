---
title: "FastAPI (3): CORS, Middleware, 그리고 서버 구조"

author: starrypark
date: 2025-10-19 00:00:00 +0900
categories: [Computer Science, Backend, FastAPI]
tags: [FastAPI, Python, CORS, Middleware, Jinja2, Session, Backend]
pin: false
math: true
mermaid: true
---

지금까지 엔드포인트를 만들고, 파일을 받아 처리하는 방법을 다뤘습니다. 이번엔 실제 서비스를 만드는 과정에서 마주치는 문제들을 다룹니다 — **CORS**, **Middleware**, **정적 파일 서빙**, **템플릿 렌더링**, **세션 관리**입니다.

---

## CORS

웹 브라우저에는 **Same-Origin Policy(동일 출처 정책)**라는 보안 규칙이 있습니다. 쉽게 말해, "다른 주소에서 온 요청은 기본적으로 막겠다"는 겁니다.

여기서 **Origin**은 `프로토콜 + 도메인 + 포트`의 조합입니다. 예를 들어:

| URL | Origin |
|---|---|
| `http://localhost:3000` | `http://localhost:3000` |
| `http://localhost:8000` | `http://localhost:8000` |
| `https://myapp.shinyapps.io` | `https://myapp.shinyapps.io` |

위 세 가지는 모두 **다른 Origin**입니다. 그래서 `localhost:3000`에서 돌아가는 프론트엔드가 `localhost:8000`의 FastAPI 서버에 요청을 보내면, 브라우저가 기본적으로 막아버립니다.

이걸 허용해주는 게 **CORS (Cross-Origin Resource Sharing)**입니다. 서버가 "이 Origin에서 오는 요청은 허용한다"고 명시해주면 브라우저가 통과시킵니다.

---

## Middleware란

**Middleware**는 요청이 엔드포인트 함수에 도달하기 전, 또는 응답이 클라이언트에 전달되기 전에 **끼어드는 레이어**입니다. 로깅, 인증, CORS 처리 등이 대표적인 활용 사례입니다.

```
클라이언트 → [Middleware] → 엔드포인트 함수 → [Middleware] → 클라이언트
```

FastAPI에서 Middleware를 추가하는 방법은 `add_middleware()`입니다.

---

## CORSMiddleware 설정

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "https://myapp.shinyapps.io",
    ],
    allow_credentials=False,
    allow_methods=["GET", "POST", "OPTIONS"],
    allow_headers=["*"],
)
```

각 옵션의 의미는 이렇습니다:

| 옵션 | 설명 |
|---|---|
| `allow_origins` | 허용할 Origin 목록. `["*"]`이면 전체 허용 (보안상 비권장) |
| `allow_credentials` | 쿠키/인증 정보 포함 여부 |
| `allow_methods` | 허용할 HTTP Method 목록 |
| `allow_headers` | 허용할 요청 헤더. `["*"]`이면 전부 허용 |

프로덕션 환경에서는 `allow_origins=["*"]`보다는 실제 Origin을 명시하는 게 안전합니다.

---

## Static Files 서빙

CSS, JavaScript, 이미지 같은 **정적 파일**을 FastAPI에서 직접 서빙할 수 있습니다.

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

프로젝트 폴더 구조가 이렇다면:

```
project/
├── app.py
└── static/
    ├── style.css
    └── script.js
```

`http://localhost:8000/static/style.css`로 CSS 파일에 접근할 수 있게 됩니다. `app.mount()`는 특정 경로를 특정 디렉토리에 연결해주는 역할입니다.

---

## Jinja2 템플릿으로 HTML 렌더링

FastAPI는 기본적으로 JSON API 서버로 쓰이지만, HTML 페이지를 직접 렌더링할 수도 있습니다. 이때 **Jinja2** 템플릿 엔진을 씁니다.

```python
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse
from fastapi import Request

templates = Jinja2Templates(directory="templates")

@app.get("/", response_class=HTMLResponse)
async def home(request: Request):
    return templates.TemplateResponse(
        "index.html",
        {"request": request, "title": "나의 FastAPI 앱"}
    )
```

`templates/index.html` 파일 안에서 Python 변수를 이렇게 씁니다:

```html
<!DOCTYPE html>
<html>
<head><title>{{ title }}</title></head>
<body>
  <h1>환영합니다, {{ title }}!</h1>
</body>
</html>
```

`{{ title }}`처럼 이중 중괄호로 변수를 렌더링합니다. 간단한 관리 페이지나 대시보드를 FastAPI 안에 포함할 때 유용합니다.

---

## 세션 관리: thread_id 패턴

FastAPI 서버는 기본적으로 **Stateless**입니다. 즉, 요청이 끝나면 서버는 이전 요청을 기억하지 않습니다. 대화 맥락이나 사용자별 상태를 유지하려면 별도의 방법이 필요합니다.

가장 단순한 방법은 **클라이언트가 고유 ID를 들고 다니는 것**입니다.

```python
from uuid import uuid4
from typing import Dict

# 서버 메모리에 임시 저장 (In-memory Store)
_SESSION_STORE: Dict[str, dict] = {}

@app.post("/start")
def start_session():
    session_id = str(uuid4())  # 고유 ID 생성
    _SESSION_STORE[session_id] = {"history": []}
    return {"session_id": session_id}

@app.post("/chat")
def chat(query: str = Form(...), session_id: str = Form(...)):
    session = _SESSION_STORE.get(session_id)
    if not session:
        raise HTTPException(status_code=400, detail="유효하지 않은 세션입니다.")

    session["history"].append(query)
    return {"history_length": len(session["history"])}
```

`uuid4()`는 전 세계에서 중복될 확률이 거의 없는 고유 문자열을 만들어줍니다. 클라이언트는 이 `session_id`를 보관했다가 매 요청에 함께 보냅니다.

단, `_SESSION_STORE`는 서버 메모리에만 저장되기 때문에 서버를 재시작하면 사라집니다. 데이터를 영구 보존하려면 Redis나 SQLite 같은 외부 저장소를 써야 합니다.

---

## 전체 서버 구조 정리

지금까지 다룬 내용을 합치면 실제 서비스에 가까운 구조가 됩니다.

```python
from fastapi import FastAPI, Request, Form, UploadFile, File, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse, JSONResponse
from uuid import uuid4
from typing import Dict

app = FastAPI(title="나의 서비스")

# CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)

# 정적 파일 및 템플릿
app.mount("/static", StaticFiles(directory="static"), name="static")
templates = Jinja2Templates(directory="templates")

# 세션 저장소
_STORE: Dict[str, dict] = {}

@app.get("/", response_class=HTMLResponse)
async def home(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    data = await file.read()
    return {"filename": file.filename, "size": len(data)}

@app.post("/chat")
async def chat(query: str = Form(...), session_id: str = Form("default")):
    return JSONResponse({"answer": f"'{query}'에 대한 답변입니다."})
```

---

## 정리

3편에 걸쳐 FastAPI의 핵심을 다뤘습니다. 흐름을 정리하면 이렇습니다.

- **1편**: HTTP 기초, 라우팅, 파라미터 (Path / Query / Form)
- **2편**: async/await, 파일 업로드, MIME Type, HTTPException
- **3편**: CORS, Middleware, Static Files, Jinja2 템플릿, 세션 관리

FastAPI는 이 구성 요소들이 명확하게 분리되어 있어서, 코드가 복잡해져도 구조를 파악하기 비교적 쉽습니다. 직접 작은 프로젝트를 만들어보는 게 가장 빠른 이해 방법입니다.

---

## References

- FastAPI 공식 문서 — CORS: <https://fastapi.tiangolo.com/tutorial/cors/>
- FastAPI 공식 문서 — Static Files: <https://fastapi.tiangolo.com/tutorial/static-files/>
- FastAPI 공식 문서 — Templates: <https://fastapi.tiangolo.com/advanced/templates/>
- Jinja2 공식 문서: <https://jinja.palletsprojects.com/>
