---
layout: post
title: "Effective Java Ch 8 General Programming"
tags: [java, effective java]
comments: true
---

지역변수, 제어구조(control structure), 라이브러리 사용, 다양한 자료형 사용, reflection, native method, 최적화, 작명 관습에 관한 내용이다.

## Item 45. Minimize the scope of local variavles.
C에서는 지역변수를 블록 앞부분에 선언한다. 하지만 자바에서는 명령문(statement)을 둘 수 있는 자리에 변수도 선언할 수 있기때문에 C에서의 습관은 고칠 필요가 있다.

* 지역변수의 유효범위를 최소화하는 가장 강력한 방법은, 처음으로 사용하는 곳에서 선언하는 것이다.
  * 근데 중간 중간 변수가 선언된 모습이 오히려 깔끔한 것 같지 않게 느껴지기도 한다. 어짜피 메소드가 한가지 일에 집중한다면, 그래서 메소드가 짧은 라인으로 작성될 수 있다면, 메소드 안에서 필요한 지역 변수들을 한번 쭉 훑은 다음에 로직을 보는 것도 좋지 않을까? (sophie)
  * 하지만, 선언된 곳과 쓰이는 곳이 멀어질수록 변수의 변화를 파악하기 힘들어서 버그를 놓치는 경우가 있을 수도 있겠다; (sophie)
  * 실제로 변수가 사용될 때쯤엔 그 변수의 자료형과 초기값이 무엇이었는지 까먹을 수 있다!
  * 지역변수를 너무 빨리 선언하면 유효범위가 앞으로 확장될 뿐아니라, 뒤쪽으로도 확장된다. 어떤 변수를 원래 사용하려던 곳 이외의 장소에서 실수로 사용하게 되면, 끔찍한 결과를 초래할 수 있겠죠.
* 거의 모든 지역 변수 선언에는 초기값이 포함되어야 한다. 다만 try-catch 블록이 사용될 때는 예외적 상황이 생길 수 있다. 
* for문이나 for-each 문의 경우 loop variable 을 선언할 수 있다. 이녀석의 유효범위는 for 다음에 오는 ()과 순환문 몸체 {} 내부로 제한된다. 따라서 순환문 변수의 내용이 밖에서도 필요하지 않다면, **while문 보다는 for문을 쓰는 것이 좋다.**

```language-java
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator(); 
while (i.hasNext()) { // BUG!
    doSomethingElse(i2.next());
}
```
while문을 쓰면 copy&paste 하면서 변수를 고쳐줘야 하므로 이런 컴파일 타임에 잡을 수 없는 버그가 생길 수있다. for문의 loop variable 은 유효범위 밖에서는 사용할 수 없으므로 위와 같은 버그는 컴파일 타임에 잡힌다. 또한, 각 for문은 서로 의존 성이 없으므로 같은 변수명을 거듭 사용해도 상관 없다. 
```language-java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
    doSomething(i);
}
```
참고. 위와같이 loop variable을 여러개 사용할 수 있다. 위의 코드는 n의 계산비용이 크기 때문에 미리 계산해두고 재계산이 없도록 했다.

* 메서드의 크기를 줄이고 특정한 기능에 집중하라. 두가지 다른 기능이 한데 모이면 한가지 기능을 하는데 필요한 지역변수의 유효범위가 다른 기능까지 확장되어 버린다.

## 46. Prefer for-each loops to traditional for loops
* for-each는 컬렉션을 순회하기 위해 필요한 성가신 변수들을 없애준다.
```language-java
// No longer the preferred idiom to iterate over a collection!
for (Iterator i = c.iterator(); i.hasNext(); ) { 
	doSomething((Element) i.next()); // (No generics before 1.5)
}

// No longer the preferred idiom to iterate over an array!
for (int i = 0; i < a.length; i++) {
    doSomething(a[i]);
}

// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
    doSomething(e);
}
```

* for-each 문은 순환문을 중첩할 때 흔히 저지르는 실수를 없애준다.
```language-java
   enum Suit { CLUB, DIAMOND, HEART, SPADE }
   enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
               NINE, TEN, JACK, QUEEN, KING }
   ...
   Collection<Suit> suits = Arrays.asList(Suit.values());
   Collection<Rank> ranks = Arrays.asList(Rank.values());
   List<Card> deck = new ArrayList<Card>();

   // Can you spot the bug?
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
           deck.add(new Card(i.next(), j.next()));

   // Fixed, but ugly - you can do better!
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
       Suit suit = i.next();
       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		  deck.add(new Card(suit, j.next()));
   }

   // Preferred idiom for nested iteration on collections and arrays
   for (Suit suit : suits)
       for (Rank rank : ranks)
           deck.add(new Card(suit, rank));	

```
* 참고. for-each는 Iterable 인터페이스를 구현하는 어떤 객체도 순회할 수 있다. 원소들의 그룹을 나타내는 자료형을 작성할 때는, Iterable 인터페이스를 구현하도록!

