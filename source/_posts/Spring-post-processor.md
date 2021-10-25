---
title: Spring post processor
date: 2021-10-25 13:34:38
categories:
- Spring
tags:
- post processor
---

Spring 中有两种 post processor，一种是 BeanFactoryPostProcessor, 另一种是 BeanPostProcessor. BeanFactoryPostProcessor 执行时机为：bean definition 加载完成之后，bean 实例化之前。而 BeanPostProcessor 则是 bean 初始化的后置处理器，包含两个方法，可以分别在初始化之前和之后执行。

总结各个扩展点的执行顺序：@PostConstruct -> InitializingBean -> initMethod -> @PreDestory -> DisposableBean -> destoryMethod

## BeanFactoryPostProcessor 使用案例

### Xml 配置的方式

不需要添加额外的注解，新建测试 bean 和 BeanFactoryPostProcessor 之后，通过 xml 关联，并在测试代码中通过 ClassPathXmlApplicationContext 加载配置即可。示例说明，测试 bean 中包含 name, age 属性，我们通过 BeanFactoryPostProcessor 在 bean definition 加载完之后修改 age 的值并将 scope type 修改为 prototype

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("------- BeanFactoryPostProcessor::postProcessBeanFactory");
        BeanDefinition bd = beanFactory.getBeanDefinition("postProcessorTestBean");

        MutablePropertyValues propertyValues = bd.getPropertyValues();
        if (propertyValues.contains("age")) {
            propertyValues.addPropertyValue("age", 24);
        }
        bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);
    }
}

public class PostProcessorTestBean {
    private String name;
    private Integer age;

    // getter + setter + toString
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="postProcessorTestBean" class="com.bin.postprocessor.PostProcessorTestBean">
        <property name="name" value="Tom"/>
        <property name="age" value="12" />
    </bean>

    <bean id="myBeanFactoryPostProcessor" class="com.bin.postprocessor.MyBeanFactoryPostProcessor"/>
</beans>
```

测试用例

```java
@Test
public void test_config_with_xml() {
    ApplicationContext ctx2 = new ClassPathXmlApplicationContext("postprocessor/bean_with_bean_factory.xml");
    PostProcessorTestBean bean2 = (PostProcessorTestBean) ctx2.getBean("postProcessorTestBean");
    System.out.println(bean2);
    System.out.println("------- Is singleton: " + ctx2.isSingleton("postProcessorTestBean"));
}
// ------- BeanFactoryPostProcessor::postProcessBeanFactory
// PostProcessorTestBean{name='Tom', age=24}
// ------- Is singleton: false
```

### Annotation 配置的方式

沿用之前的 bean 和 processor 代码，分别为他们添加 @Component 和 @Value 注解并设置值，通过 AnnotationConfigApplicationContext 加载配置执行测试

```java
@Test
public void test_config_with_annotation() {
    ApplicationContext ctx = new AnnotationConfigApplicationContext("com.bin.postprocessor");
    PostProcessorTestBean bean = (PostProcessorTestBean) ctx.getBean("postProcessorTestBean");
    System.out.println(bean);
    System.out.println("------- Is singleton; " + ctx.isSingleton("postProcessorTestBean"));
}
// ------- BeanFactoryPostProcessor::postProcessBeanFactory
// PostProcessorTestBean{name='Jack', age=22}
// ------- Is singleton; false
```

PS: 这里需要注意的是，value 并没有改变，因为完成 bean definition 加载的时候，@Value 并没有完成解析，所以修改时无效的。这一点可以看看对应的源码，后面再完善一下。从 BeanFactoryPostProcessor 的定义来说，这种用法才正确，之前的用法反而有点邪道的意思了。

## BeanPostProcessor 使用案例

和之前的 BeanFactoryPostProcessor 使用基本是一样的套路

### Xml 配置的方式

bean 沿用之前的, BeanPostProcess 如下

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("------- BeanPostProcessor::postProcessBeforeInitialization");
        if (beanName.endsWith("postProcessorTestBean")) {
            ((PostProcessorTestBean) bean).setAge(100);
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("------- BeanPostProcessor::postProcessAfterInitialization");
        if (beanName.endsWith("postProcessorTestBean")) {
            ((PostProcessorTestBean) bean).setName("Updated");
        }
        return bean;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="postProcessorTestBean" class="com.bin.postprocessor.PostProcessorTestBean">
        <property name="name" value="Tom"/>
        <property name="age" value="12" />
    </bean>

    <bean id="myBeanPostProcessor" class="com.bin.postprocessor.MyBeanPostProcessor"/>
</beans>
```

测试用例

```java
@Test
public void test_bean_post_processor_xml() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("postprocessor/bean_post_processor.xml");
    System.out.println(ctx.getBean("postProcessorTestBean"));
}
// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- BeanPostProcessor::postProcessAfterInitialization
// ------- print in test: PostProcessorTestBean{name='Updated', age=100}
```

### Annotation 配置的方式

为前面的 processor 类添加 Component 注解并通过 AnnotationConfigApplicationContext 加载即可

```java
@Test
public void test_bean_post_processor_annotation() {
    ApplicationContext ctx = new AnnotationConfigApplicationContext("com.bin.postprocessor");
    System.out.println(ctx.getBean("postProcessorTestBean"));
}
// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- BeanPostProcessor::postProcessAfterInitialization
// PostProcessorTestBean{name='Updated', age=100}
```

由此可见 initialization 和属性设置是两个概念，属性设置应该是在实例化之后，BeanPostProcessor 之前的操作，不然 age 就会改变了

## 顺便加介绍一下 initialization 的方法

### Xml 配置的方式

之前说过 BeanPostProcessor 是初始化阶段的后置处理器，初始化可以通过在 xml 中配置 init-method 实现，对应的还有销毁方法 destory-method

在之前的测试 bean 中新加方法

```java
public void init() {
    System.out.println("------- initMethod");
}

public void cleanUp() {
    System.out.println("------- destroyMethod");
}
```

在 xml 中 bean 声明部分指定对应的方法，再结合 processor 查看执行顺序

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="postProcessorTestBean" class="com.bin.postprocessor.PostProcessorTestBean">
        <property name="name" value="Tom"/>
        <property name="age" value="12" />
    </bean>

