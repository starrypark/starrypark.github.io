---
title: "LangGraph 입문: 처음 챗봇을 짤 때 반드시 알아야 할 것들"
description: LangGraph의 핵심 구성 요소인 State, Node, Edge부터 Checkpointer, ToolNode까지 — 처음 챗봇을 만들 때 알아야 할 내용을 처음부터 차근차근 정리한 노트.
author: starrypark
date: 2025-12-14 00:00:00 +0900
categories: [AI Engineering, LLM]
tags: [LangGraph, LangChain, Chatbot, Agent, StateGraph, Python, AI]
pin: false
math: true
mermaid: true
---

저는 처음에 Langchain으로 챗봇을 구축했습니다. 근데 막상 챗봇에 조건 분기를 넣거나, 메모리를 붙이거나, 툴 호출 흐름을 제어하려고 하면 한계가 생기기 시작합니다.

그런 점에서 해결책을 찾아보다가, LangGraph를 사용하게 되었습니다. LLM 호출 흐름을 **그래프(Graph)** 구조로 명시적으로 설계할 수 있게 해주는 프레임워크입니다.

---

## LangGraph?

LangGraph는 LangChain 생태계 안에 있는 라이브러리로, LLM 기반 애플리케이션을 **유향 그래프(Directed Graph)** 형태로 설계할 수 있도록 도와줍니다. 핵심 아이디어는 이렇습니다.

> "LLM 호출, 툴 실행, 조건 분기 — 이 모든 걸 노드와 엣지로 표현하자."

기존 LangChain의 Chain 방식은 선형(linear)이라 `A → B → C` 식의 흐름은 잘 동작합니다. 그런데 "조건에 따라 다음 단계가 달라지거나", "루프가 필요한" 경우에는 코드가 꼬이기 시작하죠. LangGraph는 이 문제를 **State Machine** 개념으로 풀어냅니다.

---

## 핵심 구성 요소 3가지

LangGraph 그래프는 딱 세 가지로 이루어집니다.

### State (상태)

State는 그래프가 실행되는 동안 노드들 사이에서 공유되는 **작업 일지** 같은 것입니다. 각 노드는 이 State를 읽고, 자신이 업데이트한 내용을 돌려줍니다.

```python
from typing import Annotated, TypedDict
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
```

여기서 `Annotated[list, add_messages]` 부분이 중요합니다. `add_messages`는 **Reducer 함수**인데요, 새 메시지가 들어왔을 때 기존 리스트를 덮어쓰는 게 아니라 **이어 붙이는** 동작을 정의합니다. State를 선언할 때 각 필드가 "어떻게 합쳐질 것인지"를 함께 명시해주는 방식입니다.

수식으로 쓰면, 시점 $t$에서의 State를 $S\\_t$라 할 때:

$$S_{t+1} = \text{reducer}(S_t, \Delta S_t)$$

$\Delta S\\_t$는 노드가 반환한 업데이트 값이고, reducer가 이를 기존 State에 합칩니다. `add_messages`의 경우엔 $\Delta S\\_t$를 단순히 append하는 방식입니다.

### Node (노드)

Node는 그래프 안에서 **하나의 작업 단위**입니다. Python 함수나 callable 객체면 뭐든 노드가 될 수 있습니다. State를 입력으로 받아서, 업데이트할 State의 일부를 dict로 반환하면 됩니다.

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}
```

노드 함수의 역할을 수식으로 표현하면 이렇습니다:

$$f: S \rightarrow \Delta S$$

State 전체를 받아서, 바뀐 부분만 dict로 돌려주면 됩니다. 전체를 다 반환할 필요가 없다는 점이 편리합니다.

### Edge (엣지)

Edge는 노드 사이의 **흐름**을 정의합니다. "이 노드가 끝나면 다음에 어느 노드로 갈 것인가"를 결정하는 역할입니다. 크게 두 종류가 있습니다.

| 종류 | 설명 | 코드 |
|---|---|---|
| **일반 엣지 (Normal Edge)** | 항상 다음 노드로 고정 이동 | `add_edge("A", "B")` |
| **조건부 엣지 (Conditional Edge)** | 함수 반환값에 따라 다음 노드 결정 | `add_conditional_edges("A", routing_fn)` |

---

## StateGraph 만드는 흐름

LangGraph 챗봇을 만드는 과정은 거의 항상 아래 패턴을 따릅니다.

1. **State 정의** — 그래프 전체에서 공유할 데이터 구조를 TypedDict로 선언
2. **Node 정의** — 각 작업 단계를 함수로 작성
3. **Graph 생성 및 노드 추가** — `StateGraph(State)`를 만들고 `add_node()`로 노드 등록
4. **Edge 연결** — `add_edge()` 또는 `add_conditional_edges()`로 흐름 정의
5. **Compile** — `graph.compile()`로 실행 가능한 객체 생성
6. **실행** — `graph.invoke()` 또는 `graph.stream()`으로 호출

전체 코드로 보면 이렇습니다:

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI

# 1. State 정의
class State(TypedDict):
    messages: Annotated[list, add_messages]

# 2. Node 정의
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}

# 3. Graph 생성
graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)

# 4. Edge 연결
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)

# 5. Compile
graph = graph_builder.compile()

# 6. 실행
result = graph.invoke({"messages": [("user", "안녕! 오늘 날씨 어때?")]})
print(result["messages"][-1].content)
```

