# Chapter 15. Spring WebFlux 개요 


## 15.1 Spring Webflux 탄생 배경 
- 리액티브 웹 application 구현을 위해 Spring 5.0부터 지원한 리액티브 웹 프레임워크

MVC : 서블릿 기반의 Blocking I/O 방식 
- 하나의 요청 : 하나의 스레드
- 해당 스레드의 작업이 끝나기 전까지는 해당 스레드 차단
- 대량의 트래픽에 의해 적은 수의 스레드로 안정적인 처리에 대해 비동기 Non-Blocking 방식이 필요했음

## 15.2 Webflux 기술 스택 
![image](https://github.com/xonmin/D3C/assets/27190617/cff5c7c7-e69b-413f-ac10-345fe4c8224f)

서버 
- MVC : 서블릿 기반 / tomcat 같은 서블릿 컨테이너 -> Blocking I/O 방식 동작
- Webflux : Netty 를 통한 Non-Blocking I/O

서버 API 
- MVC : 서블릿 기반 프레임워크 - 서블릿 API 사용
- Webflux : 기본 서버 엔진=Netty (Jetty나 Undertow 같은 서버 엔진에서 지원하는 리액티브 스트림즈 어댑터를 통해 리액티브 스트림즈 지원)

보안 
- MVC : Spring Security (표준 서블릿 필터) 가 서블릿 컨테이너에 통합
- Webflux : Webfilter 를 통해 Spring Security 를 사용

데이터 액세스
- MVC : Blocking I/O 방식
- Webflux : Non-Blocking I/O 지원 방식 사용

## 15.3 Webflux 요청 처리 흐름

![image](https://github.com/xonmin/D3C/assets/27190617/92263961-8e9c-48de-a7bc-20b5536dd1f3)
<img width="831" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/6b9c1650-7f08-4c63-9dc2-4e9cb4a5c88e">

> 실제 핸들러에서 리턴되는 것은 응답 데이터를 포함하고 있는 Flux or Mono Sequence
> 따라서, 메서드 호출 시 리턴된 Reactor Seq 가 즉시 어떤 작업을 수행하는 것은 아님

## 15.4 Webflux 핵심 컴포넌트 
### HttpHandler 
- HttpHandler는 다른 유형의 HTTP 서버 API로부터 request, response를 처리하기 위한 단 하나의 추상메서드를 갖는다.

```java
public interface HttpHandler {

	/**
	 * Handle the given request and write to the response.
	 * @param request current request
	 * @param response current response
	 * @return indicates completion of request handling
	 */
	Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response);

}
```

```java
public class HttpWebHandlerAdapter extends WebHandlerDecorator implements HttpHandler {

	...
    
    @Override
	public Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response) {
		if (this.forwardedHeaderTransformer != null) {
			try {
				request = this.forwardedHeaderTransformer.apply(request);
			}
			catch (Throwable ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to apply forwarded headers to " + formatRequest(request), ex);
				}
				response.setStatusCode(HttpStatus.BAD_REQUEST);
				return response.setComplete();
			}
		}
    // 1) handler() 메서드의 파라미터로 전달받은 ServerHttpRequest, ServerHttpResponse로 ServerWebExchange를 생성
		ServerWebExchange exchange = createExchange(request, response);

		LogFormatUtils.traceDebug(logger, traceOn ->
				exchange.getLogPrefix() + formatRequest(exchange.getRequest()) +
						(traceOn ? ", headers=" + formatHeaders(exchange.getRequest().getHeaders()) : ""));

    // 2) WebHandler을 호출한다.
		return getDelegate().handle(exchange)
				.doOnSuccess(aVoid -> logResponse(exchange))
				.onErrorResume(ex -> handleUnresolvedError(exchange, ex))
				.then(Mono.defer(response::setComplete));
	}
    
    ...

}
```
### Webfilter 
핸들러가 요청을 처리하기 전에 전처리 작업을 수행할 수 있도록 해준다. 애플리케이션 내에 정의된 모든 핸들러에 공통으로 동작한다.
- 파라미터로 받은 WebFilterChain 을 통해 필터 체인을 형성하여 원하는 만큼의 WebFilter를 추가할 수 있다.
```java
public interface WebFilter {
	Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);

}
```
예제 - Book 리소스에 request 시에만 로그를 출력하는 필터 
```java
@Component
public class BookLogFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String path = exchange.getRequest().getURI().getPath();
        return chain.filter(exchange).doAfterTerminate(() -> { // doAfterTerminate() : 종료 이벤트(onComplete, onError) 발생시, 수행된다.
            if (path.contains("books")) {
                System.out.println("path: " + path + ", status: " +
                        exchange.getResponse().getStatusCode());
            }
        });
    }
}
```

### HadlerFilterFunction
함수형 기반의 요청 핸들러에 적용할 수 있는 Filter로, 함수형 기반의 핸들러에서만 동작
- 따라서 **함수형 기반의 핸들러에서만 제한적으로 필터링 작업을 수행하고 싶을때 사용**
- 파라미터로 전달받은 `HandlerFunction`에 연결된다.
```java
@FunctionalInterface
public interface HandlerFilterFunction<T extends ServerResponse, R extends ServerResponse> {
	Mono<R> filter(ServerRequest request, HandlerFunction<T> next);

	...

}
```
예제 - Book 리소스에 request 시에만 로그를 출력한다. 
```java
public class BookRouterFunctionFilter implements HandlerFilterFunction {
    @Override
    public Mono<ServerResponse> filter(ServerRequest request, HandlerFunction next) {
        String path = request.requestPath().value();

        return next.handle(request).doAfterTerminate(() -> {
            System.out.println("path: " + path + ", status: " +
                    request.exchange().getResponse().getStatusCode());
        });
    }
}
```

> Webfilter vs HandlerFilterFunction
> 위 WebFliter의 구현체는 Spring Bean으로 등록되는 반면,
> HandlerFilterFunction의 구현체는 애너테이션 기반의 핸들러가 아닌 함수형 기반의 요청 핸들러에서 함수 형태로 사용되기 때문에 Spring Bean으로 등록되지 않는다.
>  - Webfilter 는 어플리케이션 내 정의된 모든 핸들러에 공통 동작(어노테이션 기반 요청 핸들러 & 함수형 기반 요청 핸들러)
>  - HandlerFilterFunction : 함수형 기반 핸들러에서만 제한적 수행 


**handlerFilterFunction 예제**
- 아래 API 요청이 오면, 정의된 Response가 내려가고, filter 안의 로그가 찍힌다.
- 함수형 요청 핸들러를 라우팅해 주는 RouterFunction의 filter() 메서드에 파라미터로 BookRouterFunctionFilter를 전달, 필터링 작업을 수행

```java
@Configuration
public class BookRouterFunction {
    @Bean
    public RouterFunction routerFunction() {
        return RouterFunctions
                .route(GET("/v1/router/books/{book-id}"),
                        (ServerRequest request) -> this.getBook(request))
                .filter(new BookRouterFunctionFilter());
    }

    public Mono<ServerResponse> getBook(ServerRequest request) {
        return ServerResponse
                .ok()
                .body(Mono.just(BookDto.Response.builder()
                        .bookId(Long.parseLong(request.pathVariable("book-id")))
                        .bookName("Advanced Reactor")
                        .author("Tom")
                        .isbn("222-22-2222-222-2").build()), BookDto.Response.class);
    }
}
```

### DispatcherHandler 
- WebHandler 인터페이스의 구현체.
- Spring MVC에서 Front Controller 패턴이 적용된 DispatcherServlet처럼 **중앙에서 먼저 요청을 전달받은 후에 다른 컴포넌트에 요청 처리를 위임**
- DispatcherHandler 자체가 Spring Bean으로 등록되도록 설계되어있고, ApplicationContext에서 HandlerMapping, HandlerAdapter, HandlerResultHandler 등의 요청 처리를 위한 위임 컴포넌트를 검색한다.

```java
ublic class DispatcherHandler implements WebHandler, PreFlightRequestHandler, ApplicationContextAware {

	@Nullable
	private List<HandlerMapping> handlerMappings;

	@Nullable
	private List<HandlerAdapter> handlerAdapters;

	@Nullable
	private List<HandlerResultHandler> resultHandlers;

	public DispatcherHandler() {
	}

	...

  // BeanFactoryUtils를 이용해 ApplicationContext로부터 HandlerMapping Bean, HandlerAdapter Bean, HandlerResultHandler Bean을 검색한 후 각각의 타입인 List 객체를 생성한다.
	protected void initStrategies(ApplicationContext context) {
		Map<String, HandlerMapping> mappingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerMapping.class, true, false);

		ArrayList<HandlerMapping> mappings = new ArrayList<>(mappingBeans.values());
		AnnotationAwareOrderComparator.sort(mappings);
		this.handlerMappings = Collections.unmodifiableList(mappings);

		Map<String, HandlerAdapter> adapterBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerAdapter.class, true, false);

		this.handlerAdapters = new ArrayList<>(adapterBeans.values());
		AnnotationAwareOrderComparator.sort(this.handlerAdapters);

		Map<String, HandlerResultHandler> beans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerResultHandler.class, true, false);

		this.resultHandlers = new ArrayList<>(beans.values());
		AnnotationAwareOrderComparator.sort(this.resultHandlers);
	}

  // List<HandlerMapping>을 Flux.fromIterable()의 원본 데이터 소스로 입력받은 후에 getHandler() 메서드를 통해 매치되는 Handler 중에서 첫번째 핸들러를 사용한다.
	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) {
			return createNotFoundError();
		}
		if (CorsUtils.isPreFlightRequest(exchange.getRequest())) {
			return handlePreFlight(exchange);
		}
		return Flux.fromIterable(this.handlerMappings)
				.concatMap(mapping -> mapping.getHandler(exchange))
				.next()
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler))
				.flatMap(result -> handleResult(exchange, result));
	}

	...
  // 실제 핸들러 호출 = invokeHandler() 내부에서 Handler 객체와 매핑되는 HandlerAdapter를 통해서 이뤄진다.
	private Mono<HandlerResult> invokeHandler(ServerWebExchange exchange, Object handler) {
		if (ObjectUtils.nullSafeEquals(exchange.getResponse().getStatusCode(), HttpStatus.FORBIDDEN)) {
			return Mono.empty();  // CORS rejection
		}
		if (this.handlerAdapters != null) {
			for (HandlerAdapter handlerAdapter : this.handlerAdapters) {
				if (handlerAdapter.supports(handler)) {
					return handlerAdapter.handle(exchange, handler);
				}
			}
		}
		return Mono.error(new IllegalStateException("No HandlerAdapter: " + handler));
	}

  // 실제 응답 처리는 handleResult() 내부에서 호출한 getResultHandler()에서 HandlerResult 객체와 매핑되는 HandlerResultHandler를 통해서 이뤄진다.
	private Mono<Void> handleResult(ServerWebExchange exchange, HandlerResult result) {
		return getResultHandler(result).handleResult(exchange, result)
				.checkpoint("Handler " + result.getHandler() + " [DispatcherHandler]")
				.onErrorResume(ex ->
						result.applyExceptionHandler(ex).flatMap(exResult -> {
							String text = "Exception handler " + exResult.getHandler() +
									", error=\"" + ex.getMessage() + "\" [DispatcherHandler]";
							return getResultHandler(exResult).handleResult(exchange, exResult).checkpoint(text);
						}));
	}

	private HandlerResultHandler getResultHandler(HandlerResult handlerResult) {
		if (this.resultHandlers != null) {
			for (HandlerResultHandler resultHandler : this.resultHandlers) {
				if (resultHandler.supports(handlerResult)) {
					return resultHandler;
				}
			}
		}
		throw new IllegalStateException("No HandlerResultHandler for " + handlerResult.getReturnValue());
	}

	...

}
```

### HandlerMapping 
MVC와 마찬가지로 request와 handler object에 대한 매핑을 정의하는 인터페이스. 
- HandlerMapping 인터페이스를 구현하는 구현 클래스로는 RequestMappingHandlerMapping, RouterFunctionMapping 등이 있다. 

```java
public interface HandlerMapping {

	...
  // 파라미터로 입력받은 ServerWebExchange에 매치되는 handler object를 리턴한다.
	Mono<Object> getHandler(ServerWebExchange exchange);

}
```

### HandlerAdapter 
HandlerAdapter는 HandlerMapping을 통해 얻은 핸들러를 직접적으로 호출을 하며, 응답 결과로 Mono<HandlerResult>를 리턴
```java
public interface HandlerAdapter {
  // 파라미터로 전달받은 handler object를 지원하는지 체크
	boolean supports(Object handler);
  // 파라미터로 전달받은 handler object를 통해 핸들러 메서드를 호출
	Mono<HandlerResult> handle(ServerWebExchange exchange, Object handler);

}
```

### 전반적인 흐름 
![image](https://github.com/xonmin/D3C/assets/27190617/81c5df3f-212e-4258-89b9-047035e8944a)
1) 클라이언트 요청
2) HttpHandler가 요청을 받고 ServerWebExchange(ServerhttpRequest, ServerHttpResponse)를 생성
3) 하여 WebFilter 체인으로 전달
4) DIspatcherHandler에게 전달
5) HandlerMapping에서 ServerWebExchange을 처리할 핸들러 조회
6) HandlerAdapater가 요청을 위임받음
7) 실제 핸들러 호출
8) HandlerResultHandler을 거쳐 response 반환