    <bean id="myBeanPostProcessor" class="com.bin.postprocessor.MyBeanPostProcessor"/>
</beans>
```

测试如下

```java
@Test
public void test_init_cleanup() {
    ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("postprocessor/bean_init_cleanup.xml");
    System.out.println(ctx.getBean("postProcessorTestBean"));
    ctx.close();
}
// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- initMethod
// ------- BeanPostProcessor::postProcessAfterInitialization
// PostProcessorTestBean{name='Updated', age=100}
// ------- destroyMethod
```

PS: close() 是 ConfigurableApplicationContext 中添加的接口，再上层就不具备这个能力了

### Annotation 配置的方式

上面的功能我们可以通过创建一个 @Configuration 类达到同样的效果。 这时，原来 bean 上的 @Component 标签需要去掉

```java
@Configuration
public class LifecycleConfig {
    @Bean(initMethod = "init", destroyMethod = "cleanUp")
    public PostProcessorTestBean getBean() {
        System.out.println("------- init LifecycleConfig");
        return new PostProcessorTestBean();
    }
}

@Test
public void test_config() {
    ConfigurableApplicationContext ctx = new AnnotationConfigApplicationContext("com.bin.postprocessor");
    System.out.println(ctx.getBean("postProcessorTestBean"));
    ctx.close();
}

// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- initMethod
// ------- BeanPostProcessor::postProcessAfterInitialization
// ------- print in test: PostProcessorTestBean{name='Updated', age=100}
// ------- destroyMethod
```

类似的还有 @PostConstruct/@PreDestory，我们为之前的测试 bean 新增两个测试方法并在此运行测试

```java
@PostConstruct
public void postConstruct() {
    System.out.println("------- invoke postConstruct");
}

@PreDestroy
public void PreDestroy() {
    System.out.println("------- invoke PreDestroy");
}

// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- @PostConstruct
// ------- initMethod
// ------- BeanPostProcessor::postProcessAfterInitialization
// ------- print in test: PostProcessorTestBean{name='Updated', age=100}
// ------- @PreDestroy
// ------- destroyMethod
```

可以看到 postConstruct 和 PreDestroy 是包在 initialization 最外层的

### 接口方式

除此之外还有一种通过实现接口来扩展的方式

```java
public class PostProcessorTestBean implements InitializingBean, DisposableBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("------- InitializingBean::afterPropertiesSet");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("------- DisposableBean::destroy");
    }
}

// ------- PostProcessorTestBean::constructor
// ------- BeanPostProcessor::postProcessBeforeInitialization
// ------- @PostConstruct
// ------- InitializingBean::afterPropertiesSet
// ------- initMethod
// ------- BeanPostProcessor::postProcessAfterInitialization
// ------- print in test: PostProcessorTestBean{name='Updated', age=100}
// ------- @PreDestroy
// ------- DisposableBean::destroy
// ------- destroyMethod
```