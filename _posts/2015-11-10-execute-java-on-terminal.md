---
layout: post
title: "터미널에서 java 실행하기"
tags: [java]
comments: true
---

## 먼저 컴파일
.java 파일이 있는 곳에 가서 `javac [파일이름].java` 하면 된다.

## 실행하기
실행하는 건 `java` 명령어로 가능한데 이게 이것만으로는 안된다. java는 클래스 이름이 겹치는 경우도 많아서 꼭 package 이름을 쓰고 실행해주어야 함!

- 만약 `test`라는 package 안에 자바 파일들이 들어 있다면 다음과 같이 실행한다.

  ```
  cd test
  java -cp .. test.[class_name]
  ```

- 한가지 예를 더보자. 만약 java class 파일의 위치가 `/home/workspace/java/Chatting/test/a.class`라면, `/home/workspace/java/Chatting/` 위치에서 다음과 같이 실행한다.

  ```
  $ pwd
  /home/workspace/java/Chatting/
  $ java -cp . test.a
  ```
