---
layout: post
title: "Java 명령의 flags, options"
tags: [java]
comments: true
---


### JVM_FLAGS
* `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=[address]`
  * From 5.0 onwards the `-agentlib:jdwp` option is used to load and specify options to the JDWP agent. (출처: [여기](https://docs.oracle.com/javase/1.5.0/docs/guide/jpda/conninv.html#Invocation))
  * jdwp 관련된 옵션이다. jdwp란, 디버거와 JVM의 통신 프로토콜이다. (참고: [오라클 사이트](http://docs.oracle.com/javase/7/docs/technotes/guides/jpda/jdwp-spec.html)) 
  * 옵션 중 suspend 값은 target VM이 main클래스가 로드 되기 직전에 suspended되면 참이라는데([여기](https://docs.oracle.com/javase/1.5.0/docs/guide/jpda/conninv.html) 참고), suspend 된다는 게... 어떤 의미인 지 감이 안온다.
* `-server`: 이 옵션을 주면, JVM의 서버 버전으로 돌린다. 서버 버전으로 돌리면, JVM 로딩은 느리나 프로그램 실행중에는 빠르게 응답할 수 있어 클라이언트 버전보다 성능이 좋다. (참고. [wikipedia](http://en.wikipedia.org/wiki/HotSpot))
* `-Xms`*n*: 메모리 초기 사이즈(in bytes). 1024의 배수여야 하고, k나 m을 붙여 단위를 쓸 수 있다.
* `-Xmx`*n*: 메모리의 maximu size. 마찬가지로 1024의 배수여야 하고, k나 m을 붙여 단위를 쓸 수 있다.
* `-XX:+UseConcMarkSweepGC`: Concurrent Mark Sweep(CMS) GC를 사용하겠다는 뜻. - **근데 CMS GC가 뭔지는 찾아봐야겠다..**
* `-XX:+UseParNewGC`: "Uses a parallel version of the young generation copying collector alongside the default collector. This minimizes pauses by using all available CPUs in parallel. The collector is compatible with both the default collector and the Concurrent Mark and Sweep (CMS) collector." (출처 [Tuning JVM Garbage Collection for Production Deployments](https://docs.oracle.com/cd/E13209_01/wlcp/wlss30/configwlss/jvmgc.html)
* `-XX:NewSize=`*n*: "Sets the size of the young generation (nursery)." - **young generation 이 뭐지?**
* `-XX:MaxNewSize=`*n*: **일단 넘어가자.** 안나온당.
* `-XX:SurvivorRatio`*n*: **일단 넘어가자.** 안나온당. 
* `-XX:MaxTenuringThreshold`*n*: **일단 넘어가자.** 안나온당. 
* `-Dproperty=value`: Set a system property value. If value is a string that contains spaces, you must enclose the string in double quotes. (출처. [coderanch](http://www.coderanch.com/t/178539/java-OCAJ/certification/java-command-line-option-good)) 
* 출처 쓰지 않은 부분은 [오라클 technotes](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/java.html) 참고하였다.

### JMX_OPTS
JMX는 자바 어플리케이션, 프로그램 등을 모니터링 할 수 있는 인터페이스 기술이다. 참고 사이트만 적어두겠다.

* [what is JMX?](https://blogs.oracle.com/jmxetc/entry/what_is_jmx) 
* http://docs.oracle.com/javase/6/docs/technotes/guides/management/agent.html
* http://docs.oracle.com/javase/6/docs/technotes/guides/management/agent.html

### SNMP_OPTS
* [SNMP 란? 간단한(?) 네트워크 관리 프로토콜](http://blog.alghost.co.kr/snmp-%EB%9E%80-%EA%B0%84%EB%8B%A8%ED%95%9C-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EA%B4%80%EB%A6%AC-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C/)
