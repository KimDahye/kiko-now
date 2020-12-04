---
layout: post
title: "Effective Java ch 6 Enums and Annotations"
tags: [java, effective java]
comments: true
---

## Item 30. Use enums instead of int constants
#### int enum pattern?
```language-java
// The int enum pattern - severely deficient!
public static final int APPLE_FUJI =0; 
public static final int APPLE_PIPPIN =1; 
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```
단점이 이래저래 많은 녀석이다.

* 형 안전성 관점에서 보면 Orange를 기대하는 메소드에 Apple관련 함수를 넘겨도 컴파일러에서 잡을 수 없다.
* 편의성 관점에서 봐도 별도의 이름공간이 없기 때문에  APPLE_ 과 ORANGE_ 같은 접두어가 필요하다. 
* 또한 인쇄가능한 문자열로 변환할 수 없고, 순차적으로 순회하거나 그룹의 크기를 알아낼 좋은 방법도 없다.

참고. String enum pattern은 더 나쁜 패턴. 문자열 비교를 하므로 성능 이슈가 있다.

#### int enum 패턴과 String enum 패턴의 대안. enum type
```language-java
public enum Apple  { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
###### 특징
* 자바의 enum 자료형은 완전한 기능을 갖춘 클래스다. 다른 언어의 enum보다 강력함. (다른 언어의 enum은 결국 int 값)
* 아이디어. enumeration constant 별로 하나의 객체를 public static final 필드 형태로 제공한다.
* 자료형의 개체수가 엄격히 통제됨. enum 자료형은 싱글턴 패턴을 일반화한 것으로, 싱글턴 패턴은 본질적으로 보면 열거 상수가 하나뿐인 enum 과 같다. 

###### 장점
* compile-time type safety: ex) Apple 형의 인자를 받는다고 선언한 메서드는 반드시 Appe 값 세 개 가운데 하나만 인자로 받는다.
* eunum 자료형은 namespace가 있기 때문에 같은 이름의 상수가 공존할 수 있도록 한다. 
* enum 자료형은 toString 메서드를 호출하면 인쇄 가능 문자열로 쉽게 변환할 수 있다.
* enum 자료형은 임의의 메서드나 필드도 추가할 수 있으며 임의의 interface 를 구현할 수도 있다.

#### 풍부한 기능을 갖춘 enum의 예시 - Planet
```language-java
// Enum type with data and behavior
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

	private final double mass;           // In kilograms
    private final double radius;         // In meters
    private final double surfaceGravity; // In m / s^2
       
    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;
       
    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
	}
       
    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
       
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
	} 
}
```
* enum 상수에 데이터를 넣으려면 instance field를 선언하고 생성자를 통해 받은 데이터를 그 필드에 저장하면 된다.
* 주의할 것은, enum은 원래 변경 불가능하므로 모든 필드는 final로 선언되어야 한다. 

```language-java
public class WeightTable {
    public static void main(String[] args) {
		double earthWeight = Double.parseDouble(args[0]);
		double mass = earthWeight / Planet.EARTH.surfaceGravity(); 

		for (Planet p : Planet.values()) 	
			System.out.printf("Weight on %s is %f%n", p, p.surfaceWeight(mass));
	}
}
```

* 모든 enum 자료형에는 static values 메서드가 기본으로 정의되어 있다. 이는 enum 상수를 선언된 순서대로 저장하는 배열을 반환한다.
* 일반적으로 유용하게 쓰일 enum이라면, top-level public 으로 선언해야 한다. 

#### 상수들이 제각기 다른 방식으로 동작해야 할 때 
```language-java
// Enum type that switches on its own value - questionable
   public enum Operation {
       PLUS, MINUS, TIMES, DIVIDE;
       
       // Do the arithmetic op represented by this constant
       double apply(double x, double y) {
           switch(this) {
               case PLUS:   return x + y;
               case MINUS:  return x - y;
               case TIMES:  return x * y;
               case DIVIDE: return x / y;
		   }
           throw new AssertionError("Unknown op: " + this);
       }
}
```
그런데 이는 예쁜 코드는 아니다. throw 문 없이는 컴파일되지 않지. 더 큰 문제는 깨지기 쉬운 코드라는 점. 새로운 상수가 추가되고도 switch case 추가 되지 않으면 컴파일은 되어도 런타임시 문제가 된다. <br>
다행히 더 좋은 방법이 있다. constant-specific class body 안에서 실제 메서드를 재정의 하는 것. (constant-specific method implementation이라고 한다.)

```language-java
// Enum type with constant-specific method implementations
public enum Operation {
    PLUS   { double apply(double x, double y){return x + y;} },
    MINUS  { double apply(double x, double y){return x - y;} },
    TIMES  { double apply(double x, double y){return x * y;} },
    DIVIDE { double apply(double x, double y){return x / y;} };
    abstract double apply(double x, double y);
}
```
이러면 새로운 상수를 추가하더라도 apply 메서드 구현을 잊을 수 없다. 잊어버리면 컴파일 오류가 나기 때문. <br>

#### 상수별로 클래스 몸체와 데이터를 갖을 때
```language-java

