# Spring Exception 처리

- 세 가지 resolver가 있고, 각 리졸버는 순서대로 dispatcherservlet에서 호출됨.
- 아래는 자세한 과정.
  - DispatcherServlet의 init과정에서 applicationContext에서 HandleExceptionResolver 타입의 bean을 가져온다.
	- applicationconext에 등록된 순서대로 resolver를 가져온다.(bean 등록 과정에서 우선순위 정하는지는 아직 모르겠음)
	- 순서는 ExceptionHandlerExceptionResolver, ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver
  - doDispatch에서 resolvers를 호출
	- modelAndview가 null이면 다음 리졸버를 호출.


- ExceptionHandlerExceptionResolver는 @exceptionHandler로 표시된 처리기를 찾고, 호출.
  - 예외 처리 메서드를 작성할 수 있어서, 세밀한 제어에 도움.
-  ResponseStatusExceptionResolver는 @ResponseStatus 나 ResponseStatusException을 처리.
- DefaultHandlerExceptionResolver는 컨트롤러 매핑 과정에서 일어나는 exception(예를 들어, 잘못된 method로 요청하는 케이스)들을 처리.


> 참고 : https://kihwan95.tistory.com/8