---
layout: post
title: "코틀린 DSL"
tags: [Kotlin]
comments: true
date: 2022-05-29
---

# 11장. DSL 만들기

11장에서는 영역 특화 언어 (DSL : Domain-Specific Language)를 사용해 표현력이 좋고 코틀린 다운 API를 설계하는 방법을 설명한다.

## 11.1 API 에서 DSL로

API라 함은 단순히 클래스들간의 상호작용도 포함된다. 

라이브러리를 만드는 사람들에게만 API를 훌륭하게 만들 책임이 있는 것이 아니다.

애플리케이션 안의 모든 클래스는 다른 클래스와 상호작용하는데 이것 또한 API의 한 부분이다.

먼저 API를 깔끔하게 작성할 수 있도록 돕는 코틀린의 기능들을 보자.

| 일반 구문 | 간결한 구문 | 사용한 언어 특성 |
| --- | --- | --- |
| StringUtil.capitalize(s) | s.capitalize() | 확장 함수 |
| 1.to(”one”) | 1 to “one” | 중위 호출 |
| set.add(2) | set += 2 | 연산자 오버로딩 |
| map.get(”key”) | map[”key”] | get 메소드에 대한 관례 |
| file.use({ f → f.read() }) | file.use { it.read() } | 람다를 밖으로 빼내는 관례 |
| sb.append(”yes”) | with (sb) { append(”yes”) } | 수신 객체 지정 람다 |

이번 장에서는 깔끔한 API에서 한걸음 더 나아가 DSL 구축을 도와주는 코틀린 기능을 살펴본다.

코틀린 언어의 다른 특성과 마찬가지로, 코틀린 DSL도 온전히 컴파일 시점에 타입이 정해진다. 따라서 컴파일 시점 오류 감지, IDE 지원 등 모든 정적 타입 지정 언어의 장점을 누릴 수 있다.

### 11.1.1 영역 특화 언어라는 개념

컴퓨터로 풀 수  있는 모든 문제를 충분히 풀 수 있는 기능을 제공하는 언어를 **범용 프로그래밍 언어(ex: Kotlin) 라고 함**

반대로, 특정 과업 또는 영역에 초점을 맞추고 그 영역에 필요하지 않은 기능을 없앤 것을 **영역 특화 언어(DSL) 라고 함**

가장 익숙한 DSL은 SQL 과 정규식이다. 이처럼 DSL은 스스로 제공하는 기능을 제한함으로써 오히려 더 효율적으로 자신의 목표를 달성할 수 있다.

범용 프로그래밍 언어는 보통 `명령적`이며, DSL 은 `선언적`이다. 

선언적 언어는 원하는 결과를 기술하기만 하면, 그 결과를 달성하기 위해 필요한 세부 실행은 실행엔진에게 맡긴다. 실행 엔진이 결과를 얻는 과정을 전체적으로 한번에 최적화하기 때문에 선언적 언어가 더 효율적인 경우가 자주 있다.

DSL의 한가지 단점은, 범용 언어로 만든 호스트 애플리케이션과 DSL을 조합하기가 어렵다는 것이다.

DSL은 자체 문법이 있기 때문에 다른 언어의 프로그램 안에 직접 포함시킬 수 없고, DSL을 호출하기 위한 별도의 방법이 필요하다. 

DSL을 파일로 만들거나, 문자열 리터럴로 저장하거나 하는 등 여러 방법이 있지만 `내부 DSL` 이라는 개념이 유명해지고 있다.

### 11.1.2 내부 DSL

독립적인 문법 구조를 가진 외부 DSL과 반대로 내부 DSL은 범용 언어로 작성된 프로그램의 일부며, 범용 언어와 동일한 문법을 사용한다.

간단히 외부/내부 DSL 을 비교해보자.

외부 DSL로는 SQL을 사용

```sql
SELECT Country.name, COUNT(Customer.id)
   FROM Country
   JOIN Customer
     ON Country.id = Customer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC
   LIMIT 1
```

내부 DSL로는 코틀린으로 작성된 데이터베이스 프레임워크인 익스포즈드(Exposed)가 제공하는 DSL 사용

```kotlin
(Country join Customer)
    .slice(Country.name, Count(Customer.id))
    .selectAll()
    .groupBy(Country.name)
    .orderBy(Count(Customer.id), isAsc = false)
    .limit(1)
```

