---
layout: post
title: "ThreadPoolExecutor"
tags: [java, effective java]
comments: true
---


generic은 프로그램을 안전하고 깔끔하게 만드는 반면 복잡성을 증가시킨다. 이번 챕터에서는 generic의 장점을 극대화시키면서, 단점은 최소화시키는 방법에 대해 알아본다!

## Item 23. Don’t use raw types in new code
#### Definition
* **generic**: A class or interface whose declaration has one or more *type parameters* is a *generic* class or interface. ex)`List<E>`
* **parameterized type**: the class or interface name followed by an angle-bracketed list of *actual type parameters* corresponding to the generic type’s formal type parameters. ex) `List<String>`, `List<Integer>`
  * 여기서 actual type parameter가 String, Integer같은 게 되는 거고, generic typs’s formal type은 E가 되는 거다.
* **raw type**: the name of the generic type used without any accompaning actual type parameters. ex) `List<E>`의 raw type은 List

#### raw type 의 단점
* 하나의 type만 들어가야 하는 collection에 다른 타입이 들어갔을 때, 이를 컴파일 타임에 찾을 수 없다. - safety 해침
* 또 해당 타입으로 casting하는 것을 일일히 해야 한다. (generic 을 쓰면 이를 컴파일이 자동적으로 해줌) - 코드를 지저분하게 만듦

#### 그런데 raw type은 왜 유지하는 걸까?
* for compatibility (호환성)

#### 이런 건 써도 괜찮다. (raw type의 대체)
* `List<Object>` : arbitrary object를 다 넣고 싶을 때 이걸 사용해라.
  *  `List`는 타입체킹을 아예 하지 않는 반면에, `List<Object>`는 컴파일러에게 아무 타입의 객체든 다 넣을 수 있다고 말하는 것이다.
  * **[주의]** `List<String>`은 `List`의 subtype이지만, `List<Object>`의 subtype은 아니다!
* `List<?>` - unbounded wildcard types: 원소의 타입이 알려지지 않았거나 타입이 별로 상관 없을 때 사용할 수 있다.
  * unbounded wildcard type이 safe, raw type은 그렇지 않다.
  * null 빼고는 `Collection<?>`에 아무것도 넣을 수 없다.
  * `Collection<?>`에서 빼낸 것에 대해 어떤 타입 유추도 할 수 없다.
  * 이런 성질 때문에 unbounded wildcard type을 쓰기 어렵다면, bounded wildcard type이나 generic method를 이용해라!

#### 예외 사항 (raw type을 쓸 수 있을 때)
* class literals - ex) `List.class`, `String[].class`, `int.class` are legal, but `List<String>.class`, `List<?>.class` are not.
* unbounded wildcard type에 `instanceof` operator를 쓰려 할 때
  - generic type은 런타임시 정보가 지워지기 때문에, unbounded wildcard type 을 제외하고 `instanceof` 연산자를 쓰는 게 허용되지 않는다. (그렇군..!)
  - 그런데 unbounded wildcard type에 `instanceof` 연산자를 사용하는 것과 raw type에 연산자를 사용하는 것이 아무런 차이가 없다. 따라서 이때는 `<?>`을 쓰는 게 코드만 지저분하게 만든다.

#### 질문
```language-java
if (o instanceof Set) {
     Set<?> m = (Set<?>) o; 
     ...
}
```

실제로 테스트 해본 코드
```language-java
public void test(Set<E> set) {
    if(set instanceof Set) {
        Set<?> m = (Set<?>) set; //직접 해보니 redundant casting 라고 뜬다.
    }
}
```

##Item 24: Eliminate unchecked warnings
- 제너릭을 사용해서 프로그램 할 때에 다양한 컴파일 경고를 보게 된다: unchecked cast warining, uncheced method invocation warning, unchecked generic array creation warning, and unchecked conversion warnings.
- 잡을 수 있는 거면 다 잡아서 unchecked warning을 없애라! 그래야 runtime시 ClassCastException이 나지 않는 typesafe한 코드를 짤 수 있다.
- 만약 warning을 없앨 수 없지만 확실히 typsafe 한 경우에만 `@SuppressWarnings(“unchecked”)` 를 붙여라. 붙일 때에도 최소한의 scope로 붙여야 중요한 warning을 잡을 수 있다. typesafe하다고 여겨지는 것들은 어노테이션을 붙여놔야, 진짜 중요한 경고들을 typesafe한 경고들 사이에서 찾아낼 수 있다.

