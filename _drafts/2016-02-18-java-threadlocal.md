---
layout: post
title: "Java ThreadLocal"
tags: [java]
comments: true
---


## 개요
#### 역할
  * [단일 쓰레드 안에서 다른 객체로의 **전파**] 같은 쓰레드 안에서 매개변수로 변수를 넘겨주지 않아도, 같은 값을 바라볼 수 있다.
  * [여러 쓰레드 안에서 값 **별도 저장**] 쓰레드 별로 별도의 값을 저장할 수 있다.
#### 활용 
  - Spring Security의 사용자 인증정보 전파 (웹서버에서 세션정보도 ThreadLocal로 전달되겠네...?!)
  - 트랜잭션 매니저의 트랜잭션 컨텍스트 전파
  - 쓰레드에 안전해야 하는 데이터 보관
* 참고 자료: [자바캔 블로그](http://javacan.tistory.com/entry/ThreadLocalUsage)

## 동작 원리
ThreadLocal 소스코드가 대략적으로 다음과 같이 생긴 것으로 보아...
```language-java
public class ThreadLocal<T> () {
    public ThreadLocal () { }
	// ...
	public T get() {
	    Thread t = Thread.currentThread();
	    ThreadLocalMap map = getMap(t);
	    if (map != null) {
	        ThreadLocalMap.Entry e = map.getEntry(this);
	        if (e != null)
	            return (T)e.value;
	    }
	    return setInitialValue();
	}

	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

   	// ...
}
```
```language-java
public class Thread implements Runnable () {
	// ...
    ThreadLocal.ThreadLocalMap threadLocals = null;
	// ... 
}
```
**각 Thread마다 각자의 map을 만들고, 그 map에 ThreadLocal을 키로 value(아래 예제에서는 Date 타입의 현재 날짜)을 저장한다.** 이러니까 Thread만 같으면 어떤 객체에서 ThreadLocal에 접근하든(전파성), thread 마다 고유의 값을 넘겨줄 수 있는 거다(별도성).

```language-java
class TestThreadLocal extends Thread {
	private ThreadLocal<Date> local = new ThreadLocal();

	@override
	public void run() {
		local.set(new Date());
		System.out.println(local.get());
	}
}
```
