`출처` [Inpa Dev JVM 내부 구조 & 메모리 영역 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8)

# 🔴 JVM

![JVM 구조](https://blog.kakaocdn.net/dn/m74dO/btrIJWO3Muu/dDZORkl1zCZVoRnhzzgGL0/img.png)

위 그림에서 붉은색으로 표시된 곳이 JVM이 동작하는 위치이다.

## 🟠 JVM 동작 방식

JVM의 역할은 .class 파일을 Class Loader를 통해 읽어 자바 API와 함께 실행하는 것이다.

![class loader 구조](https://blog.kakaocdn.net/dn/c7Bhxs/btrOnrIhmOx/QwXlPzLKq5UXekkb11FWhk/img.png)

1. java를 실행하면 JVM은 OS로 부터 메모리를 할당 받는다.
2. javac가 .java 파일을 .class 파일로 컴파일 한다.
3. Class Loader는 동적 로딩을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area에 등록한다.
4. Runtime Data Area에 로딩된 바이트 코드는 Execution Engine을 통해 해석된다.
5. 이 과정에서 Execution Engine에 의해 GC의 작동과 Thread 동기화가 이루어진다.

![class loader 세분화된 구조](https://blog.kakaocdn.net/dn/bmmGcd/btrIFHKUbDl/UXMSZ9pwyfqWQKu4ri3sk1/img.png)

위의 내용을 좀더 세분화하였고 JVM의 구성은 다음과 같다.

1. Class Loader
2. Execution Engine
    - Interpreter
    - JIT Compiler(Just In Time Compiler)
    - Garbage Collector
3. Runtime Data Area
    - Method Area
    - Heap Area
    - Stack Area
    - PC Register
    - Native Method
4. JNI(Java Native Interface)
5. Native Method Library

### 🟢 Class Loader

Class Loader는 .class 파일을 동적으로 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈이다.

로드된 .class 바이트 코드들을 엮어서 JVM의 메모리 영역인 Runtime Data Areas에 배치한다.

클래스 파일의 로딩 순서는 3단계(Loading -> Linking -> Initialization)로 구성된다.

`Loading(로드)` .class 파일을 로드하고 JVM의 메모리에 로드한다.

`Linking(링크)` 클래스 파일을 사용하기 위해 검증하는 과정이다.

- Verifying : 읽어들인 클래스가 JVM 명세에 명시된 대로 구성되었는지 검사한다.
- Preparing : 클래스가 필요로 하는 메모리를 할당한다.
- Resolving : 상수 풀 내 모든 심볼릭 레퍼런스를 다이렉트 레퍼런스로 변경한다.

`Initialization(초기화)` 클래스 변수들을 적절한 값으로 초기화한다. (static 필드들을 설정된 값으로 초기화)

### 🟢 Execution Engine

Class Loader를 통해 런타임 데이터 영역에 배치된 바이트 코드를 명령어 단위로 읽어 실행한다.

`Interpreter` 바이트 코드 명령어를 하나씩 읽어서 해석하고 바로 실행한다.

`JIT Compiler` Interpreter의 단점을 보완하기 위해 도입된 방식으로, 반복되는 코드를 발견하여 바이트 코드 전체를 컴파일하여 Native Code로 변경하고 이후에는 해당 메서드를 더 이상 Interpreter에서 실행하지 않고 캐싱해두었다가 Native Code로 직접 실행한다.

바이트 코드를 Native Code로 변환하는 것에도 비용이 발생하기 때문에 Interpreter에서 실행이 일정 기준이 넘어가면 JIT Compiler 방식으로 변경하여 실행하는 방법으로 진행된다.

`GC` Heap 메모리 영역에서 더는 사용하지 않는 메모리르 자동으로 회수해 준다. 자동으로 실행되지만 GC가 실행되는 시간은 정해져있지 않다. 특히 Full GC가 발생하는 경우 모든 스레드가 중지되기 때문에 장애가 발생할 수도 있다.

### 🟢 Runtime Data Area

JVM 메모리 영역으로 java application을 실행할 때 사용되는 데이터들을 적재하는 영역이다.

![Runtime Data Area](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnwTr9%2FbtrIMCwazct%2FYXxiCWEvxFyU9GJ0XxTkhK%2Fimg.png)

| 영역 | 용도 | 사용기간 | 쓰레드 공유 |
|-------|-------|-------|-------|
| Method(static) | JVM에서 읽어온 클래스와 인터페이스에 대한 런타임 상수 풀, 메서드와 필드, Static 변수, 메서드 바이트 코드 등을 보관 | JVM 시작시 생성 후 프로그램 종료까지, 명시적으로 null처리를 해주어야 GC 대상 | 모든 쓰레드에서 공유 |
| Runtime Constant Pool | Method Area 영역에 포함되지만 독자적 중요성을 띈다. 클래스와 인터페이스 상수, 메서드와 필드에 대한 모든 레퍼런스 저장 | 위와 동일 | 위와 동일 |
| Heap Area | 프로그램 상 데이터를 저장하기 위해 런타임 시 동적으로 할당, New 연산자를 통해 생성한 객체 또는 인스턴스와 배열을 저장 | 위와 동일 | 위와 동일 |
| Stack Area | 메서드 호출 시 생성되는 쓰레드 수행정보를 기록, Frame 저장, 메서드 정보, 지역 변수, 매개 변수, 연산 중 발생하는 임시 데이터 저장 | 메서드가 끝날때에 | 각 쓰레드별로 생성 |
| PC Register | CPU 명령어를 수행한다. 수행하는 동안 필요한 정보를 CPU내 기억장치인 레지스터에 저장, 연산 및 결과값을 메모리에 전달하기 전 기억장치 |  | 위와 동일 |
| Native Method Stack | 자바 외 언어로 작성된 네이티브 코드를 위한 메모리 | native interface 호출시, 쓰레드 종료시 생성 | 위와 동일 |

![Runtime Data Area](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbrvkNA%2FbtrIPYdYwMf%2FC5wCqtVNLvu642vOaGyCBK%2Fimg.png)

위에 설명으로 Runtime Data Area를 자세하게 다시 표현한 그림이다.

### 🟢 Method Area

JVM이 시작될 때 생성되는 공간으로 .class를 처음 메모리 공간에 올릴 때 초기화 되는 대상을 저장하기 위한 메모리 공간이다.

JVM이 동작하고 클래스가 로드될때 적재되어서 프로그램이 종료될 때까지 저장된다.

`Field Info` 멤버 변수의 이름, 데이터 타입, 접근 제어자 정보

`Method Info` 메소드 이름, return 타입, 함수 매개변수, 접근 제어자의 정보

`Type Info` Class인지 Interface인지 여부 저장, Type의 속성, Super Class의 이름

Method Area는 정적필드와 클래스 구조만을 갖고 있다.

### 🟢 Heap Area

메서드 영역과 함께 모든 쓰레드가 공유하며, JVM이 관리하는 프로그램 상에서 데이터를 저장하기 위해 런타임 시 동적으로 할당하여 사용하는 영역이다.

new 연산자로 생성되는 클래스와 인스턴스 변수, 배열 타입 등 Reference Type이 저장되는 곳이다.

힙 영역에 생성된 객체와 배열은 Reference Type으로 JVM 스택 영역의 변수나 다른 객체의 필드에서 참조가 된다.

즉, 힙의 참조 주소는 Stack Area가 갖고 있고 해당 객체를 통해서만 힙 영역에 있는 인스턴스를 핸들링 할 수 있다. 만일 참조하는 변수나 필드가 없다면 의미 없는 객체가 되어 GC에서 객체를 힙 영역에서 제거한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqEBMZ%2FbtrINtTDQbX%2FMJ2ODVUaLoVPvauKUBXsgK%2Fimg.png)

`GC` 힙영역은 GC의 대상이 되는 공간으로 GC는 효율적으로 수행되기 위해 세부적으로 5가지 영역으로 나뉘어서 실행된다.

![GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdiVa4N%2FbtrIK82DPjx%2FY2VQwZAqs7LKoebT7kucI1%2Fimg.png)

### 🟢 Stack Area

스택 영역은 int, long, boolean 등 기본 자료형을 저장할때 사용되는 임시적 정보들이 저장되는 영역이다.

![stack](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FddIT0y%2FbtrILfIfvEw%2F7IkVFuVRKNYOSHhXq53bW1%2Fimg.png)

위에서 설명한 Heap 영역에 Reference 값을 Stack에 갖고 있고 new 연산자로 생성된 객체는 Heap에 저장된다.

프로세스가 메모리에 로드될 때 스택 사이즈가 고정되어 있어, 런타임 시 스택 사이즈를 바꿀 수 없다. 만일 JVM 스택에서 프로그램 실행 중 메모리 크기를 넘었을 경우에는 StackOverFlowError가 발생한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSRMQd%2FbtrIO2VyUeh%2FsB2wwS8Sua0MswEszZEEJ1%2Fimg.png)

Runtime Data Area는 다음과 같이 표현할 수 있다.

### 🟢 PC Register

쓰레드가 시작될 때 생성되며, 현재 수행중인 JVM 명령어 주소를 저장하는 공간이다.

일반적으로 프로개름 샐행은 CPU에서 명령어를 수행하는 과정이다. 이때 CPU는 연산을 수행하는 동안 필요한 정보를 레지스터라고 하는 CPU 내 기억장치에 저장하고 이요하게 되는데 이를 Register라고 한다.

자바에서 PC Register는 CPU Register와 다르게 OS나 CPU 입장에서는 자바 또한 하나의 프로세스이기 때문에 JVM의 리소스를 이용해야 한다. 그래서 자나느 CPU에서 직접 연산을 수행하도록 하는 것이 아닌, 현재 작업 내용을 CPU에게 연산으로 제공하며 이를 위한 버퍼 공간으로 PC Register라는 메모리 영역을 갖고 있는 것이다.

### 🟢 Native Method Stack

자바 코드가 컴파일되어 생성되는 바이트 코드가 아닌 실제 실행할 수 있는 기계어로 작성된 프로그램을 실행시키는 영역이다.

또한 자바 이외에 언어로 작성된 네이트브 코드를 실행하기 위한 공간이기도 하다.