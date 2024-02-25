---
title: "node와 element"
categories:
  - study
tags:
  - html
  - css
  - javascript
  - typescript
---

# Node와 Element

- node는 html 요소, 주석, 텍스트 등을 포함.
- element는 html 요소를 지칭.
- 따라서 element는 node의 하위 개념.
- DOM API 확인시 node에 해당하는 개념인지 element에 해당하는 개념인지 잘 확인해야 함.

```javascript
console.log(Node) // Node
console.log(Element.__proto__) //  Node
```