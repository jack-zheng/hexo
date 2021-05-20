---
title: Spring 怎么实现 server 启动之后立刻执行一个服务
date: 2020-11-12 12:59:19
categories:
- Spring
tags:
- howto
---

最近遇到一个需求需要在 Spring web service 启动之后立即执行，类似一个初始化的工作，搜出来有好多实现方式，稍微记录一下他们的区别

## ApplicationListener

```java
@Component
public class InitTraceSourceEventListener implements ApplicationListener<ApplicationReadyEvent> {
    @Override
    public void onApplicationEvent(ApplicationReadyEvent applicationReadyEvent) {
        System.out.println("InitTraceSourceEventListener triggered...");
    }
}
```

或者

```java
@Configuration
public class ProjectConfiguration {
    private static final Logger log = 
   LoggerFactory.getLogger(ProjectConfiguration.class);

   @EventListener(ApplicationReadyEvent.class)
   public void doSomethingAfterStartup() {
    log.info("hello world, I have just started up");
  }
}
```

## SpringBootServletInitializer

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application extends SpringBootServletInitializer {

    @SuppressWarnings("resource")
    public static void main(final String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);

        context.getBean(Table.class).fillWithTestdata(); // <-- here
    }
}
```

## @PostConstruct

```java
@Component
public class Monitor {
    @Autowired private SomeService service

    @PostConstruct
    public void init(){
        // start your monitoring in here
    }
}

```

## ApplicationRunner

```java
import org.springframework.boot.ApplicationRunner;

@Component
public class ServerInitializer implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments applicationArguments) throws Exception {

        //code goes here

    }
}

```

## CommandLineRunner

```java
@Component
public class CommandLineAppStartupRunner implements CommandLineRunner {
    private static final Logger logger = LoggerFactory.getLogger(CommandLineAppStartupRunner.class);

    @Override
    public void run(String...args) throws Exception {
        logger.info("Application started with command-line arguments: {} . \n To kill this application, press Ctrl + C.", Arrays.toString(args));
    }
}
```

## InitializingBean

```java
@Component
class MyInitializingBean implements InitializingBean {

  private static final Logger logger = ...;

  @Override
  public void afterPropertiesSet() throws Exception {
    logger.info("InitializingBean#afterPropertiesSet()");
  }

}
```