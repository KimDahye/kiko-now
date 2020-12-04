---
layout: post
title: "[2016/01/16] 코드 리뷰"
tags: []
comments: true
---

## 피드백
* `startup_test.sh` 부터 잘 이해했어야 한다. (mvn package가 어떤 동작을 하는 건지, java -jar [jar파일] 했을 때 어떤 파일을 보고 메인클래스를 실행하는지 등)
* 컴퓨터가 저절로 잘 찾는 것은 없다. 어딘가에 분명 명세가 있고 그 명세를 바탕으로 일한다. 이 부분을 잘 파악하자.
* 오늘은 스크립트를 분석하며 어떻게 main클래스를 찾는지까지 파악하였다.
* 다음엔 이런 질문들에 답해보자.
  * 어떻게 ServiceLoader를 이용하여 상위 클래스에서 하위 클래스를 알게 되는 걸까? (ServiceLoader의 load 메서드가 어떻게 동작하는지 잘 알아보자. javadoc이 첫걸음이 될 수 있다.)
  * 메인함수의 각 state마다 뭔가 설정하는데, 그것이 어떻게 동작하는지 그 동작 방식은 모르더라도 뭐하는 녀석들인지 대략적으로 알아두자.
  * 스프링 컨테이너가 어떻게 동작하는지, 이후 HttpServer는 어떻게 동작하는지

## 오늘 배운 것들
### 일단 script 명령어들
* `>`: redirection. 왼쪽의 명령으로 나온 stdout을 오른쪽 arument를 이름으로 하는 파일에 쌓아라.
* `>>`: redirection(append)
* `|` : 왼쪽에 오는 명령의 stdout을 오른쪽에 오는 명령의 stdin으로 사용하라! 
* `$0`: the basename of the program as it was called. 
* `$1`: `$1` .. `$9` are the first 9 additional parameters the script was called with. (실행할 때 넘겨준 파라미터들)
* `$?`:  the exit value of the last run command
* 마지막에 `&`: 이 명령을 background에서 실행하자.
* `nohup`: 터미널이 꺼지더라도 이 명령을 계속 실행해라. (원래 터미널을 종료하면 그 터미널(셸)이 실행하고 있던 프로그램을 끈다.)
* `netstat`: 네트워크 상태 모니터링
* `grep`: 파일 전체를 뒤져 정규표현식에 대응하는 모든 행들을 출력
* `awk '{print $2}'`: 두번째 필드를 출력하라. (참고. awk는 각 라인에서 필드를 추출해 내는 데 필드 분리자(field separator)를 사용, 필드 분리자는 보통 하나 이상의 공백 문자이다)
* `/dev/null`: null device(쓰레기통? 아무런 반응이 없는 녀석)로 보냄. 
* `2>&1`: stderr도 stdout이랑 같이 표현해라!
>"at first, 2>1 may look like a good way to redirect stderr to stdout. However, it will actually be interpreted as "redirect stderr to a file named 1". & indicates that what follows is a file descriptor and not a filename. So the construct becomes: 2>&1." (참고 [사이트](http://stackoverflow.com/questions/818255/in-the-shell-what-does-21-mean))

### jvm flag들
* 자세히는 모르더라도 어떤 역할이다 정도는 알아야 한다. 다 조사해보자. 
* (주말동안 조사하여 새로 배운 것들을 업데이트 해두자)

### mvn package 
* `mvn package`: pom.xml을 쭉 읽고서 module과 의존관계가 있는 걸 다운받은 뒤 tar로 묶는다. 이 때 MENIFEST.MF 파일을 함께 묶는데, 여기에 classpath와 main클래스가 적혀있다.
