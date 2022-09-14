---
title: "자바 데이터 타입, 변수 그리고 배열"
categories: 
    - java
date: 2021-04-10
last_modified_at: 2022-07-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "자바의 데이터 타입 및 변수, 배열에 대해 알아보자"
---
- #### **Primitive Type(원시 타입)과 Reference Type(참조 타입)**
  - **Primitive Type(원시 타입)**<br/>
  자바에 내장되어 있는 기본형(원시) 타입으로써 미리 정의된 정수, 실수, 논리, 문자형의 타입을  제공한다. 
    - default 값이 있기 때문에 Null이 존재하지 않는다.
    - Stack(스택) 메모리에 실제 값을 저장한다.
  
  - **Reference Type(참조 타입)**<br/>
  앞서 설명한 원시 타입을 제외한 모든 타입을 참조 타입이라고 한다. (배열, 열거, 클래스, 인터페이스 타입이 여기에 속한다.)
    - Null이 존재한다. (빈 객체를 의미함)
    - Heap(힙) 메모리에 실제 값이 아닌 **실제 값이 저장된 주소 값**을 저장한다.
  
- #### **Primitive Type 종류와 값의 범위 그리고 기본 값**
![](https://images.velog.io/images/ckr3453/post/5ee1b95a-303a-4730-969d-ed7428c4b949/image.png)<br/>**(이미지 출처 : https://kingpodo.tistory.com/54)**
- #### **literal(리터럴)**
소스 코드의 고정된 값을 의미 한다. 그렇다면 Constant(상수) 와 같은 의미 일까? 아니다. 상수는 값 자체(리터럴)를 담기위해 메모리에 할당된 **공간**이다. 예를 들어보자.
  ```java
  final int test = 5;
  ```
  **test**라는 상수(공간)에 **5**라는 정수 리터럴(값 그자체)을 담았다. 
  리터럴은 상수와 똑같이 값이 변하지 않도록 저장되지만 공간과 값 그자체 라는 차이에서 엄연히 다르다. (상수는 test라는 이름을 가지고 있고 리터럴은 값 그자체이기 때문에 이름이 없다)
- #### **변수 선언 및 초기화하는 방법**
변수를 선언 및 초기화 하는 방법에는 3가지가 있다.
  - **명시적 초기화(explicit initialization)**<br/>
  가장 많이 사용하고 기본적인 방식으로써 선언과 동시에 초기화를 한다.
  ```java
  public class Test{
  	int test = 5;
  }
  ```
  **멤버 변수**의 경우 초기화를 하지 않고 선언만 할 시 타입에 해당하는 기본 값으로 초기화가 이루어진다. 그러나 **지역 변수**의 경우 초기화를 하지 않으면 컴파일 에러가 발생한다.
  (컴파일러에 의해 멤버 변수는 체크가 가능하나, 지역 변수는 체크하지 않는다.)
  ```java
  public class Test{
  	int test;	   // 정상 (컴파일 단계에서 컴파일러가 체크하여 타입에 맞는 기본값으로 부여함.)
  	int test1 = test;  // 정상
     
     public void local(){
     	int a;		// 초기화 되지않음.
     	int b = a;	// 컴파일 에러 발생 (초기화 되지 않은 변수를 부여하려 했기 때문.)
     }
  }
  ```
  되도록 선언과 동시에 초기화를 하는것이 바람직하다.
  - **생성자(constructor)**<br/>
  new 키워드를 사용, 생성자를 이용하여 초기화 하는 방법이다. 생성자는 **클래스의 인스턴스 변수 초기화 메서드**로써 인스턴스 변수를 초기화 할때 혹은 인스턴스 생성 후 실행할 작업을 위해서 사용된다. 생성자를 사용하기 위해선 클래스 내에 클래스와 동일한 이름의 메서드를 생성해야 한다. (매개변수의 타입이나 갯수를 다르게 하여 오버로딩을 활용한 초기화가 가능하다.)
  ```java
  public class TestMain {
      String a = "";
      public TestMain(){
          this.a = "기본 생성자";
      }
      public TestMain(String a){
          this.a = a;
      }

      public static void main(String[] args) {
          TestMain tm = new TestMain("생성자를 통한 초기화"); // 매개변수를 갖는 생성자 호출
          System.out.println(tm.a);	// a = "생성자를 통한 초기화"
      }
  }
 ```
  - **초기화 블록(initialized block)**<br/>
  클래스 필드의 복잡한 초기화를 담당하는 블록을 의미하며 중괄호 { } 안에 구문을 작성한다. 초기화 블록은 인스턴스 초기화 블록, 클래스 초기화 블록으로 구분 된다. 각 블록 별로 실행되는 시점이 다르며, 인스턴스/클래스 별 공통 선행 작업에 대한 코드 중복을 최소화 할 수 있다.
  
    - **인스턴스 초기화 블록(instance initialized block)**<br/>
    클래스 인스턴스가 생성 될 때마다 수행된다. **생성자 호출 이전에 수행** 되므로 각각의 생성자를 통해 인스턴스를 초기화 할때 마다 수행되어야 할 선행 작업이 있다면 코드 중복을 최소화 할 수 있다.
    - **클래스 초기화 블록(class initialized block)**<br/>
    스태틱 초기화 블록 이라고도 부르며 클래스가 로딩 될때 딱 한번 실행된다. (각 클래스당 최초 1회) **인스턴스 초기화 블록 호출 이전에 수행** 되므로 클래스 로딩 시에 수행되어야 할 공통 선행 작업을 실행할 때 유용하다. 클래스 초기화 블록은 중괄호 앞에 static을 붙인다.
      ```java
      public class TestMain {
          String a = "";
          public TestMain(){
              this.a = "기본 생성자";
              System.out.println("기본 생성자 호출");
          }
          public TestMain(String a){
              this.a = "매개변수 생성자";
              System.out.println("매개변수 생성자 호출");
          }
          
          {
          	System.out.println("인스턴스 생성");
          }
          
          static {
          	System.out.println("클래스 로딩");
          }

          public static void main(String[] args) {
              TestMain tm2 = new TestMain();
              TestMain tm1 = new TestMain("생성자를 통한 초기화"); 
              
              // 결과
              // 클래스 로딩
              // 인스턴스 생성
              // 기본 생성자 호출
              // 인스턴스 생성
              // 매개변수 생성자 호출
              
          }
      }
     ```
- #### **변수의 Scope와 Lifetime**<br/>
자바에서는 영역에 따른 변수의 종류 마다 사용할 수 있는 범위(Scope)와 생명시간(Lifetime)이 각각 존재한다.

  |**변수 종류(Variable Type)**|**범위(Scope)**|**생명시간(Lifetime)**|
  |---|---|---|
  |인스턴스 변수 (Instance Variable)|클래스 내 모든 영역 (static 영역의 경우 인스턴스를 생성해야한다.)|객체 생성 이후 ~ 메모리내에서 제거되는 시점|
  |클래스 변수 (Class Variable)|클래스 내 모든 영역|클래스 초기화 이후 ~ 프로그램 종료 시점|
  |지역 변수 (Local Variable)|해당 변수가 선언된 블록 내 모든 영역|변수 선언 이후 ~ 블록을 벗어나는 시점|

  ```java
  public class Test {
  	String instanceVar = "instanceVar"; // 인스턴스 변수, 멤버 변수라고도 불리며 클래스 내 모든 영역에서 사용 가능 하다. 그러나 static 영역의 경우, 인스턴스를 생성해야 사용 가능하다. (Instance Variable)
  	static String classVar = "classVar"; // 클래스 변수, 클래스 내에서 공유되며 어떤 곳이든 사용 가능하다. (Class Variable)
  	
  	public void testMethod() {
  		String localVar = "localVar"; // 지역 변수, Method 영역(블록)에 선언한 변수(Local Variable)
        		System.out.println(classVar); // 가능
          		System.out.println(instanceVar); // 가능
          		System.out.println(localVar); // 가능
  	}
  
  	public static void main(String[] args) {
  		// static 영역의 경우 인스턴스 변수 사용 시, 인스턴스를 생성하여 사용해야한다.
  		System.out.println(classVar); // 가능
                  System.out.println(localVar); // 에러 발생, 변수가 선언 된 영역을 벗어남.
		Test test = new Test();
		System.out.println(test.instanceVar); // 가능
	}
  ```
- #### **형변환**<br/>
**형변환** 은 어떠한 변수 혹은 상수의 타입을 다른 타입으로 바꾸는 것을 의미하며 프로그램 실행 도중 자동으로 형변환 되는 경우인 **묵시적 형변환(Type Promotion)**, 형 변환될 타입을 직접 명시하는 경우인 **명시적 형변환(Type Casting)**으로 구분 된다.
  - **묵시적 형변환(Type Promotion)**<br/>
  자동 형변환 이라고도 부르며 프로그램 실행 도중 type 별 byte 크기에 따라 작은 크기의 type이 큰 크기의 type으로 저장 될 때 자동적으로 형변환이 일어난다.
  
    - **Type 별 byte 크기**
  >byte(1) < short/char(2) < int(4) < long(8) < float(4) < double(8)
  **float이 4byte 임에도 long 보다 큰 이유는 표현할 수 있는 값의 범위가 long보다 더 크기 때문이다.**
  
  ```java
  byte byteVar = 2;
  short shortVar = byteVar // 묵시적 형변환 발생
  ```
  그러나 char 타입(표현 범위 0~65535)은 음수를 표현할 수 없기 때문에 음수를 표현할 수 있는 **byte 타입에서 char 타입으로의 묵시적 형변환은 일어나지 않는다**. (명시적 형변환의 경우는 가능)
  
  ```java
  byte byteVar = -2;
  char charVar = byteVar // 컴파일 에러 발생
  ```
  - **명시적 형변환(Type Casting)**<br/>
  강제 형변환 이라고도 부르며 Casting 연산자 ()를 사용해 타입을 직접적으로 명시함에 따라 큰 크기의 type을 작은 크기의 type으로 강제 변환 할 수 있다. 그러나 **값의 범위가 변환 하려는 값의 범위를 초과 하는 경우 값 손실이 일어나며 원래 값이 보존되지 않는다.**
  ```java
  int intVar = 2;
  byte byteVar = (byte)intVar; // 명시적 형변환, 원래 값 보존, 2를 return
  
  int intVar2 = 999999;
  byte byteVar2 = (byte)intvar; // 명시적 형변환, 원래 값을 보존하지 못함, 63을 return
  ```
  두번째 코드의 경우, int(4byte)의 byte(1byte) 값 만큼만 잘라서 저장한다.
  999999 = ~~00001111~~ ~~01000010~~ **00111111 (63을 저장)**
  
  - ** 값 뿐만 아니라 객체 참조변수도 형변환이 가능하다.**