---
title: "이펙티브자바 - 아이템12) clone 재정의는 주의해서 진행하라. (작성중)"
categories: 
    - book
date: 2023-03-01
last_modified_at: 2023-03-01
toc: true
toc_sticky: true
excerpt: "clone 재정의 시 고려 사항"
---

## 개요

`clone()`은 대상 객체의 모든 필드 및 메서드를 얕은 복사하여 반환하는 동작을 수행하는 메서드다.

> 얕은 복사란? 값 자체를 복사하는것이 아니라 주소 값을 복사한다. 즉, 원본 객체가 수정되었을 시 얕은 복사를 통해 반환된 객체또한 수정된다.

`Cloneable` 인터페이스가 제공하는 `clone()`을 재정의하여 사용할 떄 주의사항을 알아보자.


## 정리


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item13)<br/>