두 프로그램을 실행하면 같은 작업을 수행한다. 

하지만 두 번째 버전은 일반 코틀린 코드며, selectAll, groupBy, orderBy 등은 일반 코틀린 메소드다.

또한 두 번째 버전에서는 쿼리가 실행한 결과가 네이티브 코틀린 객체이기 때문에, SQL 질의가 돌려주는 결과 집합을 코틀린 객체로 변환하기 위해 노력할 필요가 없다.

내부DSL을 통해 가져온 Country.name, Count(Customer.id) 객체에 대해서 연쇄적으로 메소드 호출 가능.

외부DSL을 통해 가져오면, 일단 코틀린 객체로 한번 리턴받아 놓은 다음에 처리해야함.

### 11.1.3 DSL의 구조

DSL과 일반 API 사이에 잘 정의된 일반적인 경계가 없어 누군 DSL이라 할수도, 누군 API라고 할수도 있다.

API에는 존재하지 않지만 DSL에만 존재하는 특징은 바로 구조 또는 문법이다.

전형적인 라이브러리는 여러 메소드로 되어있고 클라이언트는 그런 메소드들을 한번에 하나씩 호출함으로써 사용한다.

코틀린 DSL 에서는 보통 람다를 중첩 시키거나 메소드 호출을 연쇄시키는 방식으로 구조를 만든다. (예로 위에 작성한 내부 DSL SQL문도 메소드 연쇄 호출)

DSL 구조의 장점은 같은 문맥을 함수 호출시마다 반복하지 않고도 재사용할 수 있다는 점이다.

gradle 에서 의존관계를 정의할때 사용하는 코틀린 DSL 을 보자. 

```kotlin
dependencies {  // 람다 중첩을 통해 구조를 만든다.
   compile("junit:junit:4.11")
   compile("com.google.inject:guice:4.1.0")
}
```

반대로 일반 명령-질의 API 를 통해 같은 의존관계 정의를 작성하면 코드 중복이 많다는 사실을 알 수 있다.

```kotlin
project.dependencies.add("compile", "junit:junit:4.11")
project.dependencies.add("compile", "com.google.inject:guice:4.1.0")
```

### 11.1.4 내부 DSL로 HTML 만들기

여기서 사용할 API는 kotlinx.html 라이브러리에서 가져온 것이다.

cell 이 하나인 표를 만드는 코드를 보자.

```kotlin
fun createSimpleTable() = createHTML().
   table {
     tr {
       td { +"cell" }
     }
   }
```

이 코드는 아래 HTML을 만들어 낸다.

```kotlin
<table>
   <tr>
     <td>cell</td>
   </tr>
</table>
```

그렇다면 직접 HTML 텍스트를 작성하지 않고 코틀린 코드로 만들려는 이유가 뭘까 ?

1) 코틀린 버전은 타입 안정성을 보장한다. `td` 는 `tr` 안에서만 사용가능하고 그렇지 않으면 컴파일되지 않는다.

2) 이 코드가 일반 코틀린 코드이기 때문에, 그 안에서 코틀린 코드를 원하는 대로 사용할 수 있다.

```kotlin
fun createAnotherTable() = createHTML().table {
  val numbers = mapOf(1 to "one", 2 to "two")
  for ((num, string) in numbers) {
    tr {
      td { +"$num" }
      td { +string }
    }
  }
}

// result 
<table>
   <tr>
     <td>1</td>
     <td>one</td>
   </tr>
   <tr>
     <td>2</td>
     <td>tow</td>
   </tr>
</table>
```

## 11.2 구조화된 API 구축: DSL 에서 수신 객체 지정 DSL 사용

### 11.2.1 수신 객체 지정 람다와 확장 함수 타입

먼저 일반 람다를 받는 buildString 함수를 정의해보자.

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit // 함수 타입인 파라미터를 정의
) : String {
    val sb = StringBuilder()
    builderAction(sb)   // 람다 인자로 StringBuilder 인스턴스를 넘긴다
    return sb.toString()
}

fun main(args: Array<String>) {
    val s = buildString {
        it.append("Hello, ")  // it는 StringBuilder 인스턴스를 가리킨다
        it.append("world")
    }
    println(s)
}

