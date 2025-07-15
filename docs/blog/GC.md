`출처` [Inpa Dev 가비지 컬렉션 동작 원리 & GC 종류 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC#garbage_collectiongc_%EC%9D%B4%EB%9E%80?)

# 🔴 Garbage Collection

## 🟠 GC란?

GC는 자바의 메모리 관리 방법 중 하나로 JVM의 Heap 영역에서 동적으로 할당했던 메모리 중 필요 없게 된 메모리 객체를 모아 주기적으로 제거하는 프로세스이다.

덕분에 개발자들은 개발에만 집중할 수 있지만 단점으로는 메모리가 언제 해제되는지 정확하게 알 수 없어 제어하기가 힘들고 GC가 동작할때 다른 쓰레드들이 모두 멈추기 때문에 오버헤드가 발생되는 문제점이 있다.

이를 Stop-The-World라고 한다.

### 🟢 STW(Stop The World)

GC를 수행하기 위해 JVM 프로그램이 멈추는 현상을 의미한다. GC가 작동하는 동안 GC 관련 Thread를 제외한 모든 Thread가 멈추게 되어 서비스가 멈추는 현상이다. 따라서 이 시간을 최소화 시키는 것이 쟁점이다.

따라서 실시간으로 멈추지 않고 작동해야 되는 프로그램의 경우 GC를 사용하지 않는 것이 더 효율적일 수 있으며 효율적으로 GC를 실행하기 위해 GC 튜닝을 진행하기도 한다.

### 🟢 GC 대상

GC의 대상은 어떻게 판별될까? GC는 특정 객체가 grabage 대상인지 판단하기 위해 도달성, 도달능력(Reachability)라는 개념을 적용한다.

객체에 레퍼런스가 있다면 Reachable로 구분되고, 객체에 유효한 레퍼런스가 없다면 Unreachable로 구분해버리고 수거해버린다.

`Reachable` 객체가 참조되고 있는 상태

`Unreachable` 객체가 참조되고 있지 않은 상태(GC 대상)

![JVM](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc80g54%2FbtrIT9fcYdX%2FIzXI82CrxVZa7ndGobFwl1%2Fimg.png)

예를 들어 new 선언문으로 생성한 객체가 Heap에 등록되고 Method Area나 Stack Area에서 주소를 참조하여 사용하고 있다. 이렇게 생성된 Heap Area 객체들이 메서드가 끝나는 등 특정 이벤트들로 인해 객체의 메모리 주소를 가지고 있는 참조 변수가 삭제된다면 위에 2번처럼 Unreachable 상태로 변하게 된다.

이러한 객체들을 주기적으로 GC가 제거해주는 것이다.

### 🟢 GC 청소 방식 (Mark And Sweep)

![Mark And Sweep](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc4UAyv%2FbtrIZ64PlNk%2FZ1y8TDx3xz6ONZsW2DttA1%2Fimg.png)

Mark-Sweep이란 다양한 GC에서 사용되는 객체를 제거하는 내부 알고리즘이다. 원리는 간단하다.

GC될 대상 객체를 식별(Mark)하고 제거(Sweep)하며 객체가 제거되어 파편화된 메모리 영역을 앞으로 압축(Compaction)시키는 과정을 수행한다.

`Mark` 먼저 Root Space로부터 그래프 순회를 통해 연결된 객체들을 찾아내 각각 어떤 객체를 참조하고 있는지 찾아서 마킹한다.

`Sweep` 참조하고 있지 않은 객체(Unreachable) 객체들을 Heap에서 제거한다.

`Compact` Sweep 후 분산된 객체들을 Heap의 시작 주소로 모아 메모리 할당된 부분과 그렇지 않은 부분으로 압축한다.(GC 종류에 따라 하지 않는 경우도 있음)

![Mark Sweep 동작 과정](https://blog.kakaocdn.net/dn/cgSNa1/btrIVgyOGYA/509WaRQ5LIjGdfNbduIMJ0/img.gif)

### 🟢 GC의 Root Space

Mark And Sweep 방식은 루트로 부터 해당 객체에 접근이 가능한지 해제의 기준이 된다. JVM GC에서 Root Space는 Heap 메모리 영역을 참조하는 Method Area, static 변수, stack area, native method stack이 갖게 된다.

## 🟠 GC 동작 과정

### 🟢 Heap 메모리 구조

JVM에서 Heap Area는 동적으로 Reference 데이터가 저장되는 공간으로서, GC 대상이 되는 공간이다.

Heap Area는 처음 설계될 때 2가지를 전제(Weak Generational Hypothesis)로 설계되었다.

1. 대부분의 객체는 금방 접근 불가능 상태(Unreachable)가된다.
2. 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

즉 대부분의 객체는 일회성으로 생성되고, 메모리에 오랫동안 남아이는 경우는 아주 드물다는 것이다.

이러한 특성으로 JVM은 Heap 영역을 Yong과 Old 2가지 영역으로 설계하였다.

![yong old](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdS5yAY%2FbtrIPXf59mc%2FtwmRkR4hOw9MpcC7BSHbak%2Fimg.png)

`Young`
- 새롭게 객체가 생성되고 할당되는 영역
- 대부분 객체가 금방 Unreachable 상태가 되기 때문에 금방 GC 된다.
- Young 영역을 대한 GC를 Minor GC라고 부른다.

`Old`
- Young 영역에서 Reachable 상태를 유지하고 살아남은 객체가 복사된 영역이다.
- Young 영역보다 크게 할당되고 영역의 크기만큼 GC가 적게 발생된다.
- Old 영영게 대한 GC를 Major GC 또는 full GC라고 부른다.

### 🟢 Yong 영역 세분화

![Yong detail](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkYzig%2FbtrIYnMIU2l%2FtXCaFeaKhyDXq2hlnalWB1%2Fimg.png)

`Eden`
- new 생성자를 통해 생성된 객체가 위치
- 정기적으로 GC 후 살아남은 객체는 survivor 영역으로 복사하여 이동함

`Survivor 0 / Survivor 1`
- 최소 1번 이상의 GC에서 살아남은 객체들이 존재하는 영역
- Survivor 영여겡는 특별한 규칙이 있는데, Survivor 0 또는 1 둘 중 하나는 꼭 비어 있어야 한다는 것이다.

### 🟢 Minor GC 과정

![Minor GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcz9lzz%2FbtrIRGyAeEV%2F6PTQpYAVBKyAgl29pVK0kk%2Fimg.png)

Young 영역은 짧게 살아남는 메모리들이 존재하는 공간이다. 모든 객체는 처음에는 Young 영역에 생성되고 Young은 Old에 비해 크기가 작기 때문에 GC를 실행하는데 걸리는 시간도 작다. 이때문에 Young 영역에서의 GC를 Minor GC라고 부른다.

1. 처음 생성된 객체는 Eden에 위치

![eden](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcXg3ZZ%2FbtrITpQET0n%2FaJSAYDEziQIKlVCvxaSpg0%2Fimg.png)

2. 객체가 계속 생성되어 Eden 영역이 꽉차면 Minor GC 시작

![minor GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtySvx%2FbtrITpwlB0D%2FJtcEmTbIYaPnHAAoNU8vB0%2Fimg.png)

3. Mark 동작을 통해 reachable 탐색

![mark](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcJl67l%2FbtrIXRd6Izv%2FZwTbTqbKkuaH71Llqzog6k%2Fimg.png)

4. Eden에서 살아남은 객체는 Survivor 0(or 1)에 복사

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbxY6Md%2FbtrIWNwtetB%2F4EpRJFGM5v7Fwk9dk0Wc9K%2Fimg.png)

5. Eden 영역에서 사용되지 않는 객체(unreachable) 메모리를 sweep

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FssSMf%2FbtrIWMYFQlZ%2FKqHBu6rr18ANKo1ngOnMQk%2Fimg.png)