// Enum type with constant-specific class bodies and data
   public enum Operation {
       PLUS("+") {
           double apply(double x, double y) { return x + y; }
       },
       MINUS("-") {
           double apply(double x, double y) { return x - y; }
       },
       TIMES("*") {
           double apply(double x, double y) { return x * y; }
       },
       DIVIDE("/") {
           double apply(double x, double y) { return x / y; }
       };
       private final String symbol;
       Operation(String symbol) { this.symbol = symbol; }
       @Override public String toString() { return symbol; }
       abstract double apply(double x, double y);
   }

// 사용은 이렇게
  public static void main(String[] args) {
       double x = Double.parseDouble(args[0]);
       double y = Double.parseDouble(args[1]);
       for (Operation op : Operation.values())
           System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
```
<br>
#### toString을 재정의 했을 때 사용할 수 있는 fromString 메서드 예시
```language-java
// Implementing a fromString method on an enum type
   private static final Map<String, Operation> stringToEnum
       = new HashMap<String, Operation>();
   static { // Initialize map from constant name to enum constant
       for (Operation op : values())
           stringToEnum.put(op.toString(), op);
   }
   // Returns Operation for string, or null if string is invalid
   public static Operation fromString(String symbol) {
       return stringToEnum.get(symbol);
   }
```
<br>
#### enum 상수끼리 공유하는 코드가 필요할 때 - strategy enum pattern
```language-java
// Enum that switches on its value to share code - questionable
   enum PayrollDay {
       MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
       private static final int HOURS_PER_SHIFT = 8;

       double pay(double hoursWorked, double payRate) {
       		double basePay = hoursWorked * payRate;
	   
	  	 	double overtimePay; // Calculate overtime pay 
	   		switch(this) {
          	case SATURDAY: case SUNDAY:
          		overtimePay = hoursWorked * payRate / 2;
          		break;
          	default: // Weekdays
            	overtimePay = hoursWorked <= HOURS_PER_SHIFT ? 0 : (hoursWorked - HOURS_PER_SHIFT) * payRate / 2; 
	   		}
      return basePay + overtimePay;
      }
  }
```
간결하지만 유지보수 관점에선 위험한 코드다. 새로운 상수를 추가했을 때, case 추가하는 것을 잊어도 컴파일 오류가 나지 않기 때문. 
좋은 방법은 다음과 같다. (strategy enum pattern)
```language-java
// The strategy enum pattern
   enum PayrollDay {
       MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY),
       WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY),
       FRIDAY(PayType.WEEKDAY),
       SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

       private final PayType payType;
       PayrollDay(PayType payType) { this.payType = payType; }

       double pay(double hoursWorked, double payRate) {
           return payType.pay(hoursWorked, payRate);
       }
       // The strategy enum type
       private enum PayType {
           WEEKDAY {
               double overtimePay(double hours, double payRate) {
                   return hours <= HOURS_PER_SHIFT ? 0 :
                     (hours - HOURS_PER_SHIFT) * payRate / 2;
               }
		   }, WEEKEND {
               double overtimePay(double hours, double payRate) {
                   return hours * payRate / 2;
		  	   }
		   };
           
           private static final int HOURS_PER_SHIFT = 8;
		   abstract double overtimePay(double hrs, double payRate);
           double pay(double hoursWorked, double payRate) {
               double basePay = hoursWorked * payRate;
               return basePay + overtimePay(hoursWorked, payRate);
		   }
	   }
   }
