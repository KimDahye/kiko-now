---
layout: post
title: "Effective Java Ch 11 Serializable"
tags: [java, effective java]
comments: true
---



이번 장에서 특히 주목할 만한 내용은 Item 78의 serialization proxy pattern. 객체 직렬화 과정에서 발생할 수 있는 많은 문제를 피하도록 도와준다.

## Item 74. Implement `Serializable` judiciously
객체를 직렬화할 수 있다는 것은, 한 VM에서 다른 VM으로 (네트워크 통신을 사용하든 디스크에 저장하는 과정은 거치든) 객체를 옮길 수 있다는 듯이다. 클래스 선언부에 `implements Serializable`를 붙이면 직렬화 가능한 객체를 만드는 클래스를 구현할 수 있다. 직렬화 가능 클래스를 만드는 것은 당장의 비용은 적어보이나, 장기적으로 봤을 땐 할 일이 매우 많고 복잡해질 수 있으니 신중해야 한다. 

#### serializable을 구현할 때 힘든 점
###### 일단 클래스를 배포하고 나면, 클래스 구현을 유연하게 바꿀 수 없다.
* Serializable을 구현하고 나면, 그 클래스의 바이트 스트림 인코딩(serialized form)도 공개 API의 일부가 되어버린다. custom serialized form을 설계하지 않고, default를 그대로 사용하는 경우, 직렬화 형식은 영원히 클래스의 원래 내부 표현방식에 종속된다. **default 직렬화 형식을 받아들이면** 그 클래스의 private, package-private 객체 필드도 공개 API가 된다. > 그럼 custom serialized form을 설계하면 괜찮나? 대신 복잡하고 어렵겠지?!
* 원래 직렬화 형식을 유지하면서도 내부 표현을 바꿀 수 있긴 하지만 (ObjectOutputStream.putFields, ObjectInputStream.readFields 이용하여) 까다롭고 소스코드에 흔적이 남는다.
* 예를 들어 직렬화 가능 클래스에는 고유한 식별 번호인 `serialVersionUID`라는 이름의 static final long 필드를 명시적으로 선언해야 하는데, 이 때 값을 지정하지 않으면 시스템이 복잡한 절차를 거쳐 식별번호를 자동 생성하여 클래스에 붙인다. 자동 생성되는 식별번호는 클래스 이름, 해당 클래스가 구현하는 모든 인터페이스 이름 등에 영향을 받기때문에 이들 중 어느것 하나라도 변경하면, 자동 생성되는 UID는 바뀐다. 따라서 **명시적으로 UID를 선언하지 않으면** 호환성이 깨져 실행 도중 InvalidClassException이 발생할 수 있다. 
* 따라서 클래스 진화라는 관점에서 직렬화는 족쇄가 될 수 있다. 아주 신중하게 설계해야 한다.

###### Serializable 을 구현하면, 버그나 security hole이 생길 가능성이 높아진다. 
* 이유는 직렬화는 extralinguistic 객체 생성 메커니즘을 따르기 때문. 역직렬화도 숨은 "생성자"인데, 생성자가 만족해야 하는 불변식이나 객체 생성중 객체 내부에 공격자가 접근할 수 없도록 해야 한다는 것을 잊기 쉽다. 기본 역직렬화 매커니즘을 그대로 사용하면 객체는 불변식 훼손, 불법 접근 문제에 쉽게 노출된다. (Item 76 참고)

###### 새 버전 클래스를 내놓기 위한 테스트 부담이 늘어난다.
* 새 릴리즈에서 직려화한 객체를 예전 릴리즈에서 역직렬화할 수 있는지, 그 역도 가능한지 검사해야 한다.
* 따라서 필요한 테스트의 양은, `직렬화 가능 클래스 수 x 릴리즈 수`에 비례한다. 
* 또한, binary compatibility 뿐 아니라, semantic compatibility(역직렬화한 객체가 원래 객체의 faithful copy인지 확인)까지 테스트해야 하므로, 테스트를 자동생성할 수 없다. 


#### 언제 어떻게 Serializable을 구현할까?
###### 기본 규칙
* 값 클래스는 Serializable 을 구현하고, ThreadPool같은 활성 개체(active entity)는 Serializable을 구현할 일이 거의 없다.

