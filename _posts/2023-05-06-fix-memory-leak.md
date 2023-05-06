---
title: "Java Memory Leak 이슈 해결 경험 공유"
categories:
  - implement
tags:
  - Java
  - MemoryLeak
---

## 문제의 시작
- Network Device들로부터 SNMP를 통해 각종 통계, 설정 정보를 모니터링하는 어플리케이션에서 문제를 발견.
  - 한 달 정도된 롱런 시험에서 해당 어플리케이션 pod memory 사용률(%)이 우상향하고 있음(80%를 넘김)을 발견.
- 여러 개의 롱런 환경을 비교해서 발견한 첫 번째 사실은 모니터링하는 device가 많을수록 메모리 증가율이 크다는 것.

## 분석
### 분석을 위한 준비
- Spring Application이 아니었으므로, 해당 pod에 [jmx exporter jar](https://github.com/prometheus/jmx_exporter)를 붙임.
- 개발 환경에 Prometheus와 Grafana를 구축.
  - [JMX Dashboard](https://grafana.com/grafana/dashboards/14845-jmx-dashboard-basic/), [Kubernetes Metrics Dashboard](https://grafana.com/grafana/dashboards/6417-kubernetes-cluster-prometheus/)
  - prometheus의 retention 주기는 3650days, storage size를 100G로 줘서 충분한 히스토리를 저장할 수 있도록 설정.

### 지표 분석
- 구축 후 이틀 뒤 heap memory 사용량 확인.
  - heap memory usage가 우상향 성향이 두드러지게 보이지 않음
  - gc가 주기적으로 잘 돌고 있었고, gc count는 미세하게 우상향. 
- ```promql```의 ```max_over_time```를 사용하여 5개의 테스트베드에서 heap usage의 최댓값이 우상향하고 있음을 파악.
- ```jmap```과 ```visualvm```을 사용하여 heapdump를 분석했더니, ```forkJoinPool``` 객체, 불필요한 객체 사용(주로 레거시 코드에 의한)이 전체 객체수의 40% 정도임을 확인.
- ```max_over_time```를 통해 ```live thread count```도 미미하게 증가함을 확인. **스레드 갯수 증가로 인해 heap memory가 증가하는 것이 아닐까 가정.**

### 지표 분석 후 액션
- 스레드 갯수 증가 요소를 제거
  - 스레드 갯수가 불확실한 ```forkJoinPool``` 대신 ```fixedThreadPool```로 대체하여 테스트
- 대대적인 코드 정리 시작
  - 신규 버전에서는 이 프레임워크를 버리기로 결정.
  - 필요한 코드만 손쉽게 가져오기 위해 코드 정리 작업을 하기로 결정.(디펜던시가 복잡하게 되어 있어서, 이 작업도 쉬운 작업은 아니었음.)

## 문제 원인 발견
- 전보다는 조금 가벼워진 코드로 롱런 시험 시작.
  - pod memory limit을 64G -> 16G로 축소.(신규 버전의 dimensioning)
- 시험 시작 7일 후, native thread를 새로 만드는 과정에서 OOM 발생.
  - 여전히 ```live thread count```가 증가하고 있음을 확인.
  - ```jmxBean```과 ```visualvm```을 통해 확인한 thread 목록에서 **이름없는 thread pool이 생성하는 thread가 증가하고 있음을 발견.**
    - code에서 이름없이 사용하는 threadPool에 모두 이름을 붙이고, 모든 thread를 다시 확인.
    - ```scheduledExecutorService```를 통해 주기적으로 호출되는 메서드에서 ThreadPool을 생성하고 지역변수로 할당하여 사용하는 로직이 있었음.
    - 메서드가 종료되어 지역변수는 스택에서 해제되지만, [thread pool을 제거하려면 shutdown을 명시적으로 해야함](https://www.baeldung.com/java-8-parallel-streams-custom-threadpool#bd-beware-of-the-memory-leak)
- ```Eclipse Memory Analyzer```를 통해 분석된 memory leak suspects에는 ```sslcontextimpl```객체가 있었음.

## 결말
- 발견된 원인을 제거한 후, 2주 정도 지난 지금, 아직까지 문제가 발생하고 있지 않다!


### 타임라인
* 2023/01/10 - 한 달 정도된 롱런 시험에서 pod memory usage가 우상향하고 있음을 발견. provisioned device가 많을수록 메모리 증가율이 큼.
    
* 2023/01/11 - 분석을 위해 jmx prometheus agent 설치 및 prometheus, grafana 구축(jmx dashboard, detailed pod resources dashboard)
    
* 2023/01/12 - heap memory usage 우상향 성향이 두드러지게 보이지 않음. gc가 주기적으로 잘 돌고 있었고, gc count는 미세하게 우상향. -&gt; non-heap 문제인가 의심.
    
* 2023/01/13 - 새 테스트 ns에서 문제되는 pod과 interacting하는 다른 pod들을 제거 후 6시간 돌려 놓음 -&gt; direct memory 등의 non-heap 쪽 문제라면 이 상황에서 memory 사용량 증가가 없어야 함.
    
* 2023/01/15(주말) - kubernetes의 pod memory 모니터링에 대해 공부 -&gt; working\_set\_size와 rss\_size가 OOM 판단 기준 중 하나. usage 자체는 값을 정확하게 확인 안해봐서 어떤 커널 지표를 사용하는지 파악 안 됨.
    
* 2023/01/16 - promql의 max\_over\_time를 사용 -&gt; 5개의 TB에서 heap usage의 최대값의 추이가 우상향. / prometheus의 retention 주기 및 storage size를 설정하지 않아 과거 2일 이전의 데이터를 볼 수 없음을 발견함.(ㅠㅠ)
    
* 2023/01/17 - heap dump를 통해 forkjoinpool object, legacy code가 사용하는 객체가 많음을 발견 / thread count도 미미하게 증가.(max\_over\_time 사용)
    
* 2023/01/18 - forkjoinpool -&gt; fixedthreadpool로 변경 / 모니터링만으로는 원인을 파악하기 어려워 동작이 불확실하거나(포크조인풀이 여기에 해당) device 갯수에 영향을 주는 코드 리팩토링 작업. 코드 리팩토링할 만 한 것은 크게 없었음.(디펜던시가 복잡하게 되어 있는 레거시라 건드리기 어려웟음)
    
* 2023/02/06 - 대대적인 코드 정리가 시작되었다. 이 프레임워크를 계속 쓰진 않을 거지만, 마이그레이션시 우리가 필요한 코드만 손쉽게 가져오기 위해 코드 정리 작업을 하기로 결정했다. 메모리릭 범인을 찾는 데에도 도움을 주기를 바라는 마음도 있었다.
    
* 2023/04/17 - 기존 패키지 코드량 -&gt; 현 패키지 코드량 비교. 그래도 메모리 릭은 발생하고 있었다.
    
* 2023/04/17 - native thread를 새로 만드는 과정에서 OutOfMemoryError 발생 -&gt; jmxBean 객체를 통해 모든 thread 조회 -&gt; live thread 갯수가 미세하게 계속 늘어나고 있음. -&gt; thread 목록을 보니, thread pool 자체가 늘어나고 있는 게 있었음. -&gt; 스케쥴링되는 메서드에서 로컬에 threadpool을 만들었음. -&gt; 객체가 해제될 때 shutdown이 호출되는지 아닌지 모르겠으나, finally 구문에서 명시적으로 shutdown을 함.