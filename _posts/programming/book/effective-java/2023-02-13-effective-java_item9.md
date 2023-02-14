---
title: "이펙티브자바 - 아이템9) try-finally 보다는 try-with-resources를 사용하라. (작성중)"
categories: 
    - book
date: 2023-02-13
last_modified_at: 2023-02-13
toc: true
toc_sticky: true
excerpt: "try-with-resources를 통한 자원 관리"
---

## 개요

자바 라이브러리는 `close()` 를 호출하여 직접 닫아줘야 하는 자원이 많다. (ex. `InputStream`, `OutputStream`, `java.sql.Connection` 등)

자바에서 이러한 자원들을 닫는 수단으로 `try-finally`를 많이 사용한다.

그러나 직접 호출해야하는 특성때문에 사용자가 자원을 닫지않고 놓치는 경우가 많아 성능 문제로 이어지기도 한다.

또한 안전망 역할로 `finalizer`를 활용한다고 해도 `finalizer` 자체가 안정적이지 않기때문에 믿음직스럽지 않다.

이번 아이템에선 이러한 `try-finally` 방식 대신 `try-with-resources` 방식을 통한 안전한 자원 관리 방법을 제안한다.

## try-finally의 문제점

`try-finally`는 보통 다음과 같이 `finally` 블럭에서 자원의 `close()`를 직접 호출하여 자원을 닫아준다.

```java
public static void fistLineOfFile(String path) throws IOException {
    BufferedReader br = new Bufferedreader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

자원 하나만 사용했을 경우에는 나쁘지 않다. 하지만 2개 이상의 자원을 사용한다면 다음과 같은 형태가 될 것이다.

```java
public static void fileCopy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } finally {
            out.close(); // 자원 반납
        }
    } finally {
        in.close(); // 자원 반납
    }
}
```

2개 이상의 자원을 다룰때 `try-finally`를 사용하면 다음과 같은 문제점이 있다.

1. 


## try-with-resources를 사용하자.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item9)<br/>