##Item 25: Prefer lists to arrays
#### generic type과 array의 차이점
- covariant vs. invariant
  - array는 covariant - 즉 if Sub is subtype of Super, then the array type Sub[] is a subtype of Super[].
  - 이와 대조적으로, generic type은 `List<A>` 가 `List<B>`의 subtype이나 supertype이 될 수 없다.
  - 이 차이로 인해 generic type이 더 typesafe 한 성질이 있다. (컴파일 때 에러를 찾을 수 있으니) 아래 예제를 보자.
``` java
//Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = “I don’t fit in”; //Throws ArrayStoreException

//Won’t compile!
List<Object> ol = new ArrayList<Long>(); //Incompatible types
ol.add(“I don’t fit in”);
```

- reified vs. erasure
  - array는 reifed. 이게 무슨 뜻이냐면, 런타임시에 원소의 타입을 결정하고 그 이후 런타임시에도 타입이 결정된 원소들의 타입을 알고 있다.
  - generic type은 erasure. 이게 무슨 뜻이냐면, 컴파일 시에 원소의 타입을 결정하고 런타임 시엔 타입 정보가 삭제된다는 뜻이다.

#### 이런 차이점 때문에 둘이 상극이다!
- generic type의 배열을 생성하는 것을 illegal. ex) `List<E>[]`, `List<String>[]`, `E[]` 요런 타입은 나올 수 없다.
- 왜 illegal인 걸까? 같이 쓰다보면 type safe  하지 않은 순간들이 찾아온다고...! generic type을 쓴다는 것은 컴파일 때 에러 없으면 runtime에 ClassCastException 없는 걸 보장하기 위함인데 array랑 같이 쓰면 그게 깨진다. 아래의 예제를 보자.

```language-java
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1]; //이게 만약 컴파일이 된다면!!!
List<Integer> intList = Arrays.asList(42);
Object[] objects = stringLists; // 배열이 covariant니까 넘어간다.
objects[0] = intList; //generic은 erasure라 runtime시에 List, List[] 이렇게만 되기 때문에 넘어간다.
String s = stringLists[0].get(0); //여기서 런타임시에 에러나겠지~!
```

#### generic 배열 생성 오류에 대한 가장 좋은 해결책은 `E[]`대신 `List<E>`를 쓰는 것!
- 성능이 저하되거나 코드가 길어질 수는 있지만,
- type safe, interoperability 증가

## Item26. Favor generic types (가능하면 제네릭 자료형으로 만들자)
제네릭 자료형을 만드는 것은 쉽진 않지만, 알아두면 아주 유용하다.
#### 제네릭 자료형 만들기 (Stack을 예제로)
- step1. add one or more type parameters to its declaration
- step2. replace all the uses of the type `Object` with the appropriate type parameter.
- step3. 그럼 일반적으로 몇가지 에러나 경고를 맞이하게 될 것이다. 특히 non-reifiable type으로 배열을 만들려고 할 때!
  - 생성자의 `elements = new E[DEFAULT_INITIAL_CAPACITY];` 를 `elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];` 로 바꾼다. unchecked 경고를 보이겠지만, 이 배열이 private field에 있고, push 할 때 E 타입만 받기 때문에 suppress warning해도 괜찮다.
  - field를 `E[]`타입에서 `Object[]`로 바꾸고, 배열에서 원소를 뽑을 때마다 `(E)`로 캐스팅해주는 방법이 있다.
  - 첫번째 방법이 한번만 해주면 되어서 더욱 선호된다.

#### 참고
- 배열보다 리스트를 쓰라고 했던 Item 25와 대치되는 이야기 아닌가? 라고 생각할 수 있지만,
- 리스트를 언제나 사용할 수 있는 것도 아니구, 성능상 배열을 쓰는 게 좋을 때도 있다. (직접 자료 구조를 만들 때엔 내부에 배열을 사용하는 것 같네. by 다혜)

## Item 27. Favor generic methods (가능하면 제너릭 메서드로 만들자)
클래스 generification 과 마찬가지로, method 를 generificate 하면 장점이 많다. static utility method들은 제너릭화 하기에 좋은 녀석들.
마찬가지로 method declaration에 type parameter를 추가하고
메소드 내용을 type parameter 사용하도록 고치면 된다.

