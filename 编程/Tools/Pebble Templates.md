# 介绍

**Pebble Templates** 是一个现代的、轻量级的 **Java 模板引擎**，语法设计深受 **Twig**（Symfony/PHP 的模板引擎）和 **Jinja2**（Python 的模板引擎）启发。

它在 Java 生态中算是“语法最友好、阅读性最好”的模板引擎之一，尤其适合 Spring Boot 项目。

# 官网

[官方文档](https://pebbletemplates.io/)

# SpringBoot集成

## 添加依赖
```xml
<dependency>  
    <groupId>io.pebbletemplates</groupId>  
    <artifactId>pebble-spring-boot-starter</artifactId>  
    <version>3.2.0</version>  
</dependency>
```

## 模板存放位置

- 默认前缀：templates/
- 默认后缀：.peb（4.x 版本默认变更为 .peb，3.x 可能是 .html 或 .twig）

推荐统一使用 .html 或 .peb 后缀，方便 IDE 高亮。
在 application.properties / application.yml 中自定义（强烈推荐）：
```yml
pebble:  
  prefix: /templates/  
  suffix: .peb  
  cache: false  
  strictVariables: true

```

## Web返回模板示例
``` java
@Controller
package com.hans.pebbletemplates.web;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.ResponseBody;  
  
import java.util.List;  
  
@Controller  
public class HomeController {  
  
    @GetMapping("/hello")  
    public String index(Model model) {  
        model.addAttribute("title", "欢迎使用 Pebble + Spring Boot");  
        model.addAttribute("age",18);  
        model.addAttribute("items", List.of("苹果", "香蕉", "橙子"));  
        return "index";   // → templates/index.peb  
    }  
  
    @GetMapping("/test")  
    @ResponseBody  
    public String test() {  
        return "test";  
    }  
}
```

[代码示例](https://github.com/XuFeiLife/pebbletemplates)
