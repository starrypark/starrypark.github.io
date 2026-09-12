# 글 분류

## 공부 주제

카테고리는 두 단계로 지정합니다. 책장은 `shelf: study`로 명시합니다.

- 통계: 공간통계, 베이지안 추론, 시계열, 생존분석, 불확실성 정량화
- 개발: 백엔드, 개발환경·배포
- AI: LLM 애플리케이션

예: `categories: [통계, 시계열]`. 다른 분야와 연결되는 개념은 태그로 표현합니다.
영문 태그는 소문자로 통일하며 핵심 키워드 3~5개를 권장합니다.
주제별 진입점은 `_data/study-topics.yml`에서 관리합니다.

## 연재

`_data/series.yml`에 연재 ID, 제목, 설명을 등록합니다.
글에는 `series: fastapi`, `series_order: 1`처럼 ID와 읽는 순서를 지정합니다.
순서는 같은 연재 안에서 중복 없는 정수로 작성합니다.

현재 연재: `spatial`, `uq`, `fastapi`, `langgraph`.
`/series/`에 전체 목차가 표시되고, 글 상단에는 현재 위치가,
하단에는 연재 이전·다음 링크가 표시됩니다. 기존 전체 글 시간순 이동도 유지됩니다.
글 본문과 `/posts/` 주소는 분류 변경의 영향을 받지 않습니다.

## 대표 이미지와 소개

각 책장의 `hero_image`, `hero_image_alt`로 이미지와 대체 텍스트를 지정합니다.
이미지는 `assets/img/shelves/`에 있고 생성 프롬프트는 `docs/shelf-images.md`에 기록합니다.
홈의 기존 일러스트는 유지합니다. 소개는 `_tabs/about.md`에서 수정합니다.

## 노출 규칙

기존 글은 별도 변경 없이 `공부`에 표시됩니다. 글의 YAML front matter에
`shelf`를 지정하면 홈의 최신 글과 해당 책장에 함께 표시됩니다.

```yaml
title: 책을 읽고 떠오른 생각
shelf: reading
categories: [독서]
```

- `study`: 공부 (생략 시 기본값)
- `reading`: 독서
- `daily`: 일상

기존 `categories`, `tags`는 세부 주제 탐색에 계속 사용합니다.
`pin: true`인 공개 글 중 최신 두 편이 ‘다시 꺼내 읽는 글’에 표시됩니다.
고정 글이 없으면 오래된 공개 글 두 편을 보여줍니다.
`hidden: true`인 글은 홈과 책장, 추천 영역에서 제외합니다.
