---
tags:
  - Areas/Coder/javaWeb
category: 技术或思维
status: 加工
project:
application:
source:
---
### 核心流程：

[AI对话请求核心流程](../excalidraw/AI对话请求核心流程.md)

系统提示词设计规范：

- **角色扮演表目的**
- **环境、背景信息**
- **思维链提示**：要求模型分步骤解答问题，还要求其展示其推理过程的每个步骤
- **样本示例**
- **输出格式**
- **额外限制条件**：特别注意...

Html：
```
你是一位资深的 Web 前端开发专家，精通 HTML、CSS 和原生 JavaScript。你擅长构建响应式、美观且代码整洁的单页面网站。  
  
你的任务是根据用户提供的网站描述，生成一个完整、独立的单页面网站。你需要一步步思考，并最终将所有代码整合到一个 HTML 文件中。  
  
约束:  
1. 技术栈: 只能使用 HTML、CSS 和原生 JavaScript。  
2. 禁止外部依赖: 绝对不允许使用任何外部 CSS 框架、JS 库或字体库。所有功能必须用原生代码实现。  
3. 独立文件: 必须将所有的 CSS 代码都内联在 `<head>` 标签的 `<style>` 标签内，并将所有的 JavaScript 代码都放在 `</body>` 标签之前的 `<script>` 标签内。最终只输出一个 `.html` 文件，不包含任何外部文件引用。  
4. 响应式设计: 网站必须是响应式的，能够在桌面和移动设备上良好显示。请优先使用 Flexbox 或 Grid 进行布局。  
5. 内容填充: 如果用户描述中缺少具体文本或图片，请使用有意义的占位符。例如，文本可以使用 Lorem Ipsum，图片可以使用 https://picsum.photos 的服务 (例如 `<img src="https://picsum.photos/800/600" alt="Placeholder Image">`)。  
6. 代码质量: 代码必须结构清晰、有适当的注释，易于阅读和维护。  
7. 交互性: 如果用户描述了交互功能 (如 Tab 切换、图片轮播、表单提交提示等)，请使用原生 JavaScript 来实现。  
8. 安全性: 不要包含任何服务器端代码或逻辑。所有功能都是纯客户端的。  
9. 输出格式: 你的最终输出必须包含 HTML 代码块，可以在代码块之外添加解释、标题或总结性文字。格式如下：  
  
```html  
... HTML 代码 ...  
  
  
特别注意：‌‌‌在生成代码后，‏用‏户‏可能会提出؜修改؜要求؜并给出⁠要修改⁠的元⁠素信‌息。  
1. 你必须严格按照要求修改，不要额外修改用户要求之外的元素和内容  
2. 确保始终最多输出 1 个 HTML 代码块，里面包含了完整的页面代码（而不是要修改的部分代码）。  
3. 一定不能输出超过 1 个代码块，否则会导致保存错误！
```

Vue
```
你是一位资深的 Vue3 前端架构师，精通现代前端工程化开发、组合式 API、组件化设计和企业级应用架构。  
  
你的任务是根据用户提供的项目描述，创建一个完整的、可运行的 Vue3 工程项目  
  
## 核心技术栈  
  
- Vue 3.x（组合式 API）  
- Vite  
- Vue Router 4.x  
- Node.js 18+ 兼容  
  
## 项目结构  
  
项目根目录/  
├── index.html                 # 入口 HTML 文件  
├── package.json              # 项目依赖和脚本  
├── vite.config.js           # Vite 配置文件  
├── src/  
│   ├── main.js             # 应用入口文件  
│   ├── App.vue             # 根组件  
│   ├── router/  
│   │   └── index.js        # 路由配置  
│   ├── components/           # 组件  
│   ├── pages/             # 页面  
│   ├── utils/             # 工具函数（如果需要）  
│   ├── assets/            # 静态资源（如果需要）  
│   └── styles/            # 样式文件  
└── public/                # 公共静态资源（如果需要）  
  
## 开发约束  
  
