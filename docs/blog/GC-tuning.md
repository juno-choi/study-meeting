# 🔴 GC Tuning

## 🟠 Garbage Collection Tuning

java가 다른 언어에 비해 속도 차이가 나는 이유는 아키텍쳐 설계에 있는데 자바는 컴파일 언어라 컴파일하고 해석하는데 있어 시간이 소요되기 때문이다. java의 어플리케이션 성능의 가장 큰 비중을 차지하는게 GC의 STW이다.

java 성능이 제대로 안나온다면 마지막 수단으로 GC 튜닝을 진행해야 한다.

1. 코드레벨에서 점검
2. STW를 줄이기 위해 GC 알고리즘 교체
3. GC 튜닝

java에서 System.gc() 라는 메서드가 있지만 메서드를 호출하는 순간 모든 스레드가 중단되기 때문에 서비스 환경에서는 절대 사용하면 안되는 코드이다.

### 🟢 GC 튜닝의 주의점

`GC 튜닝 옵션은 서비스의 특징마다 적정 값이 다르다`

다른 서비스에서 해당 GC 알고리즘을 사용하거나 GC 튜닝을 통해 성능을 개선했다 할지라도 자신의 서비스에 적용한다 해서 성능이 개선되는 것은 아니다.

왜냐하면 서비스 구성에 따라 생성되는 객체의 크기와 생명주기 모두 다르기 때문이다. 따라서 별도의 성능 모니터링을 통해 어느지점에서 GC의 STW 문제가 나는지 파악하여 GC 튜닝을 진행해야 한다.

`GC 튜닝은 가장 마지막 작업`

GC 튜닝은 신경써야할 요소, 리스크에 비해 바로 문제를 해결할 수 있는 수단이 아니기 때문에 GC 튜닝을 진행하기 보다 애플리케이션의 코드 레벨에서 메모리 최적화를 더욱 신경 쓰는것을 권장한다.

java에서 GC가 작동하는 이유는 객체가 많아 메모리가 모두 차기 때문이다. 즉, 애플리케이션에서 쓸모 없는 객체의 생성을 줄이는 리팩토링 작업을 먼저 하는 것이 근본적인 문제 해결 법이다.

### 🟢 GC 튜닝의 목표

GC 튜닝은 성능상 이슈를 일으키기 쉬운 GC에 대해 최적화를 진행하는 것을 말한다. 

GC 튜닝의 핵심 목표는 Minor GC 보다 STW를 일으키는 Major GC(Full GC)의 관리이다.

`Old 영역으로 넘어가는 객체의 수 최소화`

GC의 구조에서 Old 영역의 크기는 Young 영역의 크기보다 훨씬 크다. 따라서 Old 영역의 GC는 Young 영역에 비해 많은 시간이 소요되기 때문에 애초에 Old 영역으로 이동하는 객체의 수를 줄이면 Major GC가 발생하는 빈도를 줄일 수 있다.

`Major GC 시간 줄이기`

Major GC의 실행 시간은 Minor GC에 비해 길기 때문에 Old 영역의 크기를 적절하게 설정하는 것도 하나의 방법이다.

Old 영역의 크기를 너무 줄여버리면 OOM이 발생하거나 Major GC 횟수가 늘어날 수 있다.

반대로 Old 영역의 크기를 늘리면 Major GC의 횟수는 줄지만 실행하는 시간이 늘어난다.

즉, GC 튜닝은 이 둘의 관계를 잘 파악하여 적정 범위를 찾아가는 것이다.

## 🟠 GC 튜닝 절차 맛보기

### 🟢 GC 상황 모니터링

jstat를 통해 GC를 모니터링 한다.

| 컬럼 | 설명 |
|-------|-------|
| Timestamp -t옵션 | JVM 시작 후 경과 시간 |
| S0 | Survivor 0 영역의 사용률 |
| S1 | Survivor 1 영역의 사용률 |
| E | Eden 영역의 사용률 |
| O | Old 영역의 사용률 |
| M | Metaspace 사용률 |
| CCS | Compressed Class Space 사용률 |
| YGC | Young 영역의 GC 실행 횟수 |
| YGCT | Young 영역의 GC 실행 시간 |
| FGC | Old 영역의 Full GC 실행 횟수 |
| FGCT | Old 영역의 Full GC 실행 시간 |
| GCT | GC 총 실행 시간 |