```

## Item 31. Use instance fields instead of ordinals
#### ordinal을 남용한 사례와 그 문제점
```language-java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
   public enum Ensemble {
       SOLO,   DUET,   TRIO, QUARTET, QUINTET,
       SEXTET, SEPTET, OCTET, NONET,  DECTET;
   public int numberOfMusicians() { return ordinal() + 1; } }
```

* 상수 순서를 변경하는 순간 numberOfMusicians 메서드는 깨진다.
* 게다가 이미 사용한 정수값에 대응하는 새로운 enum 상수를 정의하는 것은 아예 불가능. <br> ex) double quartet의 연주자 수는 octet과 마찬가지로 8명.
* 새로운 상수가 나타내는 int값은 순서상 바로 앞에 오는 상수의 int 값보다 정확히 1만큼 커야 한다. 그렇지 않으면 새 상수를 추가할 수 없다.

#### 해결하는 방법은 instance field를 두는 것
```language-java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);
	
	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; } 

	public int numberOfMusicians() { return numberOfMusicians; }
}

```

**참고.** ordinal 메서드는 EnumSet이나 EnumMap처럼 일반적인 용도의 enum 기반 자료구조에서 사용할 목적으로 설계한 메서드다. 따라서 그런 자료 구조를 만들 생각이 없다면, ordinal 메서드는 사용하지 않는 것이 최선이다.

## Item 32. Use EnumSet instead of bit fields
#### Bit field enumeration
```language-java
// Bit field enumeration constants - OBSOLETE!
public class Text {
    public static final int STYLE_BOLD = 1 << 0; //1
    public static final int STYLE_ITALIC = 1 << 1; //2
    public static final int STYLE_UNDERLINE = 1 << 2; //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // Parameter is bitwise OR of zero or more STYLE_ constants
    public void applyStyles(int styles) {  /* 생략 */  }
}
```
이렇게 하면 상수들을 집합(bit field)에 넣을 때, bitwise OR 연산은 사용할 수 있다.
```language-java
   text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

* bitwise 산술연산을 통해 합집합, 교집합 등의 집합 연산도 효율적으로 실행할 수 있다.
* 하지만 int enum 패턴과 똑같은 단점을 갖고 있다... 
  * 출력했을 때 결과값을 이해하기 힘들다
  * 비트 필드에 포함된 모든 요소를 순차적으로 살펴보기 어렵다


#### EnumSet
```language-java
// EnumSet - a modern replacement for bit fields
public class Text {
   public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
   // Any Set could be passed in, but EnumSet is clearly best
   public void applyStyles(Set<Style> styles) { /* 생략 */ }
}

```
* 특정한 enum 타입의 값으로 구성된 집합을 효율적으로 표현할 수 있다.
* Set 인터페이스를 그대로 구현하며, Set이 제공하는 풍부한 기능들을 그대로 제공한다.
* 형안정성, 다른 Set 구현들과 가은 수준의 interoperability를 제공한다.
* 또 내부적으로는 bit vector를 이용하여 bit field 수준의 성능을 보장한다.
* 그러니 성능은 bit field 수준인데, 비트들을 직접 조작할 때 생길 수 있는 오류나 어수선한 코드를 피할 수 있다.

