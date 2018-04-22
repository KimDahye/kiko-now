---
layout: post
title: "Weak, Soft, Phantom Reference"
tags: [java]
comments: true
---

## Overview
- 내가 생각하는 레퍼런스 세기:<br> strong > soft > weak > (phantom ?) > unreachable <br>(왼쪽으로 갈 수록 GC에 의해 수거 안된다.)
- strong refernece: root set of reference(Java Stack,
Native Stack, Method Area)에서 참조하고 있는 reference
- soft reference: strong아니고 soft reference로만 이루어진 path가 있다면 soft reference
- weak reference: strong이나 soft도 아니고, phantom ref 없이 weak reference로 이루어진 path가 있다면 weak reference. soft가 중간에 있더라도 weak reference 가 끼어 있으면 weak reference가 되는듯.
- phantom reference: strong, soft, weak 다 아니고. soft, weak reference가 끼어 있더라도 phantom reference 가 끼어 있다면 phantom reference 인 듯!

## Weak Reference
- weakly reachable object들은 일단 GC가 돌면은 수거해가는 객체들이다.

## Soft Reference
- soft reachable obejct들은 GC가 메모리가 부족하다고 판단하면 수거해간다.
- GC가 수거해가지 못하도록 방지하는 수준은 아니지만 (이거는 strong reference), 수거 시간을 지연할 수는 있다.
- 자주 사용될수록 GC가 수거해가지 않는다.
- `[Soft Reference가 참조하고 있는 객체가 마지막으로 사용된 이후로부터 지금까지의 시간] > [옵션 설정값 N] * [힙에 남아있는 메모리 크기]` 이 조건이 만족되면 GC가 수거해간다.
- 위의 옵션 설정값 N은 JVM 옵션 설정값으로 다음과 같이 정해준다. `-XX:SoftRefLRUPolicyMSPerMB=<N>`

