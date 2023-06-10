
# 10. 애노테이션과 리플랙션 

- 애노테이션 : library가 요구하는 의미를 클래스에게 부여 가능
- 리플랙션 : 실행 시점에 컴파일러 내부 구조 분석 가능 

## 10.1 애노테이션 선언과 적용 
메타데이터를 선언에 추가시, 애노테이션을 처리하는 도구가 컴파일 시점과 실행 시점에 적절한 처리 진행 

### 10.1.1 애노테이션 적용 
코틀린에서는 `replaceWith` 파라미터를 통해 옛버전을 대신할 수 있는 패턴 제시 가능 
```kotlin 
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) { ... }
```
이를 통해 새로운 API 버전에 맞는 코드로 바꿔주는 퀵 픽스 제시 가능 

애노테이션 인자 가능한 것들
- primitive, String, Enum, 클래스 참조, 다른 Annotation class, 앞의 것들의 배열  

코틀린 애노테이션 인자 지정 문법 
- 클래스를 애노태이션 인자로 지정 시 `@MyAnnotation(MyClass::class)`
- 다른 애노테이션 인자로 지정 시, 인자로 들어가는 애노테이션 앞에 `@`를 넣어선 안된다. 
- 배열을 인자로 지정시, `@RequestMapping(path=arrayOf("/foo","/bar"))` 와 같이 `arrayOf()` 사용 
  - 자바에서 선언한 애노테이션 클래스를 사용한다면, value 라는 이름의 파라미터가 필요에 따라 자동으로 가변 길이 인자로 변환 (이때는 arrayOf 사용 x) 

애노테이션 인자를 컴파일 시점에 알 수 있어야 하므로, 프로퍼티를 인자로 사용하려면 `const` 변경자 사용 
```kotlin
const val TEST_TIMEOUT = 1000L 

@Test(timeout= TEST_TIMEOUT) fun testmethod() { ... }
```


