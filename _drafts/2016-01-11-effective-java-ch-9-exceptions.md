---
layout: post
title: "Effective Java ch 9 Exceptions"
tags: [java, effective java]
comments: true
---

## Item 57. Use exceptions only for exceptional conditions
#### 예외를 남용한 사례
```language-java
// Horrible abuse of exceptions. Don't ever do this!
try {
	int i = 0;
    while(true)
        range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e) {   
}
```
* 왜 위와 같이 이렇게 짰을까?
  * 나름대로 성능을 개선하려 했기 때문 (그렇게 코드 짜면서 성능개선 하지 말라했더니!)
  * 배열 경계 너머를 참조하지 않는지는 VM도 검사할 것이므로, for each 문 안에 컴파일러가 감춰둔 루프 종료 테스트는 중복이니까 없애야 한다는 잘못된 추론 때문이다.
* 이 추론의 세가지 오류
  * 예외는 예외적 상황에 고안된 것이라 JVM 구현자 입장에선 explicit tst만큼 빠르게 만들 이유가 없다.
  * try-catch 안에 넣어둔 코드에는 최신 JVM이 사용하는 최적화 기법의 일부가 적용되지 않는다.
  * 배열을 순회하는 표준숙어가 중복검사로 이어지리라는 생각은 틀렸다. 최신 JVM은 그런 부분도 최적화해 없앤다.
* 위와 같은 코드의 문제점
  * 의도와 반대로 성능이 더 안좋다.
  * 코드의 원래 목적을 흐린다(가독성, 유지보수 낮춤)
  * 올바른 동작을 보장할 수 없다. 관련 없는 버그 때문에 순환문이 조용히 종료되면 버그의 존재는 감춰지므로 디버깅이 어려워진다. 
#### 따라서 예외는 예외적 상황에만
  * ordinary control flow에 이용하면 안된다.
  * 표준적인 숙어대로 코딩해야지, 더 좋은 성능을 내려고 너무 머리를 굴리지 마라.
  * 잘 설계된 API는 클라이언트에게 ordinary control flow의 일부로 예외를 사용하도록 강요하지 않는다. 
#### state-dependent methods
  * 상태 종속적 메서드: 특정한 예측 불가능 조건이 만족될 때만 호출할 수 있는 메서드
  * 상태 종속적 메서드를 가진 클래스엔 대개 해당 메서드를 호출해도 되는지 알기위한 state-testing(상태 검사) 메서드가 별도로 갖춰져있다.
  * 상태검사 메서드를 제공하기 싫다면, null같은 특이값(distinguished value)가 반환되도록 구현하는 방법도 있다. 
    - 외부적인 동기화 메커니즘 없이 병렬적으로 사용될 수 있는 객체거나, 외부적 요인으로 상태변화가 일어날 수 있는 객체라면 반드시 이 방식으로 구현해야 함. (상태 검사 메서드를 호출하고 상태 종석 메서드를 호출하는 사이 시간에 상태가 변할 수 있기 때문)
  * 하지만 모든 조건이 동일하다면, 상태검사 메서드를 두는 게 대체로 바람직. 가독성 더 좋고, API를 잘못 사용하는 경우도 쉽게 발견할 수 있기 때문에. 