>>> Hello, world
```

이 코드는 이해하기 쉽지만 매번 it를 사용해 StringBuilder 인스턴스를 참조해야한다.

수신 객체 지정 람다로 바꾸면 메소드이름 앞에 it를 일일이 넣지 않아도 된다.

수신 객체 지정 람다로 바꾸어보자.

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit  // 수신객체가 있는 함수 타입의 파라미터 선언
) : String {
    val sb = StringBuilder()
    sb.builderAction()    // StringBuilder 인스턴스를 람다의 수신객체로 넘긴다 
    return sb.toString()
}

fun main(args: Array<String>) {
    val s = buildString {
        this.append("Hello, ")  // this 키워드는 StringBuilder 인스턴스를 가리킨다(생략가능)
        append("world")
    }
    println(s)
}
```

변경된 부분을 보면 파라미터 타입을 선언할 때 일반 함수 타입 대신 **확장 함수 타입을 사용했다.**

확장 함수 타입 선언은 람다의 파라미터 목록에 있던 수신 객체 타입을 파라미터 목록을 여는 괄호 앞으로 빼 놓으면서 중간에 마침표를 붙인 형태다

```kotlin
(StringBuilder) -> Unit  // 일반 함수 타입

StringBuilder.() -> Unit  // 확장 함수 타입

// 조금 더 복잡한 확장 함수 타입 
// 수신 객체 타입이 String 이며 파라미터로 두 Int 를 받고 Unit을 반환하는 확장 함수 타입 정의
String.(Int, Int) -> Unit  
```

확장 함수 타입의 변수를 정의할 수도 있다. 

정의한 확장 함수 타입 변수를 마치 확장 함수처럼 호출하거나, 수신 객체 지정 람다를 요구하는 함수에게 인자로 넘길 수 있다.

```kotlin
// appendExcl 은 확장 함수 타입의 값이다
val appendExcl : StringBuilder.() -> Unit = { this.append("!") }

fun main(args: Array<String>) {
    val sb = StringBuilder("Hi")
    sb.appendExcl()  // 확장 함수 처럼 호출 가능 

    println(sb)
    // >> Hi!

    println(buildString(appendExcl))  // 수신객체 지정람다를 요구하는 메소드에 인자로 전달가능
    // >> !
}
```

지금까지 수신객체 지정람다와 확장 함수 타입에 대해 설명했고, 이제 이런 개념을 DSL에서 어떻게 사용하는지 살펴보자.

### 11.2.2 수신 객체 지정 람다를 HTML 빌더 안에서 사용

HTML을 만들기 위한 코틀린 DSL을 보통 HTML 빌더라 부른다. 이러한 코틀린 빌더는 타입 안정성을 보장한다.

```kotlin
fun createSimpleTable() = createHTML().
   table {
     tr {
       td { +"cell" }
     }
   }
```

이 코드는 일반 코틀린 코드지 특별한 템플릿 언어 같은 것이 아니다. 

table, tr, td 는 모두 평범한 함수고 각 함수는 고차 함수로 수신 객체 지정 람다를 인자로 받는다.

tr 함수는 table 함수 안에서만 접근가능 하고, td 함수는 tr 안에서만 접근 가능한 함수다. 

다음 코드는 방금 설명한 클래스와 메소드 정의를 정리한 코드다.

```kotlin
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>() // 모든 중첩 태그를 저장한다.
    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init()         // 자식 태그를 초기화한다.
        children.add(child)  // 자식 태그에 대한 참조를 저장한다.
    }

    // 결과 HTML을 문자열로 반환한다.
    override fun toString() = "<$name>${children.joinToString("")}</$name>"
}

fun table(init: TABLE.() -> Unit) = TABLE().apply(init)

class TABLE : Tag("table") {
    // TR 태그 인스턴스를 새로 만들고 초기화한 다음에 TABLE 태그의 자식으로 등록한다.
    fun tr(init : TR.() -> Unit) = doInit(TR(), init)
}

class TR : Tag("tr") {
    // TD 태그의 새 인스턴스를 만들어서 TR 태그의 자식으로 등록한다.
    fun td(init: TD.() -> Unit) = doInit(TD(), init)
}

class TD : Tag("td")

fun createTable() =
    table {
        tr {
            td {
            }
        }
    }

>>> println(createTable()) 
<table><tr><td></td></tr></table>
```

