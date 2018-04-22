---
layout: post
title: "Effective Java (2nd Edition) Ch3 정리"
tags: [java]
comments: true
---

## 8. Obey the general contrac when override `equals`
#### 재정의 하지 않는 게 제일 좋음
아래 조건 중 하나라도 만족하면, equals메소드를 재정의 하지 않아도 된다. (클래스에 논리적 동일성 검사방법이 있는 것은 상관 없다.)

1. 각각의 객체가 고유하다면.
3. 상위 클래스에서 재정의한 equals가 하위 클래스에서 사용하기에도 적당하다면.
4. 클래스가 private 또는 package-private로 선언되었고, equals 메서드를 호출할 일이 없다면. - 근데 저자는 이럴 경우엔 반드시 다음과 같이 equals를 재정의하여 실수를 찾을 수 있도록 해야 한다고 생각하신다.
```
@Override public boolean equals ( Object o) {
     throw new AssertionError(); //Method is never called
}
```
대체로 value class들이 equals 재정의하는데 적합한 클래스들이다.

#### 해야한다면 equivalence relation을 만족해야
재정의 해야할 땐 다음의 equivalence relation을 만족해야 한다.

- **Reflective** if x is non-null reference value, `x.equals(x)` must return true. <br>(안지키기가 어려운 조건…)
- **Symmetric** For any non-null reference values x and y, `x.equals(y)` must return true if and only if `y.equals(x)` returns true. <br> (이거는 잘 일어날 수 있음. A, B 클래스의 각 equals 메소드 안에서 A 는 B까지 고려했는데, B는 A를 고려하지 않으면 symmetricity를 위반하게 된다.)
- **Transitive** For any non-null reference values x, y, z, if `x.equals(y)` returns true and `y.equals(z)` returns true, then `x.equals(z)` must return true. <br> (이것도 잘 일어날 수 있음. value를 추가하여 concrete class를 상속하는 경우. symmetricity를 맞추기 위해 super type일 때 super 에 해당하는 field로만 비교하게 되면... sub - super - sub' 순서는 다 같게 나오지만, sub - sub'은 같지 않는 경우 발생)
- **Consistent** For any non-null reference values x and y, multiple invocations of `x.equals(y)` consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified. <br>(예시로 URL 나옴. 네트워크 타고 IP주소 얻어와서 비교했다고… 호환성 때문에 고칠 수도 없다고 함.)
- For any non-null reference value x, `x.equals(null)` must return false. <br>(casting위해 instanceof 테스트 거치면 자동으로 해결되는 문제)

#### 참고사항
* instantiable 클래스를 상속하여 새로운 값 컴포넌트를 추가하면서 equals규약을 어기지 않을 방법은 없다!
* equlas 메서드를 구현할 때 instanceof 대신 getClass 메서드를 사용하면 기존 클래스를 확장하여 새로운 값 컴포넌트를 추가하더라도 equals  규약을 준수할 수 있다는 소문이 돈다... 하지만? liskov substitution principle 을 위반하게 되지..
* liskov substitution principle 이란 어떤 자료형의 중요한 속성은 하위 자료형에도 그대로 유지되어서, 그 자료형을 위한 메서드는 하위 자료형에도 잘 동작해야 한다는 원칙이다.
* super에서 getClass써서 equals 메소드 override 해버리면, 하위 자료형에서 equals쓰는 속성들 다 깨져버리게 된다.
* ex) timestamp가 Date를 상속했는데, 이는 equals의 general contract가 깨진 상태다. 따라서 이 둘을 collection에 섞어 쓰면 문제가 생김!
* 이에 대한 회피책은? **상속하는 대신 composition**

#### 정리된 규칙
(여기서 this는 java의 this keyword)