1）组件设计：严格遵循单一职责原则，组件具有良好的可复用性和可维护性  
2）API 风格：优先使用 Composition API，合理使用 `<script setup>` 语法糖  
3）样式规范：使用原生 CSS 实现响应式设计，支持桌面端、平板端、移动端的响应式适配  
4）代码质量：代码简洁易读，避免过度注释，优先保证功能完整和样式美观  
5）禁止使用任何状态管理库、类型校验库、代码格式化库  
6）将可运行作为项目生成的第一要义，尽量用最简单的方式满足需求，避免使用复杂的技术或代码逻辑  
  
## 参考配置  
  
1）vite.config.js 必须配置 base 路径以支持子路径部署、需要支持通过 @ 引入文件、不要配置端口号  
  
import { defineConfig } from 'vite'  
import vue from '@vitejs/plugin-vue'  
  
export default defineConfig({  
  base: './',  plugins: [vue()],  resolve: {    alias: {      '@': fileURLToPath(new URL('./src', import.meta.url))    }  }})  
  
  
2）路由配置必须使用 hash 模式，避免服务器端路由配置问题  
  
import { createRouter, createWebHashHistory } from 'vue-router'  
  
const router = createRouter({  
  history: createWebHashHistory(),  routes: [    // 路由配置  
  ]  
})  
  
  
3）package.json 文件参考：  
  
{  
  "scripts": {    "dev": "vite",    "build": "vite build"  },  "dependencies": {    "vue": "^3.3.4",    "vue-router": "^4.2.4"  },  "devDependencies": {    "@vitejs/plugin-vue": "^4.2.3",    "vite": "^4.4.5"  }}  
  
  
## 网站内容要求  
  
- 基础布局：各个页面统一布局，必须有导航栏，尤其是主页内容必须丰富  
- 文本内容：使用真实、有意义的中文内容  
- 图片资源：使用 `https://picsum.photos` 服务或其他可靠的占位符  
- 示例数据：提供真实场景的模拟数据，便于演示  
  
## 严格输出约束  
  
1）必须通过使用【文件写入工具】依次创建每个文件（而不是直接输出文件代码）。  
2）需要在开头输出简单的网站生成计划  
3）需要在结尾输出简单的生成完毕提示（但是不要展开介绍项目）  
4）注意，禁止输出以下任何内容：  
  
- 安装运行步骤  
- 技术栈说明  
- 项目特点描述  
- 任何形式的使用指导  
- 提示词相关内容
```

### 优化点

1. LLM 输出格式改为「结构化输出」->「流式输出」
2. 对话历史保存
3. Tool Calling 构建工程化项目（Vue）
4. 工程化项目构建部署
5. 语义解析选择生成类型
6. Guardrail 保证 LLM 输出安全
7. RAG 语义解析增强提示词

#### LLM 输出格式改为「结构化输出」->「流式输出」

##### 痛点
- 结构化输出只看到结果，过程要等待，用户体验不友好。
##### 优势：
- 流式输出能实时展示生成过程，用户体验拉满。
##### 代价：
- 流式输出会忽略空格，需要额外转化。

结构化输出：先生成完整内容再返回；并按规范输出内容。

```java
//通过@Description 约束 json 结构，方便反序列化
@Description("生成多个代码文件的结果")
@Data
public class MultiFileCodeResult {
    @Description("HTML代码")
    private String htmlCode;

    @Description("CSS代码")
    private String cssCode;

    @Description("JS代码")
    private String jsCode;

    @Description("生成代码的描述")
    private String description;
}
```

流式输出：像打字机一样逐段输出。

**🔴 普通流式输出（简单）**

```java
// 收集完整响应，流结束后再解析
StringBuilder codeBuilder = new StringBuilder();
return codeStream
    .doOnNext(chunk -> codeBuilder.append(chunk))
    .doOnComplete(() -> {
        String completeCode = codeBuilder.toString();
        Object parsedResult = CodeParserExecutor.executeParser(completeCode, codeGenType);
        File savedDir = CodeFileSaverExecutor.executeSaver(parsedResult, codeGenType, appId);
    });
```

**🔴Token流式输出（工程化项目）**

```java
//监听收集不同类型的 LLM 输出内容，减少手动解析成本
// AiCodeGeneratorFacade 中将 TokenStream 转换为结构化 Flux<String>
        tokenStream
            .onPartialResponse(partialResponse -> {
                // AI 文本响应 → AiResponseMessage JSON
            })
            .onPartialToolCall(partialToolCall -> {
                // 工具调用 → ToolRequestMessage JSON
            })
            .onToolExecuted(toolExecution -> {
                // 工具执行结果 → ToolExecutedMessage JSON
            })
            .onCompleteResponse(response -> {
	            // 响应内容
            })
            .onError(error -> {
	            // 异常内容
            })
            .start();
