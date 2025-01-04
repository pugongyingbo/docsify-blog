

Spring WebClient 是 Spring 5 引入的响应式 Web 客户端，用于执行 HTTP 请求。它是 Spring Web Reactive 模块的一部分，用于取代经典的 `RestTemplate`。

### `WebClient` 和 `RestTemplate`区别

`WebClient` 和 `RestTemplate` 都是 Spring 提供的客户端 HTTP 工具，用于执行 HTTP 请求。

它们之间有几个关键的区别：

#### 1. 响应式与阻塞式

- **WebClient**：是响应式的（Reactive），基于 Reactor 库构建，完全非阻塞，适用于异步和非阻塞的编程模型。它支持 Reactive Streams 规范，可以很好地与 Spring WebFlux 集成。

- **RestTemplate**：是阻塞式的（Blocking），在执行网络请求时会阻塞当前线程，直到请求完成。它基于 Java 的同步处理模型，适用于传统的 Spring MVC 应用程序。

#### 2. 编程模型

- **WebClient**：遵循函数式编程风格，使用 `Mono` 和 `Flux` 作为响应类型，它们是响应式编程中的核心类型，分别代表0..1和0..N个响应值的异步序列。

- **RestTemplate**：遵循命令式编程风格，使用同步的返回类型，如 `List`、`String` 或自定义的 DTO（Data Transfer Object）。

#### 3. 错误处理

- **WebClient**：错误处理是通过响应类型 `Mono` 和 `Flux` 的操作符链来完成的，例如 `onErrorResume`、`onErrorReturn` 等。

- **RestTemplate**：错误处理通常是通过 try-catch 块来捕获异常，或者配置 `ErrorHandler` 来全局处理错误。

#### 4. 性能和资源利用

- **WebClient**：由于其非阻塞特性，可以在单个线程上处理多个请求，这对于高并发场景来说更加高效，可以减少线程开销和提高吞吐量。

- **RestTemplate**：由于其阻塞特性，在高并发场景下可能会导致线程资源的浪费，尤其是在 I/O 等待时。

#### 5. 集成和配置

- **WebClient**：与 Spring WebFlux 集成得更好，可以轻松地与响应式编程的其他组件一起使用。

- **RestTemplate**：与 Spring MVC 集成得更好，通常用于传统的 Spring 应用程序。

#### 6. 支持的 HTTP 特性

- **WebClient**：支持 HTTP/2 和 WebSocket，而 `RestTemplate` 不支持。

- **RestTemplate**：虽然功能全面，但不支持 HTTP/2 和 WebSocket。

#### 7. 适用场景

- **WebClient**：适用于构建响应式应用程序，特别是在需要处理大量并发请求和需要非阻塞 I/O 操作的场景。

- **RestTemplate**：适用于传统的阻塞式应用程序，或者在响应式特性不是必需的场景。

总的来说，`WebClient` 是为响应式编程设计的，而 `RestTemplate` 是为传统的同步编程设计的。随着响应式编程的流行和 Spring 5 的推出，`WebClient` 逐渐成为推荐的选择，尤其是在构建响应式应用程序时。然而，在一些遗留系统中，`RestTemplate` 仍然被广泛使用。

### 如何使用webclient

以下是如何在 Spring 应用程序中使用 WebClient 的基本指南：

#### 1. 依赖配置

在 Spring Boot 应用中，添加 `spring-boot-starter-webflux` 依赖以获得响应式 Web 支持：

**Maven 配置:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

**Gradle 配置:**
```groovy
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-webflux'
}
```

#### 2. 创建 WebClient 实例

有三种方式创建 `WebClient` 实例：

- **使用默认设置创建**：
  ```java
  WebClient client = WebClient.create();
  ```

- **使用给定的基本 URI 初始化**：
  ```java
  WebClient client = WebClient.create("http://localhost:8080");
  ```

- **使用 `DefaultWebClientBuilder` 类构建客户端**：
  
  ```java
  WebClient client = WebClient.builder()
    .baseUrl("http://localhost:8080")
    .defaultCookie("cookieKey", "cookieValue")
    .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
    .defaultUriVariables(Collections.singletonMap("url", "http://localhost:8080"))
    .build();
  ```

#### 3. 发起请求

