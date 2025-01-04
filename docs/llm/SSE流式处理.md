## SSE（Server-Sent Events）流式处理


### 背景介绍
在使用ChatGPT等类似应用时，模型的回复是逐字逐句生成的，而不是一次性生成整个回答。这是因为语言模型需要在每个时间步骤预测下一个最合适的单词或字符。逐字生成的方式可以实现更快的交互响应，提高用户体验，避免长时间等待。

openai协议的completion API支持流式输出，当stream参数设置为true时，将会使用SSE技术流式输出结果。

这里以千问为例：

```
curl -X POST https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions \
-H "Authorization: Bearer $DASHSCOPE_API_KEY" \
-H "Content-Type: application/json" \
-d '{
    "model": "qwen-plus",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user", 
            "content": "你是谁？"
        }
    ],
    "stream":true,
    "stream_options":{
        "include_usage":true
    }
}'

```

返回结果，以[DONE]为结束标志

```
data: {"choices":[{"delta":{"content":"","role":"assistant"},"index":0,"logprobs":null,"finish_reason":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"finish_reason":null,"delta":{"content":"我是"},"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"delta":{"content":"来自"},"finish_reason":null,"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"delta":{"content":"阿里"},"finish_reason":null,"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"delta":{"content":"云的超大规模语言"},"finish_reason":null,"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"delta":{"content":"模型，我叫通义千问"},"finish_reason":null,"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"delta":{"content":"。"},"finish_reason":null,"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[{"finish_reason":"stop","delta":{"content":""},"index":0,"logprobs":null}],"object":"chat.completion.chunk","usage":null,"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: {"choices":[],"object":"chat.completion.chunk","usage":{"prompt_tokens":22,"completion_tokens":17,"total_tokens":39},"created":1726132850,"system_fingerprint":null,"model":"qwen-max","id":"chatcmpl-428b414f-fdd4-94c6-b179-8f576ad653a8"}

data: [DONE]
```


### 1. 什么是SSE？

SSE（Server-Sent Events）是一种允许服务器向客户端主动推送数据的技术。与传统的轮询相比，SSE提供了一种更高效、更实时的数据更新方式。它基于HTTP协议，使用一个持久的连接来发送数据流。

### 2. SSE的工作原理

- **客户端请求**：客户端通过HTTP请求订阅一个SSE端点。
- **服务器响应**：服务器接受请求并保持连接打开，随时准备发送数据。
- **数据推送**：服务器在有新数据时，以特定格式发送数据到客户端。
- **客户端接收**：客户端接收数据并根据需要处理（例如，更新UI）。

### 3. SSE的数据格式

SSE数据以纯文本格式发送，每条消息以一对换行符结束，并以一对换行符开始新的一条消息。每条消息可以包含以下字段：

- `data:` 实际的数据内容。
- `event:` 事件类型，客户端可以根据事件类型执行不同的操作。
- `id:` 消息的唯一标识符，用于客户端重连时确定最后接收的消息。
- `retry:` 客户端在连接断开后重连的时间间隔。

### 4. 使用WebFlux实现SSE

Spring WebFlux是Spring框架的响应式编程模块，它支持非阻塞I/O和SSE。

#### 4.1 引入依赖

首先，确保你的项目中引入了WebFlux的依赖：

```xml
<!-- Maven依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

#### 4.2 创建SSE Controller

创建一个控制器来处理SSE请求：

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;

@RestController
public class SseController {

    @GetMapping(value = "/stream-sse", produces = "text/event-stream")
    public Flux<String> streamSse() {
        // 模拟数据流
        return Flux.intervalMillis(1000)
                .map(sequence -> "data: Message " + sequence + "\n\n");
    }
}
```

#### 4.3 处理连接和错误

在实际应用中，你可能需要处理连接的建立和错误情况：

```java
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Mono;

@RestController
public class SseController {

    @GetMapping(value = "/stream-sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Mono<ServerResponse> streamSse() {
        return ServerResponse.ok()
                .body(Flux.intervalMillis(1000)
                        .map(sequence -> "data: Message " + sequence + "\n\n"),
                        MediaType.TEXT_EVENT_STREAM);
    }
}
```

### 5. 客户端订阅SSE

客户端可以使用JavaScript的`EventSource`接口订阅SSE：

```javascript
const eventSource = new EventSource('/stream-sse');
eventSource.onmessage = function(event) {
    console.log('New message:', event.data);
};
```

### 6. 注意事项

- **错误处理**：确保在服务器端妥善处理错误，并在客户端做好错误重连的逻辑。
- **性能考虑**：SSE适用于需要实时更新的场景，但也要考虑到服务器的负载和资源消耗。
- **安全性**：确保SSE端点的安全性，比如使用HTTPS来防止中间人攻击。

通过上述步骤，你可以在Java后端项目中实现SSE流式处理，为客户端提供实时的数据更新。


### 7. 如何接收流式返回

Java中可以用Spring 推荐的webclient，后续会出一篇webclient详细介绍。
大概使用如下：

```

Flux<Quote> result = client.get()
		.uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
		.retrieve()
		.bodyToFlux(Quote.class);

```





> 欢迎关注公众号：码上烟火