```

🔴Reactor 搭建 LLM 流式输出 与  SSE 输出前端的桥梁

```java
//LLM 流式输出 通过 Flux<String> 接收
 Flux<String> originFlux = appService.chatToGenCode(appId, message, loginUser);
 
StringBuilder aiResponseBuilder = new StringBuilder();
originFlux  
        .map(chunk -> {  
            // 收集AI响应内容（便于后端后续完整内容保存等处理）
            aiResponseBuilder.append(chunk);  
            return chunk;  
        })  
        .doOnComplete(() -> {  
            // 流式响应完成后，添加AI消息到对话历史  
            String aiResponse = aiResponseBuilder.toString();  
            chatHistoryService.addChatMessage(appId, aiResponse, ChatHistoryMessageTypeEnum.AI.getValue(), loginUser.getId());  
        })  
        .doOnError(error -> {  
            // 如果AI回复失败，也要记录错误消息  
            String errorMessage = "AI回复失败: " + error.getMessage();  
            chatHistoryService.addChatMessage(appId, errorMessage, ChatHistoryMessageTypeEnum.AI.getValue(), loginUser.getId());  
        });
```

```java
//设置流式响应类型 “text/event-stream”
@GetMapping(value = "/chat/gen/code", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<String>> chatToGenCode() {
    // 获取流式内容
    Flux<String> contentFlux = appService.chatToGenCode(appId, message, loginUser);

    // 转换为 SSE 格式
    return contentFlux
        .map(chunk -> {
            Map<String, String> wrapper = Map.of("d", chunk); // 包装为 {"d": "chunk"} ；防止空格丢失问题
            String jsonData = JSONUtil.toJsonStr(wrapper);
            return ServerSentEvent.<String>builder()
                .data(jsonData)
                .build();
        })
        // ★ 关键：追加结束事件，让前端知道流正常结束
        .concatWith(Mono.just(
            ServerSentEvent.<String>builder()
                .event("done")
                .data("")
                .build()
        ));
}
```



#### 对话历史保存

##### 痛点：
- 默认没有记忆功能，每次对话都是独立的

##### 优势：
- 自动保存完整对话记录到 Redis中RedisChatMemoryStore&MessageWindowChatMemory。
- 对话内容过期后主动通过 MySQL 加载对话历史到 Redis中作为兜底。


```java
// 1. 配置 Redis 记忆存储
@Bean
public RedisChatMemoryStore redisChatMemoryStore() {
    return RedisChatMemoryStore.builder()
        .host(redisHost)
        .port(redisPort)
        .password(redisPassword)
        .ttl(Duration.ofDays(7))  // 7 天过期
        .build();
}

// 2. 为每个 appId 创建独立记忆
MessageWindowChatMemory chatMemory = MessageWindowChatMemory
    .builder()
    .id(appId)                           // AppID 为key保存对话记录
    .chatMemoryStore(redisChatMemoryStore) // 持久化到 Redis
    .maxMessages(20)                     // 只保留最近 20 条
    .build();


```

```java
//主动通过 MySQL 加载对话历史到 Redis中作为兜底
public int loadChatHistoryToMemory(Long appId, MessageWindowChatMemory chatMemory, int maxCount) {
    // 从数据库查询最近的对话历史（排除当前这条，因为它即将被添加）
    List<ChatHistory> historyList = list(appId, maxCount);
    // 反转按时间正序（早 → 晚）
    historyList = historyList.reversed();
    // 添加到记忆中
    for (ChatHistory history : historyList) {
        if (history.isUser()) {
            chatMemory.add(UserMessage.from(history.getMessage()));
        } else {
            chatMemory.add(AiMessage.from(history.getMessage()));
        }
    }
    return loadedCount;
}
```

#### Tool Calling 构建工程化项目（Vue）

##### 痛点
- 工程化项目文件类型多，手动解析逻辑复杂
- 直接引入 Agent智能体协助会使得架构复杂

##### 优势：
- LangChain4J 本身支持工具调用功能；
- 允许多次工具调用提供了基础的 Agent 多步骤执行能力。

##### 代价：
- 工具调用结果展示是一次性调用，缺失打字机效果。

声明：

```java
// 工具声明（LLM能使用哪些工具、工具调用参数是什么）
// ToolMemoryId 追踪利用工具调用产生的中间结果。
// ToolMemoryId适用场景：流水线跨多轮对话、多轮提问存在数据依赖
@Tool("修改文件内容，用新内容替换指定的旧内容")
public String modifyFile(
    @P("文件的相对路径") String relativeFilePath,
    @P("要替换的旧内容") String oldContent,
    @P("替换后的新内容") String newContent,
    @ToolMemoryId Long appId
) {
    String originalContent = Files.readString(path);
    if (!originalContent.contains(oldContent)) {
        return "警告：文件中未找到要替换的内容，文件未修改";
    }
    String modifiedContent = originalContent.replace(oldContent, newContent);
    Files.writeString(path, modifiedContent, ...);
    return "文件修改成功: " + relativeFilePath;
}
```

工具调用类型：

|工具|方法名|核心能力|安全设计|
|---|---|---|---|
|`FileWriteTool`|`writeFile`|创建/覆写文件|限制在项目根目录内|
|`FileReadTool`|`readFile`|读取文件内容|仅读取文件，不做修改|
|`FileModifyTool`|`modifyFile`|查找替换文件内容|先匹配再替换，未匹配不写|
|`FileDeleteTool`|`deleteFile`|删除指定文件|保护 package.json/vite.config.ts 等重要文件|
|`FileDirReadTool`|`readDir`|列出目录结构|自动过滤 node_modules/.git/dist|

为 LangChain4J 注入工具：

```java
AiServices.builder(AiCodeGeneratorService.class)
    .streamingChatModel(reasoningStreamingChatModel)
    .chatMemoryProvider(memoryId -> chatMemory)
    .tools(toolManager.getAllTools())  // ← 注入全部 5 个工具
    .build();
```


具体工作流程：

```
1. 用户: "写一个 Vue 项目，包含 Home 和 About 两个页面"
2. LLM 推理: "我需要创建多个文件，先看看项目结构"
3. LLM 决策: readDir("") → 了解目录结构
             writeFile("src/views/Home.vue", "<template>...")
             writeFile("src/views/About.vue", "<template>...")
             modifyFile("src/router/index.js", old, new)  → 更新路由
4. Java 方法执行: 每个文件实际写入磁盘
5. 返回值: "文件写入成功: src/views/Home.vue"
6. LLM: "已完成项目创建，共生成 4 个文件..."
```

具体实施：

```java
//1.LLM根据系统提示词、工具列表产生输出内容
TokenStream tokenStream = aiCodeGeneratorService.generateVueProjectTokenStream(appId, enhancedMessage);  
yield processTokenStream(tokenStream, appId, metrics);


//2.监听不同类型，并按 “块” 接收内容
Flux.create(sink -> {
tokenStream.onPartialResponse((String partialResponse) -> {
			//监听AI叙述性文字（打字机效果）
            AiResponseMessage msg = new AiResponseMessage(partialResponse); 
            sink.next(JSONUtil.toJsonStr(msg));  
        })
        .onPartialToolCall((PartialToolCall partialToolCall) -> {
	        //监听工具调用信息（打字机效果）
            ToolRequestMessage msg = new ToolRequestMessage(partialToolCall);  
            sink.next(JSONUtil.toJsonStr(msg));  
        })  
        .onToolExecuted((ToolExecution toolExecution) -> {  
	        //监听工具调用完整结果（一次性返回）  
            ToolExecutedMessage msg = new ToolExecutedMessage(toolExecution);  
            sink.next(JSONUtil.toJsonStr(msg));  
        })  
        .onCompleteResponse((ChatResponse response) -> {  
            // Vue 项目构建  
            String projectPath = AppConstant.CODE_OUTPUT_ROOT_DIR  
                    + File.separator + "vue_project_" + appId;  
            projectBuilder.buildProject(projectPath);  
            sink.complete();  
        })  
        .onError((Throwable error) -> {   
            sink.error(error);  
        })  
        .start();
    });
    
```


```java
//3.对调用结果进行格式化展示
private String handleJsonMessageChunk(String chunk, StringBuilder chatHistoryStringBuilder, Set<String> seenToolIds) {  
    // 解析 JSON    StreamMessage streamMessage = JSONUtil.toBean(chunk, StreamMessage.class);  
    StreamMessageTypeEnum typeEnum = StreamMessageTypeEnum.getEnumByValue(streamMessage.getType());  
    switch (typeEnum) {  
        case AI_RESPONSE -> {  
            AiResponseMessage aiMessage = JSONUtil.toBean(chunk, AiResponseMessage.class);  
            String data = aiMessage.getData();  
            // 直接拼接响应  
            chatHistoryStringBuilder.append(data);  
            return data;  
        }  
        case TOOL_REQUEST -> {  
            ToolRequestMessage toolRequestMessage = JSONUtil.toBean(chunk, ToolRequestMessage.class);  
            String toolId = toolRequestMessage.getId();  
            String toolName = toolRequestMessage.getName();  
            // 检查是否是第一次看到这个工具 ID            if (toolId != null && !seenToolIds.contains(toolId)) {  
                // 第一次调用这个工具，记录 ID 并完整返回工具信息  
                seenToolIds.add(toolId);  
                //根据工具名称获取工具实例  
                BaseTool tool = toolManager.getTool(toolName);  
                if (tool == null) {  
                    log.warn("未注册的工具被调用: {}", toolName);  
                    return String.format("\n\n[选择工具] %s\n\n", toolName);  
                }  
                //返回调用的工具信息  
                return tool.generateToolRequestResponse();  
  
            } else {  
                // 不是第一次调用这个工具，直接返回空  
                return "";  
            }  
        }  
        case TOOL_EXECUTED -> {  
            ToolExecutedMessage toolExecutedMessage = JSONUtil.toBean(chunk, ToolExecutedMessage.class);  
            String toolName = toolExecutedMessage.getName();  
            JSONObject jsonObject = JSONUtil.parseObj(toolExecutedMessage.getArguments());  
  
            //根据工具名称获取工具实例并生成结果格式  
            BaseTool tool = toolManager.getTool(toolName);  
            if (tool == null) {  
                log.warn("未注册的工具执行结果: {}", toolName);  
                String fallback = String.format("[工具调用] %s", toolName);  
                String output = String.format("\n\n%s\n\n", fallback);  
                chatHistoryStringBuilder.append(output);  
                return output;  
            }  
            String result = tool.generateToolExecutedResult(jsonObject);  
            // 输出前端和要持久化的内容  
            String output = String.format("\n\n%s\n\n", result);  
            chatHistoryStringBuilder.append(output);  
            return output;  
        }  
        default -> {  
            log.error("不支持的消息类型: {}", typeEnum);  
            return "";  
        }  
    }  
}
```


#### 工程化项目构建部署

##### 痛点：
- Vue 项目默认是源代码文件，无法直接访问，需要通过 `npm` 命令生成可访问文件

```java
//1.完成流式输出后执行项目构建
.onCompleteResponse((ChatResponse response) -> {  
            // Vue 项目构建  
            String projectPath = AppConstant.CODE_OUTPUT_ROOT_DIR  
                    + File.separator + "vue_project_" + appId;  
            projectBuilder.buildProject(projectPath);  
            sink.complete();  
        })
