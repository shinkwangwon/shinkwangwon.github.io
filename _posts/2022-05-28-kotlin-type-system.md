---
layout: post
title: "코틀린 타입 시스템"
tags: [Kotlin]
comments: true
date: 2022-05-28
---

# 6장. 코틀린 타입 시스템 (Kotlin in Action)

## 6.1 널 가능성 (nullability)

- 널 가능성은 NullPointerException을 피할 수 있게 돕는 코틀린 타입 시스템의 특성이다.
- null 가능 여부를 `타입 시스템`에 추가함으로써 컴파일 시점에 오류를 감지하도록 한다.

### 6.1.1 널이 될 수 있는 타입

코틀린과 자바의 가장 중요한 차이는 코틀린 타입 시스템이 널이 될 수 있는 타입을 명시적으로 지원한다는 점이다.

자바 함수를 먼저 보자.

```java
int strLen(String s) {
   return s.length();
}
```

이 함수는 인자로 null을 넘기면 NullPointerException이 발생한다. 

코틀린으로 함수를 작성해보자.

코틀린에서 이런 함수를 작성할 때 가장 먼저 정해야할 것은 `이 함수가 null을 인자로 받을 수 있는가?` 이다.

```kotlin
fun strLen(s: String) = s.length

>>> strLen(null)
ERROR: Null can not be a value of a non-null type String
```

이 함수의 경우 null을 받을 수 없도록 선언했기 때문에 null 을 넘기면 컴파일시 오류가 발생한다. (null이 될 수 있는 변수를 인자로 넘겨도 마찬가지)

따라서 strLen 함수가 결코 실행 시점에 NPE를 발생시키지 않으리라 장담할 수 있다.

이 함수가 Null과 문자열을 인자로 받도록 하려면 타입이름 뒤에 물음표(?)를 명시해야한다.

```kotlin
fun strLenSafe(s: String?) = ...
```

어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수에 null 참조를 저장할 수 있다는 뜻이다.

물음표가 없다면 기본적으로 모두 null이 될 수 없는 타입이다.

null이 될 수 있는 타입인 변수에 대해서는 수행할 수 있는 연산이 제한된다.

```kotlin
>>> fun strLenSafe(s: String?) = s.length()

ERROR: only safe (?.) or non-null asserted(!!.) calls are allowed on a nullable receiver of type kotlin.String?
```

null이 될 수 있는 값을 null이 될 수 없는 타입의 변수에 대입할 수 없다.

```kotlin
>>> val x: String? = null
>>> var y: String = x

ERROR: Type mismatch: inferred type is String? but String was expected
```

null이 될 수 있는 타입에 대해서, 일단 null 과 비교하고 나면 컴파일러는 그 사실을 기억하고 null이 아님이 확실한 영역에서는 해당 값이 null이 아닌 것처럼 사용할 수 있다.

코틀린에서는 null 체크를 위해 (?.) 연산자를 지원한다. 잠시 뒤에 살펴보자.

```kotlin
fun strLenSafe(s: String?): Int = if(s != null) s.length else 0
```

### 6.1.2 타입의 의미

타입은 분류(classfication)를 뜻하며, 타입은 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다.

**자바**

자바에서 String 타입의 변수에는 String 이나 null 두 가지 종류의 값이 들어갈 수 있다.

이 두 종류의 값은 서로 완전히 다르고, 서로 실행할 수 있는 연산도 완전히 다르다.

실제 String 이 들어있는 변수는 String 클래스에 정의된 모든 메소드를 호출할 수 있지만, null 인 경우는 사용할 수 있는 연산이 많지 않다.

이는 자바의 타입 시스템이 null을 제대로 다루지 못한다는 뜻이다. 

변수에 선언된 타입(String)이 있지만, null 여부를 추가로 검사하기 전에는 그 변수에 대해 어떤 연산을 수행할 수 있을지 알 수 없다. (NPE 발생 가능) 

자바에서 NPE 문제를 해결하기 위해 @Nullable, @NotNull 과 같은 어노테이션을 활용할 수는 있으나, 이런 도구는 표준 자바 컴파일 절차의 일부가 아니기 때문에 일관성 있게 적용된다는 보장을 할 수 없다.

**코틀린**

코틀린에서는 널 가능 타입과, 널 불가능 타입을 구분하기 때문에 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고, 실행시점에 예외를 발생시킬 수 있는 연산을 금지시킬 수 있다.

