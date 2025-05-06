---
date: '2025-04-25T15:10:21+08:00'
draft: true
title: 'Spring_ai'
tags: ["LearningAI"]
categories: ["Learning"]
description: '基于Spring AI Alibaba调用阿里百炼平台各种模型、Function Call、RAG增强和Agent智能体'
ShowToc: true
TocOpen: true
---



---

### Spring Ai Alibaba Chat

#### 简单的流式大模型问答接口

🏷️**模型配置信息**

```yaml
spring:
  ai:
    dashscope:
      api-key: "sk-4d9ac0285a4e45cc9adcc73115de3da3"   # api key
      chat:
        options:
          model: "qwen-plus"   # 模型名称: qwen-plus, deepseek-r1
```

⭐**Api 接口代码**

```java
@RestController
@RequestMapping("/chat-memory")
public class ChatMemoryController {

    private final ChatClient chatClient;

    public ChatMemoryController(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel)
                .defaultAdvisors(new MessageChatMemoryAdvisor(new InMemoryChatMemory()))
                .defaultAdvisors()
                .build();
    }

    /**
     * 使用 Spring AI 提供的基于内存的 Chat memory方法
     */
    @GetMapping("/in-memory")
    public Flux<String> chatWithMemory(@RequestParam("prompt") String prompt,
                                       @RequestParam("chatId") String chatId,
                                       HttpServletResponse response) {
        response.setCharacterEncoding("UTF-8");

        return chatClient.prompt(prompt)
                .advisors(advisorSpec -> advisorSpec
                        .param(CHAT_MEMORY_CONVERSATION_ID_KEY, chatId)  // 上下文记忆ID，隔离不同的聊天上下文 
                        .param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 100))  // 检索长度
                .stream()
                .content();
    }
    
}
```



#### 多模型切换示例

```java
@RestController
@RequestMapping("/multi-model-chat-client")
public class multiModelChatClientController {

    private final Set<String> modelList = Set.of(
            "deepseek-r1",
            "deepseek-v3",
            "qwen-plus",
            "qwen-max"
    );

    private final ChatClient chatClient;

    public multiModelChatClientController(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel)
                .defaultAdvisors(new MessageChatMemoryAdvisor(new InMemoryChatMemory()))
                .build();
    }

    @GetMapping("/chat")
    public Flux<String> stream(
            @RequestParam("prompt") String prompt,
            @RequestParam("chatId") String chatId,
            @RequestParam(value = "modelName", required = false) String modelName,
            HttpServletResponse response
    ) {
        response.setCharacterEncoding("UTF-8");

        if (!modelList.contains(modelName)) {
            return Flux.just("model not exist!");
        }

        return chatClient.prompt(prompt)
                .options(ChatOptions.builder()  // 这里构建聊天选项，指定不同的模型
                        .withModel(modelName)  
                        .build())
                .advisors(advisorSpec -> advisorSpec  // 这里构建顾问参数，指定不同的聊天id和上下文长度
                        .param(CHAT_MEMORY_CONVERSATION_ID_KEY, chatId)
                        .param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 100))
                .stream()
                .content();
    }
}
```



### Spring AI Alibaba Image Gen

调用文生图模型示例

```java
@RestController
@RequestMapping("/image")
public class ImageController {

    private final ImageModel imageModel;

    public ImageController(ImageModel imageModel) {
        this.imageModel = imageModel;
    }

    @RequestMapping("/t2i/{prompt}")
    public void image(@PathVariable("prompt") String prompt, 
                      HttpServletResponse response) {
        ImageOptions imageOptions = ImageOptionsBuilder.builder()
                .model("wanx2.1-t2i-turbo")
                .build();
        ImageMessage imageMessage = new ImageMessage(prompt);
        ImagePrompt imagePrompt = new ImagePrompt(imageMessage, imageOptions);
        ImageResponse call = imageModel.call(imagePrompt);  // 这里会进行阻塞
        String imageUrl = call.getResult().getOutput().getUrl();
		
        // 下载图片并写入response中
        try {
            URL url = URI.create(imageUrl).toURL();
            InputStream in = url.openStream();

            response.setHeader("Content-Type", MediaType.IMAGE_PNG_VALUE);
            response.getOutputStream().write(in.readAllBytes());
            response.getOutputStream().flush();
        } catch (IOException e) {
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        }
    }
}
```



### Spring AI Alibaba TTS

调用Text2Speech模型，文本生成语音 - 1.0.0-M5.1版本