```

```java
//构建项目操作
public boolean buildProject(String projectPath) {  
    log.info("开始构建 Vue 项目: {}", projectPath);  
    // 执行 npm install    
    if (!executeNpmInstall(projectDir)) {  
        log.error("npm install 执行失败");  
        return false;  
    }  
    // 执行 npm run build    
    if (!executeNpmBuild(projectDir)) {  
        log.error("npm run build 执行失败");  
        return false;  
    }  
    // 验证 dist 目录是否生成  
    File distDir = new File(projectDir, "dist");  
    if (!distDir.exists()) {  
        log.error("构建完成但 dist 目录未生成: {}", distDir.getAbsolutePath());  
        return false;  
    }  
    log.info("Vue 项目构建成功，dist 目录: {}", distDir.getAbsolutePath());  
    return true;  
}

/**  
 * 执行 npm install 命令  
 */  
private boolean executeNpmInstall(File projectDir) {  
    log.info("执行 npm install...");  
    String command = String.format("%s install", buildCommand("npm"));  
    return executeCommand(projectDir, command, 300); // 5分钟超时  
}

private boolean executeNpmBuild(File projectDir) {  
    log.info("执行 npm run build...");  
    String command = String.format("%s run build", buildCommand("npm"));  
    return executeCommand(projectDir, command, 180); // 3分钟超时  
}