6. 살아남은 모든 객체들은 age 값이 1 증가

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDno6M%2FbtrIPYfc2VC%2FBLLkKAXLVRYKUAu3R6Vefk%2Fimg.png)

7. Eden 영역에 또 가득 차면 minor GC 발생하고 mark

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcNoD2j%2FbtrIT8gQrk9%2FvIqyTQJvZ16ByIGplen2D0%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F8XX01%2FbtrIPYsHxbi%2FFothJXlh95Tc6DSYSkZb10%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcRhJ1g%2FbtrIXPtPw5o%2FgN8t1V7BjNXaRqPjKueVX1%2Fimg.png)

8. marking 한 객체들이 비어있는 survivor 1로 이동하고 sweep

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdsj6Og%2FbtrIRHqHljJ%2F00JVcQ3KD1E8he6qejuLik%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fblilhe%2FbtrISO4eRF3%2FsMVxubaUB19Sm1XsVJE90k%2Fimg.png)

9. 살아남은 모든 객체들 age 1 증가

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd8v4Vk%2FbtrIXwVy4Uj%2FiGv5H94KNcZhQ0GufntWg0%2Fimg.png)

10. 과정 반복

![](https://blog.kakaocdn.net/dn/2WHEq/btrIUaeDqLG/qN0f490z0nGddd0Vl7ECdk/img.gif)

### 🟢 Majro GC(Full GC) 과정

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FAl6rO%2FbtrISO4bXw2%2FmkrLKFEfQKSWgZKxMcxKWK%2Fimg.png)

