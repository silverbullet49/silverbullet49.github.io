# Filter와 Interceptor에 대하여

||Filter|Interceptor|
|---|---|---|
|위치|Spring Context 외부 Servlet 컨테이너|Spring Context 내부|
|동작 방식|filter가 다른 filter를 호출하여 체이닝|등록된 interceptor가 순차적으로 호출|
|Capability|다음 filter로 요청을 넘길 때 HttpServletRequest, HttpServletResponse 교체 가능|HttpServletRequest나 HttpServletResponse는 메서드를 통한 접근/제어만 가능|
|주의사항|예외 발생시 ControllerAdive, ExceptionHandler에서 처리 불가|예외 발생시 ControllerAdive, ExceptionHandler에서 처리|

![filter와 interceptor 다이어그램](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSz6DV%2Fbtq9zjRpUGv%2F68Fw4fZtDwaNCZiCFx57oK%2Fimg.png "출처: https://mangkyu.tistory.com/173")