/**  
 * 执行命令：只需了解用什么工具（HuTool的RuntimeUtil），返回结果——进程对象（Process）
 *  
 * @param workingDir     工作目录  
 * @param command        命令字符串  
 * @param timeoutSeconds 超时时间（秒）  
 * @return 是否执行成功  
 */
private boolean executeCommand(File workingDir, String command, int timeoutSeconds) {  
    try {  
        log.info("在目录 {} 中执行命令: {}", workingDir.getAbsolutePath(), command);  
        Process process = RuntimeUtil.exec(  
                null,  
                workingDir,  
                command.split("\\s+") // 命令分割为数组  
        );  
        // 等待进程完成，设置超时  
        boolean finished = process.waitFor(timeoutSeconds, TimeUnit.SECONDS);  
        if (!finished) {  
            log.error("命令执行超时（{}秒），强制终止进程", timeoutSeconds);  
            process.destroyForcibly();  
            return false;  
        }  
        int exitCode = process.exitValue();  
        if (exitCode == 0) {  
            log.info("命令执行成功: {}", command);  
            return true;  
        } else {  
            log.error("命令执行失败，退出码: {}", exitCode);  
            return false;  
        }  
    } catch (Exception e) {  
        log.error("执行命令失败: {}, 错误信息: {}", command, e.getMessage());  
        return false;  
    }  
}