1. Use `==` operator to check if the argument is a reference to this object.
2. Use the `instanceof` operator to check if the argument has the correct type.
3. Cast the argument to the correct type.
4. For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object. (primitive type은 대개 `==` 으로 비교. float, double만 Float.compare, Double.compare method 사용. Reference type은 equals() 재귀적으로 불러서 비교.
5. 필드 비교 순서가 성능에 큰 영향을 미치므로, 가장 다를 법하거나 비교하는데 비용이 적은 것부터 비교해야한다. 때로는 중요 필드로부터 계산되는 redundant field로 비교하는 게 좋을 때도 있다. (ex. 삼각형의 넓이)
6. equals method를 재정의하였다면, symmetricity, transitivity, consistency 는 반드시 unit test를 통해 확인한다!
7. Always override hashCode when you override equals.
8. Don’t try to be too clever. (ㅋㅋ ㅇㅋ. 일반적으로 별명까지 비교하는 건 나쁜 습관이야.)
9. Don’t substitute another type for `Obeject` in the `equals` declaration. <br> (이러면 override가 아닌 overload인 거니껜~!)

## 9. Always override `hashCode` when you override `equals`
#### hashCode에 대한 general contracts
* 실행 중에 equals 비교에서 쓰였던 field값이 달라지지 않았다면 hashCode 값도 달라지지 않아야 한다.
* if `x.equals(y) == true`, then `x.hashCode() == y.hashCode()`
* if `x.equals(y) != true`, then it is highly recommended that `x.hashCode() != y.hashCode()`.
특히나 두번째 조건은 반드시 지켜야 하는 것이고 - 안지키면, HashSet, HashMap 정상작동 안함.
세번째 조건을 무시하면 O(n) 이 O(n^2)가 됨.

#### a good approximation of  hash function
ideal hash function은 input values를 예상할 수 없으므로 만들기 불가능하지만, 어느정도 쓸만하게 approximate할 수 있다.
47쪽 ~ 48쪽의 방법이 바로 good approximation

#### hash cod caching
때때로 immutable이고, 해시코드 계산 비용이 높을 때 객체 안에 해쉬코드를 캐싱해둘 수도 있다. 아래와 같이!
```
//Lazily initialized, cached hashCode
private volatile int hashCode; // 여기서 volatile 은 어떤 의미일까?

@Override
public int hashCode(){
     int result = hashCode;
     if (result == 0) {
          // 계산
     }
     return result;
}
```

## 10. Always override `toString`
#### general contract
* “the returned string should be a concise but informative representation that is easy for a person to read"
* “It is recommended that all subclasses override this method."

#### 주의 사항
* When practical, the `toString` method should return all of the interesting information contained in the object.
* 그리고 document에 `toString` 리턴 값 format을 설명할 지 말지 잘 결정해야 함. 설명해두면 다시는 바꾸지 못할 것을 염두해두기! format을 쓰던 안 쓰던 toString의 의도를 document에 잘 설명해둬야 한다.
* `toString`에 쓰인 information은 programmatic access를 줘야 한다. 안그러면 `toString`이 그 information의 실질적 API가 되버림...

## 11. clone 재정의 할 때는 신중하게. (웬만하면 안하는 게 더 좋음)
clone()을 통한 객체 생성은 생성자를 통하지 않고 객체를 만드는 것이어서 extralinguistic! (즉, 어떻게 동작할는지 예측하기가 어려움.)

#### general contract
대체적으로 너무 약한 규약들이다.

* `x.clone() != x` // will be true, but it is not absolute requirement (한글 책 번역 잘못되어 있는 건가?)
* `x.clone().getClass() == x.getClass()` //will be true, but it is not absolute requirement
* x.clone().equals(x) // will be true, but it is not absolute requirement
Copying an object will typically entail creating a new instance of its class, but it may require copying of internal data structures as well.
* No constructors are called.  //  이건 또 너무 센 규약이야!

#### clone 메소드의 예상되는 동작, 이를 위한 전제 조건
프로그래머들은 subclass에서 super.clone()을 하면 subclass의 object일 거라고 예상한다.<br>
따라서, if you override the `clone` method in a nonfatal class, you should return an object obtained by invoking super.clone().<br>그래서 super.clone이 결국 Object의 `clone` 메소드를 부를 수 있도록 해야 함. (근데 이게 언어가 강제하는 부분도 아니라고 함!)

#### Cloneable 인터페이스를 구현하려면
document에서 강제하고 있진 않지만, Cloneable 인터페이스를 구현할 거라면, 잘 동작하는 `public clone()` 메소드를 만들어줘야 한다.
(여기서 가정은 Cloneable 인터페이스를 구현하려는 이 Class의 super class들이 well-behaved clone 메소드를 갖고 있어야 한다.)

* If every field contains a primitive value or a reference to an immutable object, the returned object of `(해당 클래스로 타입캐스팅) super.clone()` may be exactly what you need. (근데 이것도 예외는 있다. serial number나 unique ID 같은 경우엔 primitive나 immutable이어도 값을 바꿔줘야겠지.)
* 그렇다면 reference to mutable objects를 갖고 있을 때엔? clone()을 재귀적으로 부르던지(Stack 예제 참고), 필요시엔 직접 deep copy를 해줘야 한다 (HashTable 예제 참고). 그런데 여기서 나오는 포인트는, 그럼 clone을 재정의하려면 final fields referring to mutable objects는 없어야 한다.

#### 주의할 점
* 생성자에서처럼 clone 메소드 안에서 override 가능한 메소드를 부르면 복제가 완성되기 전에 overriden method가 불릴 수 있어 비정상 작동을 하게 된다.
* Public clone methods should omit throwing CloneNotSupportedException. (catch로 잡아줘야 함. Why? 그래야 쓰기 편하니까 그나마.)
* Thread safe, Cloneable 클래스를 만들려면, clone 함수가 동기화되어야 한다.

#### Cloneable 을 구현할 때의 단점!
You should...

* rely on a risk-prone extralinguistic object creation mechanism
* demand unenforceable adherence to thinly documented conventions
* conflict with the proper use of final fields
* throw unnecessary checked exceptions
* require cast

#### 그래서 결론은
* copy constructor, copy factory 를 사용해라. Cloneable은 쓰지 말아라.

```java
public Yum(Yum yum); //copy constuctor
public static Yum newInstance(Yum yum); //copy factory
```

## 12. Consider implementing `Comparable`
CompareTo는 Object에 있는 메소드는 아니지만, equals 메소드와 비슷한 맥락도 있고, 구현해두면 sorting algorithm, searching algorithm 등을 쓸 수 있으니 유용하다. natural ordering이 있다면 구현하는 걸 고려하자!

#### gerneral contract of compareTo
Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws ClassCastException if the specified object’s type prevents it form being compared to this object.

* Symmetric: `sgn(x.compareTo(y) == -sgn(y.comapreTo(x))` for all x,y
* Transitive: `x.compareTo(y) > 0 && y.compareTo(z) > 0` implies `x.compareTo(z) > 0`.
* `x.compareTo(y) == 0` implies that `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`, for all z.
* **Strongly recommended**. `(x.compareTo(y) == 0) == (x.equals(y))` // the equality test imposed by the compareTo method should generally return the same results as the equals method: consistency with equals

cf) equals메소드와 다르게 compareTo 메소드는 클래스가 다른 object와는 비교하지 않아도 된다. 특히, Because the Comparable interface is parameterized, the compareTo method는 statically typed, so you don’t need to type check or cast its argument.

