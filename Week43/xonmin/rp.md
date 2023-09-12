## 14.2 Sequence 생성을 위한 Operator
### justOrEmpty
![image](https://github.com/xonmin/D3C/assets/27190617/06735043-bb17-4f8a-b487-fdeb0962fbe7)


- just()의 확장 operator
- just()와 달리, emit 데이터가 null이여도 NullPointException 대신 onComplete Signal 전송
- null이 아닐 경우 해당 데이터를 emit하는 Mono 생성

### fromIterable

![image](https://github.com/xonmin/D3C/assets/27190617/5b988417-98de-44f0-88b9-26e297403735)


- Iterable에 포함된 데이터를 emit하는 Flux 생성
- Java에서 제공하는 Iterable 구현체를 fromIterable 파라미터로 전달 가능

### fromStream

![image](https://github.com/xonmin/D3C/assets/27190617/efff8dda-ded9-4822-8893-8dbcd5cdab61)


- Stream에 포함된 데이터를 emit하는 Flux 생성
- Java의 Stream 특성상 재사용할 수가 없어, cancel, error, complete시 자동으로 닫힘

### range

![image](https://github.com/xonmin/D3C/assets/27190617/79d8543d-6f09-4917-ba74-534d4ce33a94)



- n부터 1씩 증가한 연속된 m개 emit하는 Flux 생성
- for 문처럼 특정 횟수만큼 어떤 작업 처리하고자 할 경우 사용

### defer

![image](https://github.com/xonmin/D3C/assets/27190617/907a4d50-bfef-4c76-8a8e-44324ca48315)

- 구독하는 시점에 데이터를 emit하는 Flux, Mono 생성
- lazy loading 비스무리함
- `.switchIfEmpty(Mono.just("empty"))` - 즉시 "empty" emit 함
- `.switchIfEmtpy(Mono.defer(() -> Mono.just("empty")));` - empty 일때 "empty" emit 함 

### using

![image](https://github.com/xonmin/D3C/assets/27190617/394898ff-1b8b-4b10-94d9-2f8c80c3e3d2)

- resource를 emit 하는 Flux를 생성
- 첫 번째 파라미터 = resource 
- 두 번째 파라미터 = emit하는 Flux
- 세 번째 파라미터 = `onComplete`나 `onError` 같이 종료 시그날 발생시 resource 해제 처리
- `Flux.using(() -> Files.line(path), Flux::fromStream, Stream::close)`

### generate

![image](https://github.com/xonmin/D3C/assets/27190617/00fe3f6b-edd6-48af-8e42-9bc33b94a529)


- 프로그래밍 방식으로 Signal 이벤트를 발생시키며, 동기적으로 데이터를 하나씩 순차적으로 emit할때 사용
- 첫 번째 파라미터 = emit 할 초깃값(+, -는 양수, 음수 모두 가능하다는 의미)
- 두 번째 파리미터 = V는 상태값, V로 다음 처리하는 람다표현식, v로 다음 emit할 값을 return
- 책에는 1씩 증가라고 하는데 상관 없음, V는 숫자, 객채 뭐든지 다됨, 다만 다음 상태를 표한하는 State를 return 해야함

![image](https://github.com/xonmin/D3C/assets/27190617/5fcd7007-98fd-45dd-8de5-a94f9bb244b6)


### create

![image](https://github.com/xonmin/D3C/assets/27190617/61720e9a-d4e0-42eb-9062-4fd5f40fb978)

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

![image](https://github.com/xonmin/D3C/assets/27190617/2aa634cd-7a72-4568-a28a-e145f5dffa4e)

### 비동기로 돌아가는 filterWhen

![image](https://github.com/xonmin/D3C/assets/27190617/cf1b82dd-6683-4f00-8af4-b861192adf76)

- 각 emit되는 inner sequence에서 predicate 성공하면 emit함

### skip

![image](https://github.com/xonmin/D3C/assets/27190617/cee03336-51f3-439a-ba11-21fb93f4668b)

### skip(duration)

![image](https://github.com/xonmin/D3C/assets/27190617/4e4a2426-c1f8-4ffa-9558-02f9db82b06b)

### take 

![image](https://github.com/xonmin/D3C/assets/27190617/7963428b-a1db-4d3b-ae0f-f35f3c589772)

![image](https://github.com/xonmin/D3C/assets/27190617/3ff2f672-ed00-4be9-913f-8f906314703a)
- true: 주로 request, cancel 경합이 많을때? 사용한다고함 

### take(duration)

![image](https://github.com/xonmin/D3C/assets/27190617/e4d637ab-97bb-481d-afbe-9d4ebc2a8275)

### takeLast

![image](https://github.com/xonmin/D3C/assets/27190617/6fc2f426-cc17-4ce3-a1df-421c2a55972b)

### takeUntil

![image](https://github.com/xonmin/D3C/assets/27190617/38825a9f-3045-4c26-93e1-2cecf4f2d897)

- predicate를 평가할 때 사용한 데이터가 포함됨

### takeWhile
![image](https://github.com/xonmin/D3C/assets/27190617/5063f351-5950-4dbb-93bc-fd8fa45bd893)

- predicate를 평가할 때 사용한 데이터가 포함되지 않음

### next

![image](https://github.com/xonmin/D3C/assets/27190617/548e1e07-163a-466b-95a0-c9f40f1e2f3d)

- 하나만 emit 하고 종료