Old 영역에 살아남아 있는 객체들이 존재하는 메모리 공간이다.

Old의 모든 객체들은 처음에는 Yong 영역에서 시작되었으나, GC 과정중 제거되지 않은 경우 age 임계값이 차게되어 이동된 객체들이다.

그리고 Major GC는 개체들이 계속 Young에서 Old로 이동되어 Old 영역의 메모리가 부족해지면 발생하게 된다.

1. 객체 age 값이 임계값으로 도달 (현재 8로 설정되어 있다 가정)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc3ZKIh%2FbtrITnE39P2%2FcL0xncLtqvQxqOmxmyt8V0%2Fimg.png)

2. Old 영역으로 이동, 이를 promotion이라고 부른다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FGJGPb%2FbtrIUmFRmQO%2FSZDmkA2vbBcqKcXrNiIhOk%2Fimg.png)

3. 위 과정이 반복되어 Old 영역 메모리가 부족하게 되면 Major GC가 발생한다.

![](https://blog.kakaocdn.net/dn/mnBRi/btrIUm6WXIl/obIsxDpns6wEkkKfjdvLCK/img.gif)

Old 영역의 GC는 Young 영역에 비해 큰 공간을 가지고 있어 이 공간을 제거하는데는 비교적 많은 시간이 걸리게 된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbQinW1%2FbtrIOSeHYMd%2Frd1iSnwNConXoRVWiDvDS0%2Fimg.png)

여기서 STW 문제가 발생하게 되며 Major GC가 발생할때 CPU에 부하가 걸리거나 멈춤현상, 버벅이는 현상이 일어난다.

JVM에서는 이러한 문제를 해결하기 위해 GC 알고리즘을 발전시켜왔고 현재도 진행중이다.

## 🟠 GC 알고리즘 종류

![GC 알고리즘 종류](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYyftm%2FbtrIT8g2DyE%2FMjr9HzCmMVLmZnZagtkrT1%2Fimg.png)

GC 알고리즘 종류는 java 설정을 통해 상황에 따라 선택해서 적용할 수 있다.

[자바 버전별 GC 알고리즘 (8 ~ 21)](https://imygnam.tistory.com/129)

### 🟢 Serial GC

- 서버의 CPU 코어가 1개일 때 사용하기 위해 개발된 가장 단순한 GC
- GC를 처리하는 쓰레드가 1개라 가장 STW 시간이 오래 걸린다.
- Minor GC에는 Mark-Sweep을 사용하고 Major GC에는 Mark-Sweep-Compact을 사용한다.
- 보통 실무에서 사용하는 경우는 없고 개인적으로 환경 구축할때 디바이스 성능이 안좋을 경우 사용한다.(최근에는 저렴한 디바이스도 이정도 성능보다 좋기 때문에 사용할 일이 없다.)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkBL3D%2FbtrIT91k2n7%2FOmhFna09F6duWkBh1FUFd0%2Fimg.png)

```bash
java -XX:+UseSerialGC -jar Application.java
```

다음 명령어로 java 실행시 `-XX:+UseSerialGC` GC 옵션을 설정하여 힙 메모리를 관리하도록 설정할 수 있다.

### 🟢 Parallel GC

- java 8의 default GC 알고리즘이다.
- Serial GC와 기본 동작 과정은 같지만 Young 영역의 Minor GC를 멀티 쓰레드로 실행한다.
- Serail GC에 비해 STW 시간 감소

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FckjP6E%2FbtrIVf74R58%2FLdPm5Nmd3tCLFLuknpOc2K%2Fimg.png)

```bash
java -XX:+UseParallelGC -jar Application.java 
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
```

GC 쓰레드는 기본적으로 CPU 개수만큼 할당되고 옵션을 통해 개수를 설정해줄 수 있다.

### 🟢 Parallel Old GC (Parallel Compacting Collector)

- 기존 Parallel GC를 개선한 버전
- Young 영역 뿐만 아니라 Old에서도 멀티 쓰레드로 GC 수행
- 새로운 GC 방식인 Mark-Summary-Compact 방식을 이용
- summary란 기존 sweep을 단일 쓰레드에서 실행했다면 summary는 멀티 쓰레드가 병렬적으로 실행하는 것을 의미한다.
- java17 이후로는 Parallel Old가 Parallel GC로 병합되어 따로 존재하지 않음.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcs71MN%2FbtrI0ePXZqr%2F6nI0EkNdQ7fBzMYlwewk8K%2Fimg.png)

