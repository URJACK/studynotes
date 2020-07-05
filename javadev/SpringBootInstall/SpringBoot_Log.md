# SpringBoot 日志

## 市面上日志框架的简介

|日志门面|日志实现|
|---------|---------|
|JKL\SLF4j|Log4j JUL Log4j2 Logback|

我们在这里选择Logback，它完全适配SLF4j的接口

Spring框架默认使用的是JCL的接口

SpringBoot选用的是SLF4j和Logback

## 如何使用SLF4j

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloController {
    @RequestMapping(value = "/index")
    public String hello(){
        Logger logger = LoggerFactory.getLogger(HelloController.class);
        logger.info("HelloController has received a request");
        return "index";
    }
}
```

```txt
...
2020-04-16 01:01:04.422  INFO 7116 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 5 ms
2020-04-16 01:01:04.445  INFO 7116 --- [nio-8080-exec-1] c.e.demo.controller.HelloController      : HelloController has received a request
```
