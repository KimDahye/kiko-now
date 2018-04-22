---
layout: post
title: "Effective Java (2nd Edition) Ch4 정리"
tags: [java]
comments: true
---

자바 추상화의 핵심 요소인 클래스와 인터페이스! 어떻게 하면 좀 더 편리하고, 안정적이며, 유연한 클래스와 인터페이스를 설계할 수 있을까?

## 13. Minimize the accessibility of classes and members.
#### information hiding을 해라!
- how? 적절한 modifier로!
- why?
  - 접근성을 높일수록 다양한 모듈과 coupling되어 각 개발자들이 **전체적으로** 이해해야 하고 소통해야 하기 때문에 개발 속도를 늦춘다.
  - 가장 중요한 이유는 아마 이것일 것. 오픈된 API는 나중에 수정하거나 삭제하고 싶어도 그럴 수 없기 때문에 유연성이 떨어진다.
  - information hiding이 프로그램 성능을 늦추는 것 같지만, profiling이후 성능을 저하시키는 모듈만 독립적으로 성능튜닝을 할 수 있기에 성능도 그렇게 신경쓰지 않아도 된다

#### For top level classes
- public, package-private(default)만 가능
- public 일 때는  API로 취금
- default 일 때는 내부 구현으로 취급됨.

#### For members
- 멤버에는 다음과 같은 것들이 있습니다. fields, methods, nested classes, nested interfaces
- 네가지 modifiers 모두 가능
- 기본은 private. test 를 위해서는 default도 괜찮음. 하지만 그 이상은 위험! (protected 부터는 API로 취급, public은 절대절대 안됨!)
- default를 자꾸 쓰게 된다면 클래스 설계를 다시 해서 decoupling 해야함!
- [주의] 클래스가 Serializable 을 구현했을 경우엔, private, default 필드도 API로 새어나갈 수 있음! - **스터디 첨언**: 직렬화했다는 것은 파일로 저장하거나 해서 공유하기 위한 것이니 당연히 외부로 새어나갈 수 있고, 또 readObject가 생성자를 통해 객체를 생성하는 것도 아니어서 private 필드도 바꿀 수 있다고 함. 따라서 직렬화한 정보는 프록시 패턴을 이용하고, 암호화해서 전달한다고 한다. (자세한 건 Item 74, 75 참고하자.)
- [주의] 상속, 구현할 때엔 superclass의 method의 accessibility를 낮출 수는 없다. 그래서 인터페이스 구현할 때엔 인터페이스에 있던 메소드는 public으로 구현해줘야 함!
- [예외] 클래스 추상화에 있어 핵심적인 상수인 경우, public static 으로 선언해서 API에 포함시킬 수 있음. 이때 해당 필드의 타입은 primitive이거나 immutable 클래스의 참조여야한다.

## 14. In public classes, use accessor methods, not public fields
- public classes should never expose mutable fields.
- public classes can expose immutable fields.
- Sometimes, it is  desirable for package-private or private nested classes to expose fields, whether mutable or immutable

## 15. Minimize mutability
#### Immutable  클래스를 만드는 5가지 규칙
1. mutator(setter)를 만들지 마세요!
2. 클래스가 상속되지 않도록 하세요! 하는 방법은 final로 만들던지,  생성자를 private or default 로 해서 static final factory method를 제공하든지! - **근데 default면 같은 package 안에서는 상속이 가능하지 않나?**
3. 모든 필드는 final로 만드세요! - 이건 좀 과한 규칙이긴 한데, 비용이 높은 계산에 대해 캐시용도로 사용하는 non final field를 갖는 immutable class도 많답니다 :)
4. Make all fields private. private아니면 setter 안 만든 건 소용이 없어지죠~
5. 만약 immutable로 하려는 클래스가 mutable 필드를 갖고 있다면, 이 클래스만 독점적인 접근권한을 갖도록 해주세요~! 부득이하게 이 mutable 필드에 대해 getter를 제공해야할 때엔 defensive copy를 리턴해주세요!

#### immutable 클래스의 장점
* 단순하다. 생성될 때의 상태가 객체의 lifetime 동안의 상태와 같으니까!
* thread-safe하다. 따라서 병렬 프로그래밍할 때 복잡성을 줄여준다. 또 공유해서 사용하기 좋으므로 재사용하기 좋다.
* static factory method를 사용하면, 자주 쓰이는 객체에 대해 static final로 캐싱해두고 이를 반환할 수도 있다.
* defensive copies 를 만들 필요가 없다. 그냥 넘겨줘도 어짜피 다른 이가 바꿀 수 없으니까~
* immutable 객체 자체뿐 아니라 내부 요소도 공유할 수 있다. ex) BigInteger의 negate 메소드 안에서 magnitude 는 복사되지 않는다.
* immutable class는 다른 객체의 필드로 쓰일 때도 훌륭하다. ex) Map keys, Set elements