`START`와 `END`는 LangGraph가 제공하는 특수 상수입니다. 그래프의 진입점과 종료점을 명시적으로 표시해주는 역할을 합니다.

---

## 조건부 분기: Conditional Edge

챗봇이 단순 응답만 한다면 굳이 LangGraph를 쓸 이유가 크지 않습니다. 진가는 **"상황에 따라 다른 노드로 가야 할 때"** 드러납니다.

예를 들어, LLM이 툴을 호출해야 하는 상황인지 아닌지를 판단해서 분기하는 경우를 보겠습니다:

```python
def routing_function(state: State) -> str:
    last_message = state["messages"][-1]
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tools"
    return END

graph_builder.add_conditional_edges("chatbot", routing_function)
```

`routing_function`은 State를 받아서 **다음 노드의 이름(문자열)**을 반환합니다. LangGraph는 그 반환값을 보고 어디로 갈지 결정하게 됩니다. 조건부 엣지를 쓸 때는 반환값이 반드시 등록된 노드 이름이거나 `END`여야 한다는 점을 주의해야 합니다.

---

## Memory: Checkpointer

기본 그래프는 각 `invoke()` 호출이 독립적입니다. 즉, 이전 대화를 기억하지 못합니다. 이걸 해결하는 게 **Checkpointer**입니다.

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
graph = graph_builder.compile(checkpointer=memory)

config = {"configurable": {"thread_id": "user_001"}}

# 첫 번째 턴
graph.invoke({"messages": [("user", "내 이름은 재민이야")]}, config=config)

# 두 번째 턴 — 이전 대화를 기억
graph.invoke({"messages": [("user", "내 이름이 뭐라고?")]}, config=config)
```

`thread_id`가 핵심입니다. 같은 `thread_id`로 요청하면 이전 State가 복원되어 대화 맥락이 유지됩니다. `MemorySaver`는 인메모리 저장소라 서버 재시작 시 사라집니다. 프로덕션 환경이라면 `SqliteSaver`나 Redis 기반 체크포인터를 쓰는 게 안전합니다.

---

## ToolNode: 툴 호출 붙이기

LLM이 외부 툴을 호출하는 패턴은 LangGraph에서 굉장히 자주 쓰입니다. `ToolNode`를 활용하면 LLM이 결정한 tool call을 자동으로 실행해줍니다.

```python
from langgraph.prebuilt import ToolNode
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """도시 이름을 받아 날씨를 반환합니다."""
    return f"{city}의 날씨는 맑음입니다."

tools = [get_weather]
tool_node = ToolNode(tools)
llm_with_tools = llm.bind_tools(tools)

def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

graph_builder.add_node("chatbot", chatbot)
graph_builder.add_node("tools", tool_node)

graph_builder.add_edge(START, "chatbot")
graph_builder.add_conditional_edges("chatbot", routing_function)
graph_builder.add_edge("tools", "chatbot")  # 툴 실행 후 다시 chatbot으로
graph_builder.add_edge("chatbot", END)
```

여기서 `tools → chatbot` 엣지가 핵심입니다. 툴 실행이 끝나면 다시 chatbot 노드로 돌아와서, LLM이 결과를 해석하도록 흐름을 만드는 것입니다. 이게 바로 **ReAct (Reasoning + Acting) 패턴**의 기본 구조입니다.

---

## 그래프 시각화

LangGraph는 컴파일된 그래프를 시각화하는 기능을 내장하고 있습니다. 복잡한 그래프를 디버깅할 때 특히 유용합니다.

```python
from IPython.display import Image, display

display(Image(graph.get_graph().draw_mermaid_png()))
```

처음 그래프를 설계할 때, 엣지가 의도한 대로 연결됐는지 이걸로 꼭 확인해보길 권합니다. 눈에 보이지 않으면 잘못된 연결을 잡아내기가 생각보다 어렵습니다.

---

## 처음 짤 때 자주 막히는 포인트

LangGraph로 챗봇을 처음 만들 때 흔히 놓치는 부분들을 정리하면 이렇습니다.

- **State에 `add_messages` reducer 붙이기** — 안 붙이면 메시지가 매번 덮어써집니다
- **`START` → 첫 노드 → ... → `END` 연결 확인** — 진입점과 종료점을 명시해야 그래프가 정상 실행됩니다
- **`compile()` 잊지 않기** — `graph_builder`는 설계도일 뿐, `compile()` 후에야 실행 가능한 객체가 됩니다
- **Checkpointer는 `thread_id`로 관리** — 같은 사용자의 대화는 같은 `thread_id`로 묶어야 맥락이 유지됩니다
- **툴 호출 후 다시 chatbot 노드로 돌아오는 엣지** — `tools → chatbot` 엣지가 없으면 툴 결과를 LLM이 처리하지 못합니다

LangGraph는 처음엔 설정할 게 많아 보이지만, 패턴이 손에 익으면 오히려 흐름 제어가 훨씬 명확해집니다. 위 예시 코드를 직접 실행해보는 게 가장 빠른 이해 방법입니다.

---

## References

- LangGraph 공식 문서: <https://langchain-ai.github.io/langgraph/>
- LangChain 한국어 튜토리얼 (wikidocs): <https://wikidocs.net/261576>
- LangGraph를 활용한 챗봇 구축 (wikidocs): <https://wikidocs.net/264614>
- IBM — LangGraph란 무엇인가요?: <https://www.ibm.com/kr-ko/think/topics/langgraph>
