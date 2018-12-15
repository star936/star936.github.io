---
title: FastJson的使用
date: 2018-12-15 23:20:28
tags: [fastjson]
categories: Spring
toc: true
---

> Springboot: 2.1.1
>
> FastJson: 1.2.47

> 最近项目中需要使用到Springboot,因此需要边学边写边记载下.



### FastJson的使用

#### 1. 替换Jackson

**在Springboot2中数据的序列化和反序列化默认使用的是Jackson,因此需要替换该用FastJson.**

```java
package com.example.morven.config;

import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

import java.util.List;

@Configuration
public class WebAppConfig extends WebMvcConfigurationSupport {
    /**
     * Convert message using FastJson.
     * @param converters List<HttpMessageConverter<?>
     */
    @Override
    protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        super.configureMessageConverters(converters);
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(
                SerializerFeature.PrettyFormat
        );
        fastConverter.setFastJsonConfig(fastJsonConfig);
    }
}
```

#### 2. 注解的使用

##### 2.1 @JSONField

*既可以在属性上使用也可以在getter/getter方法上使用.*

常用属性:

* name: 指定序列化和反序列化时的字段名称
* format: 指定时间的格式
* serialize: Boolean类型,是否序列化该字段
* deserialize: Boolean类型,是否反序列化该字段
* serializeUsing: 指定属性的序列化类
* deserializeUsing: 指定属性的反序列化类

##### 2.2 @JSONType

*在类上使用.*

常用属性:

* includes: 指定序列化和反序列化的字段
* ignores: 指定序列化和反序列化忽略的字段

#### 3. 思考

* **Java Bean和数据库表映射,如果我们想要在序列化或反序列化时包含一些别的属性值,那么该怎么办?**

**使用JSONField注解的serialize/deserialize属性.**

* **对于字段`name`属性,反序列化时使用名称`user_name`,序列化时使用名称`contact_name`,该怎么办?**

**在`name`的setter方法上使用@JSONField(name="user_name"),在getter方法上使用@JSONField(name="contract_name").**