***실행시점에 널 가능타입이나 널 불가능타입의 객체는 같다. 모든 검사는 컴파일 시점에 수행되기 때문에, 널가능 타입을 처리하는데 별도의 실행시점 부가비용이 들지 않는다.***

코틀린에서 널 가능 타입을 쉽게 다루는 방법을 알아보자.

### 6.1.3 안전한 호출 연산자  (?.)

코틀린이 제공하는 ?. 연산자는 null 검사와 메소드 호출(혹은 프로퍼티 접근)을 한번의 연산으로 수행한다.

```kotlin
1)
s?.toUpperCase()

2)
if(s != null) s.toUpperCase() else null
```

이 두 가지의 코드는 완전히 동일한 역할을 한다.

호출하려는 값이 null이 아니라면 ?. 연산자는 일반메소드 호출처럼 작동하고, null이면 메소드 호출은 무시되고 null 이 결과 값이 된다.

따라서 안전한 호출(?.) 연산의 결과도 null이 될 수 있는 타입인 것을 유의해야 한다.

안전한 호출(?.)은 연쇄적으로 사용가능하다.

```kotlin
class Address(val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country = this.company?.address?.country   // 연쇄 호출 
    return if(country != null) country else "Unknown"
}
```

코틀린에서는 이렇게 연달은 널 체크를 간결하게 할 수 있다.

맨 마지막 return 문에서 country가 null 인 경우에 "Unknown"을 반환하는데 코틀린을 사용하면 이런 if 문을 제거할 수 있다. 

이와 관련한 엘비스 연산자를 바로 이어서 보자.

### 6.1.4 엘비스(elvis) 연산자 (?:)

코틀린에서 제공하는 엘비스(elvis) 연산자는 null 대신 사용할 디폴트 값을 지정할 때 사용할 수 있는 연산자이다.

![No image](/assets/posts/20220528/Untitled.png)

*엘비스 프레슬리(Elvis Presley) 특유의 헤어스타일과 비슷하다고 해서 이름이 붙여졌다.*

*더 심각한 이름을 좋아하는 사람을 위해 널 복합(null coalescing) 연산자라는 이름도 있다.*

```kotlin
fun foo(s: String?) {
    val t: String = s ?: ""
}
```

이 연산자는 이항 연산자로 좌항(변수 s)을 계산한 값이 널인지 검사하고

좌항이 널이 아니라면 좌항 값을 결과로, 널이라면 우항 값을 결과로 반환한다.

엘비스 연산자를 안전한 호출(?.) 연산자와 함께 사용하는 경우가 많다.

코틀린에서는 return 이나 throw 등의 연산도 식이다. 따라서 엘비스 연산자 우항에 return , throw 등의 연산도 넣을 수 있다. 

```kotlin
// 엘비스 적용 전
fun Person.countryName(): String {
    val country = this.company?.address?.country   // 연쇄 호출  
    return if(country != null) country else "Unknown"
}

// 엘비스 적용 후
fun Person.countryName(): String = company?.address?.country ?: "Unknown"

// 엘비스 적용 후 throw
fun Person.countryName(): String = company?.address ?: throw IllegalArgumentException("No address"
```

### 6.1.5 안전한 캐스트 (as?)

코틀린에는 타입 캐스트 연산자인 as 가 있다. as로 지정한 타입으로 바꿀 수 없으면 `ClassCastException` 이 발생한다. 

as 연산자를 사용하기 전에 is 연산자(자바의 instanceof)를 통해 미리 as 연산자로 변환 가능한 타입인지 체크해볼 수 있다.

간결한 언어를 지향하는 코틀린에서는 더 나은 해법으로 as? 연산자를 지원한다.

안전한 캐스트(as?) 연산자는 지정한 타입으로 캐스팅 시도 하는데, 만약 변환할 수 없으면 null을 반환한다.

