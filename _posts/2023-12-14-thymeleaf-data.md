---
title: "thymeleaf 데이터 사용하기"
categories:
  - study
tags:
  - Java
  - SpringBoot
  - Spring
---

# 데이터 사용하기
- th:text는 기본적으로 escape를 지원.
- 의도적으로 unescape를 하려면 th:utext, [(${data})] 를 쓴다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1> 컨텐츠에 데이터 출력하기</h1>
<ul>
    <li>th:text 사용 <span th:text="${data}"></span></li>
    <li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>
</ul>
</body>
</html>
```

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

    @GetMapping("text-basic")
    public String textBasic(Model model) {
        model.addAttribute("data", "Hello <b>Spring!</b>");
        return "basic/text-basic";
    }
```

