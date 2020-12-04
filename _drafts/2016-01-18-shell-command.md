---
layout: post
title: "Shell command 정리"
tags: []
comments: true
---


오늘 리뷰하면서 나온 셸 커맨드들을 정리해둔다.

#### ps -ef 옵션
* `-e` : 모든 프로세스(-A와 같다)
* `-f` : full format으로 보여준다(자세히 보여준다)

#### grep 옵션
파일 전체를 뒤져 정규표현식에 대응하는 모든 행들을 출력한다. (출처. man 페이지)

* `-m num, --max-count=num`: Stop reading the file after num matches.
* `-v, --invert-match`: Selected lines are those not matching any of the specified patterns. (-v 옵션 다음에 나오는 패턴을 제외하고 찾는다.)

#### 셸 스크립트 문자열 체크
* if [ -n stringName ] - 문자열의 사이즈가 0 이상인지 체크, 0 이상이면 참
(출처: http://blog.daum.net/_blog/BlogTypeView.do?blogid=02Ql9&articleno=171)

#### jar 파일과, manifest 파일
* jar 파일 설명: https://docs.oracle.com/javase/tutorial/deployment/jar/manifestindex.html
* manifest 설명: manifest 파일에서 entry point 지정하기. https://docs.oracle.com/javase/tutorial/deployment/jar/appman.html

#### `mvn package`
* The package goal will compile your Java code, run any tests, and finish by packaging the code up in a JAR file within the target directory. The name of the JAR file will be based on the project’s <artifactId> and <version>. For example, given the minimal pom.xml file from be일fore, the JAR file will be named gs-maven-0.1.0.jar. (참고: https://spring.io/guides/gs/maven/)