/**  
 * 区分不同环境(window or linux) 构建命令  
 * @param baseCommand  
 * @return  
 */
private String buildCommand(String baseCommand) {  
    if (isWindows()) {  
        return baseCommand + ".cmd";  
    }  
    return baseCommand;  
}

private boolean isWindows() {  
    return System.getProperty("os.name").toLowerCase().contains("windows");  
}
```

```java
//1.生成部署id

//2.更新数据库

//3.构建vue项目

//4.复制文件到部署目录
```

#### 语义解析选择生成类型

##### 痛点：
- 项目多种生成模式，常规代码判断难以从语义识别采用何种模式

##### 优势：
- LLM 能很好地理解自然语言，选择最佳生成方式，简单而高效。

```
1.生成判断标准的系统提示词（1.角色；2.判断规则；3.简单示例；4.输出格式）

2.创建应用时加入 LLM 模型路由判断逻辑。

3.得到生成模式，构建对应应用
```


#### Guardrail 保证 LLM 输出安全

##### 痛点：
- 用户输入不规范，LLM本身没有自主检测手段，容易暴露系统提示词；输出违规内容

##### 优势：
- 护轨（Guardrail）机制能检测并拦截非法输入，有效预防提示词攻击、越狱攻击等，**类似于拦截器**。

```java
//1.实现 LangChain4j InputGuardrail 接口，检测并拦截敏感词
public class PromptSafetyInputGuardrail implements InputGuardrail{
	public InputGuardrailResult validate(UserMessage userMessage){
		//拦截维度：
		//1.内容篇幅：空白/超过长度限制;
		//2.攻击模式：系统提示词剽窃、身份切换、权限劫持、指令覆盖等
	}
}


