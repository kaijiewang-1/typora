```java
@ConfigurationProperties(prefix = "ruisa.agent")
public class AgentProxyProperties {

    /**
     * Python FastAPI base URL (no trailing slash), e.g. http://127.0.0.1:8000
     */
    private String baseUrl = "http://127.0.0.1:8000";

    public String getBaseUrl() {
        return baseUrl;
    }

    public void setBaseUrl(String baseUrl) {
        this.baseUrl = baseUrl;
    }
}
```

## **@ConfigurationProperties 注解：**

- **作用**：将配置文件（application.yml 或 application.properties）中的属性自动绑定到这个类的字段

- prefix = "ruisa.agent"

  ：指定属性的前缀

  - 只会绑定以 `ruisa.agent` 开头的配置项
  - 例如：`ruisa.agent.base-url` 会映射到 `baseUrl` 字段

**工作原理：**

```
yaml# application.yml 中的配置
ruisa:
  agent:
    base-url: http://192.168.1.100:8000
```

↓ Spring Boot 自动绑定 ↓

```
java

AgentProxyProperties.baseUrl = "http://192.168.1.100:8000"
```

## pom.xml

```xml
pom.xml 核心要素
│
├── 项目身份 (GAV)
│   ├── groupId: com.ruisa
│   ├── artifactId: ruisa-agent-gateway
│   └── version: 1.0.0
│
├── 继承关系
│   └── spring-boot-starter-parent 3.3.5
│       ├── 统一管理依赖版本
│       └── 默认插件配置
│
├── 技术栈
│   ├── Java 17
│   ├── Spring Web (HTTP/REST)
│   ├── WebSocket (实时通信)
│   └── Testing (JUnit/Mockito)
│
└── 构建工具
    └── spring-boot-maven-plugin
        ├── 打包可执行 JAR
        └── 一键启动应用
```



```java
@SpringBootApplication //@SpringBootApplication 自动扫描并注册所有带@component等注解的类为bean
@EnableConfigurationProperties(AgentProxyProperties.class)//将agentproxyproperties 注册为bean
public class RuisaAgentWebApplication {

    public static void main(String[] args) {
        SpringApplication.run(RuisaAgentWebApplication.class, args);//启动spring容器，创建和管理所有的bean
    }
}
```

## **@SpringBootApplication** 是一个组合注解，包含三个核心功能：

- `@Configuration`: 标记为配置类，可以定义Bean
- `@EnableAutoConfiguration`: 启用Spring Boot自动配置机制
- `@ComponentScan`: 自动扫描当前包及子包下的组件（Controller、Service等）

## **@EnableConfigurationProperties(AgentProxyProperties.class)**:

- 启用并注册 `AgentProxyProperties` 作为Spring Bean
- 允许将配置文件（如application.yml/properties）中的属性值自动绑定到该类的字段上



```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    private final AgentChatProxyHandler agentChatProxyHandler;

    public WebSocketConfig(AgentChatProxyHandler agentChatProxyHandler) {
        this.agentChatProxyHandler = agentChatProxyHandler;
    }
    //构造函数注入：Spring 自动将 AgentChatProxyHandler Bean 注入到此配置类
    //this.agentChatProxyHandler = agentChatProxyHandler：将注入的处理器赋值给字段


    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(agentChatProxyHandler, "/api/v1/ws/agent/chat").setAllowedOrigins("*");
    }
    //这是 WebSocketConfigurer 接口的唯一方法
    //Spring 启动时会自动调用此方法
    //用于注册 WebSocket 端点
}
```

## **@Configuration：**

- 标记该类为 Spring 配置类
- Spring 容器启动时会扫描并处理此类
- 可以定义 Bean 和配置信息

## **@EnableWebSocket：**

- **启用 Spring WebSocket 功能**
- 激活 WebSocket 相关的自动配置
- 如果没有此注解，WebSocket 功能不会生效

## **implements WebSocketConfigurer：**

- 实现 WebSocket 配置器接口

- 必须实现 `registerWebSocketHandlers()` 方法

- 用于注册 WebSocket 端点和处理器

- 实现WebSocketConfigurer接口，注册WebSocketHandler

  

  ```java
  @Component
  public class AgentChatProxyHandler implements WebSocketHandler {
  ```

  ## **@Component：**

  - 注册为 Spring Bean
  - Spring 容器启动时自动创建此类的实例

  ## **implements WebSocketHandler：**

  - 实现 WebSocket 处理器接口
  - 必须实现 5 个核心方法（后面会详细说明）

```java
@RestController
public class CameraProxyController {
```

## **@RestController：**

- 组合注解 = `@Controller` + `@ResponseBody`
- 所有方法的返回值直接写入 HTTP 响应体
- 不需要视图解析器

```java
@GetMapping(value = "/api/v1/camera/mjpeg", produces = "multipart/x-mixed-replace; boundary=frame")
public ResponseEntity<StreamingResponseBody> mjpeg() {
```

## **@GetMapping 注解：**

- **value**：请求路径 `/api/v1/camera/mjpeg`
- **produces**：指定响应的 MIME 类型

- ```
  multipart/x-mixed-replace;boundary=frame
  ```

  MJPEG 标准格式

  - 一种特殊的 multipart 格式，每个部分替换前一个部分
  - 浏览器会持续显示最新的帧，形成视频效果

- `boundary=frame`：分隔符名称，用于区分每一帧