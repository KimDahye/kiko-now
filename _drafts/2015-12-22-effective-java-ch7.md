---
layout: post
title: "Effective Java Ch 7 Methods"
tags: [java, effective java]
comments: true
---

## Item 38. Check parameters for validity
* 제한이 필요한 인자라면, 반드시 문서에 남겨야 하고 메서드 시작부분에서 검사해야 한다.
* public 메서드라면, 인자 유효성이 위반되었을 경우 발생하는 예외를 Javadoc의 @throws 태그를 사용하여 문서화한다.
```language-java
/**
* Returns a BigInteger whose value is (this mod m). This method
* differs from the remainder method in that it always returns a * non-negative BigInteger.
*
* @param m the modulus, which must be positive
* @return this mod m
* @throws ArithmeticException if m is less than or equal to 0 */
public BigInteger mod(BigInteger m) { 
	if (m.signum() <= 0)
        throw new ArithmeticException("Modulus <= 0: " + m);
 
    ... // Do the computation
}
```

* public이 아닌 메서드면, 인자 유효성을 검사할 때 assertion(확증문)을 이용한다. 확증문은 클라이언트가 패키지를 어떻게 이용하건, asserted condition(확증 조건)은 항상 **참이 되어야 한다고** 주장하는 것이다. 확증문은 확증조건이 만족되지 않으면 AssertionError를 낸다. 그런데 확증문이 활성화되지 않으면 활증문은 실행되지 않는다. (확증문을 활성화시키려면 java 인터프리터에 -ea 옵션을 줘야 함.)
```language-java
// Private helper function for a recursive sort
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset; 
	... // Do the computation
}
```
<br>

* 호출된 메서드에서 바로 이용하진 않지만 나중을 위해 보관되는 인자의 유효성을 검사하는 것은 특히 중요함. 그래서 생성자에서 인자의 유효성을 반드시 검사해야 한다.
* 하지만 이 원칙에도 예외는 있다. 유효성 검사를 실행하는 오버헤드가 너무 크거나 비현실적이면서 **유효성 검사가 메서드 내부에서 자연스럽게 이뤄지는 경우**!
* 계산과정에서 암묵적으로 유효성 검사가 이뤄질 때, 문서에 명시한 예외와 다른 예외가 던져질 수 있는데 이 때는 exception translation 숙어(Item 61)를 사용하여 문서에 명시된 예외로 변환해야 한다.
* 하지만 원래 메서드는 가능하면 일반적으로 적용될 수 있도록 설계해야 한다. 인자에 제약이 적을수록 좋음. 

## Item 39. Make defensive copies when needed
내가 만드는 클래스의 클라이언트가 invariant를 망가뜨리기 위해 최선을 다할 거란 가정 하에 방어적으로 프로그래밍해야 한다.
```language-java
 // Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;
	
	/**
	* @param start the beginning of the period
	* @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
	* @throws NullPointerException if start or end is null
	*/
	public Period(Date start, Date end) {
    	if (start.compareTo(end) > 0)
           throw new IllegalArgumentException(start + " after " + end);
		this.start = start;
		this.end   = end;
	}

	public Date start() {
	    return start;
	}

	public Date end() {
	    return end;
	}
    
    ...  // Remainder omitted
}
```
<br>
위에서 고쳐야 할 부분은 2곳이다. 첫번째로 생성자로 전달되는 변경 가능 객체를 반드시 복사해서 Period 객체의 컴포넌트로 이용해야 한다.
```language-java
// Repaired constructor - makes defensive copies of parameters
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
	if (this.start.compareTo(this.end) > 0)
		throw new IllegalArgumentException(start +" after "+ end);
	}
```
여기서 주의해야 할 점은 인자의 유효성을 검사하기 전에 (Item 38) 방어적 복사본을 만들었다는 것! 유효성 검사를 복사본에 대해서 시행하였다. 왜 이렇게 했을까? TICTOU(time-of-check/time-of-use)공격을 막기 위해.
<br>
또 고쳐야 할 부분은 바로 접근자 안! 변경 가능 내부 필드에 대한 방어적 복사본을 반환하도록 접근자를 수정해야 한다.
```language-java

// Repaired accessors - make defensive copies of internal fields
    public Date start() {
       return new Date(start.getTime());
	}

    public Date end() {
       return new Date(end.getTime());
	}

```
#### 방어적 복사본이 쓰일 때
* 위의 경우처럼 immutable class 만들 때
* 클라이언트가 제공한 객체를 내부 자료구조에 반영하는 생성자나 메서드 안에서 - ex) 클라이언트가 제공한 객체 참조를 Set 객체의 요소로 사용한다거나 Map 객체의 키로 사용한다던가..
* immutable class 가 아니더라도 변경 가능한 내부 컴포넌트에 대한 참조를 반환하기 전에는 대부분 방어적 복사를 하는 게 바람직 할 것이다. (Item 13)