#### immutable 클래스의 단점과 해결방안
* value가 다른 객체는 각각 따로 만들어져야 한다는 점! -> 시간과 공간 비용이 크죠. 특히 객체가 클 경우엔.
* 특히 "mutistep operation that generates a new object at every step, eventually discarding all objects except the final result"의 경우엔 성능 이슈가 매우 크다!
* 이럴때의 첫번째 해결책은 자주 쓰이는 multistep을 찾고 그것을 primitive 하게 만드는 것 - primitive type을 이용해서 value들 계산하라는 뜻으로 이해함. mutable (package-private) companion class를 사용하여 실제 구현된다. ex) BigInteger
* 두번째 방법으론, mutable public companion class를 제공하여 사용자가 직접 성능 개선을 하도록 열어두는 방법이 있다. 하지만 이것은 성능을 위해 필수적이라고 생각할 때만 심사숙고하여 사용하도록~ ex) StringBuilder 클래스가 String클래스의 companion 클래스 (참고. StringBuffer는 문제가 많아서 쓰지 않는 클래스!)

#### 주의할 점
* Serializable 인터페이스를 구현하도록 했고, 해당 클래스에 mutable 객체를 참조하는 필드가 있다면 readObject메서드나 readResolve메서드를 제공하거나 ObjectOutputStream.writeUnshared 와 ObjectInputStream.readUnshared 메서드를 사용해야한다. (어떤 의미인지 잘 모르겠다. Items 76에서 자세히 알려준다고 한다.)
* immutable 클래스로 만드는 것이 좋지만 부득이하게 mutable 클래스로 만든다하더라도, mutability를 최소화해야한다! 특별한 이유가 없다면 모든 필드는 final로!

## 16. Favor composition over inheritance
package 밖에서 상속을 하는 것은 위험하다. 왜 위험할까? superclass가 새롭게 release되면 subclass가 깨지는 경우가 있기 때문이다. 구체적으로 어떤 경우가 있을까?

#### 상속이 위험한 구체적 이유 (예시 포함)
- HashSet에 원소 개수를 세는 기능을 붙이기 위해 상속을 이용하였다고 하자. HashSet addAll 메소드 안에서 add 함수를 사용한다(self-use). 이런 구체적인 구현을 알아야 제대로 기능을 붙일 수 있다.
- 또한 현재 release를 기준으로 superclass보안코드를 추가한 subclass가 있다면, superclass가 새로운 버전이 나올 때 subclass의 보안수준은 깨질 수 있겠죠.
- 위의 두 문제가 모두 override 관련한 문제라면, override 하지 않고 새 메소드를 추가하는 것은 문제가 없을까? 문제 있다. super class의 다음버전에 subclass의 메소드와 signiture가 같은 메소드가 추가될 수도 있다. 이는 subclass가 compile되지 않도록 하거나 의도치 않은 override가 되게 한다.

#### 이에 대한 대안은 composition
```java
//Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
     private int addCount = 0;

     public InstrumentedSet(Set<E> s> {
          super(s);
     }

     @Override public boolean add(E e) {
          addCount++;
          return super.add(e);
     }

     @Override public boolean addAll(Collection<? extends E> c) {
          addCount += c.size();
          return super.addAll(c);
     }

     public int getAddCount() {
          return addCount;
     }
}

//Reusable forwarding class
//얘가 Set<E>를 implements하는 이유는 WrapperClass를 Set<E> 타입으로 만들고 싶어서인 것 같다. by sophie
public class ForwardingSet<E> implements Set<E> {
     private final Set<E> s;
     public ForwardingSet(Set<E> s) { this.s = s; }

     // s의 메소드로 단순 forwarding하는 메소드들은 생략
}
```

* 요런 패턴을 decorator pattern이라고 한다. 상속을 사용하면 각 concrete set마다 기능추가를 위해 각각 상속해야 하는반면, decorator 패턴을 이용하면 concrete set을 생성자에 넘김으로 해당 set에 이 기능을 덧붙여 쓸 수 있다.

