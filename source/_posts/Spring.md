---
title: Spring
date: 2017-02-17 15:49:47
tags: 日志
categories: Java

---

[TOC]

# spring-boot解决的问题

相对于spring框架，spring-boot解决了以下问题

- 简化配置
- 简化依赖
- 简化部署

## 自动配置

spring-boot提供了一些注解用于条件判断，只有当条件满足的时候才加载相应配置

- @ConditionalOnBean
- @ConditionalOnMissingBean
- @ConditionalOnClass
- @ConditionalOnMissingClass
- @ConditionalOnExpresstion
- @ConditionalOnJava
- @ConditionalOnJndi
- @ConditionalOnProperty
- @ConditionalOnResource
- @ConsitionalOnWebApplication
- @ConsitionalOnNotWebApplication

可以自定义自己的条件配置，只要实现Condition接口

```java
public class JdbcTemplateCondition implements Condition {

  @Override
  public boolean matches(ConditionContext context,
                        AnnotatedTypeMetadata metadata) {
  try {
    context.getClassLoader().loadClass(
    "org.springframework.jdbc.core.JdbcTemplate");
    return true;
  } catch (Exception e) {
    return false;
    }
  }
}
```

然后使用@Conditional注解进行配置

```java
@Conditional(JdbcTemplateCondition.class)
public Service service() {
  ...
}
```

只要当JdbcTemplate在classpath中时service才会创建

## 自定义配置

spring-boot自动配置是建立在一系列条件之上的，如果自定义了配置，比如手动注入了一个bean，spring-boot检测到这个bean已经有了，就不会再启用自动配置程序

spring-boot的配置文件在classpath下的`application.yml`中，这个配置可以在命令行启动的时候覆盖某些属性，或者全部属性

自定义错误页面，当发生错误时，可以捕获这些错误，自定义返回信息

## 测试

### Junit集成测试

```java
package cn.adam.stater.controller;

import cn.adam.stater.Application;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.boot.test.WebIntegrationTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

/**
* Created by Adam on 2016/8/24.
*/

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
// 随机端口
@WebIntegrationTest({"server.port=0"})
public class TestControllerTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void t1() throws Exception {
        String content = mockMvc.perform(MockMvcRequestBuilders.get("/t1").contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(status().isOk())
                .andDo(print())
                .andReturn().getResponse().getContentAsString();
        System.out.println(content);
    }

    @Test
    public void t2() throws Exception {
        String content = mockMvc.perform(MockMvcRequestBuilders.get("/t2").contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(status().isOk())
                .andDo(print())
                .andReturn().getResponse().getContentAsString();
        System.out.println(content);
    }

    @Test
    public void t3() throws Exception {
        String content = mockMvc.perform(MockMvcRequestBuilders.get("/t3").contentType(MediaType.APPLICATION_JSON_UTF8).sessionAttr("user", "Adam"))
                .andExpect(status().isOk())
                .andDo(print())
                .andReturn().getResponse().getContentAsString();
        System.out.println(content);
    }

}
```

### RestTemplate使用

`RestTemplate`有三个主要类别的使用方法：

- `getForObject`和`getForEntity`，使用get方法进行请求，前者和后者的区别在于后者返回`ResponseEntity`对象
- `postForObject`和`postForEntity`，使用post方法进行请求，区别同上
- `exchange`，使用自定义的`RequestEntity`进行请求，`RequestEntity`包括请求url，请求方法以及请求头部信息以及实体信息，可以最大化自定义请求，body中的键值对象必须使用`LinkedMultiValueMap`保存，否则会参数无法序列化

代码示例

```java
package cn.adam;

import cn.adam.entity.User;
import org.junit.Test;
import org.springframework.http.*;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestTemplate;

import java.net.URI;
import java.util.Map;

/**
* Created by Adam on 2017/2/9.
*/


public class Test1 {

  @Test
  public void t1() {

    RestTemplate restTemplate = new RestTemplate();
    // post请求直接在链接上加参数
    ResponseEntity<Map> entity = restTemplate.postForEntity("http://localhost:8080/t7?name=chen&age=27", null, Map.class);
    System.out.println(entity.getBody());
  }

  @Test
  public void t2() {

    RestTemplate restTemplate = new RestTemplate();

    // 请求实体
    MultiValueMap<String, Object> map = new LinkedMultiValueMap<>();
    map.add("name", "sdf");
    map.add("age", "27");

    // 请求头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

    // 组装请求
    RequestEntity<Object> requestEntity = new RequestEntity<>(map, headers, HttpMethod.POST, URI.create("http://localhost:8080/t7"));
    ResponseEntity<Map> responseEntity = restTemplate.exchange(requestEntity, Map.class);
    System.out.println(responseEntity.getBody());
  }

  @Test
  public void t3() {

    RestTemplate restTemplate = new RestTemplate();

    // 请求实体，对应controller中@RequestBody参数
    User user = new User();
    user.setAge(27);
    user.setId(100);
    user.setName("xiang");

    ResponseEntity<Map> mapResponseEntity = restTemplate.postForEntity("http://localhost:8080/t8", user, Map.class);
    System.out.println(mapResponseEntity.getBody());
  }

  @Test
  public void t4() {

    // 最常见请求方式，填充参数
    RestTemplate restTemplate = new RestTemplate();
    MultiValueMap<String, Object> map = new LinkedMultiValueMap<>();
    map.add("name", "sdf");
    map.add("age", "27");

    ResponseEntity<Map> responseEntity = restTemplate.postForEntity("http://localhost:8080/t7", map, Map.class);
    System.out.println(responseEntity.getBody());
  }
}

```