```bash
java -XX:+UseParallelOldGC -jar Application.java
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
```

### 🟢 CMS GC(Concurrent Mark Sweep)

- 어플리케이션의 실행중인 쓰레드와 GC 쓰레드가 동시에 실행되어 STW 시간을 줄이기 위해 고안된 GC
- GC 과정이 매우 복잡함
- GC 대상을 파악하는 과정이 복잡한 여러단계로 수행되기 때문에 다른 GC 대비 CPU 사용량이 높음
- 메모리 파편화 문제가 있음
- java9에는 deprecated되었고 java14에서는 중지

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbtq9xn%2FbtrIUalHp5a%2Fuoqi42gxuTywMfm5Uhcs9k%2Fimg.png)

```bash
# deprecated in java9 and finally dropped in java14
java -XX:+UseConcMarkSweepGC -jar Application.java
```

### 🟢 G1 GC(Garbage First)

- CMS GC를 대체하기 위해 java7 버전에서 최초로 고안된 GC
- java9 이상 버전부터 default GC
- 4Gb 이상의 힙메모리, STW 시간이 0.5초 정도 필요한 상황에 사용(Heap 영역 메모리 크기가 작을 경우 미사용 권장)
- 기존 GC는 Heap 영역을 물리적으로 고정된 Young/Old로 나누어 사용하였지만, G1 GC는 아예 이러한 개념 대신 `Region`이라는 새로운 개념을 도입
- Heap 영역을 Region이라는 영역으로 체스같이 분할하여 상황에 따라 Eden, Survivor, Old등 고정 할당이 아닌 동기적으로 영역을 부여
- GC는 가득찬 영역을 빠르게 회수하여 빈 공간을 확보하므로, 결국 Eden, Survivor, Old 영역에 따라 GC의 시간 차이가 없고 GC 빈도도 줄어드는 효과를 얻게 되는 원리
- 메모리 부족이나 객체 수 증가로 인해 pause 현상 문제가 있음.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcjO5N5%2FbtrI1Ob7Gbb%2FftlsnGhb10ktSzVsEpJCW0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3VAZL%2FbtrIVglH89c%2F80uC2DsGKDc1HXG57ByKak%2Fimg.png)

- G1은 이전 GC들처럼 일일히 메모리를 탐색하여 Sweep 하지 않는다. 대신 메모리가 많이 차있는 region을 인식하는 기능을 통해 메모리가 많이 차 있는 영역을 우선적으로 GC한다. 즉, 전체적으로 탐색하는 것이 아닌 영역(region) 별로 GC가 일어난다.
- 또한 이전 GC들은 Young 영역에서 Eden->survivor로 순차적 이동을 했지만 G1에서는 이동하지 않는다. 대신 G1은 더욱 효율적이라 판단되는 위치로 객체를 Reallocate(재할당) 시킨다. 예를들어 suvivor1에 존재하는 객체가 Eden 영역에 있는 것이 더 효율적이라고 판단된다면 Eden으로 이동시킨다.

```bash
java -XX:+UseG1GC -jar Application.java
```

기본값이기 때문에 굳이 설정해주지 않아도 된다.

### 🟢 Shenandoah GC
- java12에서 red hat에서 개발한 GC
- 기존 CMS가 가진 단편화, G1이 가진 pause 이슈 해결
- 강력한 Concurrency와 가벼운 GC 로직으로 heap 사이즈에 영향을 받지 않고 일정한 pause 시간 소요가 특징
- 강력한 Concurrency가 장점이지만 이를 수행하기 위해 더 많은 CPU 리소스를 활용하는 것이 단점

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlHh4s%2FbtrISNkkpMV%2FcuFxgmAT0DuafPhABn0L00%2Fimg.png)

```bash
java -XX:+UseShenandoahGC -jar Application.java
```

### 🟢 ZGC (Z Gabage Collector)

- java15에 release
- 대량의 메모리(8Mb ~ 16Tb)를 low-latency로 잘 처리하기 위해 디자인된 GC
- G1의 Region처럼 ZGC는 ZPage라는 영역을 사용하고, G1은 영역의 크기가 고정인데 비해 ZPage는 영역을 2배수로 동적으로 할당함.
- ZGC의 최대 장점은 힙 크기가 증가하더라도 STW 시간은 10ms를 절대로 넘기지 않는 다는 것
- 이 또한 CPU 사용량 증가와 튜닝 복잡성이 단점임.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIKx8x%2FbtrIT9f9qjM%2Ff1g0NHSGJI7ce9rfxPZyuk%2Fimg.png)

```bash
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -jar Application.java
```