* 대부분의 문제를 해결하는 조합은 한가지 단점이 있는데, 바로 callback frameworks 에서 사용될 때이다. 포장된 객체는 그를 포장하는 객체가 누구인지 모르기 때문에, 자기 자신에 대한 참조(this)를 넘기게 되고, callback은 wrapper를 거치지 않게 된다. [SELF problem] 사람들이 조합의 성능을 문제시하곤 하는데 실제적으로 성능에 큰 영향은 없다.

#### 상속하기 전에 물어보자.
- IS-A 관계 일때: Is every B really an A? (B가 A를 상속하고 싶을 때 다음의 질문을 해보자!)
- 상속하려는 클래스 API에 어떤 결함은 없는가? 결함이 있다면, 그 결함이 내가 만드는 subclass에 전파되어도 괜찮은 결함인가? (조합을 사용하면 결함을 적절히 가릴 수 있다.)

## 17. Design and document for inheritance or else prohibit it.
규칙 16은 상속을 위한 설계와 문서가 갖춰지지 않은 클래스를 상속할 때의 문제점을 설명하고 있다. 그렇다면 상속을 위한 설계와 문서를 갖춘다는 것은 어떤 의미일까?

#### 상속을 위한 문서
- 메서드를 재정의하면 무슨 일이 생기는지 정확하게 문서로 남겨야 한다. overridable (non-final, public or protected) 메서드를 내부적으로 어떻게 사용하는지(self use) 반드시 남겨야 한다.
- 하지만 이는 "좋은 API 문서는 메서드가 하는 일이 **무엇인지** 명시해야지, 그 일을 **어떻게** 하는지 명시해서는 안된다."는 원칙에 위배된다.
- 따라서 상속을 위한 문서는 일반 프로그래머를 위해 설계된 문서를 어지럽힐 수 있다.

#### 상속을 위한 설계
- 상속하는 클래스가 편리하게 사용하도록 신중히 고른, 내부 동작에 개입할 수 있는 protected 메서드가 제공되어야 한다.
- 위의 protected 메소드는 어떻게 고르나? 테스트해볼 뿐!
- [주의] 메서드와 필드를 protected로 선언하는 과정에서 함축된 구현관련 결정들을 **영원히** 고수해야 한다!
- 생성자는 직간접적으로 overridable 메소드를 호출해서는 안된다.
- `Cloneable`, `Serializable`을 인터페이스를 사용할 경우에는 상속용 클래스를 설계하기가 더욱 까다롭고, 일반적으로 이런 인터페이스를  상속용 클래스가 구현하는 것이 바람직하진 않다. 하지만 그래도 해야하는 경우 clone이나 readObject도 생성자와 비슷하게 동작하므로 overridable 메소드를 호출하지 않도록 주의!
- `Serializable` 인터페이스를 구현하는 상속용 클래스에 readResolve, writeReplace 메수드가 있다면, 이 두 메서드는 protected 로 선언해야 한다! (이는 상속을 허용하기 위해 구현 세부사항을 클래스 API의 일부로 오픈하는 사례 가운데 하나..)
- immutable 클래스는 아예 상속용 클래스가 될 수 없다는 것 기억하시고요~

#### 상속을 방지하는 클래스
- final로 선언
- 생성자를 private or package-private으로 선언하고 public static factory 메소드를 추가한다.

#### tip
overridable 메소드를 사용하는 코드는 동작 원리를 바꾸지 않고 overridable 메소드를 사용하지 않도록 바꿀 수 있다!<br>overrdable 메소드의 내용을 priavate helper 메소드 안으로 넣고 해당 동작을 써야 하는 곳에서 다 이 private helper method를 사용하도록 한다!

## 18. Prefer interfaces to abstract classes
Java는 여러 구현을 허용하는 자료형을 정의할 때 두가지 방법을 제공한다. 바로 interface와 abstract class!

#### interface vs abastract class
- 추상 클래스에는 메서드 구현체를 포함할 수 있다. (자바 1.8부터는 interface에도 가능하다!)
- 추상 클래스가 정의하는 자료형을 구현하기 위해서는 추상클래스를 상속해야한다. => 다중 상속 불가능!

#### interface 가 더 좋은 이유
- 이미 있는 클래스를 개조해서 새로운 인터페이스를 구현하도록 하는 것은 간단하다! just add “implements 해당_인터페이스_이름” and add the methods from interface. 추상 클래스라면 클래스 계층구조를 생각해야 해서 힘들죠.
- mixin 정의하는데 인터페이스가 이상적이다. 다중 implementation이 가능하니까.
- 비계층적 자료형 프레임워크를 만들 수 있다. 계층구조에 영향을 주지 않으니까.
- 규칙 16에 나온 wrapper class idiom을 사용하면 강력한 기능개선이 가능하다.