###### 상속 관련
* 상속을 염두에 두고 설계하는 클래스는 Serializable을 구현하지 않는 게 바람직하다. 인터페이스도 가급적 Serializable을 상속하지 말아야 한다. 이러한 클래스(인터페이스)를 상속하거나 구현하는 개발자는 많은 부담을 떠안게 되기 때문. 하지만 꼭 구현해야 하는 상황들이 있을 수 있다. (Serializable을 구현한 객체만 프레임워크에 참여할 수 있다거나)
* 상속을 고려해 설계된 클래스 중 Serializable 을 구현한 클래스로는 Throwable, Component, HttpServlet(세션 상태를 캐싱하기 위해) 등이 있다. 
* 상속 가능하면서 객체 필드를 갖는 클래스를 만들 땐, 객체 필드가 기본값으로 초기화되면 위배되는 불변식이 있는 경우에는 readObjectNoData 메서드를 반드시 추가해줘야 한다.
```java
// readObjectNoData for stateful extendable serializable classes
private void readObjectNoData() throws InvalidObjectException {
   throw new InvalidObjectException("Stream data required");
}
```

* 상속을 고려해 설계했으나 `Serializable`을 구현하지 않기로 한 상황. 상위 클래스에 parmeterless 생성자가 없다면, 하위 클래스는 직렬화할 수 없다. 이를 해결하기 위해서는 다음과 같이 protected 무인자 생성자 하나와 initialize 메서드 하나를 추가로 작성해야 한다.
```java
public abstract class AbstractFoo {
    private int x, y; //final 이 될 수 없다는 것에 주의.

    // This enum and field are used to track initialization
    private enum State {
        NEW, INITIALIZING, INITIALIZED
    }

    ;

    private final AtomicReference<State> init = new AtomicReference<State>(State.NEW);

    public AbstractFoo(int x, int y) {
        initialize(x, y);
    }

    // This constructor and the following method allow
    // subclass's readObject method to initialize our state.
    protected AbstractFoo() {
    }

    protected final void initialize(int x, int y) {
        if (!init.compareAndSet(State.NEW, State.INITIALIZING))
            throw new IllegalStateException("Already initialized");
        this.x = x;
        this.y = y;
        init.set(State.INITIALIZED);
    }

    // These methods provide access to internal state so it can
    // be manually serialized by subclass's writeObject method.
    protected final int getX() {
        checkInit();
        return x;
    }

    protected final int getY() {
        checkInit();
        return y;
    }

    // Must call from all public and protected instance methods
    private void checkInit() {
        if (init.get() != State.INITIALIZED)
            throw new IllegalStateException("Uninitialized");
    }
}
```
* * AbstractFoo 의 모든 public, protected instance 메서드는 어떤 작업을 하기 전에 반드시 checkInit 메서드를 호출해야 한다. 하위 클래스에서 객체 초기화를 다 하지 않은 상태에서 다른 메서드를 호출하면 재빨리 실패하도록 하기 위함이다.
  * init 필드가 atomic reference 필드임에 주의하자. 이 조치가 없으면, 어떤 스레드는 깨진 상태의 객체를 이용할 수 있다. 즉, 객체의 무결성이 보존되지 않을 수 있다. - [Java Atomic 타입의 이해](http://128.199.231.48/java-atomic-type/) 참고
  * compareAndSet 을 사용해 enum에 대한 참조를 원자적으로 조작하는 이 패턴은 general-purpose thread-safe satate machine 을 구현하기 좋다.

```java
// Serializable subclass of nonserializable stateful class
public class Foo extends AbstractFoo implements Serializable {
    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        // Manually deserialize and initialize superclass state
        int x = s.readInt();
        int y = s.readInt();
        initialize(x, y);
    }
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        // Manually serialize superclass state
        s.writeInt(getX());
        s.writeInt(getY());
    }
    // Constructor does not use the fancy mechanism
    public Foo(int x, int y) { super(x, y); }
    private static final long serialVersionUID = 1856835860954L;
}
```
###### inner class
* inner class는 Serializable을 구현하면 안된다. (enclosing instance와 enclosing scope의 지역변수 값을 보관하기 위해 컴파일러가 자동 생성하는 synthetic fields 가 있기 때문.) 하지만, static member class는 Serializable을 구현해도 된다.  

## Item 75. Consider using a custom serialized form
(이 장을 읽고 나서, 여기서 말하는 custom serialized form이 writeObject, readObject 메서드를 이용하여 serialize 하는 거라는 것 알게 되었다.)

#### 기본 직렬화 형식 (default serialized form)
* 해당 객체가 루트인 객체 그래프의 물리적 표현(physical representation)을 나름 효과적으로 인코딩한 것
* 객체 안에 담긴 데이터와, 해당 객체를 통해 접근할 수 있는 모든 객체에 담긴 데이터를 인코딩한다. 또한, 이 객체들이 서로 연결된 topology도 기술한다. (-> 여기서 topology라 함은... 객체들이 어떻게 연관되어 있는지 그 상태(모양)를 뜻하는 말이지 않을까?) 
* [효과적인 직렬화 형식] 해당객체가 나타내는 논리적인 데이터만 담아야 하며, 물리적 표현과는 무관해야 함.
* 따라서, 기본 직렬화 형식은 그 객체의 물리적 표현이 논리적 내용과 동일할 때만 적절하다. 

#### 기본 직렬화 형식이 적절한 경우
```language-java
// Good candidate for default serialized form
public class Name implements Serializable {
   /**
    * Last name. Must be non-null.
    * @serial
    */
   private final String lastName;
   /**
    * First name. Must be non-null.
    * @serial
    */
   private final String firstName;
          /**
    * Middle name, or null if there is none.
    * @serial
    */
   private final String middleName;
   ... // Remainder omitted
}
```
* lastName, firstName, middleName은 private필드임에도 문서화 주석이 달려있음에 주의. 이 private필드들이 public API여서 반드시 문서화 해야 한다. 
* 기본 직렬화 형식이 만족스럽다 하더라도, invariant나 보안 조건을 만족시키기 위해서는 readObject메서드를 구현해야 마땅한 경우도 많다. (Item 76, 78에서 자세히 다뤄짐)

#### 기본 직렬화 형식이 적절치 못한 경우
```language-java
// Awful candidate for default serialized form
public final class StringList implements Serializable {
   private int size = 0;
   private Entry head = null;
   
   private static class Entry implements Serializable {
       String data;
	   Entry next;
       Entry  previous; // doubly linked list임을 알 수 있다.
   }
   ... // Remainder omitted
}
```
* 논리적으로 이 클래스는 문자열 리스트를 표현한다. 물리적으로는 이 리스트는 doubly linked list이다. 기본 직렬화 형태로 가면, 모든 연결 리스트 항목과 항목 간 양방향 연결 구조가 그대로 반영될 것이다.
* 객체의 물리적 표현 형태가 논리적 내용과 많이 다를 경우에 기본 직렬화 형식을 그대로 받아들이면 아래의 4가지 문제가 생긴다.
  * 공개 API가 현재 내부 표현 형태에 영원히 종속된다.
  * 너무 많은 공간을 차지하는 문제
  * 너무 많은 시간을 소비하는 문제
  * 스택 오버플로우 문제 - 기본 직렬화 절차는 재귀적인 객체 그래프 순회를 하는데, 객체 크기가 과도하게 크지 않더라도 스택오버플로우가 날 수 있다.
* StringList의 적절한 직렬화 형식은 (리스트에 담기는 문자열의 수, 실제 문자열) 형태일 것이다. 기본 직렬화 형식에 포함되지 않는 개체 필드임을 나타내기 위해 `transient`를 사용하여 다음과 같이 바꿀 수 있다.
```language-java
// StringList with a reasonable custom serialized form
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;
    
    // No longer Serializable!
    private static class Entry {
        String data;
		Entry next;
        Entry  previous;
    }

    // Appends the specified string to the list
    public final void add(String s) { ... }
	/**
	* Serialize this {@code StringList} instance.
	*
	* @serialData The size of the list (the number of strings 
	* it contains) is emitted ({@code int}), followed by all of 
	* its elements (each a {@code String}), in the proper
	* sequence.
	*/
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        
        // Write out all elements in the proper order.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
	}

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // Read in all elements and insert them in list
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
	}
    ... // Remainder omitted
}
```
* StringList의 모든 필드가 transient인데도 writeObject가 맨 처음으로 defaultWriteObject를 호출하고 있다. readObject가 처음으로 하는 것도 defaultReadObject를 호출한다. 객체의 모든 필드가 transient여도 defaultWriteObject를 호출하면 직렬화 형식이 바뀌며, 그 결과로 유연성이 크게 향상된다. 나중에 non-transient 객체 필드를 추가하더라도 상위 및 하위 호환성이 유지된다. 
* 성능도 기본 직렬화 형식보다 훨씬 좋다.
* 객체의 논리적 상태를 구성하는 값이라는 확신이 있을 때만 non-transient 필드로 만들어라. (대부분 필드는 transient 필드여야 한다.)


#### 참고 사항
* 기본 직렬화 형식을 사용하는 경우, transient 필드들은 역직렬화 되었을 때 default value로 초기화된다. transient 필드의 값이 이런 값이 되면 안되는 경우엔, defaultReadObject를 호출하는 readObject를 구현하여 transient 필드 값을 적절히 복구해줘야 한다. (Item 76)
* 기본직렬화 형식 사용 여부에 상관 없이, 객체를 직렬화 할 때는 객체의 상태 전부를 읽는 메서드에 적용할 동기화 수단을 반드시 적용해야 한다. (이게 뭔 말일까...)
> Whether or not you use the default serialized form, you must impose any synchronization on object serialization that you would impose on any other method that reads the entire state of the object. So, for example, if you have a thread-safe object (Item 70) that achieves its thread safety by synchronizing every method, and you elect to use the default serialized form, use the following writeObject method:
```language-java
// writeObject for synchronized class with default serialized form 
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
       s.defaultWriteObject();
}
```
* 어떤 직렬화 형식 이용하건, 직렬화 가능 클래스 구현할 때 serial version UID 를 명시적으로 선언해야 한다. (Item 74)

## Item 76. Write readObject methods defensively
#### 전반적인 설명
```language-java
// Immutable class that uses defensive copying
public final class Period {
   private final Date start;
   private final Date end;
	
	/**
	* @param start the beginning of the period
	* @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
	* @throws NullPointerException if start or end is null
	*/
   public Period(Date start, Date end) {
       this.start = new Date(start.getTime());
       this.end   = new Date(end.getTime());
       if (this.start.compareTo(this.end) > 0)
           throw new IllegalArgumentException(start + " after " + end);
   }
   public Date start () { return new Date(start.getTime()); }
   public Date end ()   { return new Date(end.getTime()); }
   public String toString() { return start + " - " + end; }
   ... // Remainder omitted
}
```
```language-java
public class BogusPeriod {
	// Byte stream could not have come from real Period instance!
	private static final byte[] serializedForm = new byte[] { (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06, 0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8, 0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02, 0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f, 0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74, 0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75, 0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a, (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00, 0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf, 0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03, 0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22, 0x00, 0x78 };

	public static void main(String[] args) {
	   Period p = (Period) deserialize(serializedForm);
	   System.out.println(p);
	}
	  
	// Returns the object with the specified serialized form
	private static Object deserialize(byte[] sf) {
	    try {
	       InputStream is = new ByteArrayInputStream(sf);
	       ObjectInputStream ois = new ObjectInputStream(is);
	       return ois.readObject();
	    } catch (Exception e) {
	       throw new IllegalArgumentException(e);
		} 
	}
}

```
* Item 39에서 나온 Priod 클래스. 객체의 물리적 표현과 논리적 표현이 같다. 
* 그렇다면 `implements Serializable`만 붙이면 쉽게 직렬화할 수 있는 것 아닐까?
* No. Why?
* `readObject` 메서드가 생성자와 마찬가지로, 유효성 검사와 방어적 복사를 필요로 하기 때문이다.

#### 1차 해결. 유효성 검사만 넣기 (not perfect)
```language-java
// readObject method with validity checking
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
	s.defaultReadObject();
	// Check that our invariants are satisfied
	if (start.compareTo(end) > 0)
		throw new InvalidObjectException(start +" after "+ end);
}
```
* 이렇게 수정하면 공격자가 이상한 Period 객체를 만드는 것은 막을 수 있으나, 또다른 문제가 생긴다.
* 유효한 Period 객체로 시작하는 바이트 스트림을 만든 다음, Period객체 내부의 private 필드에 대한 참조를 추가하는 것이다.
* 이렇게 되면 공격자는 ObjectInputStream을 통해 Period 객체를 읽은 다음, 스트림에 추가된 악의적인 객체 참조(rogue object reference)를 읽는다. 다음 코드를 보자.

```language-java
public class MutablePeriod {
    // A period instance
    public final Period period;

	// period's start field, to which we shouldn't have access 
	public final Date start;

	// period's end field, to which we shouldn't have access 
	public final Date end;

	public MutablePeriod() {
        try {
            ByteArrayOutputStream bos =
                new ByteArrayOutputStream();
            ObjectOutputStream out =
                new ObjectOutputStream(bos);
			
			// Serialize a valid Period instance 
			out.writeObject(new Period(new Date(), new Date()));
			
			/*
			* Append rogue "previous object refs" for internal 
			* Date fields in Period. For details, see "Java
			* Object Serialization Specification," Section 6.4. 
			*/
			
			byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // Ref #5 
			bos.write(ref); // The start field
			ref[4]=4; //Ref#4
			bos.write(ref); // The end field
            
            // Deserialize Period and "stolen" Date references
            ObjectInputStream in = new ObjectInputStream(
            new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start  = (Date)   in.readObject();
            end    = (Date)   in.readObject();
        } catch (Exception e) {
            throw new AssertionError(e);
        }
	}
}
```
```language-java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

	// Let's turn back the clock pEnd.setYear(78); 
	System.out.println(p);

    // Bring back the 60s!
    pEnd.setYear(69);
    System.out.println(p);
}
// 결과: 
// Wed Apr 02 11:04:26 PDT 2008 - Sun Apr 02 11:04:26 PST 1978
// Wed Apr 02 11:04:26 PDT 2008 - Wed Apr 02 11:04:26 PST 1969
```
#### 2차 해결. 유효성 검사 + 방어적 복사까지
```language-java
// readObject method with defensive copying and validity checking
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
	 s.defaultReadObject();
	 
	 // Defensively copy our mutable components
	 start = new Date(start.getTime());
	 end   = new Date(end.getTime());
	 
	 // Check that our invariants are satisfied
	 if (start.compareTo(end) > 0)
		throw new InvalidObjectException(start +" after "+ end);
}
```
* Period를 공격에서 보호하려면, 다음의 두가지 사항을 지켜야 한다. (Item 39 참고)
  *  유효성 검사 이전에 방어적 복사를 시행하고 있음에 주의하자. (이래야 시간차 공격을 막을 수 있다.)
  *  방어적 복사에는 Date의 clone 메서드를 사용하지 않는다는 것도 주의하자.
* final 선언된 필드에는 방어적 복사를 할 수 없다. 따라서 start, end 필드 변수를 non-final 변수로 바꿔야 한다. 

#### 기타 참고 사항
* realease 1.4 부터 방어적 복사 없이도 악의적 객체 참조공격을 막을 수 있도록 writeUnshared, readUnshared 메서드가 ObjectStream에 추가되었으나, 규칙 77에 설명하는 ElvisStealer 공격같은 복잡한 공격에 취약하다. 따라서 **writeUnshared, readUnshared 메서드는 사용하지 말자.**
* 정리하자면, 기본 readObject 메서드면 충분한지 알아보려면 다음의 테스트를 해보자. 객체 내부의 모든 non transient 필드에 저장한 값을 인자로 받아서 유효성 검사 없이 대입해버리는 public 생성자를 추가해도괜찮다는 생각이 드는가? 그렇지 않으면 readObject메서드를 구현하여 유효성 검사 + 방어적 복사를 해야 한다. (물론 생성자에도 해야지.) **대신, serialization proxy pattern(Item 78)을 사용하는 것도 한 방법이다.**
* 상속가능한 클래스에서는, readObject 메서드도 생성자와 마찬가지로 재정의가능(overridable)메서드는 직접적이건 간접적이건 호출해서는 안된다. (Item 17)

## Item 77. For instance contorl, prefer enum types to `readResolve`
* 이번 장은 제대로 이해를 못했다...

#### 직렬화와 싱글턴
* 모든 readObject 메서드는 새로 생성된 객체를 반환한다. 이 객체는 클래스가 초기화될 당시에 만들어진 객체와 같은 객체가 아니다.
* readResolve 메서드를 이용하면 readObject가 만들어낸 객체를 다른 것으로 대체할 수 있다. 역직렬화할 객체의 클래스에 제대로 선언된 readResolve 메서드가 정의되어 있는 경우, 역직렬화가 끄나서 만들어진 객체에 대해 이 메서드가 호출된다. 그리고 새로 만들어진 객체 대신, readResolve 메서드가 반환하는 객체가 사용자에게 돌아간다. 

#### readResolve 로 싱글턴을 만들 때엔 반드시 모든 필드가 transient 여야 한다.
```language-java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    //non-transient field
    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void p`rintFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        // Return the one true Elvis and let the garbage collector
        // take care of the Elvis impersonator.
        return INSTANCE;
    }
}
```
```language-java
public class ElvisStealer {
    static Elvis impersonator;
    private Elvis payload;

