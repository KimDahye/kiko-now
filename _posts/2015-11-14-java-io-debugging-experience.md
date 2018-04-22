---
layout: post
title: "Java I/O issue debugging"
tags: [java]
comments: true
---

이슈: https://github.com/KimDahye/chat-program/issues/2

## 문제 1
- client code
  ```
  output = new PrintWriter(socket.getOutputStream(), true);
  output.print(message); //message는 String
  ```

- server code
  ```
  streamIn = new DataInputStream(new BufferedInputStream(socket.getInputStream()));
  streamIn.readUTF();
  ```

[오라클 문서](http://docs.oracle.com/javase/7/docs/api/java/io/PrintWriter.html#print(java.lang.String))를 보면  `output.print(message);`은 default encoding type으로 encoding 한다고. 즉, PrintWriter는 지 나름대로 encoding을 할 텐데 서버에서 UTF로 읽으려고 하니까 encoding protocol이 안맞아서 문제가 되었다.

## 문제 2
- client code
  ```
   String line = input.readLine();
  ```

- server code
  ```
  output.print(message);
  ```

server는 `\n`을 뒤에 붙여보내지 않는데, client는 `readLine()`을 하기 때문이다....! 끝에 라인을 붙이냐 안붙이냐도 서버, 클라이언트가 맞춰야할 프로토콜!

## 결론
**간단한 프로토콜을 잘 신경쓰자!! 웬만하면 I/O는 짝을 맞춰 쓰는 게 문제 없이 작동하겠다.**
