# Spring Controller 개발 방법

## Spring MVC 패턴
![Spring MVC 구조](https://blog.kakaocdn.net/dn/Or4T1/btqFcNAEiAD/VLPsPQcUnUC8iWw8suH3Ek/img.png)
- [출처](https://www.google.com/url?sa=i&url=https%3A%2F%2Fcodingnotes.tistory.com%2F28&psig=AOvVaw2RB6TDVahlRMrbzWUQBXsy&ust=1684160898884000&source=images&cd=vfe&ved=0CBMQjhxqFwoTCICCk7OC9f4CFQAAAAAdAAAAABAp)
### 동작 방식
- ```DispatcherServlet```이 HTTP 요청을 받는다. 
- ```HandlerMapping```과 ```HandlerAdpator```를 통해 알맞은 Controller bean에게 요청을 위임한다.
   - ```DispatcherServlet```은 ```@Controller``` 어노테이션이 붙은 클래스를 component scan시 인지한다.
- Controller가 요청을 처리하고 결과를 반환한다.
    - 결과는 view name나 data(보통 json)이다.
    - data를 리턴해도 ```HttpMessageConvertor```에 의해 json 형태로 포매팅된다.
- ```DispatcherServlet```는 Controller에게 받은 결과 종류에 따라 다르게 처리하여 HTTP 응답을 보낸다.
   - view name이 반환되면 ```DispathServlet```은 ```ViewResolver```를 통해 view 템플릿을 찾아 HTTP 응답을 보낸다.
   - data가 반환되면 ```DispathServlet```은 http response body에 해당 data를 넣고 HTTP 응답을 보낸다.


### Data를 리턴하고 싶어요.
- ```@RestController```를 써라.
- ```@RestController```는 ```@Controller``` + ```@ResponseBody``` 이다.

### View를 리턴하고 싶어요.
- ```@Controller```를 써라.
- view template을 작성한다.(JSP)

### Exception 발생하면 어떡하지?
- ```@ControllerAdvice```와 ```@ExceptionHandler```를 써라.
  - ```@RestController```라면 ```@RestControllerAdvice```를 쓰자.(```@ControllerAdvice``` + ```@ResonpseBody```)

- ControllerAdvice의 동작 방식
  - spring bean에서 발생한 모든 처리되지 않은 예외는 결국 ```DispatcherServlet```으로 가게 됨.
  - ```DispatcherServlet```에서 Exception, Throwable을 catch하고 있고, 이후 exception을 ```@ControllerAdvice``` 빈의 ```@ExceptionHandler``` 메서드 중 해당 exception을 처리할 수 있는 메서드를 찾아 전달한다.
  - ```@ControllerAdvice``` bean에서 못 찾았다면, Controller 빈의 ```@ExceptionHandler``` 메서드 중 해당 exception을 처리할 수 있는 메서드를 찾아 전달한다.
  - 그래도 못 찾으면 status code 500인 Internal Server Error를 반환한다.