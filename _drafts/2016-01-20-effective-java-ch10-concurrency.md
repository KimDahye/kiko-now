---
layout: post
title: "Effective Java Ch10 Concurrency"
tags: [java, effective java]
comments: true
---


##Item 66. Synchronize access to shared mutable data
####  `synchronized` 키워드
* 역할 1. 특정 메서드나 코드 블록을 한 번에 한 스레드만 사용하도록 보장한다. (mutually exclusiveness, 상호 배제성)
* 역할 2. synchronization 때문에, 한 쓰레드에서의 변화를 다른 쓰레드가 알아챌 수 있다. (스레드간 통신)
* 따라서, long, double 이외의 모든 변수 **(여기서 "모든"은 primitive 에 국한됨)**가 원자적으로 읽고 쓸 수 있다고 해서, 성능을 높이기 위해 동기화를 피해야 한다는 말은 아주 위험한 이야기다. 다음의 예를 살펴보자.

#### 동기화하지 않을 때의 문제점
```language-java
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested) i++; 
            }
        });

        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
* 실행한 지 1초가 지나면, main 쓰레드가 stopRequested의 값을 true로 바꾸므로 background thread가 실행되는 순환문도 그 때 중지될 거 같지만, 실제로 돌려보면 이 스레드는 절대로 종료되지 않는다.
* 그 이유는 이 프로그램이 동기화 메커니즘을 적용하지 않았기 때문에, **스레드간 통신이 적절히 이뤄지지 않았기 때문이다.** 동기화가 적용되지 않은 경우, 가상 머신은 위의 코드를 아래와 같이 바꿀 수 있다.
```language-java
//hoisting
if (!stopRequested)
  while (true)
    i++;
```
(살아있기는 하나 더 진행하지는 못하는 프로그램이 되는 것 - liveness failure라고 한다.)

#### 이를 해결하기 위한 방법
* [synchronized] 하나는 synchronized 키워드 사용하여 동기화하는 것. 쓰기 메서드와 읽기 메서드 모두에 동기화를 적용해야 제대로 동작한다. 둘다에 적용하지 않으면 아무런 효과가 없다! (**읽는 것을 바탕으로 쓸 때에만 읽기에도 동기화해주면 되지, 언제나 읽기에 동기화해야 하나?**)
```language-java
// Properly synchronized cooperative thread termination
public class StopThread {
  private static boolean stopRequested;
 
  private static synchronized void requestStop() {
    stopRequested = true;
  }
 