* 참고. 아래의 경우엔 for-each문을 사용할 수 없다. 
  * filtering. 컬렉션을 순회하다가 특정한 원소를 삭제할 필요가 있을 때엔 iterator를 명시적으로 사용하여 remove 메서드를 호출해야 한다.
  * transforming. 리스트나 배열을 순회하다가 일부나 전부의 값을 변경해야한다면, iterator나 index variable이 필요하다. 
  * parallel iteration. 여러 컬렉션을 병렬적으로 순회할 땐 iterator나 index variable이 필요하다. (앞서 살펴봤던 카드나 주사위 예제에서 버그만 무시하면 병렬 순회 사례다.)


## Item 47. Know and use the libraries
0부터 n까지 난수를 만들라고 주문하면, 상당수의 프로그래머는 아래와 같이 만든다.
```language-java
private static final Random rnd = new Random();

// Common but deeply flawed!
static int random(int n) {
   return Math.abs(rnd.nextInt()) % n;
}
```
이 코드는 세가지 문제점이 있다.

* n이 그리 크지 않은 2의 제곱수일 때, 이 함수는 오래지 않아 중복된 난수들을 만들어낸다.
* n이 2의 제곱수가 아닐 경우, 어떤 수는 다른 수들보다 평균적으로 더 자주 등장한다.
* 드문 경우이지만, 지정된 구간 밖의 난수를 토해내는 치명적 오류가 발생할 수 있다. 만약 nextInt()가 `Integer.MIN_VALUE` 를 return 하면, Math.abs도 `Integer.MIN_VALUE` 를 return, 여기에  % 적용하면 음수가 만들어진다 (n이 2의 제곱수가 아닐 때). 이러면 프로그램은 십중팔구 오류를 낼 것인데 이런 오류는 재현하기도 까다롭다.

위와 같은 문제점을 해결하는 난수발생기를 직접 만들려고 한다면, 상당한 수학적 지식이 필요하다. 하지만 다행스럽게도, java 1.2부터 포함된 Random.nextInt(int)를 사용하면 된다.

#### libary를 사용할 때의 장점
* 라이브러리가 어떻게 구현되었는지 알 필요가 없다. 이 라이브러리에 대한 전문가가 설계 및 구현하였을 것이고, 먼저 사용한 사람들이 문제를 발견하여 고쳤거나 앞으로 발견되더라도 수정될 것이기 때문.
* 실제로 하려는 일과 큰 관련성이 없는 문제에 대한 해결방법을 임의로 구현하느라 시간을 낭비하지 않아도 된다.
* 별다른 노력을 하지 않아도 성능이 점차 개선된다. 
* 시간이 흐르면서 라이브러리에 새로운 기능이 추가된다. (개발자 커뮤니티에서 요구하면 다음번 릴리즈에 그 기능이 추가되기도 함)
* 주류 개발자들과 같은 코드를 만들게 되어, 가독성이 높고 유지보수가 쉬우며 재사용성이 좋은 코드를 짜게 된다.

따라서 중요한 새 릴리즈가 나올 때마다 많은 기능이 새로 추가되는데, 그때마다 어떤 것들이 추가되었는지 알아두는 게 좋다.

#### 중요 라이브러리를 언급해보자면,
* java.lang, java.util 안에 있는 내용은 잘 알아야 하며, java.io 의 내용도 어느정도 알고 있어야 한다.
* 1.2의 java.util 안에 있는 Collections Framework 
* 1.5의 java.util.concurrent 안에 있는 고수준의 병행성 유틸리티나 저수준의 병행성 유틸리티들

## 48. Avoid float and double if exact answers are required
####float, double 의 용도
  * 넓은 범위의 값에 대해 정확도 높은 근사치를 제공할 때
  * 하지만 여기서 주의할 점은, 결국 근사치여서 정확하진 않다는 점.
  * 특히, 돈과 관계된 계산엔 적합하지 않다!

#### 정확한 계산(a.k.a 돈과 관계된 계산)이 필요할 때
  * BigDecimal: 기본 자료형보다 사용하기 좀 불편해도 괜찮고, 성능이 조금 떨어져도 상관 없을 때. 또 올림 연산이 필요한 경우
  * int or long: 성능이 중요하고, 소수점 아래 수를 직접 관리해도 상관 없으며, 계산할 수가 심히 크지 않을 때