안전한 캐스트 연산자(as?)와 엘비스 연산자(?:)를 같이 쓰는 패턴이 많다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        // 타입이 서로 일치 하지 않으면 false를 반환한다
        val otherPerson = o as? Person ?: return false  
        
        // 안전한 캐스트를 하고 나면 otherPerson이 Person타입으로 스마트 캐스트된다
        return otherPerson.firstName == firstName 
                && otherPerson.lastName == lastName        
    }
}
```

### 6.1.6 널 아님 단언 (!!)

널 아님 단언(not-null assertion)은 코틀린에서 널이 될 수 있는 타입인 어떤 값을 강제로 널이 될 수 없는 타입으로 바꿀 때 사용한다.

실제 null (변수가 null인 경우도 포함)에 대해 `!! 연산자`를 적용하면 NPE가 발생한다.

```kotlin
1  fun ignoreNulls(s: String?) {
2      val sNotNull: String = s!!  // 예외는 이 지점을 가리킨다.
3      println(s.length)
4  }

>>> ignoreNulls(null)
Exception in thread "main" java.lang.NullPointerException
	at chapter6.SafeOperationKt.ignoreNulls(SafeOperation.kt:2)
```

예외는 `s.length` 를 사용하는 라인이 아닌 단언문(!!)이 있는 위치를 가리킨다는 점을 유의하자.

근본적으로 `!!` 연산자는 컴파일러에게 "나는 이 값이 null이 아님을 잘 알고 있다. 내가 잘못 생각했다면 예외가 발생해도 감수하겠다"라고 말하는 것이다.

`!!` 연산자로 인한 예외의 stack trace에는 어떤 파일의 몇 번째 줄인지에 대한 정보는 들어있지만 어떤식에서 예외가 발생했는지에 대한 정보는 들어있지 않다. 따라서 어떤 값이 null이었는지 확실히 하기 위해 여러 `!!` 연산자를 한 줄에 쓰는 습관은 피하는게 좋다.

```kotlin
person.company!!.address!!.country // 이런식의 코드는 어떤 값으로부터 NPE가 발생했는지 추적이 어렵다.
```

**노트**

 `*!!` 연산자는 마치 컴파일러에게 소리를 지르는 것 같은 느낌이 든다. 사실 이는 의도한 것이다. 코틀린 설계자들은 컴파일러가 검증할 수 없는 단언(!!)을 사용하기보다는 더 나은 방법을 찾아보라는 의도를 넌지시 표현하려고 `!!` 라는 못생긴 기호를 선택한 것이다.*

### 6.1.7 let 함수

let 함수가 없다면 인자를 넘기기 전에 주어진 값이 널인지 검사를 해야 한다.

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array<String>) {
    var email: String? = "aa@naver.com"
    if(email != null) sendEmailTo(email) // null check 
}
```

let 함수는 null이 될 수 있는 값을 안전한 호출 연산자(?.)와 함께 사용하여 null이 아닌 값만 인자로 받는 함수에 넘기는 경우에 자주 사용 한다.

let 함수는 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다. 

만약 let 함수의 수신객체가 null 이라면 절대 람다가 실행되지 않는다.

let 함수는 람다를 수행한 결과를 리턴한다. 여러 줄이라면 마지막 줄의 결과가 리턴된다.

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array<String>) {
    var email: String? = "aa@naver.com"
    email?.let { sendEmailTo(it) }  // let 사용 
}
```

여러 값이 널인지 검사해야 한다면 let 호출을 중첩시켜 처리할 수 있으나 코드가 복잡해진다.

이런 경우 일반적인 if를 사용해 모든 값을 한꺼번에 검사하는 편이 낫다.

### 6.1.8 나중에 초기화할 프로퍼티 (lateinit)

코틀린에서는 클래스의 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화할 수 없다.

프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다.

초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용해 null로 초기화할 수 밖에 없다. 그러다보면 모든 프로퍼티 접근마다 널 체크를 넣거나 `!!` 연산자를 써야 한다.

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null // null로 초기화하기 위해 nullable 타입으로 선언해야함

    @Before
    fun setUp() { myService = MyService() }  // setUp 메소드 안에서 진짜 초기값을 지정함 

    @Test
    fun testAction() {
        Assert.assertEquals("foo", myService!!.performAction()) // 매번 널 가능성을 신경써주어야 함 
    }
}
```

코틀린에서는 이 문제를 해결하기 위해 `lateinit` 변경자를 제공한다.

`lateinit` 변경자를 붙이면 프로피터를 나중에 초기화할 수 있으며, 나중에 초기화하는 프로퍼티는 항상 `var` 여야 한다.

`val` 프로퍼티는 final 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 한다.

프로퍼티를 초기화하기 전에 접근하면 `lateinit property myService has not been initialized` 예외가 발생한다.