## Item 58. Use Checked exceptions for recoverable conditions and runntime exceptions for programming erros
![Throwable hierarchy](http://www.c-jump.com/bcc/c257c/Week05/const_images/throwable_hierarchy.png)
(그림 출처: http://www.c-jump.com/bcc/c257c/Week05/Week05.html)

#### 특징 및 사용 목적
* checked exception은 명시적으로 반드시 catch절로 잡아내든지, 밖으로 던져지도록 해야 한다. 복구할 것으로 여겨지는 상황에 대해서는 checked exception을 사용해야 한다. **호출자 측에서 상태를 복구하는 데 이용할 정보를 제공하는 메서드를 갖춰놓는 게 중요하다.** 
* unchecked error, uncheced exception (runtime exception)의 동작방식은 같다. catch로 처리할 필요가 없으며, 일반적으로 처리하지 않아도 된다. 이들이 던져졌다는 것은 복구 불가능한 상황에 직면했다는 의미이므로 적절한 오류 메시지를 내면서 프로그램이 중단된다.
* 프로그래밍 오류를 표현할 때엔 runtime exception을 사용하라. 일반적으로 `Error` 의 하위클래스는 새로 만들지 않는다. 사용자 정의 unchecked throwable 객체를 만들 때엔 `RuntimeException`의 하위 클래스로 만들자.

## Item 59. Avoid unnecessary use of checked exceptions
* checked exception을 너무 남발하면 사용하기 불편한 API가 될 수도 있다.
* 이정도밖에 할 수 있는 게 없다면 checked에서 unchecked로 바꾸는 게 좋다!
```language-java
} catch(TheCheckedException e) {
       throw new AssertionError(); 
}

//or

} catch(TheCheckedException e) {
       e.printStackTrace();        
       System.exit(1);
}
```

<br>
#### 예외 사례
* `CloneNotSupportedException`: `Cloneable`인터페이스를 구현하지 않은 객체에 clone 메서드를 호출하면 발생하는 예외. unchecked 로 하는 게 더 적합할 거라고 하는데... -> **클론이 안되었으면 다른 방향으로 복사를 시도해볼 수 있는 거아닌가? 복구 가능한 예외같기도 하구...**

#### checked exception 없애기
* 메서드가 던지는 checked 예외가 하나뿐이라면, 이걸 없앨 수 있는 방법이 있을지 고민해보는 게 좋다.
* 예외를 던지는 메서드를 둘로 나눠서 첫번째 메서드가 boolean값을 반환하도록 만드는 것이다. 아래 코드를 보자. (하지만 이게 Item 57의 state-check 메서드와 본질적으로 같기 때문에 갖고 있는 문제점도 같다. 따라서 병렬 프로그래밍 상황에서는 적합하지 않을 수 있다.)
```language-java
//This API refactoring transforms the calling sequence from this:
// Invocation with checked exception
   try {
       obj.action(args);
   } catch(TheCheckedException e) {
       // Handle exceptional condition
       ...
   }

// to this:
// Invocation with state-testing method and unchecked exception
   if (obj.actionPermitted(args)) {
       obj.action(args);
   } else {
       // Handle exceptional condition
       ...
   }
```

## Item 60. Favor the use of standard exceptions
#### 재사용되는 기본적인 unchecked exceptions 목록
<table>
  <tr>
    <td><b>Exception</b></td>
    <td><b>Occation for Use</b></td>
  </tr>
  <tr>
  	<td>IllegalArgumentException</td>
  	<td>Non-null parameter value is inappropriate</td>
  </tr>
  <tr>
  	<td>IllegalStateException</td>
  	<td>Object state is inappropriate for method invocation</td>
  </tr>
  <tr>
  	<td>NullPointerException</td>
  	<td>Parameter value is null where prohibited</td>
  </tr>
  <tr>
  	<td>IndexOutOfBoundsException</td>
  	<td>Index parameter value is out of range</td>
  </tr>
  <tr>
  	<td>ConcurrentModificationException</td>
  	<td>Concurrent modification of an object has been detected where it is prohibited</td>
  </tr>
  <tr>
  	<td>UnsupportedOperationException</td>
  	<td>Object does not support method</td>
  </tr>
</table>

* 예외를 발생시키는 조건이 해당 예외의 문서에 기술한 것과 일치해야한다. 이름만 보고 재사용하면 안되고, 의미적으로 재사용이 가능해야 한다.

## Item 61. Throw exceptions appropriate to the abstraction 
#### 추상화 수준에 맞는 예외를 던져라
상위 계층에서는 하위 계층에서 발생하는 예외를 받아서 상위 계층 추상화 수준에 맞는 예외로 바꿔 던져야 한다.

  * 그렇지 않으면 메서드가 하는 일과 뚜렷한 관련성이 없는 예외가 발생할 수 있다.
  * 추상화 수준이 높은 API가 구현 세부사항으로 오염되는 일까지 벌어진다.
  * 다음번 릴리즈에 상위 계층 구현이 바뀌면 해당 계층에서 발생하는 예외도 바뀔 것이고, 결국 클라이언트 프로그램이 깨질 가능성이 높아진다.

#### exception translation 숙어
```language-java
   // Exception Translation
   try {
       // Use lower-level abstraction to do our bidding
       ...
   } catch(LowerLevelException e) {
       throw new HigherLevelException(...);
   }
```

```language-java
// 예외 변환 숙어 사용 예시
/**
* Returns the element at the specified position in this list. * @throws IndexOutOfBoundsException if the index is out of range * ({@code index < 0 || index >= size()}).
*/
   public E get(int index) {
       ListIterator<E> i = listIterator(index);
       try {
           return i.next();
       } catch(NoSuchElementException e) {
           throw new IndexOutOfBoundsException("Index: " + index);
       }
}
```

#### exception chaining
예외 연결을 이용하면 접근자 메서드(Throwable.getCause)를 이용해 해당정보를 꺼낼 수 있다.
```language-java
// Exception Chaining
   try {
       ...  // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) { throw new HigherLevelException(cause);
}
```
```language-java
// Exception with chaining-aware constructor
   class HigherLevelException extends Exception {
       HigherLevelException(Throwable cause) {
           super(cause);
       }
}
```

#### 밖으로 예외를 던질 때 신중하자.
- 가능하다면 하위계층에서 예외가 생기지 않도록 하는 게 제일 좋다. 하위 계층 메서드에 인자를 전달하기 전에 인자의 유효성을 미리 검사하는 것도 한 방법이다.
- 하위 계층에서 생기는 문제를 상위 계층 메서드 호출자로부터 격리시키는 것도 생각해보라. 하위 계층에서 발생하는 예외를 어떤식으로 처리해버리는 것. 이래야 하는 상황이라면, 로그를 남겨 최종사용자에겐 문제를 감추지만 관리자는 나중에 분석할 수 있도록 하는 게 좋다.

## Item 62. Document all exceptions thrown by each method
메서드를 올바르게 사용하려면, 메서드에 던져지는 예외애 대한 설명이 문서에 있어야 한다.

* checked exception 은 독립적으로 선언하고, 해당 예외가 발생하는 상황을 Javadoc @throws 태그를 이용해 정확하게 밝히자.
* unchecked exception은 메서드 선언부의 throws 뒤에 나열하진 않되, Javadoc @throws 태그를 사용해서 문서를 남기자.

## Item 63. Include failure-capture information in detail messages
* 예외때문에 프로그램이 죽으면, 시스템은 자동적으로 해당 예외의 stack trace를 출력한다.
* 이 때 stack trace 정보는 예외 객체의 클래스 이름과 상세 메시지(detail message)로 구성되어 있다.
* 오류 정보를 포착해 내기 위해서는, 이 상세메시지에 오류와 관계된 모든 인자와 필드의 값을 포함시켜야 한다. ex) IndexOutOfBoundsException의 예외외 메세지에는 index의 lower bound, upper bound, index value가 포함되어야 한다.
* 상세메시지에 적절한 정보를 담는 한가지 방법은, 그 정보를 요구하는 생성자를 만드는 것이다. 다음과 같이.
```language-java
/**
* Construct an IndexOutOfBoundsException.
*
* @param lowerBound the lowest legal index value.
* @param upperBound the highest legal index value plus one.
* @param index      the actual index value.
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
   // Generate a detail message that captures the failure
   super("Lower bound: "   + lowerBound +
         ", Upper bound: " + upperBound +
         ", Index: "       + index);
   // Save failure information for programmatic access
   this.lowerBound = lowerBound;
   this.upperBound = upperBound;
   this.index = index;									
}
```
* checked exception인 경우 detail message에 등장한 정보들에 대해 접근자 메서드를 제공해야 한다. unchecked exception에 대해선 접근자 메서드를 제공하는 것이 그리 필요하진 않을 수 있지만, 갖춰두는 걸 권하기는 한다. (Item 10 참고)

## Item 64. Strive for failure atomicity
#### 실패 원자성
* 메서드 호출이 정상적으로 처리되지 못한 객체의 상태는 메서드 호출 전 상태와 동일해야 한다. 이 속성을 만족하는 메서드는 실패 원자성을 갖추었다고 한다.

#### 실패 원자성을 달성하는 방법
* 변경불가능 객체로 설계
* 실제 연산을 수행하기 전에 인자 유효성을 검사한다.
```language-java
public Object pop() {
   if (size == 0)
       throw new EmptyStackException();
   Object result = elements[--size];
   elements[size] = null; // Eliminate obsolete reference
   return result;
}
```
* 실패 가능성이 있는 코드를 전부 객체 상태를 바꾸는 코드 앞에 배치한다. ex) TreeMap에 이전에 들어가 있는 원소들과 비교 불가한 원소를 넣으려고 하면, 트리를 변경하기 전에 트리 안에서 해당 원소를 찾다가 ClassCastException이 난다. (add 하는데 왜 원소를 찾지? 모든 원소를 다 둘러본다는 얘긴가?)
* 사용빈도가 낮지만, 연산 수행 도중에 발생하는 오류를 가로채는 복구코드를 작성하는 방법도 있다. 디스크 기반의 지속성(durable) 자료구조에 주로 사용된다.
* 객체의 임시 복사본상에서 필요한 연산을 수행하고, 연산이 끝난 다음에 임시 복사본의 내용으로 객체를 바꾸는 방법도 있다. 

#### 언제나 실패 원자성을 보장해야 하나?
* 언제나 달성할 순 없다. 같은 객체를 여러 스레드가 적절한 동기화 없이 동시에 변경하면, 객체 상태의 일관성은 깨질 수 있다. ConcurrentModificationException이 발생한 뒤에는 객체 상태는 망가져 있으리라 보는 것이 좋다. 
* 예외와 달리 오류는 복구 불가능하며, 굳이 실패 원자성을 보존할 필요 없다.
* 실패 원자성을 지키지 않는 메서드의 경우, 객체 상태가 어떻게 변하는지 API문서에 명확하게 서술해야 한다.

## Item 65. Don't ignore exceptions
```language-java
// Empty catch block ignores exception - Highly suspect!
try {
 ...
} catch (SomeException e) {
}
```
* 위와 같이 절대 하지 말아라. (catch블록을 비워두지 말아라!)
* 적어도 catch 블록 안에는 예외를 무시해도 괜찮은 이유라도 주석으로 남겨둬야 한다.