```java
@RestController
@RequestMapping("/tts")
public class Text2SpeechController {

    @Value("${spring.ai.dashscope.api-key}")
    private String APIKEY;

    @RequestMapping("/gen")
    public String ttsGeneration(@RequestParam("prompt") String prompt, 
                                HttpServletResponse response) {
        
        SpeechSynthesisParam param = SpeechSynthesisParam.builder()
                .text(prompt)
                .apiKey(APIKEY)
                .model("cosyvoice-v1")
                .build();

        SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer();

        ByteBuffer audioBuffer = speechSynthesizer.call(param);

        // 3. 设置响应头（关键步骤！）
        response.setContentType("audio/mpeg"); // 根据实际音频格式调整
        response.setHeader("Content-Disposition", "inline; filename=audio.mp3");

        // 4. 将 ByteBuffer 写入响应流, try-with-resources 自动关闭资源 out 流
        try (OutputStream out = response.getOutputStream()) {
            byte[] audioBytes = new byte[audioBuffer.remaining()];
            audioBuffer.get(audioBytes);
            out.write(audioBytes);
            out.flush();
        }catch (IOException e) {
            throw new RuntimeException(e);
        }

        return "finish";
    }

}
```



### Spring AI Alibaba Function Call

提供工具给大模型调用的示例

```yaml
spring:
  ai:
    dashscope:
      api-key: "sk-4d9ac0285a4e45cc9adcc73115de3da3"   # api key
      chat:
        options:
          model: "qwen-plus"   # 模型名称: qwen-plus, deepseek-r1
    alibaba:
      toolcalling:  # func call 工具
        time:
          enabled: true
        queryInfo:
          enabled: true
        weather:
          enabled: true
```

注入 工具Bean对象

```java
@Configuration
@ConditionalOnClass({QueryInfoService.class})
@ConditionalOnProperty(prefix = "spring.ai.alibaba.toolcalling.queryInfo", name = "enabled", havingValue = "true")
public class QueryInfoConfig {

    @Bean(name = "getQueryInfoFunction")
    @ConditionalOnMissingBean
    @Description("Get the information of a LongWei.")
    public QueryInfoService getQueryInfoFunction() {
        return new QueryInfoService();
    }

}
```

工具实现

```java
public class QueryInfoService implements Function<QueryInfoService.Request, QueryInfoService.Response> {

    @Override
    public Response apply(Request request) {
        String info = "LongWei is a graduate student and 23 years old.";
        return new Response(info);
    }

    @JsonInclude(JsonInclude.Include.NON_NULL)
    @JsonClassDescription("QueryInfoFunction Service API request")
    public record Request(@JsonProperty(required = true, value = "stuName") @JsonPropertyDescription("Student Name, such as LongWei") String stuName) {}
	
    @JsonClassDescription("QueryInfoFunction Service API response")
    public record Response(@JsonProperty(required = true, value = "description") @JsonPropertyDescription("A description containing the LongWei information") String description) {
    }

}
```

⭐ 注入工具给ChatModel调用

```java
@RestController
@RequestMapping("/func")
public class FuncCallController {

    private final ChatClient ChatClient;

    public FuncCallController(ChatClient.Builder chatClientBuilder) {
        this.dashScopeChatClient = chatClientBuilder.build();
    }

    /**
     * 调用工具版 - function
     */
    @GetMapping("/chat-tool-function")
    public Flux<String> chatTranslateFunction(@RequestParam(value = "query", defaultValue = "请告诉我现在北京时间几点了") String query) {
        return chatClient.prompt(query)
                .functions("getQueryInfoFunction")
                .stream()
                .content();
    }

}
```



---

### Spring AI Alibaba Multi-Modal 

调用多模态大模型，这里示例为图文大模型(支持图片+文本输入)



```java
@RestController
@RequestMapping("/multi-modal")
public class MultiModalController {

    private final ChatClient chatClient;

    private static final String DEFAULT_MODEL = "qwen-vl-max-latest";

    private static final String TEXT_MODEL = "qwen-plus";

    public MultiModalController(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel)
                .defaultAdvisors(new MessageChatMemoryAdvisor(new InMemoryChatMemory()))
                .build();
    }

    @GetMapping("/image")
    public Flux<String> multiModal(
            @RequestParam("prompt") String prompt,
            @RequestParam("chatId") String chatId,
            @RequestParam(value = "imageURL", required = false) String imageURL,
            HttpServletResponse response
    ) throws Exception {
        response.setCharacterEncoding("UTF-8");

        boolean flag = imageURL != null && !imageURL.isEmpty();

        UserMessage userMessage;
        if (flag) {
            List<Media> media = List.of(new Media(MimeTypeUtils.IMAGE_PNG, new URI(imageURL).toURL()));
            userMessage = new UserMessage(prompt, media);
            userMessage.getMetadata().put(DashScopeChatModel.MESSAGE_FORMAT, MessageFormat.IMAGE);
        } else {
            userMessage = new UserMessage(prompt);
        }

        return chatClient.prompt(new Prompt(
                userMessage,
                DashScopeChatOptions.builder()
                        .withModel(flag ? DEFAULT_MODEL : TEXT_MODEL)
                        .withMultiModel(flag)
                        .build()
                ))
                .advisors(advisorSpec -> advisorSpec
                        .param(CHAT_MEMORY_CONVERSATION_ID_KEY, chatId)
                        .param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 100))
                .stream().content();
    }

}
```