cf) 앞의 세개 꼭 지켜야 할 조건들을 보면 equals의 조건들과 비슷하다는 것을 알 수 있다. 따라서 compareTo 메소드도 value를 추가하면서 concrete class를 상속하는 경우엔 제대로 구현할 수 없다. 우회책은 equals와 마찬가지로 상속이 아닌 조합을 쓰는 것.

cf) inconsistent with equals 인 BigDecimal은, BigDecimal(“1.0”), BigDecimal(“1.00”)을 HashSet에 넣을 때는 두개 들어가지만 TreeSet에는 하나만 들어가는 경우가 생긴다.

#### 필드 compare 방법
* compareTo 재귀적으로 호출하거나, 직접 구현, 있는 Comparator 이용
* Integral primitive fields일 경우엔 > < 연산자 사용. Float, Double인 경우엔 Float.compare, Double.compare 사용

## 스터디 중 나온 내용
int to Integer 오토박싱관련해서

```java
/*스터디 예제*/
final int a = 127;
final int b = 127;

Integer aa = a; // 얘는 compiler가 캐싱해둔 boxed Integer 줌.
Integer bb = b; // new Integer(b) 얘는 존중해줌.

if(aa==bb){
    System.out.println("tr" + aa.hashCode() + ","+ bb.hashCode()); // 출력됨
    }else {
    System.out.println("fa" + aa.hashCode() + ","+ bb.hashCode() ); //128부터는 여기가 출력됨
    }
```

이와 관련한 자료 - [int 와 Integer, AutoBoxing 과 AutoUnboxing - Vol.2](http://psvm.tistory.com/m/post/82)<br>
이와 비슷한 자료 - [tricky ternary operator in java autoboxing](http://stackoverflow.com/questions/8098953/tricky-ternary-operator-in-java-autoboxing)
