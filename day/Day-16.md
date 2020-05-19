### springboot整合freemarker

1、pom文件添加依赖

2、新建spring web项目，会自动生成application.properties.

3、在配置文件中，设置以下两个参数

```properties

#设定ftl文件路径
spring.freemarker.template-loader-path=classpath:/templates
#设定静态文件路径，js,css等
spring.mvc.static-path-pattern=/static/**
```

controller跳转页面如下：templates/action/list.ftl页面 

```java
@GetMapping("/welcome")
public String welcome(Map<String, Object> model) {
    model.put("time", new Date());
    model.put("message", "张三");
    return "action/list";
}
```


------



### springboot freemarker不渲染页面返回字符串

在集成spring boot与freemarker时，Controller不返回渲染的模板页面，而是返回模板字符串，具体如下

```java
package com.xleiy.blog.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;
import java.util.Map;

/**
 * @Copyright (c) 2018 spring_boot_blog
 * @项目名称: spring_boot_blog
 * @类名称: WelcomeController
 * @创建时间: 2018/1/3 16:33
 * @类描述：
 */
@RestController
public class WelcomeController {
    @GetMapping("/welcome")
    public String welcome(Map<String, Object> model) {
        model.put("time", new Date());
        model.put("message", "张三");
        return "welcome";
    }
}

```



#### 问题所在

把@RestController替换为@Controller注解 

**@RestController注解表示返回的内容都是HTTP Content不会被模版引擎处理的** 

下面是RestController的定义

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.stereotype.Controller;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 * @since 4.0.1
	 */
	String value() default "";

}
```



