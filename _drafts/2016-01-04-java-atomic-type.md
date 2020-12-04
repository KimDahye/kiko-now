---
layout: post
title: "Java Atomic 타입의 이해"
tags: [java]
comments: true
---

아래 내용은 [여기](http://tutorials.jenkov.com/java-concurrency/compare-and-swap.html) 를 이해하고 이를 한글로 정리한 것이다.

## Compare & Swap
먼저 아래 코드를 살펴보자.
```language-java
class MyLock {

    private boolean locked = false;

    public boolean lock() {
        if(!locked) {
            locked = true;
            return true;
        }
        return false;
    }
}
```
위의 코드는 문제가 있다. 동시간에 하나의 Thread만이 lock을 얻어야 하는데 (= lock() 메소드에서 true 가 나와야 하는데) 위의 코드는 동시에 두개의 thread가 lock을 얻을 수 있다. 이 문제를 해결하려면, check(locked 값을 확인한다) & set(locked 값을 true로 바꾼다) 과정을 atomic 하게 만들어야 한다. => `synchronized` keyword!

```language-java
class MyLock {

    private boolean locked = false;

    public synchronized boolean lock() {
        if(!locked) {
            locked = true;
            return true;
        }
        return false;
    }
}

```
위의 코드는 잘 돌아갈 것이다. 그런데 요즘의 CPU는 위와 같은 과정을 기본적으로 제공해준다. 이를 사용한 것이 AtomicBoolean, AtomicInteger 등의 Atomic 자료형이다. 위의 코드를 AtomicBoolean으로 바꾸면 아래와 같이 바꿀 수 있다.
````language-java
public static class MyLock {
    private AtomicBoolean locked = new AtomicBoolean(false);

    public boolean lock() {
        return locked.compareAndSet(false, true);
    }

}
```
<br>
##결론
Atomic 자료형이 modern cpu가 기본적으로 제공하는 synchronized compare & swap 기능을 바로 사용하므로 직접 구현할 때보다 더 빠른 성능을 보장한다.

## 참고 링크
* atomic integer 관련 링크
http://tutorials.jenkov.com/java-util-concurrent/atomicinteger.html
