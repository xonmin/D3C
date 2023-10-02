## Enum values() 

### 기존 Enum values 의 문제

Java 내  `values()`  ⇒ [API design Bug](https://mail.openjdk.org/pipermail/compiler-dev/2018-July/012242.html)

### 1. 호출시, 새로운 Array instance 생성

- `values()`  호출시, 내부에서 static 으로 만들어두고 활용하지 x,
열거형 항목들을 새로운 Array 에 반환
- 다음과 같은 테스트는 통과

```kotlin
@Test
fun enumTest() {
    val brands1 = Brand.values()
    val brands2 = Brand.values()

    println(brands1)    // @478ee483
    println(brands2)    // @1a7288a3
    assertThat(brands1 == brands2).isFalse
}
```

Enum 열거형 데이터는 동적으로 추가되지 않기에 굉장히 비효율적

- 만약 `values()` 를 집약적인(한 두) 곳에서 static 같은 한 번만 실행되는 곳에서 수행하거나 실행이 잦지 않은 곳이라면 , 큰 문제 x

문제는 라이브러리 및 프레임 워크에서 발생할 수 있음 

→ 내부적으로 얽혀 요청 당 많은 호출 / 숨겨져 있어 성능 병목현상에 대한 여부 파악이 힘들다. 

이 때문에 다음과 같이 다양한 프레임워크에서 수정 진행 

- [Spring - HttpStatus.values()](https://github.com/spring-projects/spring-framework/issues/26842)

- [mssql-jdbc](https://github.com/spring-projects/spring-framework/issues/26842)

- [kotlin - serialize](https://github.com/Kotlin/kotlinx.serialization/issues/1372)

### 2. **Collection이 아닌 변경 가능한(Mutable) array를 반환한다.**

```kotlin
val brands: Array<Brand> = Brand.values()
```

불변이 아니기에 중간에 배열 값을 조작이 가능하며, 컬렉션보다 덜 유연한 Array<E>를 리턴함 
Collection 을 위해서는 꼭 변환이 필요하다. 

```kotlin
Brand.values().toList().stream()
    .filter { it.country == "America" }

Brand.values().toMutableList().stream()
    .filter { it.country == "America" }
```

자바는 감싸줘야함..

```java
Arrays.stream(Brand.values())
    .filter(brand -> brand.getCountry().equals("America"))
    .toList();
```

이러한 이슈들로 인해 Enum.values() 사용에는 주의가 필요하며 
Kotlin 에서는 `entries` 가 1.9 부터 나온다.

## Kotlin v.1.9.0 Enum.entries

내부적 구현은 다르나 사용법은 기존 values()와 동일 

```kotlin
fun main() {
    val americaBrand = Brand.entries.filter { it.country == "America" }
    println(americaBrand)
}

enum class Brand(val country: String) {
    SAMSUNG("Korea"),
    APPLE("America"),
    GOOGLE("America"),
    MICROSOFT("America"),
}
```

구현으로는 `EnumEntries<E>` type 을 만들어 제공

```kotlin
sealed interface EnumEntries<E : Enum<E>> : List<E>
```

클래스 명으로 값에 대한 의미를 나타낼 수 있으며, 
불변형을 나타내고자 새로운 인터페이스인 EnumEntries 를 만들어 제공한다.

```kotlin
// entries 에 대한 확장 함수 
fun <E : Enum<E>> EnumEntries<E>.someExtension(): E {...}

enum class RGB {
    RED, YELLOW, ....
}

val x: RGB = RGB.entries.someExtension()
```

여기서 [entries에 대한 내용](https://github.com/Kotlin/KEEP/blob/master/proposals/enum-entries.md#examples-of-performance-issues)을 보면, 어떻게 `Enum.values()` 를 폐기할지에 대한 내용이 나옴

> Enum.values() 는 2004년에 릴리즈 된 Java 1.5 부터 사용되어 왔고, Kotlin 초기부터 사용되어져왔기에 이를 바로 사용 못하게 하는 것은 큰 혼란을 불러 일으킬 수 있다.
> 
> 
> 그러므로 IDE의 도움을 받아 부드럽게 폐기(will be softly decommissioned)해나가겠다.
> 
> - - values() 사용에 대한 우선순위를 낮추고, IDE 자동완성에서 제거한다.
> - - values() 사용을 entries 로 대체하도록 가이드하는 Soft warning을 추가한다.
> - - Kotlin 가이드 등의 문서에서 entries 를 사용하도록 업데이트한다.

