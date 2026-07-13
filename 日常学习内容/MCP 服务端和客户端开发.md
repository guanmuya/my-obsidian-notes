# 服务端
## 首先是开发所需依赖

```yaml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-server-webmvc-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```
## 编写对应的配置文件-两套

1. Stdio的形式

```xml
### application-stdio.yml  (需关闭web支持)
spring:
  ai:
    mcp:
      server:
        name: yu-image-search-mcp-server
        version: 0.0.1
        type: SYNC
        # stdio
        stdio: true
  # stdio
  main:
    web-application-type: none
    banner-mode: off
```

2. SSE的形式

```xml
### application-sse.yml   (需要关闭stdio支持)
spring:
  ai:
    mcp:
      server:
        name: yu-image-search-mcp-server
        version: 0.0.1
        type: SYNC
        # sse
        stdio: false
```

最后直接创建一个主配置文件就行，然后通过下面参数去控制对应的激活配置

```yaml
spring:  
  profiles:  
    active: stdio
```

## 测试写好的主要服务

```java
@SpringBootTest  
class ImagesSearchToolTest {  
  
    @Resource  
    private ImagesSearchTool imagesSearchTool;  
  
    @Test  
    public void test01(){  
        String s = imagesSearchTool.searchImageByKeywords("我想要一个小猫的图片");  
        Assertions.assertNotNull(s);  
    }  
}
```

## 最后可以直接在SpringBoot的主程序中直接创建对应Bean

```java
@SpringBootApplication
public class YuImageSearchMcpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(YuImageSearchMcpServerApplication.class, args);
    }

    @Bean
    public ToolCallbackProvider imageSearchTools(ImageSearchTool imageSearchTool) {
        return MethodToolCallbackProvider.builder()
                .toolObjects(imageSearchTool)
                .build();
    }
    // 这个bean也可以自己写在其他目录文件下也可以，不写在这边
}
```

使用Stdio的话就需要打成Jar包然后去放到对应的目录下面，然后去客户端那边写好相对应的`mcp-servers.json` 

```json
{  
  "mcpServers": {  
    "amap-maps": {  
      "command": "E:\\Program Files\\nodejs_24.15.0\\npx.cmd",  
      "args": [  
        "-y",  
        "@amap/amap-maps-mcp-server"  
      ],  
      "env": {  
        "AMAP_MAPS_API_KEY": "YOUR API KEY"  
      }  
    },  
    "pexels-image-search": {  
      "command": "C:\\Program Files\\Common Files\\Oracle\\Java\\javapath\\java.exe",  
      "args": [  
        "-jar",  
        "G:\\workspace\\qu-ai-agent\\qu-image-search-mcp-server\\target\\qu-image-search-mcp-service-0.0.1-SNAPSHOT.jar",  
        "--spring.profiles.active=stdio"  
      ]  
    }  
  }  
}
```