# 异常处理

参考项目：[spring-boot-template](https://coding.net/u/UnlimitedLoop/p/spring-boot-template/git)

## Controller异常

### 缺少参数异常

调用方没有提供接口所需的全部参数

异常类`MissingServletRequestParameterException`

### 参数类型转换异常

控制器接收的参数无法转换成对应的类型

异常类`MethodArgumentTypeMismatchException`

### 方法参数异常

控制器接收的参数不符合`JSR-303`定义的约束条件

会处理所有参数，然后返回不符合规则的参数的信息的集合

异常类`ConstraintViolationException`

### 实体类校验异常

控制器接收的参数组成实体后，不是所有字段都符合`JSR-303`定义的约束条件

会处理所有参数，然后返回不符合规则的参数的信息的集合

异常类`BindException`

### 404异常

## 参数校验

### 单个参数校验

要使用单个参数校验，首先要在spring中注入`MethodValidationPostProcessor`类，然后方法所在的类上使用`@Validated`注解，然后在方法参数中使用`@Validated`注解，然后加上`JSR-303`规则

如果校验不通过，会抛出`ConstraintViolationException`异常

示例

```java
@RestController
@Validated
public class TestController extends RestExceptionHandler {

  @RequestMapping(value = "t1")
  public RestResponse t1() {

    return new RestResponse();
  }

  @RequestMapping(value = "t2")
  public RestResponse t2(@Validated @Min(value = 100, message = "id#不能小于100") @RequestParam("id") int id,
                            @Validated @Min(value = 3, message = "name#长度不能小于3") @RequestParam("name") String name) {
    return new RestResponse();
  }

  @RequestMapping(value = "t3")
  public RestResponse t3(@Valid User user) {
    return new RestResponse();
  }

  @RequestMapping(value = "t4")
  public RestResponse t4(@Validated(EditGroup.class) User user) {
    return new RestResponse();
  }
}
```

`ConstraintViolationException`异常没有参数名信息，所以可以在自定义的返回信息中加上参数名，然后在解析异常的时候解析这个字符串，返回参数以及对应的提示信息

### 实体类参数校验

#### 默认校验

在实体类各个字段中加入`JSR-303`规则注解之后，使用`@Valid`注解需要校验的方法参数，只会校验`JSR-303`注解中没有设置`group`属性的规则

如果校验不通过，抛出`BindException`

#### 分组校验

在实体类各个字段加入`JSR-303`注解，并且设置`group`属性，在方法所在的类上使用`@Validated`注解，然后在方法参数中使用`@Validated`注解并且设置其中的`value`属性与`JSR-303`中`group`属性相同

如果校验不通过，抛出`BindException`

# 日志记录

`spring-boot`中有一个默认的Logback配置，并且可以集成`spring-cloud-starter-sleuth`用于记录请求，如果直接使用自己的 `logback`配置，sleuth将不会发生作用

在`spring-boot`jar包中有4个默认的logback配置文件，分别为

- defaults.xml 配置基本的日志记录格式
- console-appender.xml 控制台日志配置
- file-appender.xml 文件日志配置
- base.xml 引用了上述文件，组成一个基本可用的logback配置

这四个文件采用了logback配置中的include语法，里面的配置可以被其他logback配置文件引用 可以在`application.properties`中设置日志文件的位置

```properties
spring.application.name=spring-boot-starter
logging.file=/var/log/${spring.application.name}.log
```

预设的logback配置文件会读取这个配置，将日志文件写入这个配置

基本使用默认的logback配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <include resource="org/springframework/boot/logging/logback/base.xml" />

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

# 记录请求

`spring-cloud`中有一个服务跟踪模块`spring-cloud-starter-sleuth`，这个模块自动在日志中加入若干参数，记录请求，并且将这些参数记录到响应的首部中，用于追踪请求

这个模块必须在spring容器中才会起作用，因此将原来tomcat中记录请求的方法转移到spring中的拦截器中

代码如下：

```java
public class LogRequestInterceptor extends HandlerInterceptorAdapter {

    private static final Logger LOGGER = LoggerFactory.getLogger(LogRequestInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        StrBuilder requestInfo = new StrBuilder();
        requestInfo.append("[ ").append(request.getMethod())
                .append(" ").append(request.getRequestURI())
                .append(" ").appendln(request.getProtocol());

        Enumeration<String> headerNames = request.getHeaderNames();
        if (headerNames != null) {
            while (headerNames.hasMoreElements()) {
                String name = headerNames.nextElement();
                requestInfo.append(name).append(": ").appendln(request.getHeader(name));
            }
        }

        requestInfo.appendln("");

        Enumeration<String> parameterNames = request.getParameterNames();
        if (parameterNames != null) {
            while (parameterNames.hasMoreElements()) {
                String name = parameterNames.nextElement();
                requestInfo.append(name).append(": ").appendln(request.getParameter(name));
            }
        }

        requestInfo.append("]");

        LOGGER.info("Received {}", requestInfo.toString());

        return true;
    }
}
```

记录完整的请求报文，包括首部，起始行，实体全部信息

# Maven配置

spring-boot可以直接打包为一个可运行jar包，直接运行 需要在pom中进行配置

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>1.3.5.RELEASE</version>
    <!-- 配置生成的可运行jar包增加bash脚本，可以在linux中配置成service统一管理 -->
    <configuration>
        <executable>true</executable>
    </configuration>
    <!-- 配置打包为可运行jar包 -->
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