##Item 49. Prefer primitive types to boxed primitives
#### primitive vs. boxed primitive
* primitive 자료형은 값만 가지나, boxed primitive 자료형은 값 외에도 identity 를 가진다.
* primitive에 저장되는 값은 기능적으로 완전한 값이나, boxed primitive 자료형엔 null(nonfunctional value)이 들어갈 수 있다.
* 일반적으로 primitive 자료형이 시간이나 공간 요구량 측면에서 boxed 보다 효율적이다.

#### 주의 사항
* Integer 객체가 연산자 `<`를 만나면 auto-unboxing 된다.
* 하지만 Integer 객체에 `==`연산을 하면 값 비교를 하지 않고 identity 비교를 한다.
* 따라서 boxed primitive type에 `==` 연산자를 사용하는 것은 거의 항상 오류이다.
* 또한, primitive와 boxed primitive를 한 연산 안에 엮어 놓으면, boxed primitive가 unboxing 되는데.. 이 때  null을 primitive로 변환하려는 경우엔 nullPointerException 이 날 수 있다.
* 아래와 같이 primitive와 boxed를 혼합하여 쓸 경우 무지막지한 성능 이슈가 난다. (Item 5 에서 나온 코드)
```language-java
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
   Long sum = 0L;
   for (long i = 0; i < Integer.MAX_VALUE; i++) {
       sum += i;
   }
   System.out.println(sum);
}
```

#### 언제 boxed primitive를 쓰지?
* 컬렉션의 요소, 키, 값으로 사용할 때 - 형인자 자료형의 형인자로 사용할 때
* 리플렉션을 통해 메서드를 호출할 때

## Item 50. Avoid strings where other types are more appropriate.
* String이 값 자료형을 대신하기엔 부족하다. (numeric type, boolean type)
* String이 enum 자료형을 대신하기엔 부족하다. 
* String이 혼합 자료형을 대신하기엔 부족하다. (`String compoundKey = className + "#" + i.next();` 같이 하면 안된다!) 필드 구분자로 사용한 문자가 필드 안에 들어가버리면 문제가 생기고, 각 필드를 사용하려고 하면 매번 문자열을 parsing해야 하는데 이는 느릴뿐더러 오류 발생 가능성도 높은 과정이다. 유용한 equals, compareTo, toString 메서드도 사용하지 못하고! 이럴 때 private static 멤버 클래스로 자료형은 선언하여 사용하면 된다. 
* String은 capability를 표현하기엔 부족하다. 아래의 예시를 보자.
```language-java
// Broken - inappropriate use of string as capability!
public class ThreadLocal {
   private ThreadLocal() { } // Noninstantiable

   // Sets the current thread's value for the named variable.
   public static void set(String key, Object value);

   // Returns the current thread's value for the named variable.
   public static Object get(String key);
}
```
만일 두 클라이언트가 공교롭게도 같은 지역변수명을 사용한다면, 동일한 변수를 공유하게 되어 둘 다 오류를 낼 것이다. 이는 악의적인 클라이언트의 공격에 취약하므로 보안 문제도 생긴다.

String 키를 쓰는 대신 위조불가능 키로 바꾸면 위의 문제가 해결된다.
```language-java
public class ThreadLocal {
   private ThreadLocal() { }  // Noninstantiable

   public static class Key {  // (Capability)
       Key() { }
   }

   // Generates a unique, unforgeable key
   public static Key getKey() {
       return new Key();
   }

   public static void set(Key key, Object value);
   public static Object get(Key key);
}
```
허나 여기에도 개선의 여지는 있다. 키의 객체 메서드로 set, get 메서드를 만들면 정적 메서드가 더이상 필요 없다. 이렇게 하고 나면, 키는 더이상 스레드의 지역변수의 키가 아니라 그 자체가 스레드의 지역변수가 된다. 
```language-java
public final class ThreadLocal {
    public ThreadLocal() { }
    public void set(Object value);
    public Object get();
}
```
여기에도 개선의 여지는 남아있다. 지역변수에서 값을 꺼낼 때, Object에서 실제 자료형으로 형변환을 해야 하므로 형 안전성이 보장되지 않는다. 제너릭으로 선언하면 이 문제를 해결할 수 있다.
```language-java
public final class ThreadLocal<T> {
    public ThreadLocal() { }
    public void set(T value);
    public T get();
}
```
참고로, 이게 기본적으로 java.lang.ThreadLocal이 제공하는 API이다. 