```language-java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

**참고.** EnumSet의 한가지 단점은 자바 1.6부터 immutable EnumSet을 만들 수 없다는 것.. 다음 버전에서 해결될 거라 예측했지만 아직 안됨. Google의 Guava 라이브러리를 사용하면 immutable EnumSet 객체를 만들 수 있긴 함.

##Item 33. Use EnumMap instead of ordinal indexing
#### ordinal을 배열 index로 이용하는 경우
```language-java
	public class Herb {
       public enum Type { ANNUAL, PERENNIAL, BIENNIAL }
       private final String name;
       private final Type type;
    
       Herb(String name, Type type) {
           this.name = name;
           this.type = type;
	   }
       
       @Override public String toString() {
           return name;
	   }
	}
```

```language-java
// Using ordinal() to index an array - DON'T DO THIS!
   Herb[] garden = ... ;
   Set<Herb>[] herbsByType = // Indexed by Herb.Type.ordinal() 
      (Set<Herb>[]) new Set[Herb.Type.values().length];
   
   for (int i = 0; i < herbsByType.length; i++)
       herbsByType[i] = new HashSet<Herb>();
   
   for (Herb h : garden) 
      herbsByType[h.type.ordinal()].add(h);
   
   // Print the results
   for (int i = 0; i < herbsByType.length; i++) {
       System.out.printf("%s: %s%n", Herb.Type.values()[i], herbsByType[i]);
   }
```

* 배열은 제너릭과 호환되지 않으므로 (Item 25) 배열을 쓰려면 unchecked type casting 이 필요하며 깔끔하게 컴파일 되지 않는다.
* 거기다 배열은 index가 무엇을 표현하는지 모르므로, 수동으로 label을 붙여 출력해야한다.
* 가장 심각한 문제는 enum의 ordinal값으로 배열을 참조할 때, 정확한 int값이 사용되도록 해야 한다는 것이다. - 컴파일에서 확인할 수 있는 영역이 아니지..

#### 더 좋은 방법, EnumMap
```language-java
// Using an EnumMap to associate data with an enum
   Map<Herb.Type, Set<Herb>> herbsByType =
       new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);
   for (Herb.Type t : Herb.Type.values())
       herbsByType.put(t, new HashSet<Herb>());
   for (Herb h : garden)
       herbsByType.get(h.type).add(h);
   System.out.println(herbsByType);
```

* 짧고 깔끔하다 - 가독성이 좋다.
* 안전하다.
* ordinal 이용한 프로그램과 성능 면에서 비슷. (내부적으로 ordinal을 배열 index로 이용하는 배열을 사용하고 있기 때문)
* no unchecked casting
* label을 손수 만들 필요도 없다. enum은 출력 가능한 문자열로 알아서 변환되기 때문.
* 배열 indexing 계산하다 오류가 발생할 가능성도 없다. 
- **주의.** EnumMap 생성자는 키의 자료형을 나타내는 Class객체를 인자로 받는다 - 이런 Class 객체를 bounded type token 이라 부른다. runtime generic type 정보를 제공한다. (Item 29 참고)

#### ordinal을 배열 index로 두번이나 사용하는 경우
```language-java
// Using ordinal() to index array of arrays - DON'T DO THIS!
   public enum Phase { SOLID, LIQUID, GAS;
       public enum Transition { 
           MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT; 

	       // Rows indexed by src-ordinal, cols by dst-ordinal
	       private static final Transition[][] TRANSITIONS = {
	          { null,    MELT,     SUBLIME },
	          { FREEZE,  null,     BOIL    },
	          { DEPOSIT, CONDENSE, null    }
		   };
		   
		   // Returns the phase transition from one phase to another 
		   public static Transition from(Phase src, Phase dst) {
		      return TRANSITIONS[src.ordinal()][dst.ordinal()];
		   }
		}
	}