#### 유용히 쓰이는 예
- generic static factory method ( HashMap<K, V>의 생성자 대용으로) - 이 부분은 Java 1.7부터는 제너릭 자료형의 생성자에도 type inference가 지원된다.
- generic singleton factory pattern
```language-java
public class Main {
    private static UnaryFunction<Object> IDENTITY_FUNCTION = new UnaryFunction<Object>() {
        @Override
        public Object apply(Object arg) {
            return arg;
        }
    };

    public static <T> UnaryFunction<T> identityFunction() {
        return (UnaryFunction<T>) IDENTITY_FUNCTION;
    }

    public static void main (String[] args) {
        String[] strings = {"1", "b", "c"};
        UnaryFunction<String> sameString = identityFunction();
        for(String s : strings) {
            System.out.println(sameString.apply(s));
        }
    }

    private interface UnaryFunction<T> {
        T apply (T arg);
    }
}
```

- recursive type bound
```language-java
public interface Compareable<T> {
    int compareTo(T o);
}

// Using a recursive type bound to express mutual comparability
public static <T extends Comparable<T>> T max(List<T> list) {
    Iterator<T> i = list.iterator();
    T result = i.next();
    while(i.hasNext()) {
        T t = i.next();
        if(t.compareTo(result) > 0) result = t;
    }
}
```

## Item 28. Use bounded wildcards to increase API flexibility.
generic type은 invariant이기 때문에 확장성을 높이려면 wildcards 를 잘 살용해야 한다. 가장 기초가 되는 룰은 PECS(producer-extends, consumer-super) - 자세한 건 아래에 설명하겠다.  comparables, comparators 는 언제나 consumer 라는 걸 기억하자.

#### Stack에서 확장성 높이기
```language-java
public class Stack<E> {
      public Stack();
      public void push(E e);
      public E pop();
      public boolean isEmpty();
}
```

```language-java
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src) push(e);
}
```

```language-java
Stack<Number> numberStack = new Stack<Number>();
   Iterable<Integer> integers = ... ;
   numberStack.pushAll(integers); //위의 pushAll코드는 컴파일은 잘되지만 여기에서 에러를 뱉는다.
```


이렇게 고쳐야 확장성을 고려한 pushAll 함수가 된다.
```language-java
// Wildcard type for parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

popAll도 마찬가지. 위는 확장성이 없는 코드이고, 아래는 확장성이 있는 코드이다.
```language-java
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
       while (!isEmpty()) dst.add(pop());
}

// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

Stack을 통해 알 수 있는 규칙이 바로 - PECS(producer-extends, consumer-super)

#### PECS 를 적용한 예시
##### reduce
```language-java
static <E> E reduce(List<E> list, Function<E> f, E initVal)

 // Wildcard type for parameter that serves as an E producer
static <E> E reduce(List<? extends E> list, Function<E> f, E initVal)

```
list는 producer로 쓰이므로 extends를 붙이고, Function은 producer와 consumer로 동시에 쓰이기 때문에 그대로 둔다.

##### union
```language-java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) //여기서
public static <E> Set<E> union(Set<? extends E> s1,  Set<? extends E> s2) //이렇게 바꾼다.
```
주의할 것은 return type에는 wildcard를 쓰지 않는다는 것이다. 좀 더 유연한 코드를 만들 수 있도록 하지 않고 크라이언트 코드 안에도 와일드 카드 자료형을 명시해야 하기 때문이다. 클라이언트가 와일드카드 타입에 대해 고민하게 된다면, 그건 API가 잘못 설계된 거라고 할 수 있다.

```language-java
Set<Integer> integers = ... ;
Set<Double> doubles = ... ;
Set<Number> numbers = union(integers, doubles); //에러난다.

Set<Number> numbers = Union.<Number>union(integers, doubles); //이렇게 명시적으로 type parameter를 알려주어야 컴파일 에러가 나지 않는다.
```
##### max
```language-java
public static <T extends Comparable<T>> T max(List<T> list)
public static <T extends Comparable<? super T>> T max( List<? extends T> list) // Comparable of T는 언제나 T instance를 consume하므로 super

```