`lateinit` 프로퍼티를 의존관계 주입(DI) 프레임워크와 함께 사용하는 경우가 많다. 그런 시나리오에서는 lateinit 프로퍼티의 값을 DI 프레임워크가 외부에서 설정해준다.

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    @Autowired
    private lateinit var myService: MyService // lateinit 사용 

    @Before
    fun setUp() { myService = MyService() }

    @Test
    fun testAction() {
        Assert.assertEquals("foo", myService.performAction()) // null 체크 필요 없음
    }
}
```

### 6.1.9 널이 될 수 있는 타입 확장

널이 될 수 있는 타입에 대한 확장 함수를 정의하면 null 값을 다루는 강력한 도구로 활용할 수 있다.

String의 `isNullOrEmpty` 혹은 `isNullOrBlank` 등의 널이 될 수 있는 타입에 대한 확장함수를 보자.

```kotlin
fun String?.isNullOrBlank(): Boolean = 
  this == null || this.isBlank()
```

String? 타입은 널이 가능한 타입이기 때문에, 이 확장함수의 내부에서 this는 널이 될 수 있다.

자바에서 메소드 안의 this는 항상 객체 자신을 가리키므로 항상 널이 아니다. (자바에서 수신 객체가 null이 었다면 NPE가 발생해서 메소드 안으로 들어가지도 못한다)

코틀린에서는 이렇게 확장함수 안에서의 this는 null이 될 수 있다. 

따라서 널이 될 수 있는 타입에 대한 확장함수를 잘 정의하면 객체를 사용할때마다 널 체크를 일일히 해야하는 수고를 덜 수 있다. (확장함수를 통해 공용으로 사용하는 널체크를 만듬)

### 6.1.10 타입 파라미터의 널 가능성

코틀린에서는 함수나 클래스의 모든 타입 파라미터(T)는 기본적으로 널이 될 수 있다.

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode())  // "t"가 null이 될 수 있으므로 안전한 호출을 써야만 한다.
}

>>> printHashCode(null)
null
```

여기서 타입 파라미터 T에 대해 추론한 타입은 널이 될 수 있는 Any? 타입이다.

T에는 물음표가 붙어있지 않지만 t는 null을 받을 수 있다. 

**타입 파라미터(T)는 코틀린에서 널이 가능한 타입에는 반드시 물음표를 붙여야 한다는 규칙의 유일한 예외다.**

타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한(upper bound)를 지정해야 한다.

```kotlin
fun <T: Any> printHashCode(t: T) {  // T: Any 로 타입상한을 지정해서 null이 될 수 없음을 명시한다.
    println(t?.hashCode())
}

>>> printHashCode(null)
Error: Type paramter bound for `T` is not satisfied
```

### 6.1.11 널 가능성과 자바

자바는 널 가능성 타입을 지원하지 않는다. 다만 자바 코드에 @Nullable , @NotNull 등의 어노테이션이 있으면 코틀린도 그 정보를 활용한다.

코틀린에서 볼 때 자바의 @Nullable String 은 String? 과 같고, @NotNull String 은 String 타입과 같다.

이런 널 가능성 어노테이션이 소스코드에 없는 경우 자바의 타입은 코틀린의 `플랫폼 타입(platform type)` 이 된다. (= 기본적으로 자바의 타입은 코틀린에서 플랫폼 타입이다)

**플랫폼 타입**

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다.

플랫폼 타입은 nullable 타입으로 처리해도 되고 non-null 타입으로 처리해도 된다. 이 타입에 대해 수행하는 모든 연산은 모두 개발자 책임이라는 뜻이다.

컴파일러는 모든 연산을 허용한다.

코틀린은 보통 널이 될 수 없는 타입에 대해 널 체크를 진행하면 중복 검사라고 경고를 표시하지만, 플랫폼 타입의 값에 대해서는 널 체크 중복에 대한 아무 경고도 표시하지 않는다.

플랫폼 타입에 대해 널이 될 수 있음에도 불구하고 개발자가 널 체크를 하지 않고 사용한다면 자바와 마찬가지로 NullPointerException이 발생한다.

자바로 선언된 Person 클래스가 있다.

```java
public class Person {
    private final String name;
    
    public Person(String name) { this.name = name; }
    
    public String getName() { return name; }
}
```

코틀린에서는 getName()이 null을 리턴할지 String 을 리턴할지 알지 못한다. 따라서 개발자가 판단해서 처리해야한다.

