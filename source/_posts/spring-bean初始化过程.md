---
title: spring bean初始化过程
date: 2016-11-06 21:47:05
tags:
    - Spring
categories:
    - 技术
    - Spring
---
## 实例化过程
![spring bean实例化过程](/img/spring%20bean%E5%AE%9E%E4%BE%8B%E5%8C%96%E8%BF%87%E7%A8%8B.png)

<!-- more -->

## 验证
### applicationContext.xml:
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:spring="http://www.springframework.org/schema/tool"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tool http://www.springframework.org/schema/tool/spring-tool.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
 
    <context:annotation-config/>
    <context:component-scan base-package="com.ytf.spring"></context:component-scan>
 
    <bean id="beanPost" class="com.ytf.spring.InitBeanProcess.BeanPost"></bean>
    <bean name="animal" class="com.ytf.spring.InitBeanProcess.Animal" init-method="animalInit"
          destroy-method="animalDestroy">
        <property name="spiece" value="dog"></property>
        <property name="sex" value="male"></property>
    </bean>
 
</beans>
```
### Animal.java实现了各种接口
```java
package com.ytf.spring.InitBeanProcess;
 
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
 
//@Component
public class Animal implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, InitializingBean,DisposableBean {
    private String spiece;
    private String sex;
 
    public String getSpiece() {
        return spiece;
    }
 
    public void setSpiece(String spiece) {
        this.spiece = spiece;
        System.out.println("Set Spiece:" + this.spiece);
    }
 
    public String getSex() {
        return sex;
    }
 
    public void setSex(String sex) {
        this.sex = sex;
        System.out.println("Set Sex:" + this.sex);
    }
 
    public Animal() {
        System.out.println("Animal Instantiation");
    }
 
 
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("BeanFactoryAware.setBeanFactory");
        beanFactory.getBean(Animal.class);
    }
 
    public void setBeanName(String s) {
        System.out.println("BeanNameAware.setBeanName,beanId: " + s);
    }
 
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean.afterPropertiesSet");
 
    }
 
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("ApplicationContextAware.setApplicationContext");
    }
     
    //@PostConstruct
    public void animalInit(){
        System.out.println("Animal Init");
    }
     
    //@PreDestroy
    public void animalDestroy(){
        System.out.println("Animal Destroy");
    }
    public void destroy() throws Exception {
        System.out.println("DisposableBean.destroy");
    }
     
    public String toString(){
        return "Spiece:" + this.spiece + ";Sex:" + this.sex;
    }
}
```
### BeanPost.java实现BeanPostProcessor接口
```java
package com.ytf.spring.InitBeanProcess;
 
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
 
//@Component
public class BeanPost implements BeanPostProcessor {
 
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        if (o instanceof Animal) {
            Animal animal = (Animal) o;
            System.out.println("BeanPostProcessor.postProcessBeforeInitialization");
            animal.setSpiece("monkey");
            return animal;
        }
 
        return o;
    }
 
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        if (o instanceof Animal) {
            Animal animal = (Animal) o;
            System.out.println("BeanPostProcessor.postProcessAfterInitialization");
            animal.setSex("female");
            return animal;
        }
        return o;
    }
}
```
### ProveBeanInit.java 是测试类
```java
package com.ytf.spring.InitBeanProcess;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class ProveBeanInit {
    public static void main(String[] args){
        ClassPathXmlApplicationContext cpa = new ClassPathXmlApplicationContext("application.xml");
        Animal animal = (Animal)cpa.getBean("animal");
        System.out.println(animal);
        cpa.close();
    }
}
```
## 结果
> Animal Instantiation
Set Spiece:dog
Set Sex:male
BeanNameAware.setBeanName,beanId: animal
BeanFactoryAware.setBeanFactory
ApplicationContextAware.setApplicationContext
BeanPostProcessor.postProcessBeforeInitialization
Set Spiece:monkey
InitializingBean.afterPropertiesSet
Animal Init
BeanPostProcessor.postProcessAfterInitialization
Set Sex:female
Spiece:monkey;Sex:female
DisposableBean.destroy
Animal Destroy

可见，初始化顺序确实如图说明。

## 注意问题
1. 如果去掉BeanPost,改为Animal实现BeanPostProcessor，会导致BeanPostProcessor的两个方法不运行。
2. 网上有说使用注解@PostConstruct、@PreDestroy 代替 init-method,destroy-method，实际运行发现并不是一样的，换成注解，产生的结果如下：

> Animal Instantiation
BeanNameAware.setBeanName,beanId: animal
BeanFactoryAware.setBeanFactory
ApplicationContextAware.setApplicationContext
BeanPostProcessor.postProcessBeforeInitialization
Set Spiece:monkey
***Animal Init
InitializingBean.afterPropertiesSet***
BeanPostProcessor.postProcessAfterInitialization
Set Sex:female
Spiece:monkey;Sex:female
***Animal Destroy***
***DisposableBean.destroy***

