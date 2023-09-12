## 14.2 Sequence 생성을 위한 Operator
### justOrEmpty
![image](https://media.oss.navercorp.com/user/21795/files/1413ed7a-aba6-4c24-80c5-d4ed69288fe1)

- just()의 확장 operator
- just()와 달리, emit 데이터가 null이여도 NullPointException 대신 onComplete Signal 전송
- null이 아닐 경우 해당 데이터를 emit하는 Mono 생성

### fromIterable

![image](https://media.oss.navercorp.com/user/21795/files/36b86484-1cc3-4536-b28f-81a9819e0681)

- Iterable에 포함된 데이터를 emit하는 Flux 생성
- Java에서 제공하는 Iterable 구현체를 fromIterable 파라미터로 전달 가능

### fromStream

![image](https://media.oss.navercorp.com/user/21795/files/005bb777-b245-4b4f-a152-4a0fd303c31f)

- Stream에 포함된 데이터를 emit하는 Flux 생성
- Java의 Stream 특성상 재사용할 수가 없어, cancel, error, complete시 자동으로 닫힘

### range

![image](https://media.oss.navercorp.com/user/21795/files/bda9c1fb-7476-44b8-8900-1178e8307bcb)


- n부터 1씩 증가한 연속된 m개 emit하는 Flux 생성
- for 문처럼 특정 횟수만큼 어떤 작업 처리하고자 할 경우 사용

### defer

![image](https://media.oss.navercorp.com/user/1037/files/82524536-e062-4fe9-a262-b5f76bad36f0)

- 구독하는 시점에 데이터를 emit하는 Flux, Mono 생성
- lazy loading 비스무리함
- `.switchIfEmpty(Mono.just("empty"))` - 즉시 "empty" emit 함
- `.switchIfEmtpy(Mono.defer(() -> Mono.just("empty")));` - empty 일때 "empty" emit 함 

### using

![image](https://media.oss.navercorp.com/user/1037/files/b8d67af2-f42f-490a-82dd-f46cf290e969)

- resource를 emit 하는 Flux를 생성
- 첫 번째 파라미터 = resource 
- 두 번째 파라미터 = emit하는 Flux
- 세 번째 파라미터 = `onComplete`나 `onError` 같이 종료 시그날 발생시 resource 해제 처리
- `Flux.using(() -> Files.line(path), Flux::fromStream, Stream::close)`

### generate

![image](https://media.oss.navercorp.com/user/1037/files/cc0a47f9-28b3-4ad7-a617-f087faa18b96)

- 프로그래밍 방식으로 Signal 이벤트를 발생시키며, 동기적으로 데이터를 하나씩 순차적으로 emit할때 사용
- 첫 번째 파라미터 = emit 할 초깃값(+, -는 양수, 음수 모두 가능하다는 의미)
- 두 번째 파리미터 = V는 상태값, V로 다음 처리하는 람다표현식, v로 다음 emit할 값을 return
- 책에는 1씩 증가라고 하는데 상관 없음, V는 숫자, 객채 뭐든지 다됨, 다만 다음 상태를 표한하는 State를 return 해야함

<img width="595" alt="image" src="https://media.oss.navercorp.com/user/1037/files/c6f3327b-a66e-4d5b-9dd5-34b280595524">

### create

![image](https://media.oss.navercorp.com/user/1037/files/b73fb6ae-9757-4998-bc89-214046a3a581)

- 프로그래밍 방식으로 Signal 이벤트를 발생시키며, 한 번에 여러건의 데이터를 비동기적으로 emit 할 수 있다.
- 첫 번째 파라미터 = emitter가 emit 하는 역할을하고 이때 listener를 통해 emit 할 갯수를 전달 받고 처리하여 emit한다. 
- 두 번째 파리미터 = backpressure 설정 값

```java
    CryptoCurrencyPriceEmitter priceEmitter = new CryptoCurrencyPriceEmitter();
    
    Flux.create((FluxSink<Integer> sink) ->
                    priceEmitter.setListener(new CryptoCurrencyPriceListener() {
                        @Override
                        public void onPrice(List<Integer> priceList) {
                            priceList.stream().forEach(price -> {
                                sink.next(price);
                            });
                        }
    
                        @Override
                        public void onComplete() {
                            sink.complete();
                        }
                    }))
            .publishOn(Schedulers.parallel())
            .subscribe(
                    data -> log.info("# onNext: {}", data),
                    error -> {},
                    () -> log.info("# onComplete"));
    
    Thread.sleep(3000L);
    
    priceEmitter.flowInto(); // listener.onPrice(list) 실행, list 갯수만큼 emit
    
    Thread.sleep(2000L);
    priceEmitter.complete();
```

## 14.3 Sequence 필터링 Operator

### filter

![image](https://media.oss.navercorp.com/user/1037/files/99665e74-4891-4d3f-82eb-69d73e7d92f1)

### 비동기로 돌아가는 filterWhen

![image](https://media.oss.navercorp.com/user/1037/files/bed42272-da08-4c65-84a6-9befef88f182)

- 각 emit되는 inner sequence에서 predicate 성공하면 emit함

### skip

![image](https://media.oss.navercorp.com/user/1037/files/c2189d6d-6580-47e4-a496-45c6e2466b76)

### skip(duration)

![image](https://media.oss.navercorp.com/user/1037/files/a1b29e81-ada5-4716-bb14-6df72ef8fc3e)

### take 

![image](https://media.oss.navercorp.com/user/1037/files/e043ada3-027f-4129-b27b-86643db9a099)
![image](https://media.oss.navercorp.com/user/1037/files/7999c3a3-66f8-4101-8df9-290355328f2d)

- true: 주로 request, cancel 경합이 많을때? 사용한다고함 

### take(duration)

![image](https://media.oss.navercorp.com/user/1037/files/e94114f5-1f19-4e42-920e-78512a9472eb)

### takeLast

![image](https://media.oss.navercorp.com/user/1037/files/cb7e3906-fd50-4267-bb7e-26126145dd96)

### takeUntil

![image](https://media.oss.navercorp.com/user/1037/files/1f5a395f-020c-42a8-a9cb-8759a345c2f2)

- predicate를 평가할 때 사용한 데이터가 포함됨

### takeWhile

![image](https://media.oss.navercorp.com/user/1037/files/abe4b4a4-b0b9-493d-8b24-88a1502abce0)

- predicate를 평가할 때 사용한 데이터가 포함되지 않음

### next

![image](https://media.oss.navercorp.com/user/1037/files/dcecff4d-f2ba-453c-aa84-9a472acb3155)

- 하나만 emit 하고 종료