```kotlin
fun yellAt(person: Person) {
    println(person.name.toUpperCase() + "!!!")
}
>>> yellAt(Person(null))
NullPointerException

fun yellAtSafe(person: Person) {
    println( (person.name ?: "Anyone").toUpperCase() + "!!!")
}
>>> yellAtSafe(Person(null))
ANYONE!!!
```

코틀린에서 플랫폼 타입을 직접 선언할 수는 없다. 자바 코드에서 가져온 타입만 플랫폼 타입이 된다.

하지만, IDE나 컴파일 오류 메시지에서는 플랫폼 타입을 볼 수 있다.

```kotlin
Type mismatch: inferred type is String! but Int was expected
```

컴파일러가 표시한 `String!` 타입은 자바코드에서 온 플랫폼 타입이다. 여기서 느낌표(!) 표기는 널 가능성에 대해 아무 정보도 없다는 뜻이다. 

따라서 아래 두 선언은 모두 올바른 선언이 된다. (온전한 개발자의 판단과 책임) 

```kotlin
val s1: String? = person.name
val s2: String = person.name
```

## 6.2 코틀린의 원시 타입

### 6.2.1 원시 타입: Int, Boolean 등

자바는 원시 타입 변수에는 값이 직접 들어가고, 참조 타입 변수에는 메모리상의 주소가 들어가는데 이처럼 원시타입과 참조 타입을 구분한다.

자바는 컬렉션에 원시 타입 값을 담을 수 없기 때문에 참조 타입이 필요한 경우, wrapper 타입(Integer 등)으로 원시 타입 값을 감싸서 사용한다.

반대로 코틀린은 원시 타입과 wrapper 타입을 구분하지 않고 항상 같은 타입을 사용한다.

원시 타입과 참조 타입이 같다면 코틀린은 항상 객체로 표현하는 걸까 ? 정답은 그렇지 않다.

실행 시점에 숫자 타입은 가능한 효율적인 방식으로 표현된다.

대부분의 경우 (변수, 프로퍼티, 파라미터, 반환 타입 등) 코틀린의 Int 타입은 자바 int 타입으로 컴파일된다.

컬렉션과 같은 제네릭 클래스를 사용하는 경우에는 코틀린의 Int 타입은 Integer 타입으로 컴파일된다.

### 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등

널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없다. 

따라서 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 자바의 wrapper 타입으로 컴파일된다.

```kotlin
data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if(age == null || other.age == null) 
            return null
        return age > other.age
    }
}
```

여기서 `age: Int?` 프로퍼티는 null이 될 수 있으므로 null 체크를 진행한 후에 일반적인 값처럼 다룰 수 있다.

null 타입을 허용하기 때문에 원시타입 int가 아닌 wrapper 타입인 Integer 타입으로 컴파일된다.

### 6.2.3 숫자 변환

코틀린과 자바의 큰 차이점 중 하나는 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환 하지 않는다.

코틀린은 직접 변환 메소드를 호출해주어야 한다. 

코틀린은 Boolean을 제외한 모든 원시 타입에 대한 변환 함수를 제공한다.

예시로, 작은 범위의 숫자(Int)를 넓은 범위의 숫자(Long)로 자동 변환하는 것도 불가능하다. (자바는 자동 변환)

```kotlin
val i: Int = 1

val l: Long = i // type mismatch compile error
val l2: Long = i.toLong() // compile ok 
```

이런 wrapper 타입의 비교에서는 타입을 명시적으로 변환해서 같은 타입으로 만든후 비교해야 한다.

```kotlin
val x: Int = 1
val list = listOf(1L, 2L)

println("r1 : ${list.contains(x)}")  // false 
println("r2 : ${list.contains(x.toLong())}") // true
```

### 6.2.4 Any, Any? : 최상위 타입

자바에서 Object는 최상위 타입이듯 코틀린에서는 `Any타입`이 널이 될 수 없는 타입들의 최상위 타입이다.

자바에서는 참조 타입만 최상위 계층을 Object로 보며 원시 타입은 이런 계층이 있지 않다.

코틀린에서는 Any가 Int 등의 원시 타입을 포함한 모든 타입의 조상 타입이 된다.

자바의 Object를 코틀린에서 받아오면 플랫폼 타입인 Any! 로 취급한다. (개발자 판단으로 널 가능하게 할지 불가능하게 할지 선택 가능)

