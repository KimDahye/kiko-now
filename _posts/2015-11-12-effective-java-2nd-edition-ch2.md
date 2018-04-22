---
layout: post
title: "Effective Java (2nd Edition) Ch2 정리"
tags: [java]
comments: true
---

Ch2 는 객체의 생성과 삭제에 관한 내용이다.

## 1. Consider static factory methods instead of condsturtors.

#### static factory 를 왜 써야 하는가?
- They have names! <br>=> 잘 지어진 이름은 client에게 높은 가독성을 준다. 특히 같은 signiture로 생성자를 만들 수 없는데, 이게 이름을 구분함으로 가능해진다!
- 메소드가 불렸을 때 당장 instance를 만들지 않아도 된다. 이미 있던 인스턴스를 반환하거나 함으로써, sigleton을 유지하거나 instance pool을 유지할 수 있다. <br>=> 이런 게 가능해지면 immutable class를 만들 수도 있다. <br> => 그럼 비교를 equals로 안하고 ==으로 할 수 있게 되며 이럼 성능을 높여줄 수 있다.
- return type의 any subtype을 반환가능하다. => 유연성 높아짐
- parameterized type의 경우 type inference 가 가능 - java 1.7에서는 언어 자체적으로 지원한다고 함
- [단점인듯 장점] static factory method만 있는 클래스를 만들면 생기는 가장 큰 문제는, public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다. public static 팩터리 메서드가 반환하는 비-public 클래스도 마찬가지. <br> (근데 이래서 더 좋다는 사람도 있음. 상속보다는 조합을 쓰게 되므로!)

#### 주의 할 점 (단점)
- static factory 메소드가 다른 static 메소드와 확연히 구분되지는 않는다!

#### 스터디 하면서 언급된 사항
* serializable은 생성자를 안타고 좀 특별하게 셋팅됨.
* externalizable은 serializable과 또 다름. 데이터만 넣는 걸로 최적화된 녀석이다.
* Service Provider Framework - 사용하는 구체적인 기술 스펙과 상관없이 추상화해서 service를 넘겨받는 거에도 이 패턴이 쓰임.
* EnumSet도 factory method로 내부적으로 최적화되어 구현되어 있다는 것 짚고 넘어감.

## 2. Consider a builder when faced with many constructor parameters
필드 멤버가 많은데, 특히 대부분의 멤버가 optional일 때 어떻게 클래스를 작성할까?

#### 1) telescoping constructor pattern:
>In short, the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.

즉 쓰기가 어렵고, 또 코드 읽기도 어렵다는 말씀!

#### 2) JavaBeans Pattern - setter 폭격!

immutable 클래스를 만들 수 없음!

#### 3) 그래서 나타난 Builder Pattern
* public static 으로 Builder 클래스 내부에 선언
* Builder에 외부 클래스와 똑같이 필드 선언하고
* 필수적인것은  생성자에, 선택적인 것은 setter(return this 하는)로
* build 메소드 안에서 바깥 클래스의 private 생성자 호출

```java
public class User {
     private final int id;
     private final String name; //optional
     private final int age;  //optional

     public static class Builder {
          private final int id;
          private final String name = “default”;
          private final int age = 0;

          public Builder (int id) {
               this.id = id;
          }

          public Builder name(String name) {
               this.name = name;
          }
          public Builder age(int age) {
               this.age = age;
          }

          public User build() {
               return new User(this);
          }
     }

     private User (Builder builder) {
          id = builder.id;
          name = builder.name;
          age = builder.age;
     }
}
```
#### Builder Pattern의 특징

- [장점] **NutritionFacts 클래스는 immutable하고,** all parameter default values are in a single location(한곳에 모여있음)
- [장점] 빌더에 정의된 setter method는 builder 자신(return this!)을 반환하므로, chaining하여 깔끔하게 메소드 콜 할 수 있음! => **method로 적절한 naming을 줄 수 있다.**
``` java
User sophie = new User.Builder(1).name(“sophie”).age(20).build();
```
- [장점] 생성자와 마찬가지로 빌더 패턴을 사용하면 인자에 불변식(parameter 가 올바른 범위를 갖고 있는지 체크하는 것인가?)을 적용할 수 있다. (build method, setter method에서)
- [장점] 여러개의 varargs(인자 개수 정해지지 않는 인자 ... 붙은 거) 가질 수 있다. ( 원래는 메소드마다 마지막에 하나만 갖는데, 얘는 setter가 여러개라서)
- [장점] 또한 빌더 패턴은 유연하다. 하나의 빌더 객체로 여러 객체를 만들 수 있음. 어떤 필드의 값은 자동으로 채울 수도 있음.
- [단점] 객체를 생성하려면 우선 빌더 객체를 생성해야 한다.
- [단점] 또한 telescoping constructor pattern보다 많은 코드를 요구하기 때문에 인자가 충분히 많은 상황에서 이용해야 한다. (4개 이상)