#### 방어적 복사본 꼭 해야 할까?
* 객체의 컴포넌트로 immutable 객체를 사용했다면 방어적 복사가 필요 없다.
* 방어적 복사를 만드는 것은 사실 성능상 손해이다.
* 클라이언트가 객체 내부 상태를 변경하지 않는 게 확실하다면 방어적 복사본을 만들 필요 없다. 단, 클래스 문서에 메서드 호출자가 인자나 반환값을 변경하면 안 된다는 사실을 명시해야 한다. 

## Item 40. 메소드에 관한 소소한 tips! (Design method signatures carefully)
####  메서드 이름은 신중하게
* 최우선 목표는 이해하기 쉬우며, 같은 패키지 안에서 일관성 유지되는 이름일 것
* 두번째 목표는 좀 더 널리 합의된 사항에도 붛바하는 이름일 것 - 자바 라이브러리 API 이름 참고하기

#### convenience method 제공하는 데 너무 열 올리지 말자
* 클래스에 메서드가 너무 많으면 학습, 사용, 테스트, 유지보수 등 모든 측면에서 어려워진다.
* 인터페이스의 경우 메서드가 많으면 더 심각함 - 구현하는 사람은 다 구현해야지, 사용자는 어떤 메소드 있는지 외우기 어렵지...
* 내 생각엔, 이건 private 메소드에는 해당되지 않는 이야기인 듯. private method는 그 클래스 안에서만 쓰이는 단어같은 느낌이니까... (by sophie)

#### parameter(인자) list는 4개 이하로!
* 대부분의 프로그래머는 인자 리스트가 길어지면 기억 못함.
* 게다가 자료형이 같은 인자들이 길게 연결된 인자 리스트는 더 위험해지지 - 사용자가 순서를 바꿀 위험이 있지만 컴파일 시 오류나지 않으므로.

###### 인자 개수 줄이는 방법 3가지
* 여러 메서드로 나누자. 주의하지 않으면 너무 많은 메서드가 만들어질 수 있지만 제대로 만들면 오히려 줄일 수 있다고. orthogonoality 향상을 통해서 - 라는데 무슨 말인지 잘 모르겠음;;
* helper class 만들어서 인자를 그룹별로 나누는 것 ex) Dao를 통해 얻은 데이터를 옮길 대 DTO 만들어서 거기에 담아 옮기는 것! 
* 빌더 패턴을 고쳐서 객체 생성 대신 메서드 호출에 적용하는 것 (빌더패턴은 Item 2 참고). 이 방법은 많은 인자가 필요하나 상당수가 optional일 때 유용한 방법이다. 

#### 인자의 자료형으로는 클래스보다 interface가 낫다.
* 예를 들어 HashMap으로 쓰기보다는 Map으로 쓰는게 유연하겠죠!

#### 인자 자료형으로는 boolean을 쓰는 것보다는, 원소가 2개인 enum 자료형을 쓰는 것이 낫다.
* 가독성 때문에
* 확장성 때문에 (나중에 2개 이상의 의미를 갖는 자료형이 될 수 있으니!) 
* 각각 단위에 따른 로직은 enum 상수의 메서드에 리팩터링 해 넣을 수도 있음! (Item 30 참고)



## Item 41. Use overloading judiciously
참고. 이 부분 질문이 많으니 책과 함께 설명하며 질문도 하자. 
#### 오버로딩의 문제점
```language-java
// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
	}
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
	}
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
		};
        
        for (Collection<?> c : collections)
            System.out.println(classify(c));
	}
}
```
위의 코드는 Set, List, Unknown Collection을 순서대로 출력하지 않을까 기대하겠지만, Unknown Collection만 세 번 출력한다. 이유는 classfy 메서드가 오버로딩 되어 있으며, 오버로딩된 메서드 가운데 어떤 것이 호출될 지는 컴파일 시점에 결정되기 때문이다.

* overload 된 메서드는 정적으로 선택된다. - 컴파일 시점 자료형으로 어떤 메서드가 불릴 지 결정된다.
* override 된 메서든는 동적으로 선택된다. - 실행시점의 자료형으로 어떤 메서드가 불릴 지 결정된다. 

따라서 static 메서드임을 유지한 채 이 문제를 해결하는 가장 좋은 방법은 instanceof 연산자를 사용하는 것이다.

```language-java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "Set" :
    	   c instanceof List ? "List" : "Unknown Collection";
}
```
결론은 overriding이 일반적 규범이라면 overloading은 예외에 해당한다. 오버로딩을 사용할 때는 혼란스럽지 않게 사용할 수 있도록 주의해야 한다.