### 6.2.5 Unit 타입 : 코틀린의 void

코틀린의 Unit 타입은 자바 void와 같은 기능을 한다. 

아래 두 함수는 동일한 함수다.

```kotlin
fun f(): Unit { ... }

fun f() { ... } // Unit 생략가능
```

그럼 자바의 void와의 차이점은 무엇일까 ?

코틀린의 Unit은 일반적인 타입으로 다뤄지며, 자바의 void와 달리 타입 인자로 쓸 수 있다.

이 특성은 제네릭 파라미터를 반환하는 함수를 오버라이드할때 유용하게 쓰인다.

```kotlin
interface Processor<T> {
    fun process(): T
}

class NoResultProcessor: Processor<Unit> {
    override fun process(): Unit { // Unit 을 생략해도 된다
        // 비즈니스 로직 작성
        // 여기에선 return 을 명시할 필요가 없다
    }
}
```

자바에선 위와 같은 문제를 Void wrapper 타입을 써서 해결할 순 있으나 항상 return null 을 명시해주어야 한다.

```java
public interface Processor<T> {
    public T process();
}

public class ProcessorImpl implements Processor {
    @Override
    public Void process() {
        // 비즈니스 로직 작성
        return null;  // Void 타입에 대응할 수 있는 유일한 값인 null을 무조건 반환해야 함
    }
}
```

### 6.2.6 Nothing 타입 : 이 함수는 결코 정상적으로 끝나지 않는다.

fail 만을 던지는 메소드, 무한루프를 돌 수 밖에 없는 메소드 등에서는 결코 메소드가 정상적으로 종료되지 않는다.

이런 경우를 표현하기 위해 코틀린에는 Nothing 이라는 반환 타입을 제공한다.

Nothing 타입은 아무 값도 포함하지 않는다. 따라서 Nothing은 함수의 반환 타입이나, 반환 타입으로 쓰일 타입 파라미터(R)로만 쓸 수 있다.

```kotlin
// Nothing Class 
public class Nothing private constructor()
```

```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
    // 정상종료하는 코드를 작성하는 경우 컴파일 에러 발생
}

>>> fail("Error Occurred")
java.lang.IllegalStateException: Error Occurred
```

Nothing의 장점 중 하나는 컴파일러가 Nothing을 반환하는 함수는 결코 정상 종료되지 않음을 알고 코드 분석에 사용한다는 점이다. 

```kotlin
class Address(val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company)

fun fail(msg: String) : Nothing {
    throw IllegalStateException(msg)
}

fun main(args: Array<String>) {
    val p = Person("p_name", Company("company_name", Address("country_name")))

    // address가 null이면 fail 메소드에서 Nothing을 반환(비정상종료)하는 것을 컴파일러가 알고 있음.
    val addr = p.company.address ?: fail("No Address")

    // 위의 코드가 통과되면 addr은 무조건 null이 아님을 컴파일러가 추론할 수 있음. addr에 대한 널 체크 필요 없음.
    println(addr.country)
}
```

## 6.3 컬렉션과 배열

### 6.3.1 널 가능성과 컬렉션

컬렉션 안에 널 값을 넣을 수 있는지 여부도, 타입 인자에 물음표(?)를 붙이면 널 저장이 가능하단 의미다.

중요한 것은 컬렉션 안의 값이 널이 될 수 있는지, 컬렉션 자체가 널이 될 수 있는지를 유의해야 한다는 것이다.

```kotlin
val list1: List<Int?>    // 컬렉션 안의 값만 null이 될 수 있고, 컬렉션 자체는 null 불가
val list2: List<Int>?    // 컬렉션 안의 값은 null이 될 수 없고, 컬렉션 자체는 null 가능
val list3: List<Int?>?   // 둘다 null 이 될 수 있음
```

### 6.3.2 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션과 자바 컬렉션을 나누는 가장 중요한 특성은 코틀린에서는 읽기전용 컬렉션과 읽기/쓰기용 컬렉션을 분리했다는 점이다.

`kotlin.collections.Collection` 인터페이스는 데이터를 읽기만 가능하고 원소 추가/삭제는 불가하다.

`kotlin.collections.MutableCollection` 인터페이스는 읽기, 추가/삭제 등 모두 가능하다. 이 인터페이스는 Collection 인터페이스를 상속한다.