```

* 역시 컴파일러는 ordinal값과 배열 index 사이의 관계를 모른다.
* 따라서 TRANSITIONS 테이블을 올바르게 만들지 않았거나, Phase나 Phase.Transition 을 수정한 다음에 테이블을 제대로 고쳐놓지 않으면 프로그램은 실행도중 오류는 일으킬 것이다. 아니면 이상하게 동작하든지.

#### 더 좋은 방법, EnumMap 중첩해서 쓰기
```language-java
 // Using a nested EnumMap to associate data with enum pairs
   public enum Phase {
      SOLID, LIQUID, GAS;
       
      public enum Transition {
         MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
         BOIL(LIQUID, GAS),   CONDENSE(GAS, LIQUID),
         SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
      
         final Phase src;
         final Phase dst;
      
         Transition(Phase src, Phase dst) {
            this.src = src;
            this.dst = dst;
		 }
		
		 // Initialize the phase transition map
		 private static final Map<Phase, Map<Phase,Transition>> m =
         	 new EnumMap<Phase, Map<Phase,Transition>>(Phase.class);
         
         static {
            for (Phase p : Phase.values())
              m.put(p,new EnumMap<Phase,Transition>(Phase.class));
            for (Transition trans : Transition.values())
              m.get(trans.src).put(trans.dst, trans);
		 }
         
         public static Transition from(Phase src, Phase dst) {
            return m.get(src).get(dst);
		 }
	  }
   }
```
이러면 새로운 상태를 추가한다고 하여도, 테이블을 손수 만질 필요가 없다. 알맞은 enum 상수만 각각 추가하면 프로그램 내에서 알아서 다 해준다!

## Item 34. Emulate extensible enums with interfaces
enum 자료형은 상속이 통한 확장이 불가능하다. 이것은 단점이라 볼 수 없는데, enum 자료형을 상속한다는 것은 대체로 바람직하지 않기 때문이다. <br>
하지만 enum 자료형의 확장이 되면 좋은 경우가 단 한가지 있다. operaton code 를 만들어야 할 때! 이 효과를 내는 방법은 enum이 임의의 interface를 구현할 수 있다는 데에서 시작한다.

####  어떻게 enum 자료형을 확장하나?
```language-java
   // Emulated extensible enum using an interface
   public interface Operation {
       double apply(double x, double y);
   }

   public enum BasicOperation  implements Operation {
       PLUS("+") {
	      public double apply(double x, double y) { return x + y; } },
	   MINUS("-") {
		  public double apply(double x, double y) { return x - y; }
       },
       TIMES("*") {
		  public double apply(double x, double y) { return x * y; } },
	   DIVIDE("/") {
	      public double apply(double x, double y) { return x / y; }
	   };
     
       private final String symbol;
       
       BasicOperation(String symbol) {
           this.symbol = symbol;
       }
       
       @Override public String toString() {
           return symbol;
	   }
   }
```
<br>
이제 여기에 지수연산과 나머지 연산을 확장한다면 다음과 같이 Operation을 구현하는 새로운 enum 자료형을 만들면 된다.
```language-java
   // Emulated extension enum
   public enum ExtendedOperation implements Operation {
       EXP("^") {
           public double apply(double x, double y) {
               return Math.pow(x, y);
           }
       },
       REMAINDER("%") {
           public double apply(double x, double y) {
               return x % y;
	       }
	   };
       
       private final String symbol;
       
       ExtendedOperation(String symbol) {
           this.symbol = symbol;
       }
       
       @Override public String toString() {
           return symbol;
	   }
    }
