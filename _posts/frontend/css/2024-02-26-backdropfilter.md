---
title: "backdrop filter"
categories:
  - study
tags:
  - html
  - css
  - javascript
  - typescript
---

# backdrop filter
- "뒤" 요소를 반투명 블러 처리할 때 사용한다.
- 블러 뿐만 아니라, 인버트 등의 여러 가지 효과가 있음.
- 적용 요소에 반투명 처리를 해야 적용된다.

```css
.container {
  /* rgba를 통해 투명도를 지정해야 함. */
  background-color: rgba(255, 0, 0, .5);
  backdrop-filter: blur(10px);
}
```