주의할 점은 읽기 전용 컬렉션이라고해서 꼭 변경 불가능한 컬렉션일 필요(?)는 없다는 점이다.

읽기 전용 컬렉션과 변경가능 컬렉션이 같은 컬렉션 객체를 가리키고 있을 수 있다.

```kotlin
val list1: MutableList<Int> = mutableListOf(1,2,3)
val list2: List<Int> = list1
val list3: MutableList<Int> = list1

list3.add(4)
println(list1)  // 1, 2, 3, 4
println(list2)  // 1, 2, 3, 4
println(list3)  // 1, 2, 3, 4
```

이런 경우 list3 에서 값을 추가하면 list2는 읽기전용 컬렉션임에도 값이 변경되는 것을 알 수 있다.

이처럼 list2가 읽기전용인줄 알고 사용하고 있는 도중에, 쓰기가 가능한 list3를 통해 list2가 가리키는 내용이 달라질 수 있다. 따라서 읽기 전용 컬렉션이라고 해서 항상 Thread-Safe 하지는 않다는 점을 명심해야 한다.

### 6.3.3 코틀린 컬렉션과 자바

모든 코틀린 컬렉션은 그에 상응하는 자바 컬렉션 인터페이스의 인스턴스이다. 

자바의 컬렉션이 코틀린의 변경 가능 인터페이스를 확장한다. 

![No image](/assets/posts/20220528/Untitled1.png)

코틀린의 변경 가능한 컬렉션들이 자바의 컬렉션보다 상위 타입 이고, 코틀린의 읽기 전용 컬렉션을 그보다 한 단계 더 상위 타입으로 두어 변경가능 메소드들을 제거 했다.

이런 방식으로 코틀린은 자바 호환성을 제공하는 한편 읽기 전용 인터페이스와 변경 가능 인터페이스를 분리했다.

문제는 코틀린과 자바 코드가 섞여있는 환경에서의 컬렉션이다.

코틀린에서 생성한 읽기전용 컬렉션을 자바로 넘겼을 때, 자바에서 이 컬렉션을 변경하지 못하게 막을 방법이 없다는 것이다. 

비슷한 맥락으로 원소가 널이 될 수 없는 컬렉션을 자바로 넘기더라도 자바에서 원소로 null을 넣지 못하게 막을 방법이 없다.

따라서 자바 쪽에서 컬렉션을 변경할 여지가 있다면 아예 코틀린 쪽에서도 변경 가능한 컬렉션 타입을 사용해서 자바 코드 수행 후 컬렉션 내용이 변할 수 있음을 코드에 남겨둬야 한다.

### 6.3.4 컬렉션을 플랫폼 타입으로 다루기

자바 코드에서 정의한 타입은 코틀린에서 플랫폼 타입으로 본다고 위에서 설명했다.

플랫폼 타입의 경우 코틀린 쪽에서 널이 될 수 있는지 없는지 알 수 있는 방법이 없다.

따라서 개발자의 판단으로 널이 될 수 있게 하던지 없게 하던지 선택할 수 있다. 

컬렉션도 마찬가지로 null 판단과 추가로 읽기 전용 컬렉션으로 볼지, 변경 가능 컬렉션으로 볼지도 개발자가 선택해야 한다.

코틀린에서 자바의 컬렉션을 파라미터로 받아와서 사용한다면 대부분 원활하게 동작한다.

문제는, 컬렉션 타입이 시그니처로 들어가 있는 자바의 메소드를 오버라이드 하는 경우는 고려해야할게 많아진다.

```java
// 자바 Interface
interface Processor {
    void process(List<String> textContents);
}
```

이런 자바 인터페이스를 코틀린에서 구현해야할 때, 파라미터인 List<String> 에 대해 개발자 스스로 판단해서 오버라이드 해야한다.

자바에서는 단순히 List<String> 하나면 충분하지만 코틀린에서는 그렇지 않다. 자바 인터페이스가 어떤 맥락에서 사용되는지 정확히 파악한 후에 선택해야 한다.

- 컬렉션이 null이 될 수 있는가 ?
- 컬렉션의 원소가 null이 될 수 있는가?
- 오버라이드하는 메소드가 컬렉션을 변경할 수 있는가?

이러한 고민에 따라 코틀린에서 오버라이드한 메소드는 여러가지 시그니처를 가질 수 있게 된다. 순전히 개발자 몫이다.

