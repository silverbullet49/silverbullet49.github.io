# Throwable, Exception, Error
## Java의 오류 처리 방법
![throwable, exception, error 상속 구조](https://blog.kakaocdn.net/dn/cCoRfv/btqyGd7HANT/TNQPOSMBCCunOdqH2DikaK/img.png)
- [출처](https://www.google.com/url?sa=i&url=https%3A%2F%2Frightnowdo.tistory.com%2Fentry%2FJAVA-JAVA%25EC%259D%2598-%25EC%2598%2588%25EC%2599%25B8%25EC%25B2%2598%25EB%25A6%25AC-Throwable-Exception-and-Error&psig=AOvVaw0N_6cySey_7kuUlWH6IG-7&ust=1684161204681000&source=images&cd=vfe&ved=0CBMQjhxqFwoTCOiL_cSD9f4CFQAAAAAdAAAAABAE)
- Java에서의 오류는 Exception과 Error로 나뉜다.
  - Throwable은 이 둘을 추상화한다. 즉 오류에 관한 최상위 인터페이스이다.

- Excepton은 어플리케이션 안에서 발생한, 치명적이지 않은 오류이다.
	- 사용자 오입력, 개발자의 버그 등에 의한 오류.
- Error는 exception과는 달리 어플리케이션에 치명적인 오류이다.

||Exception|Error|
|---|---|---|
|프로그램 종료 필요?|X|O|
|발생 시기|컴파일타임/런타임|런타임|
|대표적인 예|IOException, IllegalArgumentException|OutOfMemoryError, StackOverFlowError|


## checked와 unchecked
- exception은 두 가지로 나뉜다.

||checked exception|unchecked exception|
|---|---|---|
|발생 시기|컴파일타임|런타임|
|대표적인 예|IOExceptionm, JsonProcessingException|RunTimeException, IllegalArgumentException|
|주의사항|@Transactional시 롤백하지 않음.|@Transactional시 롤백함|

- checked exception은 롤백이 안되기 때문에, 이 경우 catch에서 unchecked exception 발생시켜 롤백을 진행시킨다.