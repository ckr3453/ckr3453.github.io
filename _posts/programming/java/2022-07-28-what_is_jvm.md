---
title: "JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가"
categories: 
    - java
date: 2021-02-03
last_modified_at: 2022-07-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JVM의 동작 방식 및 구성을 알아보자"
---
- #### **JVM이란 무엇인가**
**자바 가상 머신(Java Virtual Machine, JVM)은** 시스템 메모리를 관리하면서 자바 기반 애플리케이션을 위해 이식 가능한 실행 환경을 제공합니다. 즉, 자바 언어로 작성된 프로그램을 실행시키기 위한 소프트웨어입니다.

  - **왜 굳이? 그냥 실행시키면 안되나?**
![](https://images.velog.io/images/ckr3453/post/62baf0df-d825-4785-b5ef-fecef6eb7c5c/image.png) <br/>**(이미지 출처: https://gbsb.tistory.com/2)**<br/><br/>
일반적으로 프로그램은 운영체제 위에서 실행하게 됩니다. 운영체제마다 CPU가 다르고 실행파일 형식이 다르기 때문에 각 운영체제에 맞춰 코드를 작성해야 하는 불편함이 생깁니다. 이를 보완하기 위해 자바 프로그램은 운영체제 위에 JVM이라는 독립적인 환경위에서 실행이 가능하게 설계되어 JVM만 있으면 어느 운영체제든 실행이 가능합니다. (WORA, Write Once Read Anywhere) 즉, JVM은 자바 프로그램과 운영체제 사이의 중개자 역할인 셈입니다.
	
- #### **컴파일 하는 방법**
  - **Compile**<br/>
    **컴파일이란** 사용자가 고급언어(C,C++,Java,Python,..)로 작성한 코드를 컴퓨터가 인식할 수 있는 저급언어(기계어)로 변환시켜주는 과정을 뜻하며 JDK가 가지고있는 Java compiler를 통해 컴파일 작업을 수행할 수 있습니다.
    (좀 더 정확하게는 사용자가 작성한 **.java 파일을 기계가 읽을 수 있도록 bytecode로 변환하는 과정**이며, 이 과정에서 .class 파일이 생성되며 이 파일이 bytecode를 담고 있습니다.)
  - **컴파일 및 실행과정**
    0. **진행 전, 자신의 운영체제에 맞는 JDK를 설치해야 합니다.**<br/>
    ![](https://images.velog.io/images/ckr3453/post/db1bca3a-1002-49f1-b5b3-db7f7f7542c5/image.png)
    1. 아무 에디터(메모장, 워드패드.. 등등)나 열어서 간단한 코드를 작성하고 확장자를 .java로 저장합니다.<br/>
    ![](https://images.velog.io/images/ckr3453/post/34db744a-9008-4e68-a1a2-8136a50f315d/image.png)
    2. 명령 프롬프트 창에 "javac (작성한 파일명.java)" 명령어를 실행합니다.<br/>
    3. 작성한 .java 파일이 컴파일되어 해당 위치에 .class 파일이 생성 됨을 확인할 수 있습니다.<br/>
   	![](https://images.velog.io/images/ckr3453/post/daefdc66-0617-49e8-9266-dffa85b6998f/image.png)
    4. 생성된 .class 파일을 에디터로 열어보면 자신이 작성한 소스코드가 컴파일된 내용을 볼 수 있지만 해석이 되지않아 이해하기 힘듭니다. "javap -c (파일명.class)" 명령어를 사용하면 해석된 내용을 볼 수 있습니다.<br/>
    ![](https://images.velog.io/images/ckr3453/post/2ab70eba-9d1c-4460-82cf-914ad399305b/image.png)
    5. java 명령어를 통해 잘 실행되는지 확인해 봅시다.<br/>
- #### **바이트코드(Bytecode)란 무엇인가**
바이트코드란 JVM이 이해할 수 있는 중간 단계의 언어로써 JVM이 사용자가 작성한 .java 소스 코드 파일을 운영체제에 실행가능한 명령어 집합 파일로 2차 컴파일하는 과정 중에 필요한 코드입니다. 사용자가 작성한 .java 파일이 JVM에 도달하기 전에 Java compiler에 의해 1차 컴파일이 일어나는데 이때 .java 소스 코드 파일이 바이트코드(.class)로 변환됩니다. 이처럼 JVM이 운영체제와 실제 실행환경 중간에 위치하면서 자바 프로그램은 운영체제와 실행환경에 종속받지 않고 WORA의 실현이 가능합니다.
- #### **JIT compiler란 무엇이며 어떻게 동작하는가**<br/>
![](https://images.velog.io/images/ckr3453/post/eea9d5ee-05d5-419d-8cd5-f0f7f040d6c9/image.png)<br/><br/><br/>
**JIT compiler란** Just-in-time compilation의 약자이며 interpreter 언어의 단점을 보완하기 위해 도입된 방식입니다. bytecode는 interpreter 언어로 분류되며 실행 시 한줄씩 읽고 해석하고, 그에 해당하는 기능을 실행합니다. 그렇기 때문에 속도면에서 많이 느립니다. JIT compiler는 컴파일된 bytecode를 프로그램 실행 중에(실시간으로) 해당 플랫폼에 맞는 native code로 번역해주는 특수 컴파일러 입니다. JIT 컴파일러는 같은 코드를 매번 해석하지 않고 바뀐부분만 컴파일하여 속도적인 측면에서 많이 향상됩니다.
- #### **JVM 구성 요소**
![](https://images.velog.io/images/ckr3453/post/b087aad4-c0fc-4e9a-a289-a388fceaf6c6/image.png)**(이미지 출처: https://www.notion.so/3-JVM-ff2053692a9b4665bf41420160de00e5)**<br/><br/>
JVM은 크게 Class Loader(클래스 로더), Execution Engine(실행 엔진), Runtime Data Areas(런타임 데이터 영역) 으로 구성되어 있습니다.
  - **Class Loader (클래스 로더)**
  JVM 내로 .class 파일을 load 하는 역할을 수행합니다. 클래스 로더는 Loading(로딩), Linking(링킹), Initialization(초기화) 과정을 거칩니다.
    - Loading (로딩)
    .class 파일을 바이트 코드로 읽어서 메모리에 로드합니다.
    - Linking (링킹)
    바이트 코드로 읽어온 .class 파일을 검증하는 절차를 거칩니다. (파일이 적절히 포맷되었는지, 유효한 컴파일러에 의해 생성된 것인지.. 등)
    - Initialization (초기화)
    .class 파일 내 모든 정적 변수들이 정의된 값으로 초기화 됩니다.
  - **Runtime Data Areas (런타임 데이터 영역)**
  .class 파일들이 적재되어 있으며 JVM의 메모리 역할을 수행합니다.
    - Method Area (메서드 영역)
    클래스 멤버 변수 이름, 타입, 리턴 타입 등 Class Level의 정보가 저장됩니다.
    - Heap (힙 영역)
    new 연산자로 생성 한 객체의 정보가 저장됩니다.
    - Java Thread (스레드 영역)
    스레드가 시작될 때 생성되는 영역이며 세부적으로는 PC Register, Stack(JVM Stack), Native Method Stack으로 구분할 수 있습니다. 해당 영역에선 수행중인 JVM 명령주소, 모든 스레드에 대한 런타임 스택, Native 메소드 정보를 저장합니다.
  - **Execution Engine (실행 엔진)**
  런타임 데이터 영역에 할당된 .class 파일이 실행 엔진에 의해 실행 됩니다. Interpreter, JIT Compiler 에 의해 바이트 코드를 읽고 해석됩니다.
- #### **JDK와 JRE의 차이**
![](https://images.velog.io/images/ckr3453/post/272c90e7-1fce-4727-abdc-da74d4ccdfea/image.png)
  - **JRE**
  Java Runtime Environment의 약자로 JRE는 자바 프로그램을 실행시킬 수 있게끔 환경을 구성해준다. JVM 실행환경을 포함하고 있으며 단순히 실행만을 위한 환경 구성을 지원하기 때문에 개발을 해야할 경우 JDK를 설치해야 합니다.
  - **JDK**
  Java Development Kit의 약자로 JDK는 JRE와 더불어 자바 프로그램을 개발할때 필요한 컴파일러, 툴킷 등을 제공한다. 큰 틀에서 보았을때 JDK에 JRE, JVM이 모두 포함되어 있습니다.
