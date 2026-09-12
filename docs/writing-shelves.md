# 글 분류

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