```kotlin
class ProcessorImpl : Processor {
    override fun process(textContents: List<String>?) { ... }
//    override fun process(textContents: List<String?>?) { ... }
//    override fun process(textContents: MutableList<String>?) { ... }
//    등등
}
```

### 6.3.5 객체의 배열과 원시 타입의 배열

코틀린 배열(Array)은 타입 파라미터를 받는 클래스다. 배열의 원소 타입은 그 타입 파라미터에 의해 정해진다.

코틀린 Array 클래스는 일반 제네릭 클래스처럼 생겼지만 자바의 배열( [] )로 컴파일된다.

**객체 타입 배열**

배열의 타입 인자도 항상 객체 타입이 된다. 

`Array<Int>` 타입의 배열은 boxing된 정수 배열(자바의 Integer[])이 된다.

```kotlin
val a1 = arrayOf(1, 2, 3)         // 원소가 1,2,3 인 배열
val a2 = arrayOfNulls<Int>(3)     // 원소가 모두 null인 사이즈 3 짜리 배열
val a3 = Array<Int>(3) { i -> i}  // 원소가 0,1,2 인 배열
```

**원시 타입 배열**

코틀린에서는 자바의 원시타입 배열(int[])과 같은 것을 위해 원시 타입 배열을 따로 제공한다.

원시타입배열은 각 타입에 대한 특별한 형태로 현된다. (IntArray, ByteArray 등)

```kotlin
val a1 = intArrayOf(0, 0, 0)    // 원소가 0,0,0 인 배열
val a2 = IntArray(3)            // 원소가 0,0,0 인 배열 (null 아님)
val a3 = IntArray(3) { i -> i}  // 원소가 0,1,2 인 배열
```

이미 박싱된 값이 들어있는 배열이 있다면 `toIntArray` 등의 변환 함수를 사용해 박싱하지 않은 값이 들어있는 배열로 변환 가능하다.

컬렉션을 배열로 변환해야할 땐 `toTypedArray` 메소드를 사용하면 배열로 바꿀 수 있다.

코틀린의 배열에서는 컬렉션에서 사용할 수 있는 모든 확장함수도 사용 가능하다. (filter, map 등)

다만 이런 함수가 반환하는 값은 배열이 아니라 리스트라는 점에 유의해야 한다.

![No image](/assets/posts/20220528/Untitled2.png)

## 6.4 요약

- 코틀린은 널이 될 수 있는 타입을 지원해 NullPointerException 오류를 컴파일 시점에 감지할 수 있다.
- 코틀린의 안전한 호출(?.), 엘비스 연산자(?:),  널 아님 단언(!!), let 함수 등을 사용하면 널이 될 수 있는 타입을 간결한 코드로 다룰 수 있다.
- as? 연산자를 사용해 값을 다른 타입으로 변환하거나 변환 불가능한 경우 null 로 처리할 수 있다.
- 자바에서 가져온 타입은 코틀린에서 플랫폼 타입(!)으로 취급된다. 개발자는 플랫폼 타입을 널이 될 수 있는 타입으로도, 널이 될 수 없는 타입으로도 사용할 수 있다. (개발자 판단과 책임)
- 코틀린에서 숫자를 표현하는 타입(Int 등)은 대부분 자바 원시 타입(int)로 컴파일된다.
- null이 될 수 있는 타입(Int? 등)이거나 컬렉션에 저장될 타입은 자바의 wrapper 타입(Integer)으로 컴파일된다.
- Any 타입은 다른 모든 타입의 조상 타입이며, 자바의 Object에 해당한다. Unit은 자바의 void와 비슷하다.
- 정상적으로 끝나지 않는 함수의 반환타입으로는 Nothing 타입을 사용한다.
- 코틀린 컬렉션은 표준 자바 컬렉션 클래스를 사용한다. 하지만 자바보다 개선해서 읽기 전용 컬렉션과 변경 가능 컬렉션으로 구별해서 제공한다.
- 자바 클래스를 확장하거나 자바 인터페이스를 코틀린에서 구현하는 경우 메소드 파라미터의 null 가능성과 변경 가능성에 대해 깊이 생각해야 한다.
- 코틀린의 Array 클래스는 일반 제네릭 클래스처럼 보이지만 자바 배열로 컴파일된다.
- 원시 타입의 배열은 IntArray와 같이 각 타입에 대한 특별한 배열로 표현된다.