## LRU 캐시 구현
- LRU 캐시 구현에 Soft Reference가 적합하다는 사이트([여기](http://www.javacodegeeks.com/2014/03/difference-between-weakreference-vs-softreference-vs-phantomreference-vs-strong-reference-in-java.html))도 있고, Weak Reference가 적합하다는 사이트([여기](http://d2.naver.com/helloworld/329631))도 있었다.
* Soft Reference가 적합하다고 하는 이유는 Soft Reference 자체적으로 최근에 사용될수록 GC에서 수거해가지 않는 속성이 있기 때문에 LRU 전략(Least Recently Used)을 바로 구현할 수 있기 때문.
* Weak Reference가 적합하다고 하는 이유는 다른 logic을 위해 비워야할 Heap 공간이 softly reachable 객체들에 의해 점유되기 때문에, 메모리 사용량이 늘고 따라서 GC 가 자주 돌게 되고, 이는 성능 이슈를 만든다.
* 나는 처음에 WeakHashMap 비슷한 느낌으로, 하지만 내가 실제로 Entry 클래스를 구현하지는 않으면서 SoftReference와 HashMap을 이용하여 캐시를 구현해보려 했으나, 이미 불필요해진 element를 지우는 로직이 불가능하여 포기하였다. (관련 내용은 [학습일지](../study-diary-2015-11)에 11월 26일 내용)
* HashMap과 DoublyLinkedList 이용해 간단히 구현해보았다. ([github](https://github.com/KimDahye/TIL/blob/master/java/practice/src/simpleLRU/SimpleLRUCache.java))
* **[스터디 통해 알게 된 점]** LinkedHashMap의 removeEldestEntry() 메서드를 이용하면 더 간단히 구현할 수 있다고 한다.

## Reference Queue
- SoftReference나 WeakReference는 referent가 GC 대상이 되면 GC가 이 referent 를 finalize 한뒤 자동적으로 Reference Queue (RQ)에 enqueue 한다. Soft, Weak Reference 안의 참조는 null이 된다.
- soft, weak은 RQ를 생성자에 넣을 수도 안 넣을 수도 있지만, phantom은 RQ를 꼭 생성자에 넣는다.
- **[스터디 중 정리]** Soft, Weak Reference와 Phantom reference의 가장 큰 차이는, Phantom 레퍼런스는 referent가 finalize()되고 메모리까지 회수된 이후에 RQ안에 들어가는데 - Soft, Weak의 referent는 RQ안에 들어와있다고 해서 referent의 메모리가 회수되었는지는 미지수다!

## Phantom Reference
- 어디에 쓰이나? finalizer를 대체하는 용도로 쓰인다고 함. (참고 사이트 [여기](http://resources.ej-technologies.com/jprofiler/help/doc/index.html))
- how?
  * PhantomRefernce를 상속하고, 닫아줘야 하는 중요 resource를 필드로 갖는다.
  * 생성자에서 reference queue와 resource를 주입받고, public method 로 cleanUp 메소드를 열어둔다. 이걸(A 클래스라고 하자.)
  * 쓰는데에서는 A로 이루어진 리스트에다 레퍼런스를 저장하고, 레퍼런스 큐를 만들어 A를 생성할 때 넣어준다.
  * 쓰레드에서 레퍼런스 큐를 돌면서 레퍼런스 큐에 들어온 녀석들을 정리(cleanUp 메소드 호출하여) 해준다.

## 질문
#### 1.
Reference와 씨름 하는 과정 중에 다양한 물음이 생겼다.

* Weak, Soft Refernce가 GC에 의해 자동적으로 Reference Que에 들어가게 되었을 때, 언제 어떻게 그 큐 안에서 poll()되는 걸까? 시간이 흐르면 자동적으로 GC가 poll()하는 건가?
* (poll된 이후의) Reference 객체는 언제 garbage collected 될까? reference que 안에서 poll이 되었음에도 WeakHashMap의 링크드 리스트 안에 존재할 수 있음...... 도대체 reference 객체는 그럼 언제 GC된단 말인가?
* WeakHashMap의 expungeStaleEntries 구현을 보면 Reference Queue에 들어갔다가 poll된 이후에도 Map의 링크드 리스트 안에 해당 reference 객체가 존재하고 있음을 알 수 있다. 그래서 결국 함수 안에서 prev.next = next 의 과정이 필요했다. 그래야 온전히 GC될 수 있나봄...?
* Phantom Reference 를 사용하는 예제를 만들어봤는데, 팬텀 레퍼런스가 언제 러퍼러느 큐에 들어올 지 모르겠다. 어떻게 레퍼런스 큐에 들어오게 할 수 있을까?(finalize를 대체하려고 만들었는데 finalize()를 부를 순 없잖아...)

#### 2. WeakHashMap
* `private static class Entry<K,V> extends WeakReference<Object>`
왜 `WeakReference<K>`가 아닐까?

#### 3. JVM
[오라클 문서]( http://www.oracle.com/technetwork/java/hotspotfaq-138619.html)의
"What determines when softly referenced objects are flushed?” 섹션 중에서 다음의 설명이 나온다.

"The Java HotSpot Server VM uses the maximum possible heap size (as set with the -Xmx option) to calculate free space remaining. The Java Hotspot Client VM uses the current heap size to calculate the free space.

This means that the general tendency is for the Server VM to grow the heap rather than flush soft references, and -Xmx therefore has a significant effect on when soft references are garbage collected. On the other hand, the Client VM will have a greater tendency to flush soft references rather than grow the heap."

**VM도 Server, Client가 있는 것 같네. 각각의 역할은 어떻게 되는 걸까?**

## 참고 문서
* http://www.javacodegeeks.com/2014/03/difference-between-weakreference-vs-softreference-vs-phantomreference-vs-strong-reference-in-java.html
* http://d2.naver.com/helloworld/329631
* http://resources.ej-technologies.com/jprofiler/help/doc/index.html (Help topics > CPU profiling, Removing finalizer)
* http://www.oracle.com/technetwork/java/hotspotfaq-138619.html
* 나중에 볼 문서 정리되어 있는 스택오버플로우 http://stackoverflow.com/questions/9808258/java-weak-reference-when-gc-collect-object
* https://community.oracle.com/blogs/enicholas/2006/05/04/understanding-weak-references