## Item 51. Beware the performance of string concatenation
* String은 immutable 객체이기 때문에, `+` 연산자를 사용하여 두 문자열을 연결할 때 문자열의 내용이 전부 복사된다. 따라서 `O(n^2)`!
* concatenation 연산이 많을 경우엔, `+` 연산자 말고, StringBuilder의 append연산자를 사용해라. 이녀석은 `O(n)`.

## Item 52. Refer to objects by their interfaces
규칙 40에서 인자의 자료형으로 클래스 대신 인터페이스를 사용하라고 하였다. 규칙 52는 이를 포함하는 좀 더 일반적인 규칙이다. 

* 적당한 인터페이스 자료형이 있다면, parameters, return values, variables, and fields 는 클래스 대신 인터페이스로 선언하자. - 이래야 나중에 더 적합한 구현체로 바꾸고 싶을 때 코드 한 줄로 바꿀 수 있지!
* [주의] 원래 구현이 인터페이스의 일반 규약에 없는 특별한 기능을 제공하고 있고, 원래 코드가 그 기능을 사용하고 있었다면, 적절한 인터페이스를 찾기 어려울 수 있다. 이때는 자료형으로 클래스를 사용하고 변수를 선언할 때 그 사실을 주석으로 남겨 나중에 바꿀 때 이를 인식하고 바꿀 수 있도록 해야 한다. 
* 적당한 인터페이스가 없을 때엔 객체를 클래스로 참조하는 것이 당연하다. (ex. 값 클래스, Random 클래스)
* 인터페이스가 없는 경우에는 필요한 기능을 제공하는 클래스 가운데 가장 일반적인 클래스(대개 abstract class겠지)를 클래스 계층 안에서 찾아서 이용한다.

## Item 53. Prefer interfaces to reflection
리플렉션은 생성자, 메서드, 필드를 조작할 수 있다. 컴파일 될 당시에는 존재하지도 않았던 클래스를 이용할 수 있다. 하지만 이러한 능력에는 대가가 따른다. (reflection은 runtime적인 성격이 아주 강한 녀석이라고 생각하면 될 듯!)
#### 리플렉션의 단점
  * 컴파일 시점에 자료형을 검사할 수 없으므로 이로 인한 이점들을 포기해야 한다. (예외 검사 포함하여...)
  * 리플렉션 기능을 이용하는 코드는 장황하다 - 가독성이 떨어짐.
  * 성능 문제! 리플렉션을 통한 메서드 호출 성능은 일반 메서드 호출에 비해 훨씬 낮다. (일반적인 프로그램에서 프로그램 실행중에 리플렉션을 통해 객체를 이용하려 하면 안된다!)

#### 어떨 때 사용해야 하나?
* 프로그램을 설계할 때 (not 실행할 때) ex) component-based application builder tool. 요청에 따라 클래스를 올린 다음, 리플렉션 기능을 통해 어떤 메서드와 생성자가 지원되는지를 알아낸다. 이렇게 응용프로그램을 대화형으로 만들어나갈 수 있음.
* 리플렉션이 필요한 복잡한 프로그램들 ex) 클래스 브라우저, 객체 검사도구, 코드 분석 도구 등... Spring같은 프레임워크도 리플렉션을 많이 이용하는 프로그램 중 하나겠지. (by sophie)
* 아주 제한적으로 사용할 때 - 예를 들어 객체 생성은 리플렉션으로 하고, 객체 참조는 인터페이스나 상위 클래스를 통하면 될 때.
```language-java
// Reflective instantiation with interface access
   public static void main(String[] args) {
       // Translate the class name into a Class object
       Class<?> cl = null;
       try {
           cl = Class.forName(args[0]);
       } catch(ClassNotFoundException e) {
           System.err.println("Class not found.");
           System.exit(1);
       }

       // Instantiate the class
       Set<String> s = null;
       try {
           s = (Set<String>) cl.newInstance();
       } catch(IllegalAccessException e) {
           System.err.println("Class not accessible.");
           System.exit(1);
       } catch(InstantiationException e) {
           System.err.println("Class not instantiable.");
           System.exit(1);
       }

       // Exercise the set
       s.addAll(Arrays.asList(args).subList(1, args.length));
       System.out.println(s);
}
```
* * 위의 코드가 간단해보여도 완벽한 서비스 제공자 프레임워크(Item 1)에서 쓰이는 기법이다. 
  * 물론 위의 코드에도 단점이 있다. 첫번째는 일단 3가지 runtime error를 발생시키는데, 리플렉션으로 객체를 만들지 않았더라면 컴파일 시점에 검사할 수 있는 오류들이지. 두번째는 클래스 객체를 생성하기 위해 생성자 호출로는 한줄이면 될 걸 엄청난 길이의 코드가 그 일을 하고 있다는 점.
  * 리플렉션과 관련 없지만, `System.exit(1);`으로 프로그램을 종료한다는 점을 주의하자. 이걸 쓰는 건 대체로 바람직하지 않다. VM 전체를 종료시켜 보리기 때문이다! command line utility를 비정상적으로 종료시킬 땐 적당하다.