####  interface와 abstract class의 장점을 합하자!
- abstract skeleton implementation 클래스를 두면, 기본 구현을 이 클래스에 맡겨 인터페이스와 추상클래스의 장점을 합칠 수 있다.
- simulated multiple inheritance 라는 개념도 있다. 다중상속의 단점은 피하면서 그 장점은 그대로 누릴 수 있다고 함. (실제 개발에 쓰이는 패턴일까?)
- simple implementation

#### 참고사항
- 원래 interface는 나중에 인터페이스에 메소드 추가하면, 이 인터페이스를 구현한 클래스들이 깨지게 되기 때문에 개선이 쉬운 API를 만들 때엔 적합하지 않았다.
- 하지만 자바 1.8 부터 default 메소드를 통해 이것이 가능해졌다!
- 즉 인터페이스가 짱짱맨 됐음!

## 19. Use interfaces only to define types
인터페이스를 잘못 사용하는 경우로 constant interface를 만드는 경우가 있다. 다음과 같이 인터페이스를 사용하는 경우이다.
``` java
public interface PhysicalConstants {
     static final double AVOGADROS_NUMBER = 6.02e23;
     static final double BOLTZMANN_CONSTANT = 1/38e-23;

     // 메소드 선언은 없음.
```

#### constant interface의 단점
* 클래스가 어떤 상수를 어떻게 사용하느냐는 것은 구현 세부사항인데, 상수 인터페이스를 사용하면 이것이 공개 API가 되어버린다.
* 나중에 저런 상수들을 사용하지 않게 되더라도 호환성 때문에 해당 인터페이스를 계속 구현해야 한다.

#### 상수를 표현하는 좋은 방법
* 상수가 이미 존재하는 클래스나 인터페이스에 강하게 결합하고 있는 경우엔 그 상수들을 해당 클래스나 인터페이스에 추가하는 것이다.
* 혹은 객체생성이 불가능한 유틸리티 클래스에 넣어 공개한다. - 보통 이런 유틸리티 클래스를 사용하면 클라이언트 상수 앞에 클래스 이름을 붙여야 하는데 JDK 1.5 부터 도입된 static import  기능을 이용하면 클래스 이름을 제거할 수 있다.

## 20. Prefer class hierarchies to tagged classes
#### 태그필드가 있는 클래스
```java

// Tagged class - vastly inferior to a class hierarchy!
  class Figure {
      enum Shape { RECTANGLE, CIRCLE };

      // Tag field - the shape of this figure
      final Shape shape;

      // These fields are used only if shape is RECTANGLE
      double length;
      double width;

      // This field is used only if shape is CIRCLE
      double radius;

      // Constructor for circle
      Figure(double radius) {
          shape = Shape.CIRCLE;
          this.radius = radius;
      }

      // Constructor for rectangle
      Figure(double length, double width) {
          shape = Shape.RECTANGLE;
          this.length = length;
          this.width = width;
      }

      double area() {
          switch(shape) {
            case RECTANGLE:
              return length * width;

            case CIRCLE:
              return Math.PI * (radius * radius);

            default:
              throw new AssertionError();
          }
      }

  }
```

#### 태그 필드가 있는 클래스의 단점
* enum declarations, tag fields, switch문 등의 상투적인 코드들이 잔뜩 들어가게 된다.
* 가독성이 떨어진다.
* 객체가 자기와 상관 없는 내용까지 들고 있어야 하므로 메모리 요구량이 많아진다.
* 생성자안에서 불필요한 필드를 초기화하지 않으면 필드를 final로 만들 수도 없다.
* 생성자 안에서 태그 필드를 반드시 초기화해야하며, 해당 태그에 따라 적절한 필드를 초기화해야 하는데 이는 컴파일러가 강제하지 않는부분이라 에러날 소지가 많다! (프로그래머는 항상 실수하지...)
* 또 다른 태그를 추가할라치면 무조건 소스코드를 수정해야겠지....
* 아무튼 엄청난 말썽꾸러기다!

#### 이럴때 상속을 사용해라!
* 위의 Figure를 예로 설명하자면, Figure를 abstract class로 빼고, 각 태그에 따라 다르게 구현되어야 하는부분을 맨 위의 abstract method로 뽑자.
* Figure를 상속하는 하위클래스 안에서 abstract method를 구현하고, 적절한 필드와 생성자를 구현하면 끝!

