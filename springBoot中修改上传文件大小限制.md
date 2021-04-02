# springBoot中修改上传文件大小限制

### 1、问题
当上传文件大小超过1MB，后台会提示错误：

```java
The field file exceeds its maximum permitted size of 1048576 bytes
```

原来Spring Boot工程嵌入的tomcat限制了请求的文件大小，这一点在Spring Boot的官方文档中有说明，原文如下

```java
65.5 Handling Multipart File Uploads
Spring Boot embraces the Servlet 3 javax.servlet.http.Part API to support uploading files. By default Spring Boot configures Spring MVC with a maximum file of 1Mb per file and a maximum of 10Mb of file data in a single request. You may override these values, as well as the location to which intermediate data is stored (e.g., to the /tmp directory) and the threshold past which data is flushed to disk by using the properties exposed in the MultipartProperties class. If you want to specify that files be unlimited, for example, set the multipart.maxFileSize property to -1.The multipart support is helpful when you want to receive multipart encoded file data as a @RequestParam-annotated parameter of type MultipartFile in a Spring MVC controller handler method.
```

文档说明表示，每个文件的配置最大为1Mb，单次请求的文件的总数不能大于10Mb。要更改这个默认值需要在配置文件（如application.properties）中加入两个配置


### 2、解决方法

修改上传文件大小限制
（1）在application.properties中添加文件配置

```java
spring.servlet.multipart.max-file-size = 10MB
spring.servlet.multipart.max-request-size = 20MB
```

（2）添加配置文件

```java
@Configuration
public class CommonConfig {
    @Value("${spring.servlet.multipart.max-file-size}")
    private String maxFileSize;
    @Value("${spring.servlet.multipart.max-request-size}")
    private String maxRequestSize;
    @Bean
    public MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        factory.setMaxFileSize(maxFileSize);
        factory.setMaxRequestSize(maxRequestSize);
        return factory.createMultipartConfig();
    }
```

