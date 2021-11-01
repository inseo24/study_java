* 이 글은 백기선님의 더 자바, 코드를 조작하는 다양한 방법이라는 강의를 듣고 정리한 내용입니다. 


# 자바, JVM, JDK 그리고  JRE

JVM은 바이트코드를 실행하는 표준이자 구현체(특정 벤더가 구현)다. - 바이트코드는 클래스 파일 안에 들어 있음.
이 바이트코드를 실행하는게 인터프리터와 JITCompiler, 각 native OS에 맞게 머신코드로 변경한 다음에 실행함(특정 플랫폼에 종속적)

JRE = JVM + Library

라이브러리 안에는 rt.jar 파일이나 chracterset.jar 파일, 프로퍼티 셋팅 등이 제공됨
개발하는데 필요한 도구가 제공되진 않음, javac가 없음

### JDK

- JRD + 개발에 필요한 툴
- 자바 9부터 모듈 시스템이 도입, 이 모듈 시스템을 사용해 자신만의 JRE를 만들 수도 있음(jlink)
- 이런 툴을 이용해서 서브셋을 구성할 수 있음
- 오라클에서 자바 11부터 JDK만 제공하고 JRE는 제공하지 않음

### 자바

- 프로그래밍 언어
- JDK에 들어있는 자바 컴파일러를 사용해 바이트코드(.class)로 컴파일 할 수 있음
- 오라클에서 만든 Oracle JDK 11 버전부터 사용으로 사용할 때 유료
    - 오라클 Open JDK 무료
    - 오라클 외의 아마존 등(Corretto)에서 만든 Open JDK 무료

