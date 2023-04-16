---
title: "Jackson guide"
categories:
  - implement
tags:
  - Java
  - SpringBoot
  - Spring
---
## Jackson을 쓰는 이유?
* Spring이랑 잘 맞아서 쓴다.
* RestController에서 Object만 리턴해도 Jackson으로 직렬화해준다.

```java

@Getter
class Post {
    String id;
    String title;
    int likeCount;
};

@GetMapping("/posts")
@ResponseBody()
public List<Post> getPosts() {
    List<Post> posts = postService.getPosts();
    return posts;
}
/**
 * [{
 *   "id": "abcdefef",
 *   "title": "Hello",
 *   "likeCount": 5000
 * }]
 * 
 * /

```

## jackson 과 java bean property
* ```자바빈```에는 ```프로퍼티```라는 개념이 있다.
  * ```프로퍼티```는 ```자바빈```이 관리하는 데이터들(흔히 멤버변수라 부르는 것들)이다.
  * 자바빈으로부터 ```프로퍼티```에 접근하기 위해선 메서드를 사용해야 한다. 즉, ```getter```와 ```setter```이다.
  * 이 ```getter```와 ```setter```에는 명명 규칙이 있다. 
    * get[프로퍼티명], set[프로퍼티명]이 되어야 하고, ```camel case``` 형식으로 이름지어야 한다.
* ```jackson```이 자바 객체를 직렬화할 때, **멤버변수가 아니라 프로퍼티를** 사용한다.
  * 즉, **getter에 명시된 프로퍼티와 동일한 이름으로 선언된 객체 내의 변수를 찾아 직렬화한다.** 
  * 다시 말하면, [jackson에서는 getter, setter가 없으면 직렬화/역직렬화를 못한다.](https://bactoria.github.io/2019/08/16/ObjectMapper%EB%8A%94-Property%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B0%BE%EC%9D%84%EA%B9%8C/)

## @JsonProperty
* getter/setter가 없어도 프로퍼티를 멤버변수에 매핑하여 직렬화/역직렬화를 할 수 있는 방법이 있다. ```JsonProperty``` 어노테이션을 사용하는 것이다.
* 개발자는 오히려 멤버변수와 프로퍼티를 같이 생각하는 경우가 많기 때문에, ```JsonProperty``` 어노테이션이 제공된다.
* 멤버변수에 ```JsonProperty```를 붙이면 해당 변수와 프로퍼티가 매핑이 된다.

```java
@Getter
class Post {
    UUID id;
    String title;
    @JsonProperty("likes")
    int likeCount;
};

/**
 * {
 *   "id": "abcdefef",
 *   "title": "Hello",
 *   "likes": 5000  # likeCount가 아니라 likes로 직렬화.
 * }
 */
```

## @JsonAutoDetect
* jsonProperty의 경우 지정한 멤버변수에 대해서만 프로퍼티와 매핑이 된다.
* 객체 내의 모든 멤버변수에 대해서 프로퍼티 - 멤버변수 매핑을 하고 싶다면, ```JsonAutoDetect``` 어노테이션을 사용한다.

```java
@JsonAutoDetect(JsonAutoDetect.visibility.ANY)
class Post {
    UUID id;
    String title;
    int likeCount;
};

/**
 * {
 *   "id": "abcdefef",
 *   "title": "Hello",
 *   "likeCount": 5000
 * }
 */
```

### @JsonAutoDetect visibility
* ```JsonAutoDetect``` 어노테이션은 visibility라는 옵션을 제공한다.
* 멤버변수의 접근제한자에 따라 직렬화/역직렬화에 포함시킬지 말지를 결정한다는 의미이다.
* 특별한 일이 없다면, ```JsonAutoDetect.Visibility.Any``` 를 사용하여 모든 접근제한자에 대해 직렬화/역직렬화에 포함시킨다.

## @JsonInclude
* ```JsonInclude``` 어노테이션은 ```JsonAutoDetect visibility``` 와 비슷하면서 좀 더 확장한 개념이다.
* ```NON_NULL```, ```NON_EMPTY``` 등 각 멤버변수의 상태에 따라 직렬화/역직렬화에 포함시킬지 말지를 결정한다.
* 제공하는 옵션은 이 [링크](https://fasterxml.github.io/jackson-annotations/javadoc/2.9/com/fasterxml/jackson/annotation/JsonInclude.Include.html#NON_EMPTY:~:text=Enum%20Constant%20Detail)에 있다.


```java
@JsonAutoDetect(JsonAutoDetect.visibility.ANY)
@JsonInclude(JsonInclude.Include.NON_NULL)
class Post {
    UUID id = null;
    String title = null;
    @JsonProperty("likes")
    int likeCount = 0;
};

/**
 * {
 *   "likes": 0  # null이 아니어서 직렬화에 포함.
 * }
 */
```

## @JsonValue

* class를 직렬화할 때 key-value 형태가 아니라 value로서만 전달하고 싶을 때 사용한다.
    

```java
@JsonAutoDetect
public class SwitchPort {
   long index;
   VlanId vlanId;
}

public class VlanId {
    long id;
    @JsonValue
    public long getId() {
        return this.id;
    }
}
```

```json
/**
* jsonValue 적용시.
*/
{
    "index": int,
    "vlanId": 1
}

/**
* jsonValue 미적용시.
*/
{
    "index": int
    "vlanId" : {
        "id" : int
    }
}
```

## 참고
- [자바빈과 프로퍼티](https://lee-jung-hoon.github.io/2018/09/18/programming-spring-study-004/)
- [jackson 직렬화/역직렬화와 getter/setter](https://bactoria.github.io/2019/08/16/ObjectMapper%EB%8A%94-Property%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B0%BE%EC%9D%84%EA%B9%8C/)
- [JsonInclude](https://fasterxml.github.io/jackson-annotations/javadoc/2.9/com/fasterxml/jackson/annotation/JsonInclude.html)