    private static final long serialVersionUID = 0;

    private Object readResolve() {
        // Save a reference to the "unresolved" Elvis instance
        impersonator = payload;

        // Return an object of correct type for favorites field
        return new String[] { "A Fool Such as I" };
    }
}
```
```language-java
public class ElvisImpersonator {
    // Byte stream could not have come from real Elvis instance!
    private static final byte[] serializedForm = new byte[]{
            (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05, 0x45, 0x6c, 0x76, 0x69, 0x73, (byte) 0x84, (byte) 0xe6, (byte) 0x93, 0x33, (byte) 0xc3, (byte) 0xf4, (byte) 0x8b, 0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76, 0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73, 0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c, 0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74, 0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64, 0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b, 0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
    };

    // Returns the object with the specified serialized form
    private static Object deserialize(byte[] sf) {
        try {
            InputStream is = new ByteArrayInputStream(sf);
            ObjectInputStream ois = new ObjectInputStream(is);
            return ois.readObject();
        } catch (Exception e) {
            throw new IllegalArgumentException(e);
        }
    }

    public static void main(String[] args) {
        // Initializes ElvisStealer.impersonator and returns
        // the real Elvis (which is Elvis.INSTANCE)
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;
        elvis.printFavorites();
        impersonator.printFavorites();
    }

    /** 
      * main함수 결과
      * [책] [Hound Dog, Heartbreak Hotel] \n [A Fool Such as I] 
      * [실제는?] 자바 1.8에서 돌렸을 때 에러 - byte stream이 잘못 되었는지, 
      * (Exception in thread "main" java.lang.IllegalArgumentException: 
      * java.io.InvalidClassException: Elvis; class invalid for deserialization) 
      * 에러가 난다.
      */
}
```
* 동작원리
  * 도둑 클래스 - 직렬화된 싱글턴 객체(도둑이 숨겨져 있다)를 참조하는 객체필드(payload)와 readResolve 메서드가 있다. 
  * 직렬화 스트림(serializedForm) 안에는 싱글턴의 non-transient 필드가 참조하는 대상을 도둑 객체로 바꿔 놓는다. (=> circularity 발생: 싱글턴은 도둑 참조, 도둑은 싱글턴 참조)
  * 싱글턴에 도둑이 포함되었으므로, 싱글턴이 deserialized될 때 도둑 객체의 readResolve 메서드가 먼저 실행된다. 이 때, 도둑 객체의 필드는 부분적으로(아직 readResolve가 실행되지 않은) deserialized 된 싱글턴 객체를 참조하게 된다.
  * 도둑객체는 readResolve 메서드가 실행된 후에 객체를 참조할 수 있도록 readResolve 안에서 static field(여기서 impersonator)에 이 참조를 저장해둔다.
  * 그런 후, 이 메서드는 도둑 객체를 숨겼던 비 transient 필드의 원래 자료형에 맞는 값을 반환한다. (이렇지 않으면 ClassCastException 발생) 

#### 문제 해결하기 위해, 모든 필드를 transient로 하는 방법도 있지만, 싱글턴인 경우엔 원소가 하나인 enum 자료형으로 고쳐 해결하는 편이 낫다.(Item 3)
* readResolve 를 사용하는 것은 깨지기도 쉽고 신중하게 사용해야 하기 때문.
* 아래와 같이 하라.
``` language-java
// Enum singleton - the preferred approach
public enum Elvis {
	INSTANCE;
	private String[] favoriteSongs =
	   { "Hound Dog", "Heartbreak Hotel" };
	public void printFavorites() {
	   System.out.println(Arrays.toString(favoriteSongs));
	}
}
```
#### readResolve의 accessibility
* readResolve 메서드를 final 클래스에 두는 경우엔 반드시 private으로 선언해야 한다. 
* final 클래스가 아닐 때, readResolve가 private이면 하위 클래스에는 적용되지 않는다. 
* final 클래스가 아닐 때, readResolve가 protected나 public이면, readResolve를 재정의하지 않은 모든 하위 클래스에 적용이 될 텐데 이러면 직렬화된 하위 클래스 객체를 deserialize 하면 상위 클래스 객체가 만들어져 ClassCastException이 발생할 것이다. 

## Item 78. Consider serialization proxies instead of serailized instances
#### serialization proxies
1. private static nested class (== serialization proxy)를 만든다. (outer class 객체의 논리적 상태를 간결하게 표현하는)
* 바깥 클래스를 인자 자료형으로 사용하는 생성자 하나 (일관성 검사 필요 없음, 방어적 복사 필요 없음)
```language-java
// Serialization proxy for Period class
private static class SerializationProxy implements Serializable { 
	private final Date start;
	private final Date end;
      
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
	}