이렇게 정의된 확장 함수 타입은 각 메소드에 전달할 람다의 수신 객체 타입을 순서대로 TR 과 TD로 지정할 수 있기 때문에 HTML 언어의 문법을 따르는 코드만 작성할 수 있게 된다.

각 태그는 자기 이름을 태그 안에 넣고, 자식 태그를 재귀적으로 문자열로 바꿔서 닫는 태그를 추가하는 방식으로 적절하게 표현된다.

수신 객체 지정 람다를 사용하면 DSL을 만들때 코드 블록 내부에서 이름 결정 규칙을 바꿀 수 있다. (위의 예시에서 table, tr, td 등으로 이름을 결정할 수 있는 것처럼)

### 11.2.3 코틀린 빌더: 추상화와 재사용 가능하게 하는 도구

일반 코드를 작성하는 경우 중복을 피하기 위해 반복되는 코드를 새로운 함수로 묶어서 이해하기 쉬운 이름을 붙일 수 있다.

하지만 외부 DSL인 SQL이나 HTML을 별도 함수로 분리해 이름을 부여하기는 어렵다.

반면에 내부 DSL을 사용하면 일반 코드와 마찬가지로 반복되는 내부 DSL 코드 조각을 새 함수로 묶어서 재사용할 수 있다.

부트스트랩 라이브러리를 사용한 드랍다운 메뉴 예제를 보자.

일반 HTML 버전 

```html
<div class="dropdown">
    <button class="btn dropdown-toggle">
        Dropdown
        <span class="caret"></span>
    </button>
    <ul class="dropdown-menu">
        <li><a href="#">Action</a></li>
        <li><a href="#">Another action</a></li>
        <li role="separator" class="divider"></li>
        <li class="dropdown-header">Header</li>
        <li><a href="#">Separated link</a></li>
    </ul>
</div>
```

코틀린에서 kotlinx.html 라이브러리를 사용한 버전

```kotlin
fun buildDropdown() = createHTML().div(classes = "dropdown") {
    button(classes = "btn dropdown-toggle") {
        +"Dropdown"
        span(classes = "caret")
    }
    ul(classes = "dropdown-menu") {
        li { a("#") { +"Action"} }
        li { a("#") { +"Another action"} }
        li { role = "separator"; classes = setOf("divider") }
        li { classes = setOf("dropdown-header"); +"Header" }
        li { a("#") { +"Separated link"} }
    }
}
```

div, button 등은 모두 일반 함수이기 때문에 반복되는 로직을 별도의 함수로 분리할 수 있다.

```kotlin
fun dropdownExample(): String = dropdown() {
    // dropdownButon { +"Dropdown"}  // dropdownMenu와 비슷한 맥락으로 교재에서 생략함
    dropdownMenu { 
        item("#", "Action")
        item("#", "Another action")
        divider()
        dropdownHeader("Header")
        item("#", "Separated link")
    }
}

fun UL.item(href: String, name: String) = li { a(href) { +name } }

fun UL.divider() = li { role = "separator"; classes = setOf("divider")}

fun UL.dropdownHeader(text: String) = li { classes = setOf("dropdown-header"); +text }

fun DIV.dropdownMenu(block: UL.() -> Unit) = ul("dropdown-menu", block)

fun dropdown(block: DIV.() -> Unit): String = createHTML().div("dropdown", block)

```

`dropdownExample` 함수를 보면 중복되는 불필요한 세부사항이 감춰지고 좀더 코드가 깔끔해졌다. 

item 함수를 보면 파라미터를 2개 받는 함수이고 li 함수를 호출하는 역할을 한다. li 함수가 UL클래스의 확장함수로 이므로 item 함수를 UL클래스의 확장 함수로 만들면 UL클래스에 있는 li 함수를 사용 가능하다. (`kotlinx.html` 라이브러리를 기준으로 한 클래스와 확장함수 간의 설명임)

divider 함수와 dropdownHeader 함수도 item 함수와 마찬가지로 UL 클래스의 확장함수로 만들어 li 함수를 쓰도록 했다.

dropdownMenu 함수는 ul 태그를 만드는데, 인자로 태그 내용을 채워 넣는 수신 객체 지정 람다를 받는다. 따라서 `ul { ... }` 블록을 `dropdownMenu { ... }` 블록으로 바꿀 수 있다.