## 21. Use function objects to represent strategies
#### function object 란?
가지고 있는 메서드가 다른 객체에 작용하는 메서드 하나뿐인 객체
```java
 class StringLengthComparator {
      public int compare(String s1, String s2) {
           return s1.length() - s2.length();
       }
}
```

무상태 클래스이므로 싱글턴 패턴을 따르는 게 좋다.
```java
class StringLengthComparator {
       private StringLengthComparator() { }
       public static final StringLengthComparator
           INSTANCE = new StringLengthComparator();

       public int compare(String s1, String s2) {
           return s1.length() - s2.length();
       }
}
```

해당 function object를 전달하려면 인자의 자료형이 맞아야 하므로 유연성을 주기 위해 전략 인터페이스를 정의하고 이를 구현하도록 하는 것이 필요하다.
```
// Strategy interface
   public interface Comparator<T> {
       public int compare(T t1, T t2);
}
```

해당 클래스가 특정 클래스 안에서 쓰일 때엔 익명 클래스로 선언할 수도 있다. 하지만 메소드 안에 정의하면 메소드가 불릴 때마다 객체가 만들어지므로 private static final 멤버 클래스로 전략을 구현한 다음, 전략 인터페이스가 자료형인 public static final 필드를 통해 외부에 공개하는 것이 바람직하다.

## 22. Favor static member classes over non static //4가지 nested class 에 관한 규칙
nested class의 종류로는 static member class, non-static member class, anonymous class, local class가 있다. static member class를 제외한 세가지는 전부 inner class이다.

#### static memeber class
- 가장 간단한 nested class
- has access to all of the enclosing class’s members, even those declared private. (static members에만 해당되는 얘기겠지?)
- enclosing instance와 독립적이다.
- static이니까 enclosing instance에 대해서는 모를 것이고, 따라서 enclosing class의 non-static 필드나 메소드는 부를 수 없겠다.

#### non static memeber class
- enclosing instance와 내부적으로 연결되어 있다. 따라서 enclosing instance 없이는 존재할 수 없다.
- 이 클래스의 instance method 안에서 enclosing instance의 메소드를 부르거나 enclosing instance의 참조를 얻을 수 있다. (EnclosingClass.this 를 통해서)
- non static member class의 instance 와 enclosing  instance와의 연결은 non static member class instance가 만들어지는 순간에 확립되고, 이후 수정할 수 없다.
- 따라서, non static member class instance에는 이 연결을 위한 공간이 필요하며 이 때문에 객체 생성 시간이 늘어난다.

#### anonymous class
- 이름이 없다.
- enclosing class의 멤버도 아니다!
- 사용되는 시점에 동시에 declared, instantialted 된다!
- 표현식을 준수한다면 어디서든 사용 가능함.
- nonstatic context 에서 사용될 때만 enclosing instance 를 갖는다. 하지만 static context 에서 사용된다하여도 enclosing class의 static members에 접근할 수 없다.
- 여러 인터페이스를 구현할 수 없고, 인터페이스 구현과 클래스 상속을 동시에 할 수 없다.
- 10줄 이하로 작성하는 것을 권한다. 그렇지 않으면 코드 가독성이 떨어진다.
- 익명 클래스의 클라이언트는 익명클래스가 상위 타입으로부터 상속받은 멤버들만 호출할 수 있다. 다른 멤버들은 호출할 수 없다.

#### local class
- 가장 사용빈도가 낮음
- 지역변수가 선언될 수 있는 곳이라면 어디서든 선언 하 ㄹ수 있으며, 지역 변수와 동일한 scoping rule 을 따른다.
- 익명클래스처럼 non static 문맥 안에서 선언되었을 때만 enclosing instance 를 갖고, static members를 가질 순 없다.
- 익명클래스와 마찬가지로 짧게 작성되어야 가독성을 해치지 않는다.

####  용도 정리
- enclosing instance에 접근할 필요가 없는 멤버 클래스를 정의할 때엔 always =>  static member class로!
- private static member class는 enclosing instance의 component를 표현하는 데 흔히 쓰임! ex) Map 내부의 Entry 객체 - Entry 객체의 getKey, getValue, setValue 메서드는 Map객체에 접근할 필요가 없기 때문이다.
- 익명클래스는 함수객체를 정의할 때 많이 쓰임. 프로세스 객체를 만들 때도 널리 쓰임. - 중첩 클래스가 특정한 메소드에 속해야 하고, 오직 한곳에서만 객체를 생성하면서, 필요한 특징을 갖고 있는 타입이 이미 존재한다면 사용한다.