	private static final long serialVersionUID = 234098243823485285L; // Any number will do (Item 75)
}
```

2. outer class에 writeReplace 구현 (serialzation proxy 사용하여 직렬화하도록)
```language-java
 // writeReplace method for the serialization proxy pattern
private Object writeReplace() {
	return new SerializationProxy(this);
}
```
3. writeReplace 메서드를 갖추면, 직렬화 시스템은 바깥 클래스로 직렬화된 객체를 절대로 만들지 않으나 공격자는 클래스의 불변식을 훼손하고자 바깥 클래스의 객체를 직렬화할 수 있다. 그런 공격을 막으려면 다음의 readObject메서드를 추가한다. 
```language-java
// readObject method for the serialization proxy pattern
private void readObject(ObjectInputStream stream)
       throws InvalidObjectException {
   throw new InvalidObjectException("Proxy required");
}
```

4. SerializationProxy 클래스에 자기와 논리적으로 동일한 바깥 클래스를 반환하는 readResolve 메서드 추가 (질문. readObject가 아닌 readResolve 인 이유는?)
```language-java
// readResolve method for Period.SerializationProxy 
private Object readResolve() {
    return new Period(start, end);  // Uses public constructor
}
```
* 이렇게 생성자 (혹은 정적 팩터리, 일반 메서드)를 통해 객체를 만들기 때문에 extralinguistic 특성이 대부분 제거된다!
* 따라서 deserialized 객체가 불변식을 준수하도록 별도의 수단을 동원할 필요가 없다.

#### 완성된 Period 클래스의 SerializationProxy
```language-java
public final class Period {
   private final Date start;
   private final Date end;
	
