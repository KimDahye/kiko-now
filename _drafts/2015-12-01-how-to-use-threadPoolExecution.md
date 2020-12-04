---
layout: post
title: "ThreadPoolExecutor"
tags: [java, ThreadPoolExecutor]
comments: true
---

## ThreadPoolExecution을 어떻게 사용하는가?
* [예제1](http://www.journaldev.com/1069/java-thread-pool-example-using-executors-and-threadpoolexecutor)
* [예제2](http://howtodoinjava.com/2015/03/24/java-thread-pool-executor-example/)
* `java.util.concurrent.Executors` 클래스는 Executor의 구현체를 생성해주는 static factory 클래스이다.
* `Executors.newFixedThreadPool(THREAD_NUMBER);` 등의 팩토리 함수를 이용해 ThreadPoolExecutor 인스턴스(executor라고 부르겠다)를 반환받는다.
* executor.execute() 메소드 안에 실행하고픈 Runnable 객체를 넣으면 해당 객체의 run() 메소드가 실행된다.
* executor.shutdown() 메소드를 부르면 executor 가 내부적으로 관리하고 있던 Thread Pool 안의 모든 Thread가 종료된다. (소스코드를 통해 Thread의 인스턴스 메소드인 interrupt를 호출하는 것을 확인하였다.)
* [나의 적용](https://github.com/KimDahye/chat-program/commit/b75b3acbbc3213a2dcf1a610bd38201ec00dd7bb)

## 왜 사용하는가?
* Thread Pool을 정교히 조작하여 사용할 수 있다. (max thread 수, cache 사용여부 등) 
* Thread 개수를 한정할 수 있으므로 메모리 사용량이 적어지고 이를 통해 성능 향상이 되는 것 같다.
* 풀 안에 들어갈 thread 개수가 max 값을 넘었을 때 생성된 thread에 대해 RejectedExecutionHandler 의 rejectedExecution() 메소드 안에서 처리할 수 있다.
* 이 외에도 (이건 Thread라기 보다는 Task에 더 관련된 이야기인 듯 하지만) 특정 태스크가 종료되기를 기다리거나, 태스크가 끝날 때마다 그 결과를 차례로 가져올 수 있는 등 Executor 안의 다양한 메소드를 이용하여 다양한 작업을 할 수 있다. (Effective Java 10장 item 68 참고) 

##ThreadPoolExecutor Java doc 내용
(이는 2015-12-18에 [java doc](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ThreadPoolExecutor.html)을 읽고 번역, 의역한 내용이다)
#### constructor parameters
* corePoolSize - 풀 안에 (상태가 idle이어도) 유지해야하는 쓰레드 수. (allowCoreThreadTimeOut가 setting되어 있다면 유지 안될 수 있다.)
* maximumPoolSize - 풀 안에 있을 수 있는 최대 쓰레드 수.
* keepAliveTime - 쓰레드 수가 core 보다 클 때, idle 쓰레드가 새로운 task를 할당받기 위해 기다릴 수 있는 최대 시간.
* unit - the time unit for the keepAliveTime argument
* workQueue - task가 실행되기 전에 거치는 큐. 이 큐는 오직 execute 메소드를 통해 들어온 Runnable task만 받는다. 
* threadFactory - the factory to use when the executor creates a new thread
* handler - 쓰레드 수가 max고, 큐가 가득차있을 때 task의 실행이 막힌다. 이 rejected task에 대해 handler가 실행된다. 

* **참고**. 하지만 실제로 사용할 땐 생성자에 직접 매개변수 넣어서 ThreadPoolExecutor를 생성하진 않고 위의 "ThreadPoolExecution을 어떻게 사용하는가?"에서 살펴보았듯이 대개 Executors의 팩토리 메서드를 사용하여 생성합니다.



#### Core and maximum pool sizes
- 현재 실행되고 있는 쓰레드가 core size 보다 적은데 새로운 task가 들어오면 다른 worker thread가 idle 상태여도 새로운 thread가 만들어지고 해당 task를 실행한다.
- 현재 실행되고 있는 쓰레드가 core size 보다 많고 max보다는 적다면,  a new thread will be created only if the queue is full.
- By setting corePoolSize and maximumPoolSize the same, you create a fixed-size thread pool.
- setCorePoolSize나 setMaximumPoolSize를 통해서 풀 사이즈를 동적으로 바꿀 수도 있다.

#### KeepAliveTime
- core Thread 수 보다 많은 쓰레드가 실행되고 있을 때 keepAliveTime 시간동안 idle상태이면 그 쓰레드들은 종료된다. 
- 역시 동적으로 keepAliveTime을 setting할 수 있다.

#### Queuing 
여기서 queue는 BlockingQueue의 any 구현체이다. (참고. BlockingQueue는 element를 넣거나 뺄 때 해당 작업이 이뤄질 수 있도록 기다리는 동작을 지원하는 큐를 말한다. ex. 원소 하나 꺼내려고 하는데 원소가 하나도 없다면 하나가 들어올 때까지 기다리는 것이다.)

- corePoolSize보다 적은 수의 쓰레드가 실행되고 있다면 task가 들어왔을 때 큐에 넣기보다는 바로 쓰레드를 만들어 task를 실행한다. 
- corePoolSize 만큼의 쓰레드가 실행되고 있을 때, task가 들어오면, 쓰레드가 바로 시작되지 않고 해당 태스크를 queue에 넣는다. 만약 태스크가 들어왔는데 큐가 가득 차있다면, 그 때 쓰레드를 만들어 task를 실행한다. 
- 이미 쓰레드가 max이고, queue도 가득차 있다면 해당 task는 reject 된다. 

#### Queuing 전략
- Direct handoffs (default choice)
  - SynchronousQueue를 사용하면 쓸 수 있는 전략이다.
  - 큐에 task를 놓지(holding) 않고 바로 thread로 던저(hands off) 실행해버리는 전략이다. => 큐 사이즈가 0이라고 생각하면 쉽다!
  - task를 즉시 실행 할 수 있는 쓰레드가 없을 때, 큐에 task를 넣으려고 하면 그 시도는 실패하고 새로운 쓰레드가 생성되어 그 task를 실행한다. 
  - 이 전략은 내부적인 의존관계가 있는 task 집합에 대해 처리할 수 있다. (This policy avoids lockups when handling sets of requests that might have internal dependencies.) 왜냐면 보통 unbounded maximumPoolSizes 로 설정되므로 의존관계가 있는 task들이 있어도 다른 task들을 처리할 수 있다.
  - 일반적으로 새로 들어온 task를 reject하지 않기 위해 unbounded maximumPoolSizes를 요구한다. 하지만 이는 task 평균 실행속도보다 유입되는 task속도가 더 빠를 때 unbounded thread grouth의 가능성이 있음을 유의해야 한다!
- Unbounded queues
  - LinedBlockingQueue (without a predefined capacity)등의 unbounded queue를 사용하는 전략이다.
  - 큐가 무한대니까 corePoolSize 이상의 쓰레드가 생성될 수 없겠지. 따라서 maxPoolSize가 셋팅되어도 영향을 줄 수 없다.
  - 이 전략은 각각의 task가 서로 독립적일 때 적합한 전략이다. - 왜냐면 corePoolSize만큼 task가 서로 내부 의존관계를 갖고 있으면 그 그룹이 계속 실행되고 있을 수가 있다. 그럼 더이상 다른 Thread가 실행되지 못한다. 따라서 task가 독립적으로 실행되어야 거절되는 task가 적어진다. ex) web page server 
  - task request가 몰리더라도 잘 견딜 수 있겠지만, 이 역시 유입되는 task 속도가 task의 실행 속도보다 높다면 큐가 무한대로 커질 수 있는 가능성을 지니고 있다.  
- Bounded queues
  - ArrayBlockingQueue 등의 bounded queue를 사용하는 전략이다.
  - finite maximuPoolSize를 사용한다면 자원의 고갈을 막을 수 있겠죠.
  - 하지만 튜닝이나 컨트롤이 더 어렵다는 단점이 있다!
  - 큐 사이즈와 maxPoolSize는 서로 trade-off 관계! 큐 사이즈를 크게 잡고 풀 사이즈를 작게 잡으면, CPU와 OS자원, context-switching 비용을 최소화할 수 있다. 하지만 throughput은 낮겠지. 큐 사이즈를 작게 잡고 풀 사이즈를 크게잡으면, CPU를 바쁘게 하나 과도히 태스크가 몰려 (threshold를 넘기면) 오히려 throuput을 낮추게 될 수 있다.. 
## 공부하면서 생긴 질문
* Q. executor 내부의 thread를 **선택적으로** interrupt하거나 다른 메소드를 부를 수 있을까? <br> 못한다고 한다. executor를 필드로 갖고 있는 클래스에 해당 Thread의 reference를 필드로 저장해두고 있어야만 가능하다.

* Q. excutor 내부의 thread중에 interrupt 된 녀석이 있다면, 그 녀석은 thread pool에서 자동적으로 사라지는 건가(removed out)? <br> 
HashSet<Worker>로 worker 쓰레드를 들고 있는듯! 그럼 쓰레드가 끝나면 가바지 컬렉터에 의해 HashSet에서 정리되려나?

* Q. 큐잉전략으로 unbounded queue를 쓰는 것은 왜 task가 상호 독립적일 때에 적합하다는 걸까?