#### 오버로딩의 혼란을 피하는 방법
* 같은 수의 인자를 갖는 두개의 오버로딩 메서드를 API에 포함시키지 않는다.
* varargs 를 사용하는 경우엔 아예 오버로딩하지 않는 전략을 취해야 한다. (예외적인 경우는 Item 42를 참고)
* 생성자는 다른 이름의 메서드를 만들 수 없어 이 규칙이 버거울 수 있다. 이럴 땐 static factory 패턴을 이용하는 방법이 있을 수 있다. (Item 1)
* 예외 - 같은 수의 인자더라도, 형식 인자 가운데 적어도 하나가 확실히 다르다면, 오버로딩 할 수 있다. (확실히 다르다는 말은 형변환 할 수 없다는 뜻)
```language-java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<Integer>();
        List<Integer> list = new ArrayList<Integer>();
        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```
위의 경우가 오버로딩이 잘못된 예이다. 이전에는 오토박싱이 없어서 int와 Integer는 확실히 달랐지만 지금은 그렇지 않거든. (지금은 list.remove(i)는 지정된 인덱스에 있는 원소를 제거하는 구실을 한다!) 의도대로 동작하게 하려면 아래와 같이 코드를 수정해야 한다.
```language-java
for (int i = 0; i < 3; i++) {
    set.remove(i);
	list.remove((Integer) i); // or remove(Integer.valueOf(i)) 
}
```
<br>
#### 규칙을 어기고 싶다면
이미 있는 클래스를 개선하여 새로운 인터페이스를 도입했다면, 새로운 인터페이스에 대해 확장된 메서드로 오버로딩하고 싶을 것이다. 이때는 프로그래머가 어떤 메서드가 호출될 지 알 수 없어도, 똑같이 동작하게 하여 문제가 없도록 하면 된다. 표준적인 방법은 다음과 같이, 좀 더 구체적인 오버로딩 메서드가 좀 더 일반적인 오버로딩 메서드를 호출하도록 하는 것. 
```language-java
//여기서 contnetEquals는 인터페이스. StringBuffer는 이를 구현한 구현 클래스이다.
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```
<br>
## Item 42. Use varargs judiciously
#### 기본 개념
* 자바 1.5 부터 공식적으로 variable arity method(가변 인자 메서드)라 불리는 varargs 메서드가 추가되었다. 
* 이 메서드는 지정된 자료형의 인자 0개 이상 받을 수 있다. 
* 클라이언트에서 전달한 인자 수에 맞는 배열이 자동 생성되고, 모든 인자가 이 배열에 대입된 뒤, 해당 배열이 메서드에 인자로 전달된다.

#### varargs 사용 예
###### 0개 이상의 인자
```language-java
 // Simple use of varargs
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
	return sum;
}
```

######1개 이상의 인자
```language-java
// The right way to use varargs to pass one or more arguments
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
		return min; 
}
```

#### 마지막의 배열 인자, varargs로 바꿔야 하나?
반드시 그래야 하는 것은 아니다. Arrays.asList 를 예로 들어 설명하겠다.
```language-java
public class ArrayAsList {
    public static void main(String[] args) {
        Integer[] arr = new Integer[2];
        int [] arr2 = new int[2];
        for(int i = 0; i <arr.length; i++) {
            arr[i] = i;
            arr2[i] = i;
        }
        System.out.println(Arrays.asList(arr)); 
        System.out.println(Arrays.asList(arr2));
    }
}
```
여기서 `System.out.println(Arrays.asList(arr))`는 array를 출력하고 싶을 때 사용하는 숙어였다. 그런데 이 숙어의 문제점은 Arrays.asList가 인자로 배열을 받을 때엔 primitive type에 대해 컴파일 에러가 났었다는 것이다. varargs로 바꾼 뒤에 컴파일 에러는 나지 않는다. 하지만 결과가 의도대로 나오지 않는다. 
```
[0, 1]
[[I@15301ed8]
```
개선된(?) Arrays.asList 메서드는 객체 참조를 모아 배열로 만드는데, 그 결과로 int 배열 arr2에 대한 참조가 담긴 길이 1짜리 배열, 즉 배열의 배열이 만들어졌기 때문이다. // 근데 왜 primitive  type에 대해서 이렇게 동작하는지... 왜 배열로 인식 못하는 건지...?
<br>
**결론: varargs 는 정말로 임의 개수의 인자를 처리할 수 있는 메서드를 만들어야 할 때만 사용하자.**
<br>
참고로, 자바1.5부터 어떤 자료형의 배열이라도 문자열로 변환할 수 있도록 설계된 Arrays.toString 메서드가 나왔다.