```
새로 만든 연산들은 기존 연산들이 쓰였던 곳에는 어디든 사용할 수 있다. API가 BasicOperation이 아니라 Operation을 사용하도록 작성되어 있기만 하다면.

#### 클라이언트 코드
```language-java
public static void main(String[] args) { 
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]); 

	test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test( Class<T> opSet, double x, double y) { 
	for (Operation op : opSet.getEnumConstants())
           System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

```language-java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]); 
	test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

* 첫번째 방법은 ExtendedOperation.class를 test로 전달하는데, 이 class 리터럴은 bounded type token (Item 29 참고) 구실을 한다. 
* 두번째 방법은 bouded wildcard type인 Collection<? extends Operation> 을 인자의 자료형으로 사용하는 경우다. 코드의 복잡성은 줄어들었지만, EnumSet이나 EnumMap을 사용할 수 없기 때문에 여러 자료형에 정의한 연산들을 함께 전달할 수 없다. (이 부분 잘 이해안됨)php
* **참고** enum을 interface를 이용해 확장하는 방법은 한가지 단점이 있다. 바로 enum마다 필요한 코드(여기서는 symbol을 저장하고 꺼내는 메서드)가 중복된다는 점. 이 점은 helper class를 따로 둬서 중복을 제거할 수 있을 것이다.

## Item 35. Prefer annotations to naming patterns
#### naming pattern
JUnit 테스트 프레임워크에서 테스트 메서드 이름을 test 로 시작해야 한다는 규칙 같은 거.

##### 단점
- 철자를 틀리면 알아채기 힘들다.
- 특정한 프로그램 요소에만 적용되도록 만들 수 없다. ex) 클래스 이름을 test로 시작하게 한다고 해서 클래스의 메서드 전체를 실행하진 않는다.
- 프로그램 요소에 인자를 전달할 마땅한 방법이 없다. ex) 테스트가 특정 예외를 발생해야 할 때, 예외의 자료형을 전달해야 하는데...

#### annotation 사용하기
###### 어노테이션 정의
```language-java
   // Marker annotation type declaration
   import java.lang.annotation.*;
   /**
    * Indicates that the annotated method is a test method.
    * Use only on parameterless static methods.
    */
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface Test {
   }
```
- @Retention(RetentionPolicy.RUNTIME)은 Test가 실행시간에도 유지되어야 하는 어노테이션이라는 뜻.
- @Target(ElementType.METHOD)은 테스트가 메서드 선언부에만 적용할 수 있는 어노테이션이라는 뜻. 

###### Test어노테이션의 사용예시
```language-java
// Program containing marker annotations
public class Sample {
	@Test public static void m1() { } // Test should pass public static void m2() { }
	@Test public static void m3() { // Test Should fail
           throw new RuntimeException("Boom");
       }
	public static void m4() { }
	@Test public void m5() { } // INVALID USE: nonstatic method public static void m6() { }
	@Test public static void m7() { // Test should fail
           throw new RuntimeException("Crash");
       }
	public static void m8() { }
}
```

###### 테스트 실행기 프로그램 - 리플렉션 이용
```language-java
// Program to process marker annotations
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class testClass = Class.forName(args[0]);

        for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(Test.class)) { tests++;
                try {
                    m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                } catch (Exception exc) {
                    System.out.println("INVALID @Test: " + m);
                }
			}
		}
        
        System.out.printf("Passed: %d, Failed: %d%n", passed, tests - passed);
    }
}
```
* 리플렉션을 통해 호출된 메서드가 예외를 발생시키면 해당 예외는 InvocationTargetException으로 포장된다. 위의 프로그램은 이 예외를 받으면 테서트 메서드가 만든 실제 예외를 담은 테스트 실패 보고서를 출력하는데, 실제 예외는 InvocationTargetException에 정의된 getCause메서드를 호출하면 구할 수 있다.
* 테스트 메서드를 호출했을 때 InvocationTargetException이 아닌 예외가 발생하면, 그것은 컴파일 시에 발견하지 못한, 잘못 사용된 Test 어노테이션이 있다는 뜻. 객체 메서드나, 인자를 받는 메서드, 또는 접근 불가능 메서드에 어노테이션을 붙이면 발생.

#### 1개의 인자를 받는 어노테이션 사용하기
###### 인자를 받는 어노테이션 자료형
```language-java
// Annotation type with a parameter
import java.lang.annotation.*;
   /**
    * Indicates that the annotated method is a test method that
    * must throw the designated exception to succeed.
    */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
    	/** “the Class object for some class that extends Exception”
		 * bounded type token 의 사용예시 
		 */   
        Class<? extends Exception> value();
	}
