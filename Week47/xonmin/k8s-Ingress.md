

### 5.4 Ingress 를 통한 서비스 외부 노출 

#### 인그레스가 필요한 이유 

- 로드밸런서 서비스는 자신의 public IP 주소를 가진 로드밸런서가 필요하지만, 
  인그레스는 한 IP 주소로 여러 서비스에 접근이 가능하도록 지원 
- client 가 HTTP 요청을 인그레스에 보낼 때, 요청한 호스트와 경로에 따라 요청을 전달할 서비스가 결정 

![image](https://github.com/xonmin/D3C/assets/27190617/e2a13c9e-b252-41c4-a06b-51302581f8f7)


#### 인그레스(Ingress)란 ? 
외부에서 들어온 HTTP / HTTPS 를 클러스터 내부 서비스로 라우팅 
![image](https://github.com/xonmin/D3C/assets/27190617/f3757288-748c-47b4-afaf-dfff17fbe59f)

- 인그레스는 어플리케이션 계층(HTTP) 에서 동작하며, 서비스가 할 수 없는 쿠키 기반 세션 어피니티 등과 같은 기능을 제공 

#### 인그레스 컨트롤러
- 인그레스를 사용하기 위해서는 클러스터 내에 **인그레스 컨트롤러**가 실행되어야 한다.
  - 현재 k8s 에서 공식 지원하는 Ingress Controller 는 AWS, GCP, Nginx 
	- 구글 쿠버네티스 엔진 : HTTP 로드밸런싱 기능을 이용해 인그레시 기능 제공
	- Minikube는 애드온 제공을 통해 인그레스 기능을 시험할 수 있도록 함
 
```yml 
> minikube **addons** list

- addon-manager: enabled

- dashboard: disabled

- default-storageclass: enabled 

- efk: disabled

- freshpod: disabled

- gvisor: disabled

- heapster: disabled

- ingress: disabled // 현재 활성화되지 않음 

- logviewer: disabled

- metrics-server: disabled


> minikube addons enable ingress 

> kubectl get po --all-namespaces | grep ingress

NAMESPACE   NAME                            READY STATUS    RESTARTS   AGE
kube-system nginx-ingress-controller-gdts0  1/1   Running   0          18m
```


### Ingress Class 
Ingress 는 여러 Ingress Controller에 의해 구현될 수 있으며, 이 때 다른 구성으로 구현될 수 있기 때문에 
각 인그레스에서는 클래스를 구현해야하는 **컨트롤러 이름** 과 추가적인 구성이 포함된 IngressClass 를 참조클래스에서 지정해야 한다.
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
```

### 인그레스 리소스 생성 
- https://github.com/luksa/kubernetes-in-action/blob/master/Chapter05/kubia-ingress.yaml
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com // 인그레스가 해당 도메인 이름을 서비스에 매핑 
    http:
      paths:
      - path: /        // 모든 요청은 kubia-nodeport 서비스의 80 로 전달 
        backend:
          serviceName: kubia-nodeport 
          servicePort: 80
```

**인그레스로 서비스 액세스하도록** 

ingress IP 주소
```shell
kubectl get ingress 

NAME              HOSTS   ADDRESS        PORTS AGE 
cafe-ingress-uri  *       123.123.123.44 80    88s
```


인그레스 동작 방식 
- client -> target DNS 조회 -> DNS 서버에서 인그레스 컨트롤러 IP return 
- client ->  인그레스 컨트롤러에 HTTP 요청  (host 헤더에 target 정보 지정)
   -> 해당 헤더 확인을 통해 인그레스 컨트롤러가 액세스하고자하는 서비스 결정 후 해당 엔드포인트 오브젝트로 파드 IP 조회 후 요청 전달 
![image](https://github.com/xonmin/D3C/assets/27190617/a041b052-3a63-479f-b09e-776b4518e538)


#### 5.4.3 하나의 인그레스로 여러 서비스 노출


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com // 동일 호스트의 다른 경로로 n 개 서비스 연결 (서로 다른 호스트도 가능)
    http:
      paths:
      - path: /foo        
        backend:
          serviceName: kubia
          servicePort: 80
      - path : /bar 
        backend:
          serviceName: bard
          servicePort: 80
      
```


##### TLS 트래픽(HTTPS) 처리하도록 인그레스 구성 
- https://kubernetes.io/docs/concepts/services-networking/ingress/#tls

클라이언트와 컨트롤러 간 통신에 대해선 암호화되지만, 
컨트롤러와 백엔드 파드 간의 통신에 대해서는 암호화되지 않는다. 
- 파드에서 실행 중인 애플리케이션은 TLS 를 지원할 필요가 없기 때문이다. 
- ex_ 파드가 웹서버를 실행중인 경우, HTTP 트래픽만 허용하도록 한 후 인그레스 컨트롤러가 TLS 와 관련한 처리는 담당할 수 있도록 할 수 있다. 
- 이 때, 컨트롤러가 위와 같은 동작을 위해서는 **인증서와 개인 키** 모두 인그레스에 첨부해야하며, 
  이를 k8s secret 에 저장하여 인그레스 매니페스트에 참조한다. 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

해당 시크릿을 생성 후 인그레스 오브젝트를 업데이트하면 `https-example.foo.com` 에 대한 HTTPS 요청 수락 가능 

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80

```
