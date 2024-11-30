## Spring AI
Spring AI 项目由 Spring 官方开源并维护的 AI 应用开发框架，该项目目标是简化包含人工智能（AI）功能的应用程序的开发，避免不必要的复杂性。该项目从著名的 Python 项目（例如 LangChain 和 LlamaIndex）中汲取灵感，但 Spring AI 并非这些项目的直接移植，该项目的成立基于这样的信念：下一波生成式 AI 应用将不仅面向 Python 开发人员，还将遍及多种编程语言。

> 要求JDK17以上

## Spring AI Alibaba 
Spring AI Alibaba 开源项目基于 Spring AI 构建，是阿里云通义系列模型及服务在 Java AI 应用开发领域的最佳实践，提供高层次的 AI API 抽象与云原生基础设施集成方案，帮助开发者快速构建 AI 应用。



### 功能介绍
* 开发复杂 AI 应用的高阶抽象 Fluent API — ChatClient
* 提供多种大模型服务对接能力，包括主流开源与阿里云通义大模型服务（百炼）等
* 支持的模型类型包括聊天、文生图、音频转录、文生语音等
* 支持同步和流式 API，在保持应用层 API 不变的情况下支持灵活切换底层模型服务，支持特定模型的定制化能力（参数传递）
* 支持 Structured Output，即将 AI 模型输出映射到 POJOs
* 支持矢量数据库存储与检索
* 支持函数调用 Function Calling
* 支持构建 AI Agent 所需要的工具调用和对话内存记忆能力
* 支持 RAG 开发模式，包括离线文档处理如 DocumentReader、Splitter、Embedding、VectorStore 等，支持 Retrieve 检索





## chat 项目创建

> 注意：这里使用的是通义千问的api-key

1. 可以通过 https://start.spring.io/ 创建一个包含 Spring AI 的初始项目;
2. 引入springAI alibaba依赖；（自动引入SpringAI）
3. 配置yaml文件api-key(阿里百炼平台开通获取)
4. 注入ChatClient即可对话

```
spring:
  application:
    name: ai-test

  ai:
    dashscope:
      api-key: *****

```

> 可以看到代码很简洁

```

private ChatClient chatClient;

    public AiController(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    @GetMapping("/chat")
    public String chat(String input) {
        return this.chatClient.prompt()
                .user(input)
                .call()
                .content();
    }
```




## Function call流程
> 大模型需要支持此功能



### Spring AI 实现流程



### 步骤
1. 注册函数并调用大模型
2. 校验是否返回tool结果并组装参数
3. 调用获取function结果
4. 根据function结果，用户输入再次调用大模型
5. 返回最终大模型结果

> SpringAI 只需定义一个返回 java.util.Function 的 @Bean 定义，并在调用 ChatModel 时将 bean名称作为选项进行注册。

> FunctionCallback.java 接口和配套的 FunctionCallbackWrapper.java 工具类包含了底层实现代码，它们是简化Java 回调函数的实现和注册的关键。


```
protected ToolResponseMessage executeFunctions(AssistantMessage assistantMessage) {

    List<ToolResponseMessage.ToolResponse> toolResponses = new ArrayList<>();

    for (AssistantMessage.ToolCall toolCall : assistantMessage.getToolCalls()) {

       var functionName = toolCall.name();
       String functionArguments = toolCall.arguments();

       if (!this.functionCallbackRegister.containsKey(functionName)) {
          throw new IllegalStateException("No function callback found for function name: " + functionName);
       }

       String functionResponse = this.functionCallbackRegister.get(functionName).call(functionArguments);

       toolResponses.add(new ToolResponseMessage.ToolResponse(toolCall.id(), functionName, functionResponse));
    }

    return new ToolResponseMessage(toolResponses, Map.of());
}


```





### 使用ChatClient调用function

1. 声明


```
public class MockWeatherService implements Function<MockWeatherService.Request, Response> {

    @Override
    public Response apply(Request request) {
        if (request.city().contains("杭州")) {
            return new Response(String.format("%s%s晴转多云, 气温32摄氏度。", request.date(), request.city()));
        }
        else if (request.city().contains("上海")) {
            return new Response(String.format("%s%s多云转阴, 气温31摄氏度。", request.date(), request.city()));
        }
        else {
            return new Response(String.format("暂时无法查询%s的天气状况。", request.city()));
        }
    }

    @JsonInclude(JsonInclude.Include.NON_NULL)
    @JsonClassDescription("根据日期和城市查询天气")
    public record Request(
            @JsonProperty(required = true, value = "city") @JsonPropertyDescription("城市, 比如杭州") String city,
            @JsonProperty(required = true, value = "date") @JsonPropertyDescription("日期, 比如2024-08-22") String date) {
    }


}


```

2. 调用

```

 @GetMapping("/function/chat")
    public String functionChat(String subject) {
        return chatClient.prompt()
                .function("getWeather", "根据城市查询天气", new MockWeatherService())
                .user(subject)
                .call()
                .content();
    }
```

### 使用ChatModel调用function

1. 声明

```
@Service
public class MockOrderService {
    public Response getOrder(Request request) {
        String productName = "尤尼克斯羽毛球拍";
        return new Response(String.format("%s的订单编号为%s, 购买的商品为: %s", request.userId, request.orderId, productName));
    }

    @JsonInclude(JsonInclude.Include.NON_NULL)
    public record Request(
            @JsonProperty(required = true, value = "orderId") @JsonPropertyDescription("订单编号, 比如1001***") String orderId,
            @JsonProperty(required = true, value = "userId") @JsonPropertyDescription("用户编号, 比如2001***") String userId) {
    }
    public record Response(String description) {
    }
}

```

2. 注入

```
@Configuration
public class AiConfig {
    @Bean
    @Description("根据用户编号和订单编号查询订单信息")  //function的描述
    public Function<MockOrderService.Request, MockOrderService.Response> getOrderFunction(MockOrderService mockOrderService) {
        return mockOrderService::getOrder;
    }

}

```

3. 调用

```

   @GetMapping("/v2/function/chat")
    public String functionChat2(String input) {
        UserMessage userMessage = new UserMessage(input);
        ChatResponse response = chatModel.call(new Prompt(List.of(userMessage),
                DashScopeChatOptions.builder().withFunction("getOrderFunction").build()));
        return response.getResult().getOutput().getContent();
    }
```