이런식으로 내부 DSL을 사용하면 추상화와 재사용을 통해 기존 코드를 개선할 수 있다.

## 11.3 invoke 관례를 사용한 더 유연한 블록 중첩

invoke 관례를 사용하면 객체를 함수처럼 호출할 수 있다. 

하지만 이 기능은 일상적으로 사용하라고 만든 기능이 아니라는 점에 유의해야한다. 

invoke 관례를 남용하면 `1()` 과 같이 이해하기 어려운 코드가 생길 수 있다.

```kotlin
operator fun Int.invoke() { println(this) }
1()
>> 1
```

하지만 DSL에서는 invoke 관례가 유용할때가 있는데 알아보자.

### 11.3.1 invoke 관례: 함수처럼 호출할 수 있는 객체

관례는 특별한 이름이 붙은 함수를 일반메소드 호출 구문으로 호출하지 않고 더 간단한 구문으로 호출할 수 있게 지원하는 기능이다.

7장에 나왔던 예시를 다시 보면, `get` 이라는 이름의 함수를 정의하고 `operator` 키워드를 붙이면 get() 메소드로 호출하지 않고 `[ ]` 를 이용한 구문으로 메소드를 호출할 수 있다.

```kotlin
operator fun Point.get(index: Int): Int {
	return when(index) {
		0 -> x
		1 -> y
		else -> throw IndexOutOfBoundException("Invalid coordinate $index")
	}
}

val p = Point(10, 20)
println(p[1]) // p.get(1) 로 호출하지 않고 p[1] 로 호출 가능

>> 20
```

invoke 관례도 마찬가지 역할을 한다. 다만 `invoke`는 `get` 과 달리 각괄호(`[ ]`) 대신 괄호(`()`)를 사용한다.

`operator` 키워드가 붙은 `invoke` 메소드 정의가 들어있는 클래스의 객체를 함수처럼 호출할 수 있다.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {  // Greeter 안에 invoke 메소드 정의
        println("$greeting, $name!")
    }
}

fun main(args: Array<String>) {
    val greeter1 = Greeter("Hello")
    greeter1("Bob")  // Greeter 인스턴스를 함수처럼 호출 
}

>> Hello, Bob!
```

`greeter(”Bob”)` 은 내부적으로 `greeter.invoke(”Bob”)` 로 컴파일 된다. 

invoke 메소드의 시그니처에 대한 요구사항은 없기 때문에 원하는 대로 파라미터 개수나 타입을 지정할 수 있다.

여러 파라미터 타입을 지원하기 위해 invoke 메소드를 오버로딩할 수도 있다.

### 11.3.2 invoke 관례와 함수형 타입

일반적인 람다 호출 방식(람다 뒤에 괄호를 붙이는 방식)이 실제로는 invoke 관례를 적용한 것이다. 

인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스(FunctionN)를 구현하는 클래스로 컴파일된다.

각 함수형 인터페이스 안에는 인터페이스 이름이 가리키는 개수만큼 파라미터를 받는 invoke 메소드가 있다.

```kotlin
interface Function2<in P1, in P2, out R> {
    operator fun invoke(p1: P1, p2: P2) : R
}
```

람다를 함수처럼 호출하면 이 관례에 따라 invoke 메소드 호출로 변환된다.

이런 사실을 알면 복잡한 람다를 여러 메소드로 분리할 수 있다.  (invoke 메소드가 수행될테니 invoke안에서 처리할 로직을 여러메소드를 분리시킬 수 있음)

기존 람다를 여러 함수로 나눌 수 있으려면 함수 타입 인터페이스를 구현하는 클래스를 정의해야 한다.

이때 기반 인터페이스를 `Function<P1, ..., PN, R>` 타입이나 `(P1, ..., PN) → R` 타입으로 명시해야한다. (아래 예시에선 `(Issue) → Boolean` 타입으로 명시)

```kotlin
data class Issue(
    val id: String,
    val project: String,
    val type: String,
    val priority: String,
    val description: String
)

// 함수 타입을 부모 클래스로 사용 (Function1 인터페이스로 컴파일됨)
class ImportantIssuePredicate(val project: String) : (Issue) -> Boolean { 
    override fun invoke(issue: Issue): Boolean {  // invoke 메소드 오버라이드 
        return issue.project == project && issue.isImportant()
    }