#### 요약
빌더 패턴은 인자가 많은 생성자나 static factory 가 필요한 클래스를 설계할 때, 특히 대부분의 인자가 선택적 인자인 상황에 유용하다. (telescoping보다 가독성이 좋고, java bean 사용할 때보다 thread safe - immutable 하니까)

#### 참고사항
Class.newInstance는 컴파일 시점에 예외 검사가 가능해야 한다는 규칙을 깨뜨려 안쓰는 게 좋다.

## 3.  Enforce the singleton property with a private constructor or an enum type
#### private constructor
싱글톤을 구현하는 우리가 흔히 쓰는 방법 두가지는 다음과 같다.

* 생성자를 private 로 선언하고 public static final로 선언한 필드변수에 인스턴스 저장
* 생성자를 private 로 선언하고
`public static ClassType getInstance() { return INSTANCE; }`


위의 두 방법 모두 private 생성자를 reflection 통해 부를 수 있음에 주의해야 한다.

성능에 대한 것은, 두번째 방법이 method를 이용하니까 첫번째 방법의 성능이 더 좋다고 생각할 수 있지만, 최신 JVM은 static fiactory method호출을 거의 항상 inline으로 처리하기 때문에 성능상의 차이는 거의 없다고 봐도 된다.

위의 두 방법으로 구현한 싱글턴 클래스를 직렬화하려면 `implements Serializable`을 추가하는 것으로는 부족. 모든 필드를 `transient`로 선언, `readResolve` method 추가해야함. (그렇게 하지 않으면 deserialize할 때 새로운 객체가 생기게 된다.)

#### 원소가 하나뿐이 enum
* JDK 1.5부터는 싱글턴 구현할 때 새로운 방법을 사용할 수 있는데 바로 원소가 하나뿐인 enum 자료형을 쓰는 것!
* 직렬화가 자동으로 처리되고, 리플렉션을 통한 공격에도 안전하다고.
* 저자는 원소가 하나뿐인 enum 자료형이야 말로 싱글턴을 구현하는 가장 좋은 방법이라고 주장한다.

## 규칙4. Enforce noninstantialbility with a private constructor. 객체 생성을 막을 때는 private 생성자를 사용하자!
객체 생성을 막을 때는 언제냐? java.lang.Math 같은 유틸리티 클래스처럼, static field와 static method만 사용하는 경우.
private consturctor를 사용하고, 그 안에서 throw AssertionError를 두면, 클래스 안에서 생성되는 것도 알아차릴 수 있음.

#### 참고사항
* AssertionError는 real에서는 밖으로 던져지진 않는다고 함.

## 규칙5. Avoid creating unnecessary objects
객체를 필요할 때마다 만드는 것보다 재사용하는 편이 더 빠르고 우아함(?). 변경 불가능 객체는 언제나 재사용할 수 있다!

#### case 1. String
`"hello"`자체가 객체이므로,
```java
String s = new String("hello"); // 이렇게 하면 안되고
String s = "hello"; // 이렇게 하시오.
```

#### case 2. constant
메소드 안에 상수로 쓸 만한 것들은 넣지 마세요. 매번 객체를 만들게 되니까.  
(@@ 참고로 Calendar 는  tread-safe 하지 않음 - synchronized, thread local을 이용. java8에서는 Calendar를 대체하는 새로운 게 나옴)