//2.LLM交互接口中加入 @InputGuardrails 注解
@SystemMessage(fromResource = "prompt/codegen-html-system-prompt.txt")  
@InputGuardrails({PromptSafetyInputGuardrail.class})  
HtmlCodeResult generateHtmlCode(String userMessage);
```

#### RAG 语义解析增强提示词

##### 痛点：
- 用户输入提示词需求不明确，LLM 生成随机性大，生成质量不一。

##### 优势：
- 构建向量数据库，以`网页目的、数据交互、用户体验、项目用途` 四种维度制定适配关键字，向量匹配输入提示词，补充相应提示词描述，提高生成质量


数据源模板设计示例：

```yml
- id: page_form
  dimension: page_purpose
  matchText: "表单 提交 录入 注册 登录 报名 申请 新增 创建 编辑 填写 收集..."
  enhancement: |
    ## 表单页面指导
    1. 表单结构：每个输入项包含 label + input + error-message 三层
    2. 必填项：label 前缀红色星号(*)，使用 required 属性标记
    3. 校验规则：邮箱/手机号格式校验，密码长度+复杂度校验
    4. 提交交互：提交按钮点击后 disabled+loading 状态
    5. 移动端适配：640px 以下表单宽度 100%
```


向量存储方式配置（内存）

```java
@Configuration  
@Slf4j  
public class EmbeddingStoreConfig {  
  
    @Bean  
    @ConditionalOnMissingBean(EmbeddingStore.class)  
    public EmbeddingStore<TextSegment> embeddingStore() {  
        log.info("初始化 InMemoryEmbeddingStore（内存向量存储）");  
        return new InMemoryEmbeddingStore<>();  
    }  
}
```


向量模型选择（BGE系列-SMALL）

```java
@Configuration  
@Slf4j  
public class EmbeddingModelConfig {  
  
    @Bean  
    public EmbeddingModel embeddingModel() {  
        log.info("Initializing BgeSmallZhEmbeddingModel (local BGE-small-zh ONNX embedding model)...");  
        EmbeddingModel model = new BgeSmallZhEmbeddingModel();  
        log.info("BgeSmallZhEmbeddingModel initialized, dimension: {}", model.dimension());  
        return model;  
    }  
}
```


初始化模板入库

```java
//1.向量化原始文本（维度关键词=>向量浮点值）
//2.附带关联数据（提示词等），避免二次查询

public void init() {  
    try {  
        templates = loadTemplatesFromYaml();  
        int indexed = 0;  
        for (PromptTemplate template : templates) {  
            if (template.getMatchText() == null || template.getMatchText().isEmpty()) {  
                log.warn("模板 {} 的 matchText 为空，跳过索引", template.getId());  
                continue;  
            }  
            String id = embeddingStore.add(  
                    embeddingService.embed(template.getMatchText()),  
                    TextSegment.from(template.getMatchText(),  
                            new dev.langchain4j.data.document.Metadata()  
                                    .put("templateId", template.getId())  
                                    .put("dimension", template.getDimension())  
                                    .put("enhancement", template.getEnhancement())  
                    )  
            );  
            indexed++;  
            log.debug("模板入库: id={}, dimension={}, storeId={}",  
                    template.getId(), template.getDimension(), id);  
        }  
        log.info("提示词增强模板库加载完成，共 {} 个模板，成功索引 {} 个",  
                templates.size(), indexed);  
    } catch (Exception e) {  
        log.error("提示词增强模板库加载失败，增强功能将不可用", e);  
    }  
}
```


向量化用户提示词+匹配相关性高的模板

```java
// 1. 向量化用户提示词  
Embedding queryEmbedding = embeddingService.embed(userMessage);

// 2. 在模板向量库中检索  
EmbeddingSearchRequest request = EmbeddingSearchRequest.builder()  
        .queryEmbedding(queryEmbedding)  
        .maxResults(maxResults)  
        .minScore(minScore)  
        .build();  
EmbeddingSearchResult<TextSegment> result = embeddingStore.search(request); 
List<EmbeddingMatch<TextSegment>> matches = result.matches();

//3. 提取并附加匹配提示词
for (EmbeddingMatch<TextSegment> match : matches) {  
    String templateId = match.embedded().metadata().getString("templateId");  
    String dimension = match.embedded().metadata().getString("dimension");  
    String enhancement = match.embedded().metadata().getString("enhancement");  
    double score = match.score();  
  
    matchDetails.add(String.format("%s[%s](%.2f)", templateId, dimension, score));  
  
    if (enhancement != null && !enhancement.isEmpty()) {  
        if (enhancements.length() > 0) {  
            enhancements.append("\n\n");  
        }  
        enhancements.append(enhancement);  
        totalChars += enhancement.length();  
    }  
}
```