    // 람다에서 실행할 로직을 분리 
    private fun Issue.isImportant(): Boolean {
        return type == "Bug" &&
                (priority == "Major" || priority == "Critical")
    }
}

fun main(args: Array<String>) {
    val issue1 = Issue("IDEA-154", "IDEA", "Bug", "Major", "desc1")
    val issue2 = Issue("KT-12", "Kotlin", "Feature", "Normal", "desc2")
    
    val predicate = ImportantIssuePredicate("IDEA")

    for (issue in listOf(issue1, issue2).filter(predicate)) {
        println(issue.id)
    }

//    또는 이렇게 작성해도 동일 
//    for (issue in listOf(issue1, issue2).filter { predicate(it) }) {
//        println(issue.id)
//    }
}
```

위 예시는 술어(predicate)의 로직이 너무 복잡해서 한 람다로 표현하기 어려웠다.

그래서 람다를 메소드로 나누고 메소드에 뜻을 명확히 알 수 있는 이름을 붙인 것이다.

람다를 함수 타입 인터페이스(FunctionN)를 구현하는 클래스로 변환하고 그 클래스의 invoke 메소드를 오버라이드하면 이러한 리팩토링이 가능한 것이다.

```
람다는 익명함수를 대체할 수 있는 코드블럭의 개념

람다는 변수에 할당 가능 => 함수타입 파라미터를 가지는 함수에 람다를 인자로 전달 가능
인라인하는 람다를 제외한 람다는 모두 FunctionN 과 같은 함수형 인터페이스로 컴파일 됨
함수타입 파라미터도 FunctionN 과 같은 함수형 인터페이스로 컴파일됨
FunctionN은 N 개의 파라미터를 갖는 함수형 인터페이스(추상메소드가 하나인 인터페이스)

함수타입 파라미터를 가지는 함수에서 인자로 전달받은 람다를 실행시키면 내부적으로 FunctionN 인터페이스가 가지고 있는 invoke() 메소드가 수행됨
즉, 람다를 함수처럼 호출하면(= 파라미터로 전달받은 람다를 호출하면) 내부적으로 FunctionN의 invoke() 메소드가 수행됨.
(invoke 관례라는 것이 invoke 메소드를 구현한 클래스를, 객체명()으로 호출하면 invoke 메소드가 수행되는 것임)

이런 invoke 관례를 통해서, 함수타입(FunctionN)을 구현(implements)하는 클래스를 작성하면, 복잡한 람다를 여러 메소드로 분리할 수 있다. 
함수타입을 구현한 클래스는 invoke 메소드를 오버라이드 해야하고 invoke 메소드안에는 람다 본문을 작성하면 된다. 
invoke() 메소드안에 람다 본문이 들어가기 때문에 invoke() 메소드에서 복잡한 부분을 다른 메소드로 뺴내고 빼낸 메소드에 적절한 이름을 붙여 람다 본문을 보기 쉽게 작성할 수 있다.

또한, 함수타입을 구현한 클래스를 객체로 생성할 경우에, 일반람다를 어떤 변수에 넣는것과 동일한 문맥으로 외부에서 호출가능한 객체를 만들 수 있다. 
함수타입을 파라미터로 받는 함수에(=람다를 필요로하는 곳) 인자로 함수타입을 구현한 클래스(= invoke()메소드가 있는 클래스)의 객체 자체를 람다처럼 넘길 수 있음(invoke() 메소드가 람다의 본문)
쉽게 말해 함수타입을 구현한 클래스로 객체를 생성하면 그 객체자체가 람다가 될수있다(invoke() 메소드 안에있는 부분이 람다 본문이 됨).

람다를 실행하는것은 결국 함수형 인터페이스(FunctionN)의 invoke() 메소드를 수행하는 것이다.
따라서 함수형 인터페이스(=함수타입)를 implements 하는 클래스에서 invoke()를 구현하고 실행하면 결국 invoke() 메소드 본문내용이 람다의 본문이 되는것이다.
객체.invoke() 수행은 너무 길기 때문에 객체()로 호출 가능한 것이 invoke의 관례이다.
```

### 11.3.3 DSL의 invoke 관례: 그레이들에서 의존관계 정의

앞에서 봤던 gradle 코드를 다시보자.

```kotlin
dependencies {
   compile("junit:junit:4.11")
   compile("com.google.inject:guice:4.1.0")
}
```

위에 코드처럼 중첩된 블록구조를 허용하면서, 넓게 펼쳐진 형태의 함수 호출구조도 제공하는 API를 만들고 싶다.

쉽게 말해 아래 처럼 하나의 API에 대해 2가지 형식을 모두 지원하고 싶다. (유연성을 위해)

```kotlin
// 일반 API 호출
dependencies.compile("junit:junit:4.11")