	/**
	* @param start the beginning of the period
	* @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
	* @throws NullPointerException if start or end is null
	*/
	public Period(Date start, Date end) {
	   this.start = new Date(start.getTime());
	   this.end   = new Date(end.getTime());
	   if (this.start.compareTo(this.end) > 0)
	       throw new IllegalArgumentException(start + " after " + end);
	}

	public Date start () { return new Date(start.getTime()); }
	public Date end ()   { return new Date(end.getTime()); }
	public String toString() { return start + " - " + end; }
	... // Remainder omitted

	// outer class에 writeReplace 구현 (serialzation proxy 사용하여 직렬화하도록)
	private Object writeReplace() {
		return new SerializationProxy(this);
	}

	// outer class 객체를 직렬화할 수 없도록 원천봉쇄!
	private void readObject(ObjectInputStream stream)
	       throws InvalidObjectException {
	   throw new InvalidObjectException("Proxy required");
	}

	// Serialization proxy for Period class
	private static class SerializationProxy implements Serializable { 
		private final Date start;
		private final Date end;
		  
		SerializationProxy(Period p) {
		    this.start = p.start;
		    this.end = p.end;
		}

		//readResolve() 메서드 추가
		private Object readResolve() {
  		 	return new Period(start, end);  // Uses public constructor
		}