```

###### 인자를 받는 어노테이션의 사용 예제
```language-java
 // Program containing annotations with a parameter
 public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // Test should pass
        int i = 0;
        i = i / i; 
    }
    
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // Should fail (wrong exception)
        int[] a = new int[0];
        int i = a[1];
	}
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // Should fail (no exception)
}
```

###### 테스트 실행기 
```language-java
if (m.isAnnotationPresent(ExceptionTest.class)) {
       tests++;
       try {
           m.invoke(null);
           System.out.printf("Test %s failed: no exception%n", m);
       } catch (InvocationTargetException wrappedEx) {
           Throwable exc = wrappedEx.getCause();
           Class<? extends Exception> excType =
               m.getAnnotation(ExceptionTest.class).value();
           if (excType.isInstance(exc)) {
               passed++;
           } else {
               System.out.printf(
                   "Test %s failed: expected %s, got %s%n",
                   m, excType.getName(), exc);
           }
       } catch (Exception exc) {
           System.out.println("INVALID @Test: " + m);
       }
}
```

#### 인자를 배열로 받는 어노테이션 사용하기
```language-java
// Annotation type with an array parameter
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
   Class<? extends Exception>[] value(); 
}
```


```language-java
// Code containing an annotation with an array parameter 
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() {
    List<String> list = new ArrayList<String>();
    // The spec permits this method to throw either
    // IndexOutOfBoundsException or NullPointerException
    list.addAll(5, null);
}
```

```language-java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("Test %s failed: no exception%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause(); 
        Class<? extends Exception>[] excTypes = 
            m.getAnnotation(ExceptionTest.class).value();
        int oldPassed = passed;
        for (Class<? extends Exception> excType : excTypes) {
            if (excType.isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("Test %s failed: %s %n", m, exc);
    }
}
```

## Item 36. Consistently use the Override annotation
#### 버그 찾아보기
```language-java
// Can you spot the bug?
   public class Bigram {
       private final char first;
       private final char second;
       public Bigram(char first, char second) {
           this.first  = first;
           this.second = second;
       }
       public boolean equals(Bigram b) {
           return b.first == first && b.second == second;
       }
       public int hashCode() {
           return 31 * first + second;
       }
       public static void main(String[] args) {
           Set<Bigram> s = new HashSet<Bigram>();
           for (int i = 0; i < 10; i++)
               for (char ch = 'a'; ch <= 'z'; ch++)
                   s.add(new Bigram(ch, ch));
           System.out.println(s.size());
       }
   }
```
* Set이니까 개수는 26이 되어야 할 거 같지만, 결과는 260! why? equals 메소드를 오버라이딩 한 게 아니라 오버로딩했거든.
* @Override 어노테이션을 메소드를 붙여줬다면 컴파일을 통해 에러를 잡을 수 있었을 것이다.
* @Override 어노테이션을 쓸만 한 데엔 다 붙이는 게 좋을 거 같다! :) (다만 추상 메서드나 인터페이스의 메서드를 구현할 땐 안붙여도 어짜피 컴파일 에러난다.)
* 추상 클래스나 인터페이스의 경우, 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 abstract 이든 아니든 어노테이션을 붙일 필요가 있다. 실수로 메서드를 더하는 일을 막기 위해서!

## Item 37. Use marker interfaces to define types
#### marker interface
* 아무 메서드도 선언하지 않은 인터페이스
* 해당 클래스가 어떤 속성을 맞고한다는 사실을 표시한다. ex) Serializable

#### marker annotation과의 차이점
* marker interface는 자료형이라는 점! marker annotation은 자료형이 아니다.
* 적용 범위를 좀 더 세밀하게 지정할 수 있다. 예를 들어 특정한 인터페이스를 구현한 클래스에만 적용할 수 있는 표식이 필요하다면, 표식 인터페이스가 그 특정 인터페이스를 extends 하면 된다. 

#### marker annotation
* 프로그램 안에서 어노테이션 자료형을 쓰기 시작한 뒤에도 더 많은 정보를 추가할 수 있다.

#### 적합한 상황
* 새로운 자료형 (without methods)을 정의하고자 한다면 marker interface
* 클래스나 인터페이스 이외의 프로그램요소에 표식을 달고 싶고, 앞으로 표식에 더 많은 정보를 추가할 가능성이 있다면 maker annotation