  private static synchronized boolean stopRequested() {
    return stopRequested;
  }
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(new Runnable() {
      public void run() {
        int i = 0;
        while (!stopRequested())  
          i++; 
      }
    });
    
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    requestStop(); 
  }
}
```
* [volatile] 위의 예제에서 동기화 메서드가 하는 일은 동기화가 없이도 원자적이다. 순전히 스레드간 통신 문제를 해결하기 위해 동기화한 것이다. 이럴 때엔 동기화 비용을 줄이면서 스레드간 통신문제를 해결하는 방법이 있다. 바로 `volatile` 키워드를 사용하는 거다. **volatile로 선언하면, 상호배제성을 실현하진 않지만, 어떤 스레드건 가장 최근에 기록된 값을 "언젠가는" 읽도록 보장한다.** (메모리 오더링을 보장해주진 않기 때문에 최신 값이 다른 스레드에 바로 읽히지는 않을 수 있다.)
```language-java
// Cooperative thread termination with a volatile field
public class StopThread {
private static volatile boolean stopRequested;
       public static void main(String[] args)
               throws InterruptedException {
           Thread backgroundThread = new Thread(new Runnable() {
               public void run() {
                   int i = 0;
                   while (!stopRequested)
i++; }
           });
           backgroundThread.start();
           TimeUnit.SECONDS.sleep(1);
           stopRequested = true;
       }
}
```
#### volatile을 잘못 쓴 예제
```language-java
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;
   public static int generateSerialNumber() {
       return nextSerialNumber++;
}
```
* 여기서는 동기화가 필요하다. 왜냐면 generateSerialNumber() 안의 연산이 원자적이지 않기 때문. 이런 오류를 safety failure라고 한다. 이걸 해결하기 위한 방법은 synchronized 를 붙이거나, AtomicLong 클래스를 사용하는 것.
```language-java
private static final AtomicLong nextSerialNum = new AtomicLong();
   public static long generateSerialNumber() {
       return nextSerialNum.getAndIncrement();
}
```
#### liveness failure, safety failure을 피하려면
* 가장좋은 방법은 변경 가능 데이터를 공유하지 않는 것! 공유하려면 immutable 데이터를 공유하라! 이렇게 정했다면 문서에 남겨 이 지침이 위배되지 않도록 주의하자.
* 특정한 스레드만이 객체를 변경할 수 있도록 하고, 변경이 끝난 뒤에야 다른 스레드와 공유하도록 할 때는 객체 참조를 공유하는 부분에만 동기화를 적용하면 된다.  객체 참조를 가져온 스레드는 객체가 더이상 수정되지 않는 한, 동기화 없이 객체의 내용을 읽을 수 있다. 이런 객체를 effectively immutable 객체라고 한다. 또 이런 객체 참조를 다른 스레드로 전달하는 행위를 safe publication이라고 부른다. => **스터디 하면서 정리한 것은** effectively immutable 객체는 mutability를 최소화한 객체라고 생각하면 되고, safe publication은 객체가 잘 만들어지고 나서 다른 스레드가 참조를 시작할 수 있는 게 safe pubictaion이다. (자세한 것은 다음에 읽게 될 Java Concurrency in practice 를 읽어보자.)

#### 스터디 중 나온 내용
* spinlock....뭐하면서 나온 내용이었지?

## Item 67. Avoid excessive synchronization
과도한 동기화를 하면 프로그램의 정확성(deadlock, nondetemininistic behavior)과 성능에 문제가 생길 수 있다. 먼저 정확성에 문제가 생기는 경우부터 알아보자.

#### 동기화 메서드나 블록 안에서 클라이언트에게 프로그램 제어 흐름을 넘기지 마라.
**이부분 observer  패턴 보고 다시 보자. 잘 이해가 안됨...ㅠㅠ**<br>
다시 말해, overidable 메서드나 클라이언트가 제공한 function object 메서드를 호출하지 말라. (이런 메서드는 동기화 영역이 존재하는 클래스 관점에서 보면 alien methods라고 한다.) 무슨 일을 하는지, 얼마나 걸릴지 예측할 수 없기 때문이다.

###### nondeterminitstic behavior 예시
```language-java
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> set) { super(set); }

	private final List<SetObserver<E>> observers =
	   new ArrayList<SetObserver<E>>();

	public void addObserver(SetObserver<E> observer) {
		synchronized(observers) {
		   observers.add(observer);
		}
	}
   
   	public boolean removeObserver(SetObserver<E> observer) {
    	synchronized(observers) {
        	return observers.remove(observer);
		} 
	}
       
	private void notifyElementAdded(E element) {
		synchronized(observers) {
		   for (SetObserver<E> observer : observers)	
    		   	observer.added(this, element);
        }
	}

    @Override public boolean add(E element) {
       boolean added = super.add(element);
       if (added)
           notifyElementAdded(element);
       return added;
	}
	@Override public boolean addAll(Collection<? extends E> c) { 
		boolean result = false;
		for (E element : c)
			result|=add(element); //callsnotifyElementAdded 
		return result;
	}
}
```

```language-java
public interface SetObserver<E> {
   // Invoked when an element is added to the observable set
   void added(ObservableSet<E> set, E element);
}
```

```language-java
public static void main(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<Integer>(new HashSet<Integer>());
	set.addObserver(new SetObserver<Integer>() {
		public void added(ObservableSet<Integer> s, Integer e) {
	    	System.out.println(e);
	    	if (e == 23) s.removeObserver(this); // (*)
        }
	});
    
    for (int i = 0; i < 100; i++)
	    set.add(i);
}
```
위의 코드에서 (*) 의 코드 때문에 `CurrentModificationException` 이 발생한다. synchronized를 붙였어도 리스트 순회가 이루어지고 있는 도중에 리스트에서 원소를 삭제하려했기 때문. 같은 쓰레드 내에서 동일한 객체에 대한 락을 걸었기 때문에 reentrace가 가능했기 때문이다.<br>

###### deadlock 예시
```language-java
// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<Integer>() {
	public void added(final ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService executor = Executors.newSingleThreadExecutor();
            final SetObserver<Integer> observer = this;
               	try {
                	executor.submit(new Runnable() {
                       public void run() {
                           s.removeObserver(observer);
                       }
                   	}).get();
               } catch (ExecutionException ex) {
                   throw new AssertionError(ex.getCause());
               } catch (InterruptedException ex) {
                   throw new AssertionError(ex.getCause());
               } finally {
                   executor.shutdown();
               }
		}
	}
});
```
reentracne를 막기 위해 background 스레드를 만들어 얘가 s.removeObserver 를 호출하는데, 이 메서드는 observers에 락을 걸려 한다. 하지만 락을 걸 수 없다. 왜냐면 main 스레드가 이미 락을 걸고 있기 때문이다. main thread는 background 스레드가 구독 해제를 끝내기를 기다리면서 락을 계속 들고 있는데, 그래서 deadlock 상태가 되는 것이다.

#### 문제 해결
ailen 메서드를 동기화 영역 밖에서 호출하라(open call).
```language-java
// Alien method moved outside of synchronized block - open calls
   private void notifyElementAdded(E element) {
       List<SetObserver<E>> snapshot = null;
       synchronized(observers) {
           snapshot = new ArrayList<SetObserver<E>>(observers);
       }
       for (SetObserver<E> observer : snapshot)
           observer.added(this, element);
}
```
사실 ailen 메서드를 동기화 영역 밖으로 옮기는 문제라면, 자바 1.5부터 추가된 CopyOnWriteArrayList라는 Concurrent collection을 사용할 수도 있다.  CopyOnWriteArrayList는 내부 배열을 통째로 복사하는 방식으로 쓰기 연산을 지원한다. 병렬 상황에서 순회 연산이 압도적으로 많을 때, 사용하기 적절하다.
```language-java
 // Thread-safe observable set with CopyOnWriteArrayList
   private final List<SetObserver<E>> observers =
       new CopyOnWriteArrayList<SetObserver<E>>();
   
   public void addObserver(SetObserver<E> observer) {
       observers.add(observer);
   }
   
   public boolean removeObserver(SetObserver<E> observer) {
       return observers.remove(observer);
   }
   
   private void notifyElementAdded(E element) {
       for (SetObserver<E> observer : observers)
           observer.added(this, element);
}
```
#### 성능 이슈
* 멀티코어 세서에서 동기화의 진짜 비용은 락을 거느라 소비되는 CPU 시간이 아니라, 병렬성을 활용할 기회를 잃는다는 것, 그리고 모든 코어가 동일한 메모리 뷰를 보도록 하기 위해 필요한 delay가 더 큰 비용이다.
* 동기화가 지나치면 VM의 최적화도 제한된다.
* mutable 클래스의 경우, 병렬적으로 이용될 클래스이거나, 내부적인 동기화를 통해 외부에서 전체 객체의 락을 걸 때보다 높은 병행성을 달성할 수 있을 때만 thread safety를 갖도록 구현해야 한다. - 그렇지 않다면 내부적인 동기화는 하지 마라. 대신 스레드 안정성을 보장하지 않는 클래스라고 문서에 남겨두자. ex) StringBuffer -> StringBuilder
* 클래스 내부적으로 동기화 메커니즘을 적용해야 한다면, lock splitting, lock striping, nonblocking concurrency control과 같은 기법을 사용할 수 있다.
* static 필드를 변경하는 메서드가 있을 때는 해당 필드에 대한 접근을 반드시 동기화해야 한다. 외부적으로 동기화할 방법이 없기 때문이다.... (**왜?**)
* 중요한 것은 동기화 영역 안에서 하는 작업의 양을 제한해야 한다!

## Item 68. Prefer executors and tasks to threads
이책의 초반에는 간단한 work queue 예제가 있었다. 하지만 제대로 만들지 않을 경우, 여러 오류를 유발할가능성이 있는 복잡한 코드였다. 이제는 이런 코드를 만들 필요가 없다. 자바 1.5부터 추가된 Exectuor Framework를 사용하면 된다.

#### Executor
```language-java
ExecutorService executor = Executors.newSingleThreadExecutor();   
executor.execute(runnable);
executor.shutdown();
``` 

* executor service를 사용하면 할 수 있는 게 많다. 
  * 특정 태스크가 종료되기를 기다릴 수 있다. 
  * 임의의 태스크가 종료되기를 기다릴 수 있다.
  * 실행자 서비스가 자연스럽게 종료되기를 기다릴 수 있다.
  * 태스크가 끝날 때마다 그 결과를 차례로 가져올 수도 있다.
* 큐의 작업을 처리할 때 스레드를 여러개 만들고 싶다면, java.util.concurrent.Executors 클래스의 static factory methods를 이용해 `ThreadPoolExecutor`라 부르는 실행자 서비스를 생성할 수도 있다. 
  * 보통 작은 프로그램이거나 부하가 크지 않은 서버는 Executors.newCachedThreadPool을 사용. (큐에 담지않고 바로 스레드를 만든다.)
  * 부하가 큰 환경에서는 Executors.newFixedThreadPool 을 이용해 스레드 개수가 고정된 풀을 만들거나, 많은 부분을 직접 제어하기 위해 ThreadPoolExecutor 클래스를 직접 사용한다.

#### task, executor service
* work queue를 직접 구현하는 것은 삼가라. 또 thread를 직접 이용하는 것도 피하라. - Thread는 작업의 단위이면서 작업을 실행하는 메커니즘이기도 했다. 이제는 이게 분리되어 작업은 Task, 실행 메커니즘은 ExecutorService가 담당하는 것이다. 작업과 실행 메커니즘을 분리해야 실행 정책을 더 유연하게 정할 수 있다. 
* java.util.Timer을 대신하는 ScheduledThreadPoolExecutor도 정의되어 있다. 여러 스레드를 이용할 뿐 아니라, 태스크 안에서 unchecked exception 이 발생한 상황도 우아하게 복구해낸다.
* 참고. Task에는 두가지가 있다. Callable(값을 반환하는 것), Runnable

#### 스터디 중 나온 내용
###### Excutor에 대해
* Executors.newCachedThreadPool (Queue로 SynchrnousQueue를 쓴다)
  * 큐 사이즈가 0. 여기에 넣으면 바로 꺼내진다. 꺼내갈 사람이 없으면 block되거나(synchronous니까) reject 된다.
  * 대신 효율적이다. 큐에 넣었다 빼내지는 그 시간이 절약되니까. 
* Executors.newFixedThreadPool(Queue로 LinkedBlockingQueue를 쓴다)
  * linked list니까 큐 사이즈가 무한대!
* Executors.newScheduledTrheadPool 
  * Timer를 그룹으로 쓸 수 있는 형태라고 생각하면 된다.


## Item 69. 저수준 cuncurrient utility(wait, notify)보다 고수준 concurrent utility를 이용하라.
(참고. 이번 제목은 내가 이해가 더 잘되도록 임의로 바꿨음.) <br>
자바 1.5부터 고수준 병행성 유틸리티가 포함되어, 예전엔 wait, notify로 했던 일들을 대신하게 되었다. 저수준 벼행성 유틸리티를 사용하는 것이 어렵기 때문에 고수준 유틸리티를 반드시 이용해야 한다. 고수준 유틸리티에는 크게 Executor Framework, Cuncurrent Collecton, Synchronizer가 있다. Executor Framwork은 68장에서 다루었으므로 이번 장에서는 나머지 두개를 다룬다.

#### Concurrent Collection
* 이 컬렉션들은 병행성을 높이기 위해 동기화를 내부적으로 처리한다. 
* 따라서 컬렉션 외부에서 병행성을 처리하는 것은 불가능하다. 락을 걸어봐야 아무 효과도 없고 속도만 느려진다.
* 따라서 클라이언트는 이 컬렉션에 대한 메서드 호출을 원자적으로 작성할 수 없다. 대신 몇가지 기본 연산들을 원자적으로 묶어 제공한다. ex) putIfAvsent(key, value)

###### ConcurrentMap 예시
```language-java
// Concurrent canonicalizing map atop ConcurrentMap - not optimal
private static final ConcurrentMap<String, String> map =
   new ConcurrentHashMap<String, String>();

