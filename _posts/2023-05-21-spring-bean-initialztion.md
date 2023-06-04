# Bean에 있는 멤버 초기화 방법

- 빈에 있는 멤버를 초기화할 때 의존성인 경우에는 @RequiredArgsConstructor 어노테이션으로 쉽게 처리한다.
- 그런데 우리가 사용하는 멤버들은 아래와 같이 의존성이 아닌 케이스도 많다.

```java
@Slf4j
@RequiredArgsConstructor
@Repository
public class RestClient {
    private final RestTemplate restJsonTemplate;
    HttpHeaders headers = new HttpHeaders(){{
        setContentType(new MediaType("application", "json", StandardCharsets.UTF_8));
        setAccept(List.of(MediaType.APPLICATION_JSON));
    }}

    public <T> T get(String uri, Class<T> classType) {
        ResponseEntity<T> response = restJsonTemplate.exchange(uri, HttpMethod.GET, new HttpEntity<>(headers), classType);

        if (!response.getStatusCode().is2xxSuccessful()) {
            String errorMessage = Optional.ofNullable(response.getBody())
                    .map(Object::toString)
                    .orElse("response body received was null");

            throw new ResponseStatusException(response.getStatusCode(), errorMessage);
        }

        return response.getBody();
    }
}

```

## Java의 Double Bracket 초기화는 Anti-Pattern이다.

### bean의 멤버를 참조할 수 있어서 생길 수 있는 오류
- 우선 아래처럼 사용하면 Bean이 생성되지 않는다.

```java
@Slf4j
@RequiredArgsConstructor
@Repository
public class RestClient {
    private final RestTemplate restJsonTemplate;
    HttpHeaders headers = new HttpHeaders(){{
        headers.setContentType(new MediaType("application", "json", StandardCharsets.UTF_8));
        headers.setAccept(List.of(MediaType.APPLICATION_JSON));
    }}

    public <T> T get(String uri, Class<T> classType) {
        ResponseEntity<T> response = restJsonTemplate.exchange(uri, HttpMethod.GET, new HttpEntity<>(headers), classType);

        if (!response.getStatusCode().is2xxSuccessful()) {
            String errorMessage = Optional.ofNullable(response.getBody())
                    .map(Object::toString)
                    .orElse("response body received was null");

            throw new ResponseStatusException(response.getStatusCode(), errorMessage);
        }

        return response.getBody();
    }
}

```

- 이 코드의 문제는 headers에 있다. 
  - braket안에서 headers를 접근하고 있다. 컴파일할 때는 문제가 없다.
  - 그러나 headers가 생성되기 전에 headers에 사용하고 있기 때문에 ```NullPointerException```이 발생한다.


### Memory Leak 위험
- double bracket 초기화 방식에서는 bean의 멤버들과 같은 외부 레퍼런스를 참조한다.
- [링크](https://stackoverflow.com/questions/1958636/what-is-double-brace-initialization-in-java#:~:text=2.%20You%27re%20potentially%20creating%20a%20memory%20leak!)와 같이 dobule bracket 방식으로 초기화된 객체를 리턴하는 메서드가 있다면, 이 객체가 부모 클래스들을 다 레퍼런싱하고 있기 때문에 memory leak이 발생할 수 있다.

### 비효율적인 클래스 생성
- double bracket 초기화 방식은 익명 내부 클래스를 만들고, 부모 메서드를 호출하여 부모 멤버 변수를 초기화하는 방식이다.
- [따라서 double bracket 초기화를 해당 객체에서 여러 번할 경우 여러 개의  내부 클래스가 만들어진다.](https://stackoverflow.com/questions/1958636/what-is-double-brace-initialization-in-java#:~:text=Every%20time%20someone%20uses%20double%20brace%20initialisation%2C%20a%20kitten%20gets%20killed.)

## 멤버를 초기화할 때는 @PostConstruct를 사용한다.

```java
@Slf4j
@RequiredArgsConstructor
@Repository
public class RestClient {
    private final RestTemplate restJsonTemplate;
    HttpHeaders headers = new HttpHeaders();

    @PostConstruct
    public void init() {
        headers.setContentType(new MediaType("application", "json", StandardCharsets.UTF_8));
        headers.setAccept(List.of(MediaType.APPLICATION_JSON));
    }

    public <T> T get(String uri, Class<T> classType) {
        ResponseEntity<T> response = restJsonTemplate.exchange(uri, HttpMethod.GET, new HttpEntity<>(headers), classType);

        if (!response.getStatusCode().is2xxSuccessful()) {
            String errorMessage = Optional.ofNullable(response.getBody())
                    .map(Object::toString)
                    .orElse("response body received was null");

            throw new ResponseStatusException(response.getStatusCode(), errorMessage);
        }

        return response.getBody();
    }
}
```

- @PostConstruct는 빈이 만들어지고, 의존성 주입이 끝난 후에 호출되는 콜백 메서드이다.

> 스프링 컨테이너 생성 → 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료