#### case 3. adapter pattern에서 adapter 여러번 만들 필요 없다.
아래의 내용은 adapter pattern 을 공부해봐야 이해될 것 같다. 일단 정리해둠.
> 또 뷰 라고도 부르는 어댑터의 경우를 생각해보면, 어댑터는 실제 기능 수행은 후면 객체(backing object)에 위임하며, 후면 객체에 대한 또 다른 인터페이스를 제공하는 객체다. 후면 객체가 관리하는 것 이외의 정보는 따로 저장하지 않으므로, 특정 객체에 대한 어댑터를 하나 이상 만들 필요는 없다. (예를 들어, Map 인터페이스의 keySet은 Map 객체의 Set 뷰를 반환. 얼핏 생각하면 keySet을 호출할 때마다 새로운 Set 객체가 반환될 것 같지만, 같은 Map에 keySet을 여러번 호출하면 실제로는 같은 Set 객체가 반환된다.



#### case 4. use primitive type (not boxed primitive type)
아래와 같이 unintentional autoboxing이 일어나지 않도록 주의하세염!
```java
Long sum = 0L; //여기가 문제가 되겠죠?
for (long i = 0; i < Integer.MAX_VALUE; i++) {
     sum += i;
}
```

#### 하지만 주의할 점
일반적으로 독자적인 객체 pool 을 유지하는 건 코드 복잡성을 늘린다. 객체 생성 비용이 극단적으로 높은 게 (ex - db connetion) 아니라면 하지 마라. 메모리 요구량이 많아져 성능도 저하됨. **재사용이 가능하다면 새로운 객체를 만들면 안되지만, 새로운 객체를 만들어야 한다면 재사용하면 안됨!**

## 6. eliminate obsolete object references
```
public class Stack {
     private Object[] elements;

     //생략

     public Object pop() {
          if(size == 0) throw EmptyStackException();
          return elements[—size];
     }
}
```
위의 스택 예제는 불필요한 레퍼런스지만 element에 저장하고 있다. 따라서 GC 수집하지 않고, 메모리 요구량 증가!<br>

다음과 같이 null 로 reference 없애야 한다.
```
public Object pop() {
     if(size == 0) throw EmptyStackException();
     Object result = elements[—size];
     elements[size] = null; // Eliminate obsolete reference
     return result;
 }
```

하지만, 객체 참조를 null 처리하는 것은 norm(표준 규정)이라기 보다는 exception이어야 함. (예외적인 거여야 함!)

그럼 이 예외적인 경우는 언제? 바로 자체적으로 메모리를 관리할 때.

####캐시도 메모리 누수가 흔히 발생하는 장소!
해결하는 방법은 WeakHashMap을 가지고 캐시를 구현하는 것. (캐시 바깥에서 키를 찹조하고 있을 때만 값을 보관하면 될 때 쓸 수 있는 전략)

#### 또한 콜백도 메모리 누수가 흔히 발견됨!
callback을 명시적으로 제거하지 않을 경우 적절한 조취를 취하기 전까지는 메모리가 점유되어 있다. 어떻게 해결? GC가 즉시 처리할 수 있도록 callback의 weak reference만 저장하는 것

```
//Finalizer Guardian idiom
public class Foo {
     // Sole purpose of this object is to finalize outer Foo object
     private final Object finalizerGuardian = new Object() {
           @Override protected void finalize() throws Throwable {
               … // Finalize outer Foo object
          }
     };
     … // Remainder omitted
}
```

## 규칙7. Avoid finalizer
- finalizer란 녀석... 언제 실행될 지 모름. 긴급한 작업을 종료자 안에서 처리하면 안됨!!!!! (종료자 실행시점은 GB 수집 알고리즘 전략에 의해 결정된다.)
- 그럼 파일이나, 스레드처럼 명시적으로 반환하거나 삭제해야 하는 자원을 포함하는 객체의 클래스는 어떻게 작성하나? 명시적인 종료 메서드를 하나 정의하고 그 메서드를 호출해야 한다. 객체 종료를 보장하기 위해 try-finally문과 쓰여야 함.
- finalizer 사용이 적합한 경우는? 명시적 종료 메서드 호출을 잊은 경우에 대비하는 안전망으로서의 역할, 네이티브 피어(native peer)와 연결된 객체를 다룰 때.  (네이티브 피어 객체는 일반 객체가 아니므로 GB가 알 수 없을 뿐더러 자바측 피어 객체가 반환될 때 같이 반환할 수도 없다. 네이티브 피어가 중요한 자원을 점유하고 있지 않다고 가정한다면 종료자는 그런 객체의 반환에 걸맞다.)
- 주의할 거 하나더. finalizer chaining은 자동으로 이뤄지지 않음. 그래서 아래와 같은 idiom을 쓴다고 합니다.



## 읽으면서 들었던 질문
#### static factory method vs. Factory Method in Design pattern
* static factory method는 그 클래스 안에서 생성자 대신 이 클래스의 인스턴스를 만들 수 있는 api인 것이고.
* factory method pattern은 [여기](http://www.tutorialspoint.com/design_pattern/factory_pattern.htm) 예제를 보면 쉽게 알 수 있는데 한 interface를 구현한 여러개의 concrete class 가 있으면, 이거에 대한 이름을 DI받아서 반환해주는 Factory class 가 따로 있는 패턴.

####  serialize에서 transient  뜻?
직렬화 할 때 그 필드는 직렬화에 포함하지 않겠다는 뜻이다. ([자바의 정석] 참고함)

#### native peer는 뭐지?
JVM이 OS 위에 자기만의 영역을 할당받는데 native는 이 영역을 벗어나 있는 코드이다. Java에서 C를 콜할 수도 있는 게 native. 하지만 컨트롤할 수 있는 영역이 아니고, 디버깅도 못하니깐, 버그 있으면 JVM이 내려가 버린다. 따라서 가급적 native는 쓰지 말자. (but 안드로이드는 쓴다....!)

## 아직 답을 찾지 못한 질문
* 원소가 하나뿐인 Enum 자료형은 어떻게 싱글턴이 되는 걸까? => Enum 을 공부해보자.
* weak reference 관련 naver d2 자료([여기](http://d2.naver.com/helloworld/329631)) 읽어봤는데 weakly reachable object나 unreachable object나 GC가 가비지인지 판별할 때, 다 가비지로 판별되는 것 아닌가? 근데 어떻게 캐시에 사용될 수 있다는 거지?