public static String intern(String s) {
   String previousValue = map.putIfAbsent(s, s);
   return previousValue == null ? s : previousValue;
}
```

```language-java
// Concurrent canonicalizing map atop ConcurrentMap - faster!
public static String intern(String s) {
	String result = map.get(s);
	if (result == null) {
       result = map.putIfAbsent(s, s);
       if (result == null)
		   result = s;
	}
   
    return result;
}
```
* 병행성이 높을 뿐 아니라, ConcurrentHashMap은 아주 빠르다.
* Collections.synchronizedMap 이나 Hashtable 대신 ConcurrentHashMap을 사용하도록 하자. 

###### Blocking operation
* 컬렉션 인터페이스 가운데 몇몇은 blocking 연산이 가능하도록 확장되기도 했다. ex) BlockingQueue 의 take연산
* ThreadPoolExecutor를 비롯한 대부분의 ExecutorService 구현은 BlockingQueue를 사용한다.

#### Synchronizer
스레드들이 서로 기다릴 수 있도록 하여, 상호 협력이 가능하게 한다.

###### CountDownLatch
```language-java
// Simple framework for timing concurrent execution
public static long time(Executor executor, int concurrency, final Runnable action) throws InterruptedException {
	final CountDownLatch ready = new CountDownLatch(concurrency); 
	final CountDownLatch start = new CountDownLatch(1);
	final CountDownLatch done = new CountDownLatch(concurrency); 

	for (int i = 0; i < concurrency; i++) {
       executor.execute(new Runnable() {
           public void run() {
               ready.countDown(); // Tell timer we're ready
               try {
                   start.await(); // Wait till peers are ready
                   action.run();
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt(); // [스터디 중 나온 코멘트] 이 예외 무시하지말고 이렇게 꼭 처리해라. 이렇게 해야 예외로 인해 프로그램이 종료되지 않으면서도, 밖에 그대로 interrupt를 발생시킨다.
               } finally {
                   done.countDown();  // Tell timer we're done
               }
			}
		});
	}
	
	ready.await();
	long startNanos = System.nanoTime();
	start.countDown(); // And they're off!
	done.await(); // Wait for all workers to finish 
	return System.nanoTime() - startNanos;
}
```
* 이 코드에서 주의할 것은 executor는 주어진 병행성 수준만큼의 스레드를 생성할 수 없으면, 테스트가 결코 끝나지 않는다는 것 - 이런 문제를 thread starvation deadlock 이라고 한다.
* 특정 구간의 실행시간을 잴 때는 System.currentTimeMills 대신 System.nanoTime을 사용해야 한다! (정밀하고, 시스템의 실시간 클락 변동에 영향을 받지 않기 때문에)

#### 저수준 병행성 유틸리티 wait, notify
###### wait idiom
```language-java
// The standard idiom for using the wait method
synchronized (obj) {
    while (<condition does not hold>) 
    //한글책에서는 "이 조건이 만족되지 않을 경우에 순환문 실행" 이라 번역됨
		obj.wait(); // (Releases lock, and reacquires on wakeup)
    
    ... // Perform action appropriate to condition
}
```

###### 조건이 만족되지 않았는데 스레드가 깨어나는 이유
**여기 다 이해 안됨 ㅠㅠ 일단 복붙**

* (1) 다른 스레드가 notify를 호출한 덕에 (2) wait 상태에서 벗어나긴 했는데, (1)과 (2) 사이에 또 다른 스레드가 락을 획득해서 해당 락이 보호하는 상태를 바꿔버린 경우.
* 조건이 만족될 수 없는 상황에서 다른 스레드가 우연히 또는 악의적으로 notify를 호출한 경우. 외부로 공개된 객체에 wait를 호출하면 이런 상황에 노출된다. 클래스 외부로 공개된 객체에 정의된 동기화 메서드 안에서 wait를 호출하면 이런 문제가 생긴다.
* wait를 호출하여 대기 중인 스레드를 깨우는 스레드가 너무 "너그러운" 경우. 예를 들어, 대기 중인 스레드 가운데 소수만이 대기 조건을 만족하는 데도 notifyAll을 호출해 버리는 경우.
* 드물지만, notify 호출 없이도 대기 중인 메서드가 깨어나는 경우. (spurious wakeup이라고 부른다.)

###### notify vs. notifyAll
* notify는 대기중인 스레드가 있다는 가정하에 하나만 깨우고, notifyAll은 전부다 깨운다.
* notify 대신 notifyAll을 사용하는 것이 바람직하다. 클래스 외부로 공개한 객체에 실수로, 또는 악의적으로 notify를 호출하는 상황에 대비해서 wait를 순환문에 넣어 두었듯이, notifyAll을 사용하면 다른 스레드가 우연히, 또는 악의적으로 wait를 호출하는 상황에 대비할 수 있다.

#### 스터디 중 나온 내용
###### Concurrent collection
* 동기화 필요 없지. 속도도 꽤 빠르다.
* 딱 한가지 단점은, 메모리를 무지하게 먹는다는 것. (JDK 6 or 7부터 개선되고 있다.)
* 그리고 특정 operation들은 매우 비싸다. docs를 잘 확인하고 써야 한다.

###### CountDownLatch 사용법
* CountDownLatch가 주어진 count값으로 초기화된다. (`new CountDownLatch(count)` count는 int형 변수)
* 해당 CountDownLatch 객체의 await() 메서드는 다른 쓰레드에서 countDown()이 불리면 count에서 하나씩 감소해가다가 0이 되면 그 즉시 return 된다. 
* 참고로 CountDownLatch는 리셋되진 않는다. 리셋되는 비슷한 기능을 원한다면 CyclicBarrer를 사용하라.
* [java docs](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html)에서 따온 sample example
```language-java
/**
  *The first is a start signal that prevents any worker from proceeding until the driver is ready for them to proceed;
  * The second is a completion signal that allows the driver to wait until all workers have completed.
  */
 
 class Driver { // ...
   void main() throws InterruptedException {
     CountDownLatch startSignal = new CountDownLatch(1);
     CountDownLatch doneSignal = new CountDownLatch(N);

     for (int i = 0; i < N; ++i) // create and start threads
       new Thread(new Worker(startSignal, doneSignal)).start();

     doSomethingElse();            // don't let run yet
     startSignal.countDown();      // let all threads proceed
     doSomethingElse();
     doneSignal.await();           // wait for all to finish
   }
 }

 class Worker implements Runnable {
   private final CountDownLatch startSignal;
   private final CountDownLatch doneSignal;
   Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
      this.startSignal = startSignal;
      this.doneSignal = doneSignal;
   }
   public void run() {
      try {
        startSignal.await(); // 메인쓰레드에서 startSignal.countDown()을 하도록 기다린다.
        doWork();
        doneSignal.countDown();
      } catch (InterruptedException ex) {} // return;
   }

   void doWork() { ... }
 }