### Spring AI Alibaba Cloud Rag 

使用阿里提供的云知识库，进行Rag检索增强

```java
@RestController
@RequestMapping("/rag-chat")
public class CloudRagController {

    @Resource
    private ICloudRagService cloudRagService;

    @GetMapping("/bailian/knowledge/importDocument")
    public void importDocument() {
        cloudRagService.importDocuments();
    }

    @GetMapping("/bailian/knowledge/generate")
    public Flux<String> generate(@RequestParam(value = "message",
            defaultValue = "你好，请问你的知识库文档主要是关于什么内容的?") String message,
                                 HttpServletResponse response) {
        response.setCharacterEncoding("UTF-8");
        return cloudRagService.retrieve(message).map(x -> x.getResult().getOutput().getText());
    }
}
```

```java
@Slf4j
@Service
public class CloudRagService implements ICloudRagService {

    private static final String indexName = "Rag测试";  // 知识库名称

    @Value("classpath:juc.pdf")
    private Resource ragResourceFile;

    private static final String retrievalSystemTemplate = """
			Context information is below.
			---------------------
			{question_answer_context}
			---------------------
			Given the context and provided history information and not prior knowledge,
			reply to the user comment. If the answer is not in the context, inform
			the user that you can't answer the question.
			""";

    private final ChatClient chatClient;

    private final DashScopeApi dashscopeApi;

    public CloudRagService(ChatClient.Builder builder, DashScopeApi dashscopeApi) {
        DocumentRetriever retriever = new DashScopeDocumentRetriever(dashscopeApi,
                DashScopeDocumentRetrieverOptions.builder().withIndexName(indexName).build());
		// 注入检索增强配置
        this.dashscopeApi = dashscopeApi;
        this.chatClient = builder
                .defaultAdvisors(new DocumentRetrievalAdvisor(retriever, retrievalSystemTemplate))
                .build();
    }

	// api导入信息到知识库
    @Override
    public void importDocuments() {
        String path = saveToTempFile(ragResourceFile);

        DashScopeDocumentCloudReader reader = new DashScopeDocumentCloudReader(path, dashscopeApi, null);
        List<Document> documentList = reader.get();
        log.info("{} documents loaded and split", documentList.size());

        VectorStore vectorStore = new DashScopeCloudStore(dashscopeApi, new DashScopeStoreOptions(indexName));
        vectorStore.add(documentList);
        log.info("{} documents added to dashscope cloud vector store", documentList.size());
    }

    private String saveToTempFile(Resource springAiResource) {
        try {
            File tempFile = File.createTempFile("juc", ".pdf");
            tempFile.deleteOnExit();

            try (InputStream inputStream = springAiResource.getInputStream();
                 FileOutputStream outputStream = new FileOutputStream(tempFile)) {
                byte[] buffer = new byte[4096];
                int bytesRead;
                while ((bytesRead = inputStream.read(buffer)) != -1) {
                    outputStream.write(buffer, 0, bytesRead);
                }
            }

            return tempFile.getAbsolutePath();
        }
        catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Flux<ChatResponse> retrieve(String message) {
        return chatClient.prompt().user(message).stream().chatResponse();
    }
}
```



### Spring Ai  alibaba Agent

这里调用阿里百炼平台搭建的 Agent智能体对象

```yaml
spring:
  ai:
    dashscope:
      api-key: "sk-4d9ac0285a4e45cc9adcc73115de3da3"
      workspace-id: "llm-tc6cf2ai0o4spxq2"
      chat:
        options:
          model: "qwen-plus"
      agent:
        app-id: "cc619000958d409c8f96fd87608a490d"
```

```java
@Slf4j
@RestController
@RequestMapping("/agent")
public class TestAgentController {

    private String APP_ID = "cc619000958d409c8f96fd87608a490d";

    private DashScopeAgent agent;

    public TestAgentController(DashScopeAgentApi dashScopeAgentApi) {
        this.agent = new DashScopeAgent(dashScopeAgentApi);
    }

    @GetMapping("/bailian/agent/stream")
    public Flux<String> stream(@RequestParam(value = "prompt") String prompt,
                               HttpServletResponse response) {
        response.setCharacterEncoding("UTF-8");

        return agent.stream(new Prompt(prompt, DashScopeAgentOptions.builder().withAppId(APP_ID).build())).map(resp -> {
            AssistantMessage app_output = resp.getResult().getOutput();
            return app_output.getContent();
        });
    }
}
```



---

