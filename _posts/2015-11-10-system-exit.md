---
layout: post
title: "Java에서 System.exit()의 의미"
tags: [java]
comments: true
---

하나의 post로 쓰기엔 너무 자질구레한 것 같지만... 그래도?

- main 함수 안에서 `System.exit(0);` 이건 왜하는거지? 그냥 끝내면 안되나?   
  - main 함수의 리턴 타입이 void니까 종료했을 때 값을 반환하려면 System.exit() 을 사용한다. 보통 정상종료 했을 때 0을 반환, 에러 있을 때 1을 반환한다고 함.