// DSL API (invoke) 호출
dependencies {
   compile("junit:junit:4.11")
}
```

dependencies 객체는 `DependencyHandler` 클래스의 인스턴스다. `DependencyHandler` 안에는 `compile` 과 `invoke` 메소드 정의가 들어있다. 

`DependencyHandler` 의 구현을 간략화해서 보면 아래와 같다. 

꽤 적은 양의 코드지만 이렇게 정의한 invoke 메소드로 인해 DSL API의 유연성이 훨씬 커질 수 있다.

```kotlin
class DependencyHandler {
    fun compile(coordinate: String) {  // 일반적인 명령형 API 정의
        println("Added dependency on $coordinate")
    }

    operator fun invoke(body: DependencyHandler.() -> Unit) {  // "invoke"를 정의해 DSL 스타일 API 제공 
        body()
    }
}

fun main(args: Array<String>) {
    val dependencies = DependencyHandler()

    dependencies.compile("kotlin-stdlib:1.0.0") // 첫번째 호출
    
    dependencies {
        compile("kotlin-reflect:1.0.0") // 두번째 호출 
    }
//    내부적으로는 이렇게 호출된다.
//    dependencies.invoke({ 
//        this.compile("kotlin-reflect:1.0.0")
//    })
}

>> Added dependency on kotlin-stdlib:1.0.0
>> Added dependency on kotlin-reflect:1.0.0
```

## 11.4 실전 코틀린 DSL

### 11.4.1 중위 호출 연쇄: 테스트 프레임워크의 should

KotlinTest 라이브러리의 DSL에서 중위 호출 활용방법을 살펴본다.

str 에 들어있는 값이 “kot”로 시작하지 않으면 이 단언문은 실패하는 테스트이다. 

```kotlin
// KotlinTest
str should startWith("kot")

// JUnit
assertTrue(str.startsWith("kot"))
```

이러한 기능을 하는 KotlinTest의 간략한 구현은 이렇다.

```kotlin
interface Matcher<T> {
    fun test(value: T)
}

// 평범한 클래스라면 첫글자가 대문자일텐데, DSL 에서는 그런 일반적인 명명 규칙을 벗어나야할 때가 있음.
class startWith(val prefix: String) : Matcher<String> {
    override fun test(value: String) {
        if(!value.startsWith(prefix))
            throw AssertionError("String $value does not start with $prefix")
    }
}

// infix 키워드 필요
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)
```

### 11.4.2 원시 타입에 대한 확장 함수 정의: 날짜 처리

java.time API와 코틀린을 사용해 날짜를 조작하는 DSL을 정의해본다.

```kotlin
val Int.days: Period  // Int 타입의 확장 프로퍼티인 days 정의 (Period 반환)
   get() = Period.ofDays(this)

val Period.ago: LocalDate  // Period 의 확장 프로퍼티 정의 
   get() = LocalDate.now() - this  // 뺄셈은 LocalDate의 minus 메소드 호출

val Period.fromNow: LocalDate
   get() = LocalDate.now() + this

fun main(args: Array<String>) {
    println(1.days.ago)      // 현재기준 1일 전
    println(1.days.fromNow)  // 현재기준 1일 후 
}