```

## Item 70. Document thread safety
#### thread safety 문서화의 중요성
* 클래스의 객체나 static 메서드가 병렬적으로 이용되었을 때, 어떻게 동작하느냐를 문서화해놓지 않으면 프로그래머는 이 클래스를 이용할 때 임의로 어떤 가정을 해야 한다. 하지만 이 가정이 틀렸을 경우, 프로그램은 동기화를 충분히 하지 못하거나, 너무 과도하게 하게 된다. 둘 다 심각한 문제를 야기한다.
* 병렬적으로 사용해도 안전한 스레드가 되려면, 어떤 수준의 thread safety를 제공하는지 문서에 명확하게 남겨야 한다.

#### thread safety level
* **immutable** - 이 클래스로 만든 객체들은 상수다. 외부적인 동기화 메커니즘 없이도 병렬적 이용이 가능. ex) String, Long, BigInteger
* thread-safe
  * **unconditionally thread-safe** - 이 클래스의 객체들은 변경가능하나, 적절한 내부 동기화 메커니즘을 갖추고 있어 외부 동기화 메커니즘 없이도 병렬적 이용이 가능. ex) Random, ConcurrentHashMap
  * **conditionally thread-safe** - 무조건 thread safe 수준과 거의 비슷한데, 몇몇 스레드는 외부적 동기화가 있어야(conditionally) 병렬적으로 이용이 가능하다. ex) Collections.synchronized 계열 메서드가 반환하는 wrapper 객체들. 이런 객체의 iterator는 외부적 동기화 없이 병렬적으로 이용할 수 없다.
  * **non tread-safe** - 병렬적으로 사용하려면 외부 동기화 필요. ex) ArrayList, HashMap
  * **thread-hostile** - 메서드 호출 부분을 외부적 동기화로 다 감싸더라도, 안전하지 않다. 이런 클래스가 되는 것은 보통 동기화 없이 static data를 변경하기 때문. (고의로 이러면 안된다!) ex) System.runFinalizersOnExit (deprecated)


#### 주의사항
* 조건부 스레드 안정성 클래스에 대한 문서를 만들때 신중해야 한다. 어떤 순서로 메서드를 호출할 때 외부 동기화 메커니즘을 동원해야 하는지, 그 순서로 메서드를 실행하려면 어떤 락을 사용해야 하는지 명시해야 한다. 
* 보통은 객체 자체에 락을 걸면 되는데, 예외도 있다. view 역할을 하는 객체의 경우 클라이언트는 원래 객체에 대해 동기화를 해야 한다.
```language-java
Map<K, V> m = Collections.synchronizedMap(new HashMap<K, V>());
   ...
