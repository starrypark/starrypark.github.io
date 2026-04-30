---
title: "LangGraph에서 기억력 구현하기: MemorySaver, Checkpointing, 그리고 대화 히스토리"
description: LangGraph가 상태를 저장하고 이어받는 방식을 밑바닥부터 정리한 노트. MemorySaver, 히스토리 누적 패턴, thread_id 세션 분리, 영구 저장소 교체까지 다룹니다.
author: starrypark
date: 2026-04-30 09:00:00 +0900
categories: [AI, LangGraph]
tags: [langgraph, llm, memory, checkpointing, chatbot, python, langchain]
math: true
mermaid: true
---

## 개요

LangGraph로 대화형 AI를 만들다 보면 가장 먼저 맞닥뜨리는 벽이 있습니다.

> "왜 아까 한 얘기를 기억 못 하지?"

이 글에서는 LangGraph가 상태(state)를 어떻게 저장하고, 다음 호출에 어떻게 이어주는지를 밑바닥부터 살펴봅니다.

---

## 1. 왜 LLM은 기본적으로 기억이 없는가

LLM 자체는 *stateless*입니다. `model.invoke("어제 내가 뭐 먹었다고 했지?")`를 호출하면, 모델은 이 메시지 하나만 보고 답합니다. 앞서 오간 대화는 완전히 없던 일이 됩니다.

흔히 쓰는 해결책은 **프롬프트에 대화 기록을 직접 붙여 넣는** 것입니다. LangGraph는 이 패턴을 *그래프 상태(graph state)* 와 *체크포인터(checkpointer)* 로 체계화합니다.

---

## 2. LangGraph 상태 기계 한 줄 요약

LangGraph는 노드(node)와 엣지(edge)로 이루어진 **유향 그래프(DAG + 조건 분기)**입니다. 각 노드는 `state` 딕셔너리를 받아 수정된 딕셔너리를 반환하고, 그래프는 그 결과를 다음 노드로 전달합니다.

```
START → node_A → node_B → node_C → END
           ↑                 │
           └─────────────────┘  (조건에 따라 루프 가능)
```

한 번의 `graph.invoke(input, config)` 호출이 끝나면, **최종 상태 전체가 체크포인터에 저장됩니다.** 다음 호출 때 같은 `thread_id`를 넘기면, 저장된 상태를 불러와 이어서 실행합니다.

---

## 3. MemorySaver: 인메모리 체크포인터

LangGraph가 기본 제공하는 가장 단순한 체크포인터가 `MemorySaver`입니다.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Optional

class MyState(TypedDict):
    user_input: str
    answer:     Optional[str]

def chat_node(state: MyState) -> MyState:
    # 실제로는 LLM 호출
    return {**state, "answer": f"You said: {state['user_input']}"}

builder = StateGraph(MyState)
builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)

checkpointer = MemorySaver()
graph = builder.compile(checkpointer=checkpointer)
```

호출할 때 `config`에 `thread_id`를 지정하면, 그 스레드의 상태가 메모리에 누적됩니다.

```python
cfg = {"configurable": {"thread_id": "session-001"}}
result1 = graph.invoke({"user_input": "안녕"}, cfg)
result2 = graph.invoke({"user_input": "방금 내가 뭐라고 했지?"}, cfg)
```

다만 `MemorySaver`는 프로세스가 살아 있는 동안만 유효합니다. 서버 재시작 시 초기화되므로, 운영 환경에서는 `SqliteSaver` 또는 `PostgresSaver`를 사용합니다.

---

## 4. 대화 히스토리를 상태에 누적하기

체크포인터가 상태를 이어준다고 해도, LLM에게 "이전 대화"를 보여주려면 **히스토리를 상태 필드에 직접 쌓아야** 합니다.

### State 정의

```python
from typing import TypedDict, Optional, List, Dict

class ChatState(TypedDict):
    user_input:   str
    answer:       Optional[str]
    chat_history: Optional[List[Dict[str, str]]]  # [{"role": "user", "content": "..."}, ...]
```

### 히스토리를 LLM에 넘기기

LangChain의 메시지 객체(`SystemMessage`, `HumanMessage`, `AIMessage`)로 변환하면 어떤 모델이든 일관되게 처리할 수 있습니다.

```python
from langchain_core.messages import SystemMessage, HumanMessage, AIMessage
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

def chat_node(state: ChatState) -> ChatState:
    history = state.get("chat_history") or []
    messages = [SystemMessage(content="당신은 친절한 AI 어시스턴트입니다.")]

    # 최근 10턴만 포함 (컨텍스트 길이 관리)
    for msg in history[-10:]:
        if msg["role"] == "user":
            messages.append(HumanMessage(content=msg["content"]))
        else:
            messages.append(AIMessage(content=msg["content"]))

    messages.append(HumanMessage(content=state["user_input"]))
    answer = llm.invoke(messages).content
    return {**state, "answer": answer}