[https://medium.com/@javachampions/java-is-still-free-c02aef8c9e04](https://medium.com/@javachampions/java-is-still-free-c02aef8c9e04)

### 타 프로그래밍 언어 지원

- 사실상 .class 파일만 있으면 JVM이 실행을 해줌
- 자바와의 의존성이 타이트하지 않아서 다른 언어로 코딩해도 그 언어를 컴파일 했을 때 .class파일이 만들어지거나 자바 파일을 만들어주기만 하면 JVM을 활용할 수 있음
- JVM 기반으로 동작하는 프로그래밍 언어
    - 클로저, 그루비, JRuby, Jython, Kotlin, Scala

참고자료

- JIT 컴파일러: [https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/](https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/)
- JDK, JRE 그리고 JVM: [https://howtodoinjava.com/java/basics/jdk-jre-jvm/](https://howtodoinjava.com/java/basics/jdk-jre-jvm/)
- [https://en.wikipedia.org/wiki/List_of_JVM_languages](https://en.wikipedia.org/wiki/List_of_JVM_languages)

# JVM 구조

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45c1d9b4-9ee1-4e9a-815a-aba0e8a62da2/Untitled.png)

### 클래스로더

- .class에서 바이트 코드를 읽고 메모리에 저장
- 로딩: 클래스를 읽어오는 과정
- 링크: 레퍼런스를 연결하는 과정
- 초기화: static 값들 초기화 및 변수에 할당

### 메모리

- 메소드 영역에는 부모 클래스 이름, pull package name, 메소드, 변수 등이 저장된다. 공유 자원이다.
- 힙 영역에는 객체를 저장한다. 공유 자원이다.
- 스택 영역에는 쓰레드마다 런타임 스택을 만들고, 그 안에 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다. 쓰레드가 종료하면 런타임 스택도 사라진다.
- PC(Program Counter) 레지스터: 쓰레드마다 쓰레드 내 현재 실행할 스택 프레임을 가리키는 포인터가 생성된다.
- 네이티브 메서드 스택(JNI): 쓰레드마다 생김, 네이티브 메서드 호출할 때 쓰이는 별도의 메서드 스택이라고 생각하면 됨
    - [https://javapapers.com/core-java/java-jvm-run-time-data-areas/#Program_Counter_PC_Register](https://javapapers.com/core-java/java-jvm-run-time-data-areas/#Program_Counter_PC_Register)
    - 메서드에 native라는 키워드가 붙어 있고, 그 구현을 자바가 아닌 C나 C++로 한 걸 의미
    - 예를 들어, Thread.currentThread() 메서드는 자바로 구현된게 아니라 C로 구현되어 있음.
    - 직접 native 메서드를 만들 수도 있음, 하지만 나는 써보기만 했지 만들어 본 적은 없다.
    - JNI(Java Native Interface)는 native로 구현된 메서드를 호출할 수 있게 해주는 인터페이스
        - [https://schlining.medium.com/a-simple-java-native-interface-jni-example-in-java-and-scala-68fdafe76f5f](https://schlining.medium.com/a-simple-java-native-interface-jni-example-in-java-and-scala-68fdafe76f5f)
    - 실제 구현된 그 자체를 네이티브 메서드 라이브러리라고 부름, 이 라이브러리는 항상 JNI를 통해서 써야 한다.

### 실행엔진

- 인터프리터: 바이트코드를 한줄씩 실행
- 반복적인 코드가 발생하면 JITCompiler로 보내서 미리 바꿔둠
- GC(Garbage Collector): 더이상 참조되지 않는 객체를 모아서 정리함
    - GC는 이해도 하고, 경우에 따라 커스터마이징도 해야할 때가 있음, 사용할 GC를 선택할 경우도 있음
    - Throughput 위주의 GC와 Stop The World 줄이는 GC가 있다. 그 중 적절한 걸 사용
    - 서버 운영 중에 많은 객체가 생성하고, response time이 굉장히 중요하면 Stop The World, 즉 GC를 사용하면서 생기는 pause 연산(멈춤 현상)을 최소화할 수 있는 GC 사용이 좋을 것
    - 자바 성능과 관련된 책에서 GC를 다룸

참고

- [https://www.geeksforgeeks.org/jvm-works-jvm-architecture/](https://www.geeksforgeeks.org/jvm-works-jvm-architecture/)
- [https://dzone.com/articles/jvm-architecture-explained](https://dzone.com/articles/jvm-architecture-explained)
- [http://blog.jamesdbloom.com/JVMInternals.html](http://blog.jamesdbloom.com/JVMInternals.html)

# 클래스 로더

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4dd4b206-6480-4095-b754-c69a842b43ba/Untitled.png)

### 클래스 로더

- 로딩, 링크, 초기화 순으로 진행
- 클래스 로더는 계층 구조로 되어있고 기본적으로 3가지 클래스 로더가 제공된다.

- 로딩
    - 클래스 로더가 .class 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만들고 "메소드" 영역에 저장한다.
    - 이 때 메소드 영역에 저장하는 데이터
        - FQCN(Fully Qualified Class Name) - 패키지 경로까지 포함한 패키지 이름 등
        - 클래스, 인터페이스, enum
        - 메소드와 변수
    - 로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성해 힙에 저장
    - 부트 스트랩 클래스 로더 - JAVA_HOME\lib에 있는 코어 자바 API를 제공, 최상위 우선순위를 가진 클래스 로더다. 네이티브 코드로 되어 있어서 볼 수 없음
    - 플랫폼 클래스로더 - JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다
    - 애플리케이션 클래스로더 - 애플리케이션 클래스패스(애플리케이션 실행할 때 주는 -classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치)에서 클래스를 읽음
    - 우리가 쓰는 대부분의 코드들은 AppClassLoader가 읽고, path에 추가한 것도 대부분 AppClassLoader가 읽음
    - 클래스를 읽어올 때 가장 부모인 부트스트랩에 부탁을 하고 없으면 아래 쪽으로 내려감, 만약 본인도 못 읽으면 ClassNotFoundException이 발생함(pom.xml에 의존성 제대로 추가안했을 때 생김)

- 링크
    - Verify, Prepare, Resolve(optional) 3단계로 나뉨
    - 검증: .class 파일 유효한지 체크
    - Preparation: 클래스 변수(static 변수)와 기본값에 필요한 메모리를 준비
    - Resolve: 심볼릭 메모리 레퍼런스를 메서드 영역에 있는 실제 레퍼런스로 교체한다. 나중에 사용할 때 교체하기도 하므로 Optional.

- 초기화
    - Static 변수의 값을 할당. (static 블럭이 있다면 실행된다.)