Set<K> s = m.keySet();  // Needn't be in synchronized block
   ...
synchronized(m) {  // Synchronizing on m, not s!
   for (K key : s)
	   key.f();
}
```
* 클라이언트에게 외부로 공개된 락을 제공하는 것은 Dos 공격에 취약하다. 그런 공격을 막는 한가지 방법은 동기화 메서드를 쓰는대신 private lock object를 이용하는 것이다. private lock object는 클래스 바깥에서 이용할 수 없으므로 클라이언트는 객체의 동기화 매커니즘에 개입할 수 없다. (그런데 이 방법은 무조건 thread safety 수준에 적합하지, 조건부 thread-safety에는 적합하지 않다. 조건부는 특정 순서로 메서드를 호출할 때 클라이언트가 어떤 락을 획득하게 되는지 문서에 남겨야 하기 때문.)
```language-java
// Private lock object idiom - thwarts denial-of-service attack
private final Object lock = new Object();
public void foo() {
   synchronized(lock) {
		... 
   }
}
```

## Item 71. Use lazy initialization judiciously
#### 초기화 지연이란?
* 필드 초기화를 실제로 그 값이 쓰일 때가지 미루는 것이다.
* 기본적으론 최적화 기법.
* 클래스나 객체 초기화 과정에서 발생하는 harmful circularity를 해소하기 위해서도 사용한다.

#### 언제 해야 하나?
* 최적화 기법이 다 그렇듯, 정말로 필요하지 않으면 하지 마라.
* 필드 사용 빈도가 낮고, 초기화 비용이 높다면 쓸만할 것이다. 정말로 그런지 확실히 아는 방법은 초기화 지연 적용 전후의 성능을 비교해 보는 것이다.

#### 초기화 지연 기법들 (스레드 안정성 보장)
* 여기서 나온 모든 방법은 numirical primitive type의 초기화 지연에도 동일하게 쓰일 수 있다. (다만 null 체크가 아닌, 0인지 체크해야 한다.) 
* 지연하지 말고 일반적으로 초기화해라.
```language-java
// Normal initialization of an instance field
private final FieldType field = computeFieldValue();
```

* (initialization circularity를 피하고 싶다면) use synchronized accessor 
```language-java
// Lazy initialization of instance field - synchronized accessor
private FieldType field;