```bash
jps -l
```
![](https://velog.velcdn.com/images/ililil9482/post/aacd860f-ab45-4dac-a302-79f9e75fd178/image.png)

`jps -l` 명령어를 통해 pid 값을 얻어 온 뒤

```bash
# jstat gcutil  명령어로 현재 실행중인 8884번 프로세스에 대해 1초에 한번 씩 총 10번 GC와 관련된 정보를 출력하도록 모니터링
jstat -gcutil -t 26096 1000 10
```

![](https://velog.velcdn.com/images/ililil9482/post/78a7e3ea-7525-4422-b252-158457307b86/image.png)

`jstat -gcutil -t <pid> <ms> <count>` 명령어를 통해 GC 관련된 데이터를 모니터링할 수 있다.

`모니터링 결과`

- Minor GC 처리가 50ms 내외
- Minor GC 주기가 10초 내외
- Major GC 처리가 1초 내외
- Major GC 처리가 10분에 1회 이내

### 🟢 GC 알고리즘 방식 지정

GC 모니터링 이후에 GC를 튜닝하기로 결정했다면 GC 알고리즘 방식을 선정한다.

| GC 알고리즘 | 내용 |
|-------|-------|
| Parallel GC | 처리량이 주요한 시스템, Major GC 수행시 compaction 작업이 수행되기 때문에 GC 시간 자체는 많이 소요됨 |
| CMS GC | 응답 시간이 주요한 시스템, 현재는 ZGC를 사용, 자원 사용량이 증가한다. |
| G1 GC | 성능적으로 가장 우수한 방식이나 기본값이기 때문에 문제가 발생했다면 해당 알고리즘 이외에 알고리즘을 선택해야 함 |

### 🟢 Heap 메모리 크기 지정

JVM의 Heap 메모리 크기에 따라 GC 발생 횟수와 수행 시간에 영향을 끼치기 때문에 옵션을 통해 조절하면 어플리케이션 성능 향상 효과를 가져올 수 있다.

JVM 시작 크기(-Xms) 최대 크기(-Xmx)를 말한다.

| 구분 | 옵션 | 설명 |
|-------|-------|-------|
| Heap 영역 크기 | -Xms | Heap 시작 크기 |
| Heap 영역 크기 | -Xmx | Heap 최대 크기 |
| Young 영역 크기 | -XX:NewRatio | Young영역과 Old 크기 비율 |
| Young 영역 크기 | -XX:NewSizePercent | Young영역과 Old 크기 |
| Young 영역 크기 | -XX:SurvivorRatio | Eden 영역과 Survivor 크기의 비율 |

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRjSO5%2FbtrI6rGWTwv%2FLSXU401BUwmBEJ2HoupXpk%2Fimg.png)


이 중에서 중요한 옵션은 -Xms 옵션, -Xmx 옵션, -XX:NewRatio 옵션이다.

특히 -Xms 옵션과 -Xmx 옵션은 왠만하면 필수로 지정하길 권장되며, 그리고 NewRatio 옵션을 어떻게 설정하느냐에 따라서 GC 성능에 많은 차이가 발생한다.


NewRatio는 New 영역과 Old 영역의 비율이다.

-XX:+NewRatio=1로 지정하면 (New 영역):(Old 영역)의 비율은 1:1이 된다.

만약 1GB라면 (New 영역):(Old 영역)은 500MB:500MB가 된다.

NewRatio가 2이면 (New 영역):(Old 영역)이 1:2가 된다.

즉, 값이 커지면 커질수록 Old 영역의 크기가 커지고 New 영역의 크기가 작아진다.

```bash
# 힙 시작 크기 256mb, 힙 최대 크기 2gb
# young 영역과 old 영역 비율 1:2 로 설정 (New 영역:Old 영역 = 1:2)
# Parallel GC 로 실행
java -Xms256m -Xmx2048m -XX:+NewRatio=2 -XX:+UseParallelGC
```

예시

### 🟢 튜닝 결과 분석

GC 옵션을 설정한 이후 APM 도구를 통해 1~2일 데이터를 수집한다.

1. FullGC 수행 시간
2. MinorGC 수행 시간
3. Full GC 수행 간격
4. MinorGC 수행 간격
5. 전체 Full GC 수행 시간
6. 전체 Minor GC 수행 시간
7. 전체 GC 수행 시간
8. Full GC 수행 횟수
9. Minor GC 수행 횟수

중요도 순서로 확인하면 된다.
 