// now = 2022-01-13
>> 2022-01-12  // 1.days.ago
>> 2022-01-14  // 1.days.fromNow
```

### 11.4.3 멤버 확장 함수: SQL을 위한 내부 DSL

`클래스 안`에서 확장 함수와 확장 프로퍼티를 선언하는 것을 **멤버 확장(member extensions)** 이라 부른다.

멤버 확장을 사용하는 예시로 익스포즈드 프레임워크에서 제공하는 SQL을 위한 내부 DSL을 보자.

```kotlin
object Country : Table() {
    val id = integer("id").autoIncrement().primaryKey()
    val name = varchar("name", 50)
}
```

이 선언은 데이터베이스 테이블이 대응된다. 익스포즈드 프레임워크의 `SchemaUtils.create(Country)` 메소드를 호출하면 테이블이 생성된다.

```sql
CREATE TABLE IF NOT EXISTS Country (
  id INT AUTO_INCREMENT NOT NULL,
  name VARCHAR(50) NOT NULL,
  CONSTRAINT pk_Country PRIMARY KEY (id)
)
```

익스포즈드 프레임워크의 Table 클래스는 데이터베이스 테이블에 대해 정의할 수 있는 모든 타입을 정의한다.

```kotlin
class Table {
   fun integer(name: String) : Column<Int>
   fun varchar(name: String, length: Int) : Column<String>
   // ... 
}
```

autoIncrement 나 primaryKey 같은 메소드를 사용해 각 칼럼의 속성을 지정하는데, 이 때 **멤버 확장**이 쓰인다.

Column 클래스에 대해 이런 메소드를 호출할 수 있다. 각 메소드는 자신의 수신 객체를 다시 반환하기 때문에 메소드 연쇄 호출이 가능하다.

```kotlin
object Country : Table() {
    val id = integer("id").autoIncrement().primaryKey()
    val name = varchar("name", 50)
}

class Table {
    fun <T> Column<T>.primaryKey(): Column<T>
    fun Column<Int>.autoIncrement(): Column<Int>
    // ...
}

// 클래스 밖에서 확장함수를 정의하면 아무데서나 호출 가능 (= 일반확장)
// fun Column<Int>.autoIncrement(): Column<Int>
```

primaryKey, autoIncrement 메소드는 Table 클래스의 멤버다. 따라서 Table 클래스 밖에서 이 함수들을 호출할 수 없다. (Table 이라는 맥락이 없으면 칼럼의 프로퍼티 정의는 의미가 없음) 

Table 클래스 내부가 아닌 밖에서, Column 클래스에 대해 일반 확장 함수를 정의하면 Table 밖에서도 호출 가능.

이렇게 멤버 확장으로 정의하는 이유는 메소드가 적용되는 범위를 제한하기 위함이다. 

여기서는 추가적으로 확장 함수의 수신객체타입을 제한하는 기능을 볼 수 있다. 자동증가칼럼(autoIncrement)는 Int 타입만 가능하도록 타입을 제한했다.

아래 예시는  질의는 미국에 사는 모든 고객의 이름을 출력한다.

```kotlin
val result = (Country join Customer)
    .select { Country.name eq "USA" } // WHERE Country.name = "USA" 라는 SQL에 해당

result.forEach { println(it[Customer.name]) }
```

eq 메소드는 중위표기법으로 적었다는 사실과 또 다른 멤버 확장임을 알 수 있다.

eq 메소드는 Column에 대한 확장 함수이고 SqlExpressionBuilder 클래스 내부에 정의된 멤버 확장이기 때문에 SqlExpressionBuilder 클래스 밖에서는 사용 불가능하다.

```kotlin
fun Table.select(where: SqlExpressionBuilder.() -> Op<Boolean>) : Query

object SqlExpressionBuilder {
    infix fun<T> Column<T>.eq(t: T) : Op<Boolean>
}
```

## 11.5 요약

- 내부 DSL은 여러 메소드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 있는 패턴이다
- 수신 객체 지정 람다는 람다 본문 안에서 메소드를 결정하는 방식을 재정의함으로써 여러 요소를 중첩시킬 수 있는 구조를 만들 수 있다
- 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수 타입이다. 람다를 파라미터로 받아서 사용하는 함수는 람다를 호출하면서 람다에 수신 객체를 제공한다
- 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면 코드를 추상화하고 재활용할 수 있다
- 중위 호출 인자로 특별히 이름을 붙인 객체를 사용하면 특수 기호를 사용하지 않는 실제 영어처럼 보이는 DSL을 만들 수 있다
- 원시 타입에 대한 확장을 정의하면 날짜 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다
- invoke 관례를 사용하면 객체를 함수처럼 다룰 수 있다
- KotlinTest 라이브러리는 단위 테스트에서 읽기 쉬운 단언문을 지원하는 내부 DSL을 제공한다
- 익스포즈드 라이브러리는 데이터베이스를 다루기 위한 내부 DSL을 제공한다.