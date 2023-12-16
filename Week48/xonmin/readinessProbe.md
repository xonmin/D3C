# 5.5 Readiness Probe

파드의 레이블이 서비스의 파드 셀렉터와 일치하는 경우, 파드는 서비스의 엔드포인트로 포함된다. 
하지만, 새 파드가 만들어지자마자 해당 파드가 즉시 요청 처리할 준비가 돼 있지 않을 수 있다. 
다음과 같은 상황이 이에 해당한다. 

- 파드 구성에 시간이 걸리거나, 데이터를 초기 로드하는데 시간이 필요한 경우
- 첫 번째 사용자 요청이 너무 오래걸리거나 사용자 경험에 영향을 미치는 것을 방지하고자 웜업 등의 절차가 필요한 경우   

이러한 상황에 파드가 준비가 되어있는 지 레디니스 프로브(Readiness Probe) 를 통해 해당 파드가 요청을 수신할 준비가 되어있는 지 확인 가능하다. 

  
### Readiness Probe 

- 라이브니스 프로브(Liveness Probe)와 불안전한 컨테이너를 자동으로 다시 시작해 원활한 상태의 어플리케이션 상태를 유지할 수 있는데 레디니스 프로브 또한 비슷하게 정의할 수 있다. 
- 레디니스 프로브는 주기적으로 호출되어 특정 파드가 클라이언트 요청을 수신할 수 있는 지 결정한다. 
	- 이 때, 각 컨테이너 별로 준비에 대한 정의는 각각 다르게 정의할 수 있음 
	- GET req 혹은 특정 url 경로 호출, 전체적인 항목 검사 모두 가능 

#### 레디니스 프로브 유형 
- exec commend 프로브의 경우 컨테이너 상태를 통해 프로세스 종료 상태 코드로 결정 


```yaml
...
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

- HTTP Probes : `httpGet` 으로 설정하며 GET 요청을 컨테이너로 보낸 후, 응답의 HTTP 상태 코드를 통해 컨테이너 준비 여부를 확인 
```yaml
livenessProbe:
  httpGet:
    httpHeaders:
      - name: Accept
        value: application/json

startupProbe:
  httpGet:
    httpHeaders:
      - name: User-Agent
        value: MyUserAgent
```
- TCP probes : 컨테이너의 지정된 포트로 TCP 연결 -  소켓이 연결되면 컨테이너가 준비된 것으로 간주 

```yaml
spec:
  terminationGracePeriodSeconds: 3600  # pod-level
  containers:
  - name: test
    image: ...

    ports:
    - name: liveness-port
      containerPort: 8080
      hostPort: 8080

    livenessProbe:
      httpGet:
        path: /healthz
        port: liveness-port
      failureThreshold: 1
      periodSeconds: 60
      # Override pod-level terminationGracePeriodSeconds #
      terminationGracePeriodSeconds: 60
```

#### 레디니스 프로브 동작 
- 컨테이너가 시작될 땐, 첫번째 레디니스 확인 이전에 구성 가능한 시간이 경과되기까지 `initialDelaySeconds` 설정을 통해 기다리도록 구성이 가능하다. 
- 그 이후 주기적으로 해당 프로브를 호출하고 결과에 따라 작동한다. 
- 만약 파드가 준비되지 않았다고 한다면 엔드포인트에서 해당 파드는 제거되고 요청이 들어오지 않는다. 
  파드가 다시 준비되면 엔드포인트에 다시 추가된다. 
- 중요한 점은 레디니스 프로브 결과에 대해 실패하더라도 해당 파드가 종료되거나 다시 시작하지 않는다. 
  해당 점이 라이브니스 프로브와의 차이
  -  물론 연속적으로 레디니스 프로브에 실패할 경우 `failureThreshold`와 `terminationGracePeriodSeconds` 설정을 통해 unhealthy 한 파드로 여겨 제거/재시작을 할 수 있다. 

#### 레디니스의 중요성 

파드 그룹이 다른 파드(DB)에서 제공하는 서비스에 의존한다고 가정
- 만약 프론트엔드 파드 중 하나에 연결 문제가 발생하여 더이상 Db에 연결할 수 없는 경우, 
  해당 시점에 파드가 해당 요청을 처리할 준비가 되지 않았다는 신호를 **레디니스 프로브**를 통해 kubernetes 에 알릴 수 있다. 
  이를 통해 문제가 있는 프론트엔드 파드에 요청을 다른 문제 없는 파드로 정상 요청할 수 있도록 하여 서비스 이용에 차질 없이 운용해나갈 수 있다. 

운용상 레디니스 프로브를 추가하지 않는 경우, 파드 시작에 대해 바로 서비스 엔드포인트에 추가된다. 
이 때 서비스가 준비가 안된 경우 클라이언트에서 `Connection refused` 유형의 에러를 볼 수 있게 되니 
반드시 추가하도록 하자. 

또한 레디니스 프로브에 파드 종료 코드를 포함해선 안된다. 
- 파드가 종료할 때, 실행되는 애플리케이션 종료 신호를 받자마자 연결 수락을 중단하는데 이러한 특성을 이용해
  종료 절차가 시작되는 즉시 레디니스 프로브가 실행하도록 만들어 파드가 모든 서비스에서 확실하게 제거하도록 해야할 수 있다고 생각할 수 있다. 
- 하지만 쿠버네티스는 파드를 삭제하자마자 모든 서비스 엔드포인트에서 해당 파드를 제거하기 때문에 레디니스에서 파드 종료에 대한 처리를 할 필요가 없다. 