		private static final long serialVersionUID = 234098243823485285L; // 아무숫자나 상관 없음
	}
}
```

#### SerializationProxy 의 장점
* 가짜 바이트 스트림 통한 공격 방지
* 내부 필드 탈취 공격 방지
* (item 76 의 해결책과 비교했을 때 더 좋은점) 필드를 final 로 선언할 수 있음. 따라서 진정한 immutable 클래스 만들 수 있음!  
* (item 76 의 해결책과 비교했을 때 더 좋은점) 직렬화 중 유효성 검사를 할 필요도 없다.
* (item 76 의 해결책과 비교했을 때 더 좋은점) 역직렬화된 객체가 애초에 직렬화된 객체와 다른 클래스가 되도록 만들 수 있다. 

#### 역직렬화된 객체가 직렬화된 객체와 다른 클래스가 되는 예시, EnumSet
```language-java
// EnumSet's serialization proxy
   private static class SerializationProxy <E extends Enum<E>>
           implements Serializable {
       // The element type of this enum set.
       private final Class<E> elementType;
       
       // The elements contained in this enum set.
       private final Enum[] elements;
       
       SerializationProxy(EnumSet<E> set) {
           elementType = set.elementType;
           elements = set.toArray(EMPTY_ENUM_ARRAY);  // (Item 43)
	   }
       
       private Object readResolve() {
           EnumSet<E> result = EnumSet.noneOf(elementType);
           for (Enum e : elements)
               result.add((E)e);
           return result;
       }

       private static final long serialVersionUID = 362491234563181265L;
}
```
* 64개 원소 있는 EnumSet es(RegularEnumSet 타입)를 직렬화 
* es에 다섯개 원소 추가
* 직렬화했던 것을 역직렬화하면 JumboEnumSet 타입으로 나온다!

#### Serialization proxy의 제약
* 클라이언트가 확장할 수 있는 클래스에 적용할 수 없다. (item 17)
* 객체 그래프에 circularity가 있으면 적용 할 수 없다.
* 비용이 조금 더 든다. 

## 스터디 내용
#### Serializable 기억 해야 할 것
* private 필드도 공개된다.
* serialVersionUID 는 꼭 명시적으로 지정해줘라.
* serializable 일 때, 상속관계는 잘 안하지만 - 직렬화가 가능하지 않은 부모를 상속하는 직렬화 가능 자식 클래스가 있을 때, 이 자식이 직렬화가 된다는 것 주의해라. 따라서 자식이 deserialize 할 때, readObject를 콜하고, 여기서 super()를 콜하기 때문에 부모에 **빈 껍데기 생성자를 만들어둬야 한다.** (+부모 필드변수 초기화해줘야 한다면, initialize 메서드도 추가해야겠지.)
  * 좀 더 자세히 이 이유를 설명하자면, 객체가 serialize 될 때, 객체 그래프 자체를 덤프뜨는데 - 이 때 이 그래프는 serializable을 구현 안한데까지 올라가게 된다. 따라서 일반 객체는 Object 까지 올라가겠고, 위와 같은 경우엔 부모 클래스까지만 올라가게 될 것이다. 따라서 이 부모를 생성할 수 있는 메커니즘이 필요한 거다.
* readObject 메서드는 숨겨진 생성자다. 생성자에 들어가는 유효성검사나 방어적 복사, readObject 메서드에 꼭 넣어줘야 한다.


#### 실무에서 어떻게 쓰이나?
* 실무에선, 성능 이슈도 있고 언어외적인 요소라 컨트롤하기 까다로운 이슈도 있어 serializable 를 순수하게 쓰지 않는다. 캐시에서는 직렬화기능이 필수적인데, 이 캐시들에서 custom하게 serialize 를 구현해놓는다. 
* 실무에서는 externalize를 더 많이 쓴다.
  * 이에 대해 찾아보니 논의가 있다. [스택오버플로우](http://stackoverflow.com/questions/817853/what-is-the-difference-between-serializable-and-externalizable-in-java)를 보니, externalizable은 reflection 성능이 안좋았을 때 이를 사용하는 serializable 성능이 문제가 되어서 직렬화 과정의 customize를 강제(?)하는 etxernalizable이 나왔는데 - 현재는 reflection 성능도 좋아졌고, JBoss등에서 customized 직렬화 기능도 제공하고 그래서 externalizable은 고대 유물(relic)이라고 평한 답변이 있다.
