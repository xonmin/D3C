
### 10.1.3 애노테이션을 활용한 JSON 직렬화 제어 

JSON에는 객체의 타입이 저장되지 않기 때문에 JSON 데이터로부터 인스턴스를 만들기 위해서는 타입 인자로 클래스 명시 

`Person("Alice", 20) -> 직렬화 -> {"age":29, "name":"Alice"}`
`Person("Alice", 20) <- 역직렬화 <- {"age":29, "name":"Alice"}`

- 객체를 직렬화할 때 기본적으로 모든 프로퍼티를 직렬화하며, 프로퍼티 이름을 키로 사용한다. 
- 애노테이션을 사용하면 이런 동작을 변경할 수 있음
- 이번 절에서는 `@JsonExclude`, `@JsonName` 이라는 두 애노테이션을 다룸 
- `@JsonExclude` : 직렬화나 역직렬화 시 해당 프로퍼티 무시 (jackson - `@JsonIgnore`)
- `@JsonName`: 프로퍼티를 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름을 쓰게 할 수 있음 
 
 ```kotlin 

data class Person(
  @JsonName("alias")
  val firstName: String, 
  
  @JsonExclude
  val age: Int? = null
)
```
- 직렬화 대상에서 제외할 프로퍼티의 경우 반드시 디폴트 값 선언 필요 
  - 역직렬화 시, 새로운 인스턴스 생성 불가 

### 10.1.4 애노테이션 선언 

```kotlin
annotation class JsonExclude
```

- 애노테이션 클래스는 오직 선언이나 식과 관련있는 메타데이터의 구조를 정의하기 때문에 내부에 아무 코드가 들어있을 수 x 
- 그런 이유로 컴파일런ㄴ 애노테이션 클래스에서 본문을 정의하지 못하게 막는다. 
- 파라미터가 있는 애노테이션 정의시, 애노테이션 클래스의 주 생성자에 파라미터 선언해야 함 
```kotlin
annotation class JsonName(val name: String)
```

일반 클래스의 주 생성자 선언 구문과 동일하지만, 애노테이션 클래스에서는 모든 파라미터 앞에 `val` 추가 

자바 어노테이션 선언과 비교 
```java 
public @interface JsonName {
  String value();
}
```

코틀린 애노테이션에서는 `name` 이라는 프로퍼티를 사용했지만, 자바 애노테이션에서는 `value` 라는 메서드를 사용 
- 자바
	- 자바에서의 `value` 메서드는 특별하다. 
	- 어떤 애노테이션을 적용할 때 `value`를 제외한 모든 애트리뷰트에는 이름 명시 
- 코틀린 
  - 애노테이션 적용 문법은 일반적인 생성자 호출과 같음 
  - 인자의 이름 명시를 위해서는 이름 붙은 인자 구문 사용하거나, 이름을 생략할 수 있다. 
  - `@JsonName(name="first_name") == @JsonName("first_name")` 

### 10.1.5 메타애노테이션 : 애노테이션을 처리하는 방법 제어 
자바와 마찬가지로 코틀린 애노테이션 클래스에도 애노테이션 붙일 수 있음 
**메타애노테이션** : 애노테이션 클래스에 적용할 수 있는 애노테이션 (ex_ `@Target`)

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

- 메타애노테이션을 직접 생성해야할 때, `ANNOTATION_CLASS` 대상으로 지정 
```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annotation class MyBinding
```
- 대상을 PROPERTY로 지정한 애노테이션은 자바 코드에서 사용할 수 없음
- 자바에서 해당 애노테이션을 사용하기 위해서는 `AnnotationTarget.FIELD`를 두번째 대상으로 추가해야한다. 

> `@Retention` 애노테이션
> 정의 중인 애노테이션 클래스를 소스 수준에서만 유지할지, .class에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지 지정하는 애노테이션 
> 자바 컴파일러는 기본적으로 애노테이션을 .class 파일에는 저장하지만, 런타임에는 사용할 수 없게 한다.
> 하지만, 대부분의 애노테이션은 런타임에도 사용할 수 있어야 하므로 코틀린에서는 기본적으로 애노테이션의 retention 을 `RUNTIME` 으로 지정 

### 10.1.6 애노테이션 파라미터로 클래스 사용

어떤 클래스를 선언 메타데이터로 참조해야할 경우 

ex_ 제이키드 라이브러리 내 `@DeserializeInterface`
- 해당 애노테이션은 프로퍼티에 대한 역직렬화 제어시 사용 
- 인터페이스의 인스턴스를 직접 만들 수 없으므로 역직렬화 시 어떤 클래스를 사용해 인터페이스를 구현할지 지정해주는 역할 

```kotlin
interface Company(
 val name: String 
) 

data class CompanyImpl(ovverrid val name: String): Company 

data class Person(
  val naem: String, 
  @DeserializeInterface(CompanyImpl::class) val company: Company
)
```

KClass 는 자바 java.lang.Class` 타입과 같은 역할을 하는 코틀린 타입 
- 코틀린 클래스에 대한 참조 저장시 KClass 타입 사용 
- CompanyImpl::class 타입은 KClass<CompanyImpl>이며, 해당 타입은 DeserializeInterface의 파라미터 타입인 `KClass<out Any>`의 하위 타입 
- 따라서 DeserializeInterfacedml 인자로 Any 를 확장하는 모든 클래스에 대한 참조 전달 가능 

### 10.1.7 애노테이션 파라미터로 제네릭 클래스 받기 
```kotlin
interface ValueSerializer<T> {
  fun toJsonValue(value: T): Any? 
  fun fromJsonValue(jsonValue: Any?): T
}
```
`@CustomSerializer` 는 ValueSerializer 인터페이스를 구현을 통해 역/직렬화의 로직을 변경 가능 

```kotlin
data class Person(
  val name: String, 
  @CustomSerializer(DateSerializer::class) val birthDate: Date 
```

ValueSerializer 클래스는 제네릭 클래스라서 타입 파라미터가 존재 
- 따라서 ValueSerializer 타입을 참조하기 위해서는 항상 타입 인자를 제공해야 한다. 
- 하지만 해당 애노테이션이 어떤 타입에 대해 쓰일지 알 수 없으므로 여기서는 *(스타프로젝션)을 사용

```kotlin
annotation class CustomSerializer (
  val serializerClass: KClass<out ValueSerializer<*>>
)
```
여기서는 CustomSerializer가 ValueSerializer를 구현하는 클래스만 인자로 받아야함을 명시해야함 
- ex) Date 가 ValueSerializer를 구현하지 않으므로 @CustomSerializer(Date::class) 금지시켜야 함 

정리 : 
- 클래스를 인자로 받아야하면 애노테이션 파라미터 타입에 `KClass<out 허용할 클래스 이름>`
- 제네릭 클래스를 인자로 받아야 한다면 `KClass<out 허용할 클래스 이름<*>>` 
