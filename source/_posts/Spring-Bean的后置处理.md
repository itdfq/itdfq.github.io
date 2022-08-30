---
title: Spring Bean的后置处理
top: false
cover: false
toc: true
mathjax: true
date: 2022-01-05 11:24:15
password:
summary: Bean的后置方式
tags:
- 原创
- IOC
categories:
- java
---
## 初始化Bean的后置处理
---
#### BeanPostProcessor
BeanPostProcessor定义了两种bean的处理方法
 1. postProcessBeforeInitialization：
	初始化前回调
	```java
	 /**
	     * bean加载之前处理
	     *
	     * @param bean
	     * @param beanName
	     * @return
	     * @throws BeansException
	     */
	    @Override
	    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	      
	        return bean;
	    }
	```

 2. postProcessAfterInitialization
	初始化之后回调，返回可以是原始实例，也可以是包装好的实例； 如果为null ，则不会调用后续的 BeanPostProcessors
	

	```java
	/**
	     * bean加载之后处理
	     *
	     * @param bean
	     * @param beanName
	     * @return
	     * @throws BeansException
	     */
	    @Override
	    public Object postProcessAfterInitialization(final Object bean, String beanName) throws BeansException {
	    
	        return bean;
	    }
	```

#### DestructionAwareBeanPostProcessor
1. 继承BeanPostProcessor，额外多了两个方法
2. postProcessBeforeDestruction 在销毁之前将此 BeanPostProcessor 应用于给定的 bean 实例，例如调用自定义销毁回调

	```java
	 /**
	     * 销毁前的后置处理
	     * @param bean
	     * @param beanName
	     * @throws BeansException
	     */
	    @Override
	    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
	
	    }
	```

 3. requiresDestruction 是否需要后处理器销毁，默认是true

	```java
	
    /**
     * 是否需要销毁
     * @param bean
     * @return
     */
    @Override
    public boolean requiresDestruction(Object bean) {
        return DestructionAwareBeanPostProcessor.super.requiresDestruction(bean);
    }
	```

#### InstantiationAwareBeanPostProcessor
 1. postProcessBeforeInstantiation
 初始化之前处理,默认是null

	```java
	 /**
	     * 默认是null
	     * 初始化之前处理
	     * @param beanClass
	     * @param beanName
	     * @return
	     * @throws BeansException
	     */
	    @Override
	    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
	        return InstantiationAwareBeanPostProcessor.super.postProcessBeforeInstantiation(beanClass, beanName);
	    }
	```

 2. postProcessAfterInstantiation
	 初始化之后是否需要填充字段,在 bean 被实例化之后，通过构造函数或工厂方法，但在 Spring 属性填充（从显式属性或自动装配）发生之前执行操作。
	 返回：如果应该在 bean 上设置属性，则为true ； 如果应跳过属性填充，则为false
	 
	```java
	 /**
	     * 初始化之后是否需要填充字段
	     * 在 bean 被实例化之后，通过构造函数或工厂方法，但在 Spring 属性填充（从显式属性或自动装配）发生之前执行操作。
	     * @param bean
	     * @param beanName
	     * @return 如果应该在 bean 上设置属性，则为true ； 如果应跳过属性填充，则为false
	     * @throws BeansException
	     */
	    @Override
	    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
	        return InstantiationAwareBeanPostProcessor.super.postProcessAfterInstantiation(bean, beanName);
	    }
	```


 3. postProcessProperties
 		在工厂将给定的属性值应用于给定的 bean 之前对给定的属性值进行后处理，无需任何属性描述符。
 		返回：应用于给定 bean 的实际属性值（可以是传入的 PropertyValues 实例），或者null继续处理现有属性，但特别继续调用postProcessPropertyValues （需要为当前 bean 类初始化PropertyDescriptor ）

	```java
	 /**
	     * 在工厂将给定的属性值应用于给定的 bean 之前对给定的属性值进行后处理，无需任何属性描述符
	     * @param pvs
	     * @param bean
	     * @param beanName
	     * @return 应用于给定 bean 的实际属性值（可以是传入的 PropertyValues 实例），或者null继续处理现有属性，
	     * 但特别继续调用postProcessPropertyValues （需要为当前 bean 类初始化PropertyDescriptor ）
	     * @throws BeansException
	     */
	    @Override
	    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
	        return InstantiationAwareBeanPostProcessor.super.postProcessProperties(pvs, bean, beanName);
	    }
	```
#### MergedBeanDefinitionPostProcessor

 1. postProcessMergedBeanDefinition

	```java
	/**
	     * 对指定 bean 的进行后处理
	     * @param beanDefinition
	     * @param beanType
	     * @param beanName
	     */
	    @Override
	    public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
	
	    }
	```

 2. resetBeanDefinition
	
	
	```java
	/**
	     * 通过名称重置指定bean
	     * 指定名称的 bean 定义已重置的通知，并且此后处理器应清除受影响 bean 的所有元数据
	     * @param beanName
	     */
	    @Override
	    public void resetBeanDefinition(String beanName) {
	        MergedBeanDefinitionPostProcessor.super.resetBeanDefinition(beanName);
	    }
	```