```language-java
// Won’t compile - wildcards can require change in method body!
public static <T extends Comparable<? super T>> T max( List<? extends T> list) {
    Iterator<T> i = list.iterator();  // 여기가 에러나므로 Iterator<? extends T> i = list.iterator(); 이렇게 고쳐야 한다.
    T result = i.next();
    while (i.hasNext()) {
      T t = i.next();
      if (t.compareTo(result) > 0)
      result = t;
    }
    return result;
}
```

##### swap
```language-java
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
어떻게 선언하든 기능상의 차이는 없지만(둘다 any list를 받을 수 있다),  두번째꺼를 사용한다고 한다. 왜냐면 두번째 것이 더 간단하기 때문에.
type parameter가 단 한번 나타난다면 wildcard로 대체하자!

```language-java
public static void swap(List<?> list, int i, int j) {
       list.set(i, list.set(j, list.get(i)));
}
```

근데 위와 같이 구현하면 컴파일 에러가 난다. List<?>에는 null만 넣을 수 있기 때문이다. 이럴 때에 helper method를 이용한다. 다음과 같이~!
```language-java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

## Item 29. Consider typesafe heterogeneous containers
제너릭 일반용법에 따르면, 컨테이너별로 형인자 개수는 고정되어 있다. 하지만 컨테이너 대신 키를 제네릭으로 만들면 그런 제약이 없는 typesafe heterogeneous container를 만들 수 있다. 그런 컨테이너는 Class 객체를 키로 쓰는데, 그런 Class 객체를 type token 이라고 부른다.
(컴파일 시간 자료형이나 실행시간 자료형 정보를 메스드들에 전달할 목적으로 class literal을 사용하는 경우, 그런 class literal을 type token 이라 한다.)

#### typesafe heterogeneous container 의 예시 - Favorite 클래스
```language-java
// Typesafe heterogeneous container pattern - implementation
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();
  public <T> void putFavorite(Class<T> type, T instance) {
    if (type == null)
      throw new NullPointerException("Type is null");
    favorites.put(type, instance);
  }

  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type)); //dynamic cast (동적 형변환)
  }
}

```
unbounded wildcard를 사용했으니, 이 맵에 아무것도 넣을 수 있는 게 아닌가? 사실은 그 반대다! 와일드카드 자료형이 중첩되어 있다는 것에 주의하자. 와일드 카드 자료형이 쓰인 곳은 맵이 아니라 키다. 모든 키가 different parmeterized type을 가질 수 있다는 의미이다. 즉 Class<String>인 키도 있고, Class<Integer>인 키도 있을 수 있다.

#### Favorite 클래스의 두가지 한계
##### 1. 악의적인 사용자가 Favorite 객체의 형안전성을 깨트릴 수 있다.
다음과 같이 putFavorite을 고치면 넣을 때에 key와 value의 type을 맞출 수 있다.
```language-java
// Achieving runtime type safety with a dynamic cast
public <T> void putFavorite(Class<T> type, T instance) {
       favorites.put(type, type.cast(instance)); //instance가 type 클래스의 객체인지 여기서 확인된다.
}
```

##### 2. non-refiable type 은 저장할 수 없다.
- 즉, String이나 String[]은 저장할 수 있지만, `List<String>`은 저장할 수 없다! 이유는 `List<String>`의 Class 객체를 얻을 수 없기 때문. 다시 말해, `List<String>.class`는 문법적으로 옳지 않은 표현이다.
- 이것은 한편으로 다행스러운 일이기도 한데, `List<String>`이나 `List<Integer>`나 List.class를 공유하기 때문에 만일 non-refiabe type을 저장할 수 있었더라면 Favorites객체는 올바르게 작동하지 않을 것이다.
- 이걸 피할 수 있는 완벽한 방법은 없다. (super type token이라는 기법이 만들어지긴 했는데 여기도 문제는 있다.)

#### annotations API (...이 부분은 아직 잘 이해 안된다)
- 어노테이션 API는 bounded type token을 무지하게 이용하는 예시다.
- annotated element는 키로 annotation type을 갖는 typesafe heterogeneous container이다.
- `subClass` 라는 instance method를 이용하여 클래스 객체를 형변환을 할 수 있다.

```language-java
// Use of asSubclass to safely cast to a bounded type token
   static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null; // Unbounded type token
    try {
      annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
      throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(
      annotationType.asSubclass(Annotation.class));
}
```
