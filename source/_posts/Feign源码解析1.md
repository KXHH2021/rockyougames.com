---
title: Feign源码解析：初始化过程（一） 
date: 2023-01-10 20:00:00
categories:
  - spring boot/cloud
tags:
  - Spring全家桶
  - Feign源码
description: Feign相关的bean不少，有一些是因为我们的Feign相关注解而引入的，有一部分是因为spring的自动装配来自动引入的。
cover: https://s2.loli.net/2024/01/20/sRI1GFVQ2Sk8Epz.png
---
![image.png](https://s2.loli.net/2024/01/20/sRI1GFVQ2Sk8Epz.png)

## 前言

本篇文章，先讲下Feign相关的beanDefinition，beanDefinition就是bean的设计图，bean都是按照beanDefinition来制造的。

Feign相关的bean不少，有一些是因为我们的Feign相关注解而引入的，有一部分是因为spring的自动装配来自动引入的。

今天讲讲因为我们注解引入的那些。

## EnableFeignClients引入的FeignClientSpecification

如果项目用到Feign，在@SpringBootApplication注解的主类上，我们一般还会加上@EnableFeignClients注解。

```
package a.b.c；
    
@SpringBootApplication
@EnableFeignClients
public class DemoApplication

```
实际上，spring的beanFactory初始化一般就是分两步，第一步，收集beanDefinition列表，第二步，根据beanDefinition生成并初始化bean。

收集beanDefinition相当重要，但是beanDefinition从哪来来呢，一开始的时候，都是框架默认注册了几个，应用自身的beanDefinition一般要从主类来，也就是上面说的@SpringBootApplication注解的主类。

一开始就会去解析主类上的注解，包名；解析包名的原因是，拿到包名后，默认就会去扫描这个包名下的class，再找到注解了@configuration、@controller、@service、@component之类的bean作为beanDefinition；解析注解的原因是，可以根据注解，引入更多的beanDefinition。

以 **@EnableFeignClients**为例，该注解的定义如下：


```
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients

```
所以这里会通过Import引入更多的beanDefinition。

```
org.springframework.cloud.openfeign.FeignClientsRegistrar

public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    // 1
    registerDefaultConfiguration(metadata, registry);
    // 2
    registerFeignClients(metadata, registry);
}

```
上面的1处，如下：


```
Map<String, Object> defaultAttrs = metadata.getAnnotationAttributes(EnableFeignClients.class.getName(), true);
String name = "default." + metadata.getClassName();
registerClientConfiguration(registry, name, defaultAttrs.get("defaultConfiguration"));

private void registerClientConfiguration(BeanDefinitionRegistry registry, Object name, Object configuration) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(FeignClientSpecification.class);
    builder.addConstructorArgValue(name);
    builder.addConstructorArgValue(configuration);
    registry.registerBeanDefinition(builder.getBeanDefinition());
}

```

主要是获取@EnableFeignClients这个注解相关的默认属性，然后注册了一个FeignClientSpecification类型的bean。

这个FeignClientSpecification类很简单，属性就两个：


```
public class FeignClientSpecification implements NamedContextFactory.Specification {

	private String name;

	private Class<?>[] configuration;
}

```
一个name，一个configuration，其实就是代表了一个要如何创建和配置FeignClient的配置，包含了如何创建feign的encoder、decoder等。


```
A custom @Configuration for all feign clients. Can contain override @Bean definition for the pieces that
make up the client, for instance feign.codec.Decoder, feign.codec.Encoder, feign.Contract.

```
我们举个例子，如下的代码，



```
package a.b.c；
    
@SpringBootApplication
@EnableFeignClients
public class DemoApplication

```
最终生成的 **FeignClientSpecification beanDefinition，beanName为：default.a.b.c.DemoApplication.FeignClientSpecification**，属性：


**name：default.a.b.c.DemoApplication**

configuration：空数组

## FeignClient注解引入的FeignClientSpecification#

说完这个，继续说如下的二处：


```
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    // 1
    registerDefaultConfiguration(metadata, registry);
    // 2
    registerFeignClients(metadata, registry);
}

```
2处内部，是首先获取@EnableFeignClients注解的类所在的包名，然后在这个包名下扫描注解了@FeignClient的类，扫描到的这些类，那肯定是弄了一个beanDefinition了，这个可以倒推，毕竟每个@FeignClient注解的类最终都变成了一个bean嘛。

里面有一点值得讲，FeignClient注解中有一个属性，叫configuration，它支持你指定一个class，里面可以覆盖FeignClient内部的部分组件，如Feign的Encoder等


```
public @interface FeignClient {
    String value() default "";
    Class<?>[] configuration() default {};
}

```
这个你指定的配置类，是可以被多个FeignClient复用的，所以，spring内部也是只会存一份，这一份配置，就通过一个创建一个类型为FeignClientSpecification.class的bean来保存。

以如下为例：


```
@FeignClient(name = "demoService")
public interface DemoServiceFeignClient

```
由于没有指定configuration属性，这里生成的FeignClientSpecification bean中，name就是demoService，configuration就是null：


```
public class FeignClientSpecification implements NamedContextFactory.Specification {

	private String name;

	private Class<?>[] configuration;
}

```
如果指定configuration属性：


```
@FeignClient(name = "demoService", configuration = CustomConfig.class)
public interface DemoServiceFeignClient
FeignClientSpecification中的configuration就会是CustomConfig.class。

```

这个FeignClientSpecification.class的bean的名字，默认则会是demoService.FeignClientSpecification。

## FeignClient对应的beanDefinition

接下来，就是注册一个FeignClient对应的beanDefinition了。

详细的源码可以看这里：org.springframework.cloud.openfeign.FeignClientsRegistrar#registerFeignClient

FeignClient的bean是较难创建的，所以这里是用了工厂bean：FeignClientFactoryBean


```
String name = getName(attributes);

FeignClientFactoryBean factoryBean = new FeignClientFactoryBean();
factoryBean.setBeanFactory(beanFactory);
factoryBean.setName(name);
factoryBean.setContextId(contextId);
factoryBean.setType(clazz);
factoryBean.setRefreshableClient(isClientRefreshEnabled());
BeanDefinitionBuilder definition = BeanDefinitionBuilder.genericBeanDefinition(clazz, () -> {
    factoryBean.setUrl(getUrl(beanFactory, attributes));
    factoryBean.setPath(getPath(beanFactory, attributes));
    ...

    return factoryBean.getObject();
});

BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className, qualifiers);
BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);

```
这里的beanname，一般也就是FeignClient的全路径名。

## 汇总

最终其实就引入了三个bean：

![image.png](https://s2.loli.net/2024/01/20/xeJgCnhcfukZzK8.png)

首先是由@enableFeignClients引入的FeignClientSpecification，然后是@enableFeignClients注解所在的包名下的由@FeignClient注解引入的FeignClientSpecification，再一个就是FeignClient本身的bean（一个factoryBean）