`WebClient` 支持 GET、POST、PUT、DELETE 等多种 HTTP 方法。以下是一些基本的请求示例：

- **GET 请求**：
  ```java
  Mono<String> result = webClient.get()
      .uri("/users/{id}", 1)
      .retrieve()
      .bodyToMono(String.class);
  ```

- **POST 请求（发送 JSON）**：
  ```java
  
  **流式请求**
  Flux<Person> personFlux = ... ;
  
  Mono<Void> result = client.post()
  		.uri("/persons/{id}", id)
  		.contentType(MediaType.APPLICATION_STREAM_JSON)
  		.body(personFlux, Person.class)
  		.retrieve()
  		.bodyToMono(Void.class);
  
  **JSON请求**
  Mono<User> result = webClient.post()
      .uri("/users")
      .bodyValue(new User("John", "Doe"))//等同于.body(personMono, Person.class)
      .retrieve()
      .bodyToMono(User.class);
  ```

#### 4. 处理响应

响应可以通过 `retrieve()` 方法链处理，然后使用 `bodyToMono()` 或 `bodyToFlux()` 来获取响应体：

- **同步处理**：
  ```java
  String result = resultMono.block();
  ```

- **异步处理**：
  ```java
  resultMono.subscribe(
      response -> { /* 处理响应 */ },
      error -> { /* 处理错误 */ }
  );
  ```

* 处理400、500异常

  ```java
  Mono<Person> result = client.get()
  		.uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
  		.retrieve()
  		.onStatus(HttpStatusCode::is4xxClientError, response -> ...)
  		.onStatus(HttpStatusCode::is5xxServerError, response -> ...)
  		.bodyToMono(Person.class);
  ```

  

#### 5. 配置和自定义

`WebClient` 可以配置超时、默认头、过滤器等：

例如以下选项：

uriBuilderFactory：自定义的uriBuilderFactory，用作基本URL。

defaultUriVariables：展开URI模板时使用的默认值。

defaultHeader：每个请求的标头。

defaultCookie：每个请求的Cookie。

defaultRequest：消费者自定义每个请求。

filter：对每个请求进行客户端筛选。

exchange策略：HTTP消息读写器自定义。

clientConnector:HTTP客户端库设置。

- **连接超时

  ```java
  import io.netty.channel.ChannelOption;
  
  HttpClient httpClient = HttpClient.create()
  		.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);
  
  WebClient webClient = WebClient.builder()
  		.clientConnector(new ReactorClientHttpConnector(httpClient))
  		.build();
  
  ```

- **读写超时

  ```java
  import io.netty.handler.timeout.ReadTimeoutHandler;
  import io.netty.handler.timeout.WriteTimeoutHandler;
  
  HttpClient httpClient = HttpClient.create()
  		.doOnConnected(conn -> conn
  				.addHandlerLast(new ReadTimeoutHandler(10))
  				.addHandlerLast(new WriteTimeoutHandler(10)));
  
  // Create WebClient...
  ```

  

  - 每个请求设置超时

    ```java
    return webClient
        .method(this.httpMethod)
        .uri(this.uri)
        .headers(httpHeaders -> httpHeaders.addAll(additionalHeaders))
        .bodyValue(this.requestEntity)
        .retrieve()
        .bodyToMono(responseType)
        .timeout(Duration.ofSeconds(2)) 
    ```

    

  

- **配置请求头：
  
  ```java
  @Configuration
  public class WebClientConfig {
      @Bean
      public WebClient webClient() {
          return WebClient.builder()
              .baseUrl("https://echo.apifox.com")
              .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
              .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
              .build();
      }
  }
  
  ```
  
- - **添加过滤器**：
```
  
  ```java
  WebClient webClient = WebClient.builder()
      .filter((request, next) -> {
          ClientRequest newRequest = ClientRequest.from(request)
              .header("header1", "value1")
              .build();
          return next.exchange(newRequest);
      })
      .build();
```

`WebClient` 提供了强大的错误处理机制，可以方便地处理网络请求中出现的错误和异常情况。它支持自定义错误处理器，可以根据需要定义错误处理逻辑。同时，`WebClient` 可以轻松地与 Spring Boot 的其他组件集成，如 Spring Data、Spring Security 等，使得在构建基于微服务的响应式应用程序时更加方便和灵活。