#### 성능 이슈
* 성능이 중요한 환경이라면 varargs 사용에 더욱 신중해야 한다. - 아무래도 배열을 만들고 초기화하는 비용이 들기 때문
* 만약 95%정도는 메서드를 호출할 때 2개 이하의 인자가 전달된다면 아래의 코드처럼 오버로딩 메서드를 준비하는 게 좋다.
```language-java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
<br>
##Item 43. Return empty arrays or collections, not nulls
#### null을 반환할 때의 문제점
null을 반환하면 다음과 같은 코드가 나온다.
```language-java
private final List<Cheese> cheesesInStock = ...;
    /**
     * @return an array containing all of the cheeses in the shop,
     *     or null if no cheeses are available for purchase.
    */
    public Cheese[] getCheeses() {
       if (cheesesInStock.size() == 0)
           return null;
       ...
}
```
```language-java
Cheese[] cheeses = shop.getCheeses();
if (cheeses != null && Arrays.asList(cheeses).contains(Cheese.STILTON))
    System.out.println("Jolly good, just the thing.");
```
클라이언트 코드에 null에 대한 체크가 없을 수 있다면 

* 더 로직에 집중된 코드를 만들 수 있고, 
* null체크를 빼먹었을 때에 향후 생길 에러가 없겠지.

#### 어떻게 하나?
```language-java
// The right way to return an array from a collection
private final List<Cheese> cheesesInStock = ...;
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
   
/**
 * @return an array containing all of the cheeses in the shop.
 */
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

```language-java
// The right way to return a copy of a collection
public List<Cheese> getCheeseList() {
	if (cheesesInStock.isEmpty())
    	return Collections.emptyList(); // Always returns same list
    else
        return new ArrayList<Cheese>(cheesesInStock);
}
```
<br>
##Item 44. Write doc comments for all exposed API elements
자세한 건 javadoc 문법은 Oracle의 "How to Write Doc Comments" [웹 페이지](http://www.oracle.com/technetwork/articles/java/index-137868.html) 참고

#### 문서화 범위
* 좋은 API 문서를 만들려면 exported class, interface, constructor, method, field declaration 위에 문서화 주석을 달아야 한다.
* 유지보수가 쉬운 코드를 만들려면 unexported class, interface, constructor, method, field declaration 에도 달아야 한다.

#### 메서드를 위한 주석
* 메서드가 계승을 위한 메서드가 아니라면, 메서드가 무엇을 하는지를 설명해야함 (어떻게 그 일을 하는지 설명해서는 안 됨)
* 선행조건(클라이언트가 메서드를 호출하려면 반드시 참이어야 하는 조건)과 후행조건(메서드 실행이 성공적으로 끝난 다음에 만족되어야 하는 조건들)을 나열해야 함. 선행조건은 대부분 `@throws` 태그를 기술.
* 메서드의 side effect 에 대해서도 문서화 해야 함. 예를들어 background thread로 실행한다면 그런 사실이 문서화 되어야 함.
* 메서드의 Thread-Safety에 대해서도 남겨야 한다.
* 완벽하게 기술하려면 인자마다 `@param` 태그, 반환값에 `@return`태그, 모든 예외에 `@throws`태그도 달아야 한다. 

#### 유용한 태그들
* `{@code}` 태그 - 코드 서체, HTML 마크업이나 javdoc 태그가 위력을 발휘하지 못함
* `{@leteral}` 태그 - HTML 마크업이나 javdoc 태그가 위력을 발휘하지 못함

#### 이외의 규칙들
* 모든 문서화의 첫번째 문장 - 해당 주석에 담긴 내용을 요약한 것.
* 제네릭 자료형이나 메서드에 주석을 달 때는 모든 자료형 인자들을 설명해야 한다.
* enum자료형에 주석을 다 ㄹ때는 상수 각각에도 주석을 달아 주어야 한다.
* 어노테이션 자료형에 주석을 달 때는 모든 멤버에도 주석을 달아야 한다. 
* 클래스가 스레드에 안전하건 말건, 그 안정성 수준을 문서로 남겨야 한다. (Item 70)
* 직렬화 가능한 클래스라면 그 직려로하 형식도 문서에 남겨야 한다.(Item 75)
* javadoc 에는 메서드 주석을 "상속"하는 기능도 있다.
* 문서화 주석에 있을지 모르는 오류를 줄이는 간단한 방법은 javadoc 결과 만들어진 HTML 파일들을 validity checker로 검사해보는 것. (W3C-validator 참고)
* Oracle의 "How to Write Doc Comments" 웹 페이지에 나온 규칙들이 지켜지는지 검사하는 IDE 플러그인도 있다. 

