---
title: "class loading"
categories:
  - study
tags:
  - Java
---
# Class Loading
- 클래스 로더는 java class들을 **동적으로** jvm에 로딩한다.
- 자바 클래스들은 메모리에 한번에 적재되지 않고, 어플리케이션이 필요할 때 적재된다.

## 빌트인 클래스 로더
- 빌트인 클래스로더에는 system class loader, extention class loader, bootstrap class loader가 있다.
- 이들은 hierarchy를 가진다. 

- 부트 스트랩 클래스로더
  - 최상위 클래스로더이다. primitive 클래스 등의 JDK interal class들을 로딩한다.
  - JVM 내부에 작성되어 있다.(그러므로 native code)
  - classLoader들도 일종의 클래스인데, classLoader를 로딩해주는 클래스로더가 바로 부트스트랩 클래스로더이다.

- extention 클래스 로더
  - jdk의 ext 디렉토리 밑의 클래스들을 load한다고 한다.
  - 부트스트랩 클래스로더 클래스의 자식 클래스이다.


- 시스템 클래스 로더
  - 개발자가 만든 클래스들을 로딩한다.
    - 클래스패스 경로에 있는 클래스 파일들을 읽고, 로딩한다.


### Delegation model
- 빌트인 클래스 로더들은 delegation model을 따른다.
  - class를 로딩할 때, 부모 클래스 로더에게 로딩을 위임한다.
  - 최상위 클래스 로더가 로딩에 실패한다면, 바로 아래 자식 클래스 로더가 로딩을 시도한다.


### visibility principle
- 하위 클래스 로더는 상위 클래스 로더가 로딩한 클래스들을 알 수 있다.
- 반대는 불가하다.

### unique principle
- 하위 클래스 로더는 상위 클래스가 로딩한 클래스를 다시 로딩하지 않아야 한다.
- delegation scheme을 통해 이를 보장한다.


## 클래스 로딩 시점
### 클래스 로딩 시점
- 인스턴스 생성
- final이 아닌 정적 변수 사용
- 정적 메서드 호출

### 클래스 로딩이 안 되는 경우
- 클래스에 접근하지 않을 때
- final 정적 변수 사용


## 클래스 로딩 프로세스
- 클래스 로딩에는 loading, link, initialization이 있다.

- loading
  - .class의 byte array를 Class 형태로 메모리에 올리는 것(메소드 영역에 저장)
- link
  - verification, prepare, resolution이라는 세 단계가 있다.
- initialztion
  - **static 변수들을 개발자가 작성한 코드로 초기화한다.**


## 싱글톤 
- 내부 클래스의 경우에도 사용될 때 로딩이 된다.
- 이를 이용해 조금 더 효율적인 싱글톤 클래스를 작성할 수 있다.


```java
//Initialization-on-demand holder idiom 
public class HolderSingleton {

    private HolderSingleton() {
    }

    public static HolderSingleton getInstance() {
        return Holder.instance;
    }

    private static class Holder {
        public static final HolderSingleton instance = new HolderSingleton();
    }
}
```