---
layout: post
title:  "[Spring Boot 2.0 大怪兽] 弃用的WebMvcConfigurerAdapter"
date:   2019-10-21 00:51:12
tags: SpringBoot
categories: SpringBoot
---

* TOC
{:toc}

### 精通SpringBoot2.0-弃用的WebMvcConfigurerAdapter

**org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration**

在这个类中，Spring Boot给我们配置好了视图解析器、静态资源、消息转换器、区域信息解析器、首页、欢迎页等等内容。

Spring Boot并不仅仅给我们自动配置好了关于Spring MVC的配置，而且为了方便的进行扩展，还给我们提供了

WebMvcConfigurerAdapter这个适配类，只需要我们自己实现了这个类，就可以自定义比如视图解析器、区域信息解析器这些组件。

##### 在Spring Boot2.0版本中，WebMvcConfigurerAdapter这个类被弃用了。
```java
@Deprecated
public abstract class WebMvcConfigurerAdapter implements WebMvcConfigurer {
```

那么现在如何拓展MVC的相关配置呢

1. #### 继承WebMvcConfigurationSupport

   ```java
      protected void addInterceptors(InterceptorRegistry registry) {
      }
   
     /**
      Override this method to add view controllers.
      @see ViewControllerRegistry
      */
      protected void addViewControllers(ViewControllerRegistry registry) {
      }
   
   
   ```

   可以看到该类下有非常多的add方法，可以继承这个类并且自己实现。但是这里有一个问题，你如果继承这个类实现mvc的扩展，则关于springboot mvc 相关的自动配置，则会失效，可以查看一下自动配置类

   ```java
   @Configuration
   @ConditionalOnWebApplication(type = Type.SERVLET)
   @ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
   @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
   @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
   @AutoConfiΩgureAfter({ DispatcherServletAutoConfiguration.class,
         ValidationAutoConfiguration.class })
    public class WebMvcAutoConfiguration {
   ```

   

其中一个条件就是**@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)**，只有当容器中没有WebMvcConfigurationSupport这个类型的组件的时候，才会启动自动配置。

##### 所以当我们继承WebMvcConfigurationSupport之后，除非你自己对代码把控的相当的好，在继承类中重写了一系列有关WebMVC的配置，否则可能就会遇到静态资源访问不到，返回数据不成功这些一系列问题了。



2. #### 实现WebMvcConfigurer接口

   ​    我们知道，Spring Boot2.0是基于Java8的，Java8有个重大的改变就是接口中可以有default方法，而default方法是不需要强制实现的。上述的WebMvcConfigurerAdapter类就是实现了WebMvcConfigurer这个接口，所以我们不需要继承WebMvcConfigurerAdapter类，可以直接实现WebMvcConfigurer接口，用法与继承这个适配类是一样的。如下面这个实现了消息转换器：

   ```java
   @Configuration
   public class WebMvcConfig implements WebMvcConfigurer {
       @Bean("111")
       public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
           RequestMappingHandlerAdapter adapter = new RequestMappingHandlerAdapter();
           List<HttpMessageConverter<?>> converters = adapter.getMessageConverters();
           MappingJackson2HttpMessageConverter jsonConverter = new MappingJackson2HttpMessageConverter();
           List<MediaType> supportedMediaTypes = new ArrayList<>();
           MediaType textMedia = new MediaType(MediaType.APPLICATION_FORM_URLENCODED, Charset.forName("UTF-8"));
           supportedMediaTypes.add(textMedia);
           MediaType jsonMedia = new MediaType(MediaType.APPLICATION_JSON, Charset.forName("UTF-8"));
           supportedMediaTypes.add(jsonMedia);
           jsonConverter.setSupportedMediaTypes(supportedMediaTypes);
           converters.add(jsonConverter);
           adapter.setMessageConverters(converters);
           return adapter;
       }
   }
   ```

##### 所以这两种方法都可以作为拓展Spring mvc 的方法，但是前者的方式会导致自动配置全部消失，需要你全部重新实现，而后者则不会导致自动配置失效，可以根据自己的需求有选择的配置mvc，所以开发中首选第二种方式。



   

   

   