---
layout: post
title: "spring의 MessageSource"
tags: [java, spring]
comments: true
---


### MessageSource가 어떻게 동작하나?
- [사용 예시](http://changpd.blogspot.kr/2013/05/localeresolver-messagesource.html)
- 내부적으로 ResourceBundle을 사용하네. (ResourceBundleMessageSource의 경우)

### 그럼 ResourceBundle은 어떻게 동작하나?
- [ResourceBundle simple test 예제](http://www.avajava.com/tutorials/lessons/how-do-i-read-a-properties-file-with-a-resource-bundle.html?page=1)
- [ResourceBundle로 locale 설정하기](https://docs.oracle.com/javase/tutorial/i18n/resbundle/propfile.html)

### ResourceBundle을 바로 쓰는 것도 기능적으로 문제 없는데, 왜 한번 감싸진 MessageSource를 쓴 걸까?
- 성능 이슈? 오히려 감싸면서 성능이슈가 더 많아지지 않을까?
- No. 답은 ResourceBundleMessageSource docs에 나와있었다! ResourceBundleMessageSource이 캐싱으로 더 빠른 성능을 제공한다!
>"This MessageSource caches both the accessed ResourceBundle instances and the generated MessageFormats for each message. It also implements rendering of no-arg messages without MessageFormat, as supported by the AbstractMessageSource base class. The caching provided by this MessageSource is significantly faster than the built-in caching of the java.util.ResourceBundle class." by [spring framework docs](http://docs.spring.io/autorepo/docs/spring-framework/3.2.6.RELEASE/javadoc-api/org/springframework/context/support/ResourceBundleMessageSource.html)