synchronized FieldType getField() { 
	if (field == null)
        field = computeFieldValue();
    return field;
}
```


* (for performance on a **static** field) use lazy initialization holder classs idiom
```language-java
	// Lazy initialization holder class idiom for static fields
	private static class FieldHolder {
		static final FieldType field = computeFieldValue();
	}
	
	static FieldType getField() { return FieldHolder.field; }
```

* (for performance on an instance field) use double-check idiom 
  * 필드는 반드시 `volatile`로 선언해야 한다.
```language-java
// Double-check idiom for lazy initialization of instance fields private volatile FieldType field;
FieldType getField() {
    FieldType result = field;
    if (result == null) {  // First check (no locking)
        synchronized(this) {
            result = field;
            if (result == null)  // Second check (with locking)
                field = result = computeFieldValue();
		} 
	}
    return result;
}
```

* (double-check idiom의 variant 1. 여러번 초기화되어도 상관 없을 때) single-check idiom
```language-java
// Single-check idiom - can cause repeated initialization! private volatile FieldType field;

private FieldType getField() {
   FieldType result = field;
   if (result == null)
       field = result = computeFieldValue();
   return result;
}
```

* (double-check idiom의 variant 2. 모든 스레드가 필드 재계산 해도 상관없고, 필드 자료형이 long, double이 아닌 primitive type일 때) racy single-check idiom
  * single check idiom에서 `volatile` 키워드 빠짐
  * String 객체는 해시코드 값을 캐시하기 위해 이 숙어를 사용한다.

### 스터디 중 나온 내용
* lazy initialization 하도록 스터디 중 손코딩 해봄.
* 기억해야 하는 것은
  - 이전의 정석은 volatile keyword + double checking
  - 이것이랑 똑같은 게 holder class pattern

## Item 72. Don't depend on the thread scheduler
#### thread scheduler에 의존하지 마라
* Thread scheduler란? "Thread scheduler in java is the part of the JVM that decides which thread should run.(출처:[javapoint.com](http://www.javatpoint.com/thread-scheduler-in-java))"
* 안정적이지도 않고, 이식성(portability)이 보장되지 않는다. 
* 안정적이고, 즉각 반응하며 (responsive), 이식성이 좋은 프로그램을 만드는 가장 좋은 방법은 runnable thread의 평균 수가 프로세서 수보다 너무 많아지지 않도록 하는 것이다.
* runnable thread의 수를 일정수준으로 낮추는 기법의 핵심은 각 스레드가 필요한 일을 하고 나서 다음에 할 일을 기다리게 만드는 것. 스레드는 필요한 일을 하고 있지 않을 때는 실행중이어서는 안된다. Executor framework 관점에서 보면, 스레드 풀의 크기를 적절히 정하고, 태스크의 크기는 적당히 작게, 그리고 서로 독립적으로 만들라는 뜻.
* 또한 스레드는 busy-wait해서는 안된다. 즉, 무언가 일어나길 기다리면서 공유 객체를 반복적으로 검사해대면 안된다. busy-wait 하는 스레드는 스케쥴러 변화에 취약하고, 프로세서에 높은 부담을 주고, 다른 스레드가 할 수 있는 일의 양을 줄인다. 아래와 같이 구현하면 안된다!!
```language-java
// Awful CountDownLatch implementation - busy-waits incessantly!
public class SlowCountDownLatch {
   private int count;
   public SlowCountDownLatch(int count) {
       if (count < 0)
           throw new IllegalArgumentException(count + " < 0");
       this.count = count;
   }
  
