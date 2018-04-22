---
layout: post
title: "Java ServerSocket과 Socket의 차이점"
tags: [java]
comments: true
---

## C API로 이해하기
- server에서 쓰일 때
  ```
  new ServerSocket(DEFAULT_PORT) //C의 socket()과 bind(), listen()이 합쳐진 것
  ```

- client에서 쓰일 때
  ```
  Socket socket = new Socket(serverAddress, DEFAULT_PORT);
  // C의 socket(), connect() 가 합쳐진 것
  ```

## 오늘의 질문
java의 method들이 어떤 역할을 하는지, 대략적으로 어떻게 구현되어 있는지 (는 몰라도 되나?) 를 알려고 찾아보면 Oracle 문서를 보게 되는데... Oracle 문서만으로는 뭔가 정확히 잘 모르겠는 느낌?! 예를 들어, printWriter, BufferedReader, InputSteamReader 얘네들도 어떻게 돌아가는 건지 자세히 알고 싶은데 다큐먼트만으로는 잘 모르겠다. 이해될 때까지 읽어보는 게 답인가?
- 이에 대해, 이전 블로그 글을 옮기고 있던 2018년 4월의 2년차 개발자인 내가 답해보면(ㅋㅋ), 자세히 알고 싶다면 document만으론 안된다. source code를 다운받아서 보는 게 가장 좋다고 말하고 싶다 :)