```

### update_history 노드: 히스토리 누적의 핵심

대화가 끝날 때마다 `user_input`과 `answer`를 히스토리에 추가하는 **터미널 노드**를 두는 것이 깔끔한 패턴입니다.

```python
def update_history(state: ChatState) -> ChatState:
    history = list(state.get("chat_history") or [])
    if state.get("user_input"):
        history.append({"role": "user",      "content": state["user_input"]})
    if state.get("answer"):
        history.append({"role": "assistant", "content": state["answer"]})

    # 히스토리가 너무 길어지지 않도록 최근 20개만 유지
    if len(history) > 20:
        history = history[-20:]
    return {**state, "chat_history": history}
```

그래프 구조는 아래처럼 `update_history → END`를 공통 종착점으로 만듭니다.

```python
builder.add_edge("chat",           "update_history")
builder.add_edge("update_history", END)
```

---

## 5. 전체 흐름 한눈에 보기

```mermaid
flowchart LR
    A([START]) --> B[merge_context]
    B --> C[classify_intent]
    C -->|chat| D[general_chat]
    C -->|calc| E[run_calc]
    D --> F[update_history]
    E --> F
    F --> G([END])
```

1. `START` → 이전 체크포인트(state)가 자동으로 로드됩니다.
2. 각 노드는 state를 수정해 다음 노드로 전달합니다.
3. `update_history`가 `chat_history`를 누적한 뒤 반환합니다.
4. `END` → 체크포인터가 최종 state를 저장합니다.
5. 다음 `invoke` 호출 시 4번에서 저장한 state를 이어받습니다.

---

## 6. thread_id: 대화 세션의 식별자

같은 그래프를 여러 사용자가 동시에 쓸 때, `thread_id`가 각 세션을 분리합니다.

```python
# 사용자 A
cfg_a = {"configurable": {"thread_id": "user-alice"}}
graph.invoke({"user_input": "내 이름은 Alice야"}, cfg_a)

# 사용자 B (A의 기록과 완전히 분리됨)
cfg_b = {"configurable": {"thread_id": "user-bob"}}
graph.invoke({"user_input": "내 이름은 뭐야?"}, cfg_b)
# → "죄송합니다, 말씀해 주시지 않아서 알 수 없어요."
```

웹 앱이라면 세션 ID나 사용자 ID를 `thread_id`로 쓰면 됩니다.

---

## 7. 영구 저장소로 교체하기

`MemorySaver`를 `SqliteSaver`로 교체하면 재시작 후에도 대화가 유지됩니다.

```python
from langgraph.checkpoint.sqlite import SqliteSaver

with SqliteSaver.from_conn_string("checkpoints.db") as checkpointer:
    graph = builder.compile(checkpointer=checkpointer)
```

PostgreSQL을 쓰려면 `langgraph-checkpoint-postgres` 패키지를 설치하고 아래처럼 연결합니다.

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:pass@localhost:5432/mydb"
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    graph = builder.compile(checkpointer=checkpointer)
```

인터페이스는 `MemorySaver`와 완전히 동일하므로, 코드 변경 없이 교체가 가능합니다.

---

## 8. 컨텍스트 길이 관리 전략

히스토리를 무한정 쌓으면 토큰 비용이 급증하고 컨텍스트 한계에 부딪힙니다. 몇 가지 전략을 조합해서 씁니다.

| 전략 | 설명 |
|---|---|
| **슬라이딩 윈도우** | 최근 N 턴만 유지 (`history[-10:]`) |
| **요약(Summarization)** | 오래된 대화를 LLM으로 요약해 압축 |
| **중요도 필터링** | 특정 태그(entity, fact)가 붙은 메시지만 보존 |
| **토큰 카운팅** | `tiktoken` 등으로 실시간 토큰 수 체크 후 trim |

슬라이딩 윈도우가 구현이 가장 단순하고 예측 가능해서 프로토타입에 적합합니다.

---

## 9. 자주 겪는 실수

### `chat_history`를 State에 넣었는데 LLM에 안 넘김

State에 필드만 추가하고 노드 안에서 꺼내 쓰지 않으면 당연히 기억이 없습니다. `chat_node` 안에서 `state.get("chat_history")`를 반드시 꺼내야 합니다.

### `update_history`를 일부 경로에서 건너뜀

그래프에 여러 브랜치가 있을 때, 특정 경로가 `update_history`를 거치지 않으면 그 대화는 히스토리에 남지 않습니다. 모든 종착 엣지가 `update_history → END`를 거치도록 설계해야 합니다.

### State 필드를 덮어쓰지 않고 반환

```python
# 잘못된 예: chat_history가 사라짐
return {"answer": answer}

# 올바른 예: 기존 state를 spread한 뒤 수정된 필드만 덮어씀
return {**state, "answer": answer}
```

---

## 10. 정리

LangGraph의 기억력은 마법이 아닙니다. **체크포인터**가 state를 저장·복원하고, **`chat_history` 필드**가 대화 맥락을 누적하며, **`update_history` 노드**가 매 턴마다 이를 갱신합니다. 이 세 가지를 제대로 연결하면, LangGraph 위에서 자연스럽게 대화 맥락을 이어가는 AI를 만들 수 있습니다.

---

> **참고 자료**
> - [LangGraph 공식 문서 — Persistence](https://langchain-ai.github.io/langgraph/concepts/persistence/)
> - [LangChain — Message Types](https://python.langchain.com/docs/concepts/messages/)
> - [LangGraph Checkpointers](https://langchain-ai.github.io/langgraph/reference/checkpoints/)