   public void await() {
       while (true) {
           synchronized(this) {
               if (count == 0) return;
           }
	   }
   }

   public synchronized void countDown() {
      if (count != 0)
		  count--;
   }
}
```
<br>

#### `Thread.yield`나 스레드 우선순위에 의존하지 마라.
* 다른 스레드에 비해 충분한 CPU 시간을 받지 못하는 스레드가 있어 프로그램이 겨우 동작하더라도, `Thread.yield`를 사용하지 마라. 임시방편이지 이식성은 없다. 바람직한 해결책은 프로그램 구조를 바꿔 concurrently runnable 스레드의 수를 줄이는 것이다. 
* 스레드 우선순위도 자바 플랫폼에서 가장 이식성이 낮은 부분 가운데 하나다.
* `Thread.yield`를 테스트의 병행성 수준을 높이는 데 사용하는 경우도 있었으나, 제대로 동작하리라는 보장이 없는 방법이었다. 병행성 테스트가 목적이라면 `Thread.yield`대신 `Thread.sleep(1)`을 사용해야 한다.

##  Item 73. Avoid thread groups
* 스레드 시스템이 제공하는 기본적인 추상화 단위에는 스레드, 락, 모니터 이외에도 스레드 그룹(thread group)이라는 것이 있다. 스레드 그룹은 원래 applet을 격리시켜 보안 문제를 피하고자 고안된 것이었으나 그 목적을 달성하지 못했다. 
* ThreadGroup이 제공하는 기능은 많지 않았는데 그 중 특정 Thread 기본연산을 여러 스레드에 동시에 적용할 수 있는 기능이 있었다. 근데 이 기본연산 가운데 상당수가 폐기되었다. 나머지는 별로 쓰이지 않는다. 
* ThreadGroup API 의 thread-safety도 취약하다.
* ThreadGroup.uncaughtException 메서드는 어떤 쓰레드가 무점검 예외를 던졌을 때 제어권을 가져오는 유일한 수단이었으나 1.5부터는 Thread의 setUncaughtExceptionHandler메서드가 같은 기능을 제공한다.
* **결론.** ThreadGroup 은 이제 폐기된 추상화 단위다. 쓰지 마라.  스레드를 논리적인 그룹으로 나누는 클래스를 만들어야 한다면, thread pool executor을 이용하라.





