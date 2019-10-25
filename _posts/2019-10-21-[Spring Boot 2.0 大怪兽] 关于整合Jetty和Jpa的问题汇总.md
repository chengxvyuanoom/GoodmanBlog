---
layout: post
title:  "[Spring Boot 2.0 大怪兽] 关于整合Jetty和Jpa的问题汇总"
date:   2019-10-21 00:51:12
tags: SpringBoot
categories: SpringBoot
---

* TOC
{:toc}

### 整合SpringBoot 2.0.1 问题汇总

1. **关于jetty的容器问题**

   之前版本的Jetty配置类:

   > ```java
   > @Bean
   > public EmbeddedServletContainerFactory servletContainer() {
   >   JettyEmbeddedServletContainerFactory factory = 
   >       new JettyEmbeddedServletContainerFactory();
   >   factory.setPort(jettyPort);
   >   factory.setContextPath(jettyContext);
   >   factory.setSessionTimeout(sessionTimeout, TimeUnit.HOURS);
   >   return factory;
   >  }
   > ```

   2.0以后的Jetty配置类:

   > ```Java
   > @Bean
   > public ConfigurableServletWebServerFactory servletContainer() {
   >   JettyServletWebServerFactory factory = new JettyServletWebServerFactory();
   >   factory.setPort(jettyPort);
   >   factory.setContextPath(jettyContext);
   >   return factory;
   > }
   > ```

   

   EmbeddedServletContainerFactory 变成了 ConfigurableServletWebServerFactory

   JettyEmbeddedServletContainerFactory 变成了 JettyServletWebServerFactory



2. **整合JPA的问题**

   首先看Pom文件配置

   > ```xml
   > <!-- JPA -->
   > <dependency>
   >     <groupId>org.springframework.boot</groupId>
   >     <artifactId>spring-boot-starter-data-jpa</artifactId>
   >     <exclusions>
   >         <exclusion>
   >             <groupId>org.hibernate</groupId>
   >             <artifactId>*</artifactId>
   >         </exclusion>
   >         <exclusion>
   >             <groupId>org.aspectj</groupId>
   >             <artifactId>aspectjrt</artifactId>
   >         </exclusion>
   >     </exclusions>
   > </dependency>
   > ```

   

   直接运行会报一个错误如下:

   > required a bean named 'entityManagerFactory' that could not be found.

   

   解决办法，添加如下配置类：

   > ```java
   > @Configuration
   > public class EclipseLinkJpaConfig extends JpaBaseConfiguration {
   >     protected EclipseLinkJpaConfig(
   >             DataSource ds, JpaProperties properties,
   >             ObjectProvider<JtaTransactionManager> jtm,
   >             ObjectProvider<TransactionManagerCustomizers> tmc) {
   >         super(ds, properties, jtm, tmc);
   >     }
   > 
   >     @Override
   >     protected AbstractJpaVendorAdapter createJpaVendorAdapter() {
   >         return new EclipseLinkJpaVendorAdapter();
   >     }
   > 
   >     @Override
   >     protected Map<String, Object> getVendorProperties() {
   >         Map<String, Object> map = Maps.newHashMap();
   >         map.put(PersistenceUnitProperties.WEAVING, "false");
   >         map.put(PersistenceUnitProperties.DDL_GENERATION,
   >                 "create-or-extend-tables");
   >         return map;
   >     }
   > }
   > ```

### Lombak 踩过的坑

1. lombak建造有参构造函数，必须构造一个无参构造函数，否则会引发一个Jpa的Bug


### Spring Boot JPA 的使用心得



1. 对于一些公共的Entity类，比如 id ，created ，deleted 要使用Wrap类，不能使用基础类型，否则你虽然set了具体的value，数九也会出现Null的情况


2. 关于JPA 与Jackson的嵌套循环，栈溢出的 问题

   网络上提供出了重夺
   

   

   

   