---
title: "이펙티브자바 - 아이템8) finalizer와 cleaner 사용을 피하라.(작성중)"
categories: 
    - book
date: 2023-02-06
last_modified_at: 2023-02-07
toc: true
toc_sticky: true
excerpt: "finalizer와 cleaner 사용을 지양하자."
---

## 개요

자바의 객체 소멸은 가비지 컬렉터가 담당한다. 그 외에도 `finalizer`와 `cleaner`라는 두가지 객체 소멸자를 제공한다. 

그러나 두가지 모두 **예측할 수 없고, 느리고, 상황에따라 위험할 수 있기 때문**에 기본적으로 '사용하지 말것'을 권장하고 있다.

왜 기본적으로 쓰지 말아야 하는지 알아보자.

## 수행 시점 및 수행 여부를 보장하지 않는다.

객체에 접근 할 수 없게된 후 (=소멸 대상) finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없다.

즉, 제때 실행되어야 하는 작업은 절대 할 수 없다.

finalizer와 cleaner의 수행 속도는 전적으로 **가비지 컬렉터 알고리즘**에 달렸다.(가비지 컬렉터 구현마다 천차만별)

finalizer와 cleaner가 수행되리라는 보장도 없다. 시스템이 종료될때까지 소멸대상 객체를 소멸 시키지 못할수도 있다는 의미다.

실행을 보장하는 `System.runFinalizersOnExit`와 `Runtime.runFinalizersOnExit` 메서드가 존재하지만 치명적 결함으로 인하여 deprecated 되었다.

> 치명적 결함 -> 다른 스레드가 소멸대상의 객체에 접근하고 있어도 실행해 버린다는 매우 치명적인 결함이 있다.

## 예외 발생 무시 (finalizer 한정)

finalizer 동작 중 발생한 예외는 무시되며 처리할 작업이 남아있어도 그 순간 바로 종료된다.

보통 예외가 발생하면 스레드를 중단 시키고 스택 내용을 출력하지만, finalizer에서 예외 발생 시 경고조차 출력하지 않는다.

잡지 못한 예외 때문에 훼손된 객체가 남아있어서 어떻게 동작할 지 예측할 수 없게 된다.

(cleaner는 사용하는 라이브러리가 자신의 스레드를 통제하기 때문에 이런 문제가 발생하지 않는다.)

## 성능 문제


## 보안 문제


## 📣 Reference


[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item8)<br/>
[]()<br/>