* 드물지만 리플렉션은 실행시점에 존재하지 않는 클래스나 메서드, 필드에 대한 종속성(dependency)을 관리하는데 적합하다. ex) 어떤 패키지의 버전이 여러가지 버전이고, 그 전부를 지원하는 또다른 패키지를 구현해야 할 때. (와닿지는 않지만, 그냥 그렇구나 정도로 넘어감)

##Item 54. Use native methods judiciously
#### native methods
* C나 C++ 등의 native programming language로 작성된 메서드

#### native methods의 기존 용도
* registry나 file lock 같은 특정 플랫폼에 고유한 기능을 이용할 수 있다.
* 이미 구현되어 있는 라이브러리를 이용할 수 있으며, 그 라이브러리를 통해 기존 데이터를 활용할 수 있다.
* 네이티브 메서드를 사용하면 성능이 중요한 부분의 처리를 네이티브 언어에 맡길 수 있다. 

#### native methods의 단점
* 네이티브 메서드로 성능을 개선할 필요 없다. 현재 JVM이 훨씬 빠르다! 초기 릴리즈에 비해 신중하게 최적화되었기 때문. 오히려 네이티브 메서드를 쓸 때 더 성능이 나빠질 수도 있다.
* 네이티브 언어는 안전하지 않으므로 (Item 39 참고), 네이티브 메서드를 이용하는 프로그램은 메모리 훼손 문제를 일으킬 수 있다. 
* 플랫폼 종속적이라 이식성이 낮다.
* 디버깅이 어렵다.
* glude code를 작성해야 하므로 가독성이 낮다.


## Item 55. Optimize judiciously
- 최적화를 하면서 프로그램을 짜기보다는 좋은 프로그램을 만들려고 노력하자. 
- 좋은 프로그램이란? information hiding이 잘 되어 있는 프로그램 (모듈화, 추상화가 잘 되어 있는 프로그램) 좋은 프로그램을 만들면 모듈끼리 독립적이라 최적화하기도 쉽다.
- 다만, API의 경우엔 한번 배포된 후 성능문제가 발견되면 고치기 쉽지 않으니, 처음부터 설계할 때 내리는 결정들이 성능에 어떤 영향을 끼칠지 생각해야 한다. 
- 최적화를 하자고 결정했다면, 프로파일링 도구의 도움을 받아 최적화 영역을 찾고, 구현에 쓰인 알고리즘부터 검토하자. 저수준의 최적화를 해봐야 알고리즘이 잘못되었다면 최적화 수준이 도토리 키재기정도 일 것. 최적화 후 이것이 정말 최적화되었는지 꼭 확인해봐야 한다. 
- 최적화를 나름 했는데 성능이 더 나빠지는 경우도 있다. 현재 자바의 성능 모델이 제대로 정의되어 있지 않고, JVM마다 릴리즈마다, 프로세서마다, 하드웨어 마다 성능이 다르기 때문이다.

##Item 56. Adhere to generally accepted naming conventions
- 컨벤션을 지키지 않으면, 읽는 사람이 혼란스러워 사용하기가 어렵다.
- 또한 유지보수할 때도 혼란이 된다.

#### typographical
예시로 설명을 대신한다. 

- Package: `com.google.inject`, `org.joda.time.format` (약어 사용 가능)
- Class or Interface: `Timer`, `FutureTask`, `LinkedHashMap`, `HttpServlet`
- Method or Field: `remove`, `ensureCapacity`, `getCrc` 
- Constant Field: `MIN_VALUE`, `NEGATIVE_INFINITY`
- Local Variable: `i`, `xref`, `houseNumber` (약어 사용 가능)
- Type Parameter: T, E, K, V, X, T1, T2


#### grammatical
- Enum, Class : 명사나 명사구.
- Interface: Class와 비슷하지만, able, ible의 어미가 붙기도 한다. 
- 어노테이션은 별다른 규칙 없는듯.
- 메서드는 일반적으로 동사나 동사구(목적어 포함). 특히 boolean을 반환하면 is, has로 시작하는 경우가 대부분.
- toString, toArray // asList (뷰를 반환할 때)
- static factory method: `valueOf`, `of`, `getInstance`, `newInstance`, `getType`, `newType` 
