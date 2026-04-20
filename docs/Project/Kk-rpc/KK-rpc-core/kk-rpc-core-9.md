---
sidebar_position: 9
---

# 启动机制和注解驱动

## 一、需求分析

- 参考服务提供者和服务消费者代码例子，发现现在使用框架的时候还是不够简便，因此需要优化框架的易用性，通过建立合适的**启动机制和注解驱动机制**，帮助开发者最少只用一行代码，就能轻松使用框架



## 二、设计方案

### 启动机制设计

- 其实很简单，把所有启动代码封装成一个 专门的启动类 或方法，然后由服务提供者/服务消费者调用即可。
- 但有一点需要注意，服务提供者和服务消费者需要初始化的模块是不同的，比如服务消费者不需要启动 Web 服务器
  - 所以需要针对服务提供者和消费者分别编写一个启动类，如果是二者都需要初始化的模块，可以放到全局应用类 `RpcApplication` 中，复用代码的同时保证启动类的可维护、可扩展性
- 参考 doubbo ：https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/api/

### 注解驱动设计

- 参考 doubbo 的注解驱动，开发者只需要在服务提供者实现类打上一个 `Dubboservice` 注解，就能快速注册服务;同样的, 只要在服务消费者字段打上一个 `DubboReference` 注解，就能快速使用服务
- 而且由于现在的 Java 项目基本都使用 Spring Boot 框架，所以 Dubbo 还贴心地推出了 Spring Boot starter，用更少的代码在 Spring Boot 项目中使用框架
  - 参考文档：https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/starter/
- 因此可以参考 doubbo ，给本 rpc 框架创建一个 Spring Boot Starter 项目，并通过注解驱动框架的初始化，完成服务注册和获取引 用
- 实现注解驱动并不复杂，有2种常用的方式:
  1. 主动扫描: 让开发者指定要扫描的路径，然后遍历所有的类文件，针对有注解的类文件，执行自定义的操作
  2. 监听 Bean 加载: 在 Spring 项目中，可以通过实现 `BeanPostProcessor` 接口，在 Bean 初始化后执行自定义的操作

## 三、开发实现

### 启动机制

- 在 rpc 项目中新建包名 `bootstrap` ，所有和框架启动初始化相关的代码都放到该包下

#### 1. 服务提供者启动类

- 新建 `ProviderBootstrap` 类，先直接复制之前服务提供者示例项目中的初始化代码，然后略微改造，支持用户传入自己 要注册的服务

- 由于在注册服务时，我们需要填入多个字段，比如服务名称、服务实现类，参考代码如下:

  ```java
          // 注册服务
          String serviceName = UserService.class.getName();
          LocalRegistry.register(serviceName, UserServiceImpl.class);
  ```

- 因此可以将这些字段进行封装，在 model 包下新建 `ServiceRegisterInfo` 类，代码如下

  ```java
  package com.lhk.kkrpc.bootstrap;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  /**
   * 服务注册信息类
   */
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class ServiceRegisterInfo<T> {
  
      /**
       * 服务名称
       */
      private String serviceName;
  
      /**
       * 实现类
       */
      private Class<? extends T> implClass;
  }
  
  ```

- 这样一来，服务提供者的初始化方法只需要接受封装的注册信息列表作为参数即可，简化了方法

  ```java
  package com.lhk.kkrpc.bootstrap;
  
  import com.lhk.kkrpc.RpcApplication;
  import com.lhk.kkrpc.config.RegistryConfig;
  import com.lhk.kkrpc.config.RpcConfig;
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  import com.lhk.kkrpc.registry.LocalRegistry;
  import com.lhk.kkrpc.registry.Registry;
  import com.lhk.kkrpc.registry.RegistryFactory;
  import com.lhk.kkrpc.server.tcp.VertxTcpServer;
  
  import java.util.List;
  
  /**
   * 服务提供者启动类
   */
  public class ProviderBootstrap {
      public static void init(List<ServiceRegisterInfo<?>> serviceRegisterInfos){
  
          // 初始化 RPC 配置 (从 application.properties 文件中读取配置)
          RpcApplication.init();
          // 获取 RPC 配置对象
          final RpcConfig rpcConfig = RpcApplication.getRpcConfig();
  
          for (ServiceRegisterInfo<?> serviceRegisterInfo : serviceRegisterInfos) {
              String serviceName = serviceRegisterInfo.getServiceName();
              // 本地注册服务
              LocalRegistry.register(serviceName, serviceRegisterInfo.getImplClass());
  
              // 注册服务到注册中心
              RegistryConfig registryConfig = rpcConfig.getRegistryConfig();
              Registry registry = RegistryFactory.getInstance(registryConfig.getRegistry());
              ServiceMetaInfo serviceMetaInfo = new ServiceMetaInfo();
              serviceMetaInfo.setServiceName(serviceName);
              serviceMetaInfo.setServiceHost(rpcConfig.getServerHost());
              serviceMetaInfo.setServicePort(rpcConfig.getServerPort());
              try {
                  registry.register(serviceMetaInfo);
              } catch (Exception e) {
                  throw new RuntimeException(serviceName + " 注册失败",e);
              }
          }
  
          // 启动 tcp 服务
          VertxTcpServer vertxTcpServer = new VertxTcpServer();
          vertxTcpServer.doStart(rpcConfig.getServerPort());
  
      }
  }
  ```

- 这样，想要在服务提供者项目中使用 RPC 框架，就非常简单了。只需要定义要注册的服务列表，然后一行代码调用`ProviderBootstrap.init()` 方法即可完成初始化

  ```java
  package com.lhk.example.provider;
  
  import com.lhk.example.common.service.UserService;
  import com.lhk.kkrpc.bootstrap.ProviderBootstrap;
  import com.lhk.kkrpc.bootstrap.ServiceRegisterInfo;
  
  import java.util.ArrayList;
  import java.util.List;
  
  /**
   * 服务提供者示例（针对测试 kk-rpc-core）
   */
  public class ProviderExample {
  
      public static void main(String[] args) {
          // 要注册的服务列表
          List<ServiceRegisterInfo> serviceRegisterInfos = new ArrayList<>();
          ServiceRegisterInfo userServiceRegisterInfo = new ServiceRegisterInfo(UserService.class.getName(),UserServiceImpl.class);
          serviceRegisterInfos.add(userServiceRegisterInfo);
          // 服务提供者初始化并启动
          ProviderBootstrap.init(serviceRegisterInfos);
      }
  }
  ```



#### 2. 服务消费者启动类

- 服务消费者启动类的实现较为容易，因为它不需要注册服务、也不需要启动 Web 服务器，因此只需要执行 `RpcApplicatio n.init()` 完成框架的通用初始化即可

  ```java
  package com.lhk.kkrpc.bootstrap;
  
  import com.lhk.kkrpc.RpcApplication;
  
  /**
   * 服务消费者启动类
   */
  public class ConsumerBootstrap {
  
      /**
       * 初始化 RPC 配置
       */
      public static void init(){
          // 初始化 RPC 配置 (从 application.properties 文件中读取配置)
          RpcApplication.init();
      }
  }
  
  ```

- 服务消费者代码调整

  ```java
  package com.lhk.example.consumer;
  
  import com.lhk.example.common.model.User;
  import com.lhk.example.common.service.UserService;
  import com.lhk.kkrpc.bootstrap.ConsumerBootstrap;
  import com.lhk.kkrpc.proxy.ServiceProxyFactory;
  
  /**
   * 服务消费者示例（针对测试 kk-rpc-core）
   */
  public class ConsumerExample {
  
      public static void main(String[] args) throws InterruptedException {
          ConsumerBootstrap.init();
          // 获取代理
          UserService userService = ServiceProxyFactory.getProxy(UserService.class);
          User user = new User();
          user.setName("lhk");
          // 远程调用
          User newUser = userService.getUser(user);
          if (newUser != null) {
              System.out.println(newUser.getName());
          } else {
              System.out.println("user == null");
          }
          userService.getUser(user);
  
          Thread.sleep(10000);
  
          userService.getUser(user);
      }
  }
  
  ```



### Spring Boot Starter 注解驱动

- 为了不和已有项目的代码混淆，创建一个新的项目模块，专门用于实现 Spring Boot starter 注解驱动的 RPC 框架



#### 1. Spring Boot Starter 项目初始化

- 新建一个 Spring Boot 项目 `kk-rpc-spring-boot-starter`

  - JDK 和 java 版本选择 >= 8 即可
  - Spring Boot 版本选择 2.6.13
  - 选择依赖 Spring Configuration Processor

- 创建好后，移除pom.xml 中无用的插件代码

  ```xml
  <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>${spring-boot.version}</version>
      <configuration>
          <mainClass>com.yupi.yurpc.springboot.starter.YuRpcSpringBootStarterApplication</mainClass>
          <skip>true</skip>
      </configuration>
      <executions>
          <execution>
              <id>repackage</id>
              <goals>
                  <goal>repackage</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  
  ```

- 引入自定义 rpc 框架依赖

  ```xml
          <dependency>
              <groupId>com.lhk</groupId>
              <artifactId>kk-rpc-core</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
  ```



#### 2. 定义注解

- 实现注解驱动的第一步是定义注解，要定义哪些注解呢?怎么知道应该定义哪些注解呢?
  - 可以参考知名框架 Dubbo 的注解。
  - 比如:
    1. `@EnableDubbo`: 在 Spring Boot 主应用类上使用，用于启用 Dubbo 功能
    2. `@DubboComponentScan`: 在 Spring Boot 主应用类上使用，用于指定 Dubbo 组件扫描的包路径
    3. `@DubboReference`: 在消费者中使用，用于声明 Dubbo 服务引用
    4. `@DubboService`: 在提供者中使用，用于声明 Dubbo 服务
    5. `@DubboMethod`: 在提供者和消费者中使用，用于配置 Dubbo 方法的参数、超时时间等
    6. `@DubboTransported`: 在 Dubbo 提供者和消费者中使用，用于指定传输协议和参数，例如传输协议的类型、端口等
- 遵循最小可用化原则，这里只需要定义3个注解
  - `@EnableRpc`
  - `@RpcReference`
  - `@RpcService`
- 新建 `annotation` 包，将所有注解代码放到该包下

##### 1. @EnableRpc

- `@EnableRpc`: 用于全局标识项目需要引入 RPC 框架、执行初始化方法

- 由于服务消费者和服务提供者初始化的模块不同，我们需要在 `@EnableRpc` 注解中，指定是否需要启动服务器等属性

  ```java
  package com.lhk.kkrpcspringbootstarter.annotation;
  
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  /**
   * 启用 rpc 注解
   */
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface EnableRpc {
  
      /**
       * 默认需要启动服务端
       * @return
       */
      boolean needServer() default true;
  }
  
  ```

- 当然，也可以将 `@EnableRpc` 注解拆分为两个注解(比如 `@EnableRpcProvider`、`@EnableRpcConsumer`)，分别用于标识服 务提供者和消费者，但可能存在模块重复初始化的可能性

##### 2. @RpcService

- `@RpcService`: 服务提供者注解，在需要注册和提供的服务类上使用

- `@RpcService` 注解中，需要指定服务注册信息属性，比如服务接口实现类、版本号等(也可以包括服务名称)

  ```java
  package com.lhk.kkrpcspringbootstarter.annotation;
  
  import com.lhk.kkrpc.constant.RpcConstant;
  import org.springframework.stereotype.Component;
  
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  /**
   * 服务提供者注解（用于注册服务）
   */
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Component
  public @interface RpcService {
  
      /**
       * 服务接口类
       */
      Class<?> interfaceClass() default void.class;
  
      /**
       * 服务版本号
       */
      String serviceVersion() default RpcConstant.DEFAULT_SERVICE_VERSION;
  }
  
  ```

##### 3. @RpcReference

- `@RpcReference`: 服务消费者注解，在需要注入服务代理对象的属性上使用，类似 Spring 中的 `@Resource` 注解

- `@RpcReference` 注解中，需要指定调用服务相关的属性，比如**服务接口类**(可能存在多个接口)、**版本号**、**负载均衡器**、**重** **试策略**、**是否 Mock 模拟调用**等

  ```java
  package com.lhk.kkrpcspringbootstarter.annotation;
  
  import com.lhk.kkrpc.constant.RpcConstant;
  import org.springframework.stereotype.Component;
  
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  /**
   * 服务提供者注解（用于注册服务）
   */
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Component
  public @interface RpcService {
  
      /**
       * 服务接口类
       */
      Class<?> interfaceClass() default void.class;
  
      /**
       * 服务版本号
       */
      String serviceVersion() default RpcConstant.DEFAULT_SERVICE_VERSION;
  }
  
  ```

#### 3. 注解驱动

- 在 starter 项目中新建 `bootstrap` 包，并且分别针对上面定义的3个注解新建启动类

##### 1. Rpc 框架全局启动类 RpcInitBootstrap

- 这个类的需求是，在 Spring 框架初始化时，获取`@EnableRpc` 注解的属性，并初始化 RPC 框架

- 怎么获取到注解的属性呢?

  - 可以实现 Spring 的 `ImportBeanDefinitionRegistrar` 接口，并且在 `registerBeanDefinitions` 方法中，获取到项目的注 解和注解属性

  ```java
  package com.lhk.kkrpcspringbootstarter.bootstrap;
  
  import com.lhk.kkrpc.RpcApplication;
  import com.lhk.kkrpc.config.RpcConfig;
  import com.lhk.kkrpc.server.tcp.VertxTcpServer;
  import com.lhk.kkrpcspringbootstarter.annotation.EnableRpc;
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.beans.factory.support.BeanDefinitionRegistry;
  import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
  import org.springframework.core.type.AnnotationMetadata;
  
  /**
   * Rpc 框架启动
   */
  @Slf4j
  public class RpcInitBootstrap implements ImportBeanDefinitionRegistrar {
  
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          // 获取 @EnableRpc 注解的属性值
          boolean needServer = (boolean) importingClassMetadata.getAnnotationAttributes(EnableRpc.class.getName())
                  .get("needServer");
  
          // Rpc 框架初始化，初始化 RPC 配置 (从 application.properties 文件中读取配置)
          RpcApplication.init();
          // 获取 RPC 配置对象
          RpcConfig rpcConfig = RpcApplication.getRpcConfig();
  
          // 启动服务器
          if (needServer) {
              // 初始化服务端
              // 启动 tcp 服务
              VertxTcpServer vertxTcpServer = new VertxTcpServer();
              vertxTcpServer.doStart(rpcConfig.getServerPort());
          }else {
              log.info("不启动 server");
          }
      }
  }
  
  ```

  - 上述代码中，我们从 Spring 元信息中获取到了 `@EnableRpc` 注解的 `needServer` 属性，并通过它来判断是否要启动服务器



##### 2. Rpc 服务提供者启动类 RpcProviderBootstrap

- 服务提供者启动类的作用是，获取到所有包含 `@Rpcservice` 注解的类，并且通过注解的属性和反射机制，获取到要注册的服务信息，并且完成服务注册

- 怎么获取到所有包含 `@Rpcservice` 注解的类呢?

  - 可以主动扫描包，也可以利用 Spring 的特性监听 Bean 的加载。

    - 此处选择后者，实现更简单，而且能直接获取到服务提供者类的 Bean 对象。
    - 只需要让启动类实现 `BeanpostProcessor` 接口的 `postProcessAfterInitialization` 方法，就可以在某个服务提供者 Bean 初始化后，执行注册服务等操作了

    ```java
    package com.lhk.kkrpcspringbootstarter.bootstrap;
    
    import com.lhk.kkrpc.RpcApplication;
    import com.lhk.kkrpc.config.RegistryConfig;
    import com.lhk.kkrpc.config.RpcConfig;
    import com.lhk.kkrpc.model.ServiceMetaInfo;
    import com.lhk.kkrpc.registry.LocalRegistry;
    import com.lhk.kkrpc.registry.Registry;
    import com.lhk.kkrpc.registry.RegistryFactory;
    import com.lhk.kkrpcspringbootstarter.annotation.RpcService;
    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.config.BeanPostProcessor;
    
    /**
     * Rpc 服务提供者启动
     */
    public class RpcProviderBootstrap implements BeanPostProcessor {
    
        /**
         * Bean 初始化后执行，注册服务
         * @param bean
         * @param beanName
         * @return
         * @throws BeansException
         */
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            Class<?> beanClass = bean.getClass();
            RpcService rpcServiceAnnotation = beanClass.getAnnotation(RpcService.class);
            if (rpcServiceAnnotation != null) {
                // 获取服务接口
                Class<?> interfaceClass = rpcServiceAnnotation.serviceInterface();
                // 默认值处理如果没有指定接口，则获取第一个接口
                if (interfaceClass == void.class) {
                    interfaceClass = (Class<?>) beanClass.getInterfaces()[0];
                }
                // 获取服务名
                String serviceName = interfaceClass.getName();
                // 获取服务版本
                String serviceVersion = rpcServiceAnnotation.version();
    
                // 注册服务
                // 本地注册
                LocalRegistry.register(serviceName, beanClass);
                // 获取 RPC 配置对象
                RpcConfig rpcConfig = RpcApplication.getRpcConfig();
                // 注册服务到注册中心
                RegistryConfig registryConfig = rpcConfig.getRegistryConfig();
                Registry registry = RegistryFactory.getInstance(registryConfig.getRegistry());
                ServiceMetaInfo serviceMetaInfo =  new ServiceMetaInfo();
                serviceMetaInfo.setServiceName(serviceName);
                serviceMetaInfo.setServiceVersion(serviceVersion);
                serviceMetaInfo.setServiceHost(rpcConfig.getServerHost());
                serviceMetaInfo.setServicePort(rpcConfig.getServerPort());
                try {
                    registry.register(serviceMetaInfo);
                } catch (Exception e) {
                    throw new RuntimeException(serviceName + " 注册失败",e);
                }
            }
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
    }
    
    ```

    - 其实上述代码中，绝大多数服务提供者初始化的代码都只需要从之前写好的启动类中复制粘贴，只不过换了一种参数获 取方式罢了



##### 3. Rpc 服务消费者启动类 RpcConsumerBootstrap

- 和服务提供者启动类的实现方式类似，在 Bean 初始化后，通过反射获取到 Bean 的所有属性，如果属性包含 `@RpcReferenc`e 注解，那么就为该属性动态生成代理对象并赋值

  ```java
  package com.lhk.kkrpcspringbootstarter.bootstrap;
  
  import com.lhk.kkrpc.proxy.ServiceProxyFactory;
  import com.lhk.kkrpcspringbootstarter.annotation.RpcReference;
  import org.springframework.beans.BeansException;
  import org.springframework.beans.factory.config.BeanPostProcessor;
  
  import java.lang.reflect.Field;
  
  /**
   * Rpc 服务消费者启动
   */
  public class RpcConsumerBootstrap implements BeanPostProcessor {
  
      /**
       * Bean 初始化后执行，注入服务
       * @param bean
       * @param beanName
       * @return
       * @throws BeansException
       */
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          Class<?> beanClass = bean.getClass();
          Field[] declaredField = beanClass.getDeclaredFields();
          // 遍历对象所有属性
          for (Field field : declaredField) {
              RpcReference rpcReference = field.getAnnotation(RpcReference.class);
              if (rpcReference != null) {
                  // 为属性生成代理对象
                  Class<?> interfaceClass = rpcReference.interfaceClass();
                  if (interfaceClass == void.class){
                      interfaceClass = field.getType();
                  }
                  field.setAccessible(true);
                  Object proxy = ServiceProxyFactory.getProxy(interfaceClass);
                  try {
                      field.set(bean, proxy);
                      field.setAccessible(false);
                  } catch (IllegalAccessException e) {
                      throw new RuntimeException("为字段注入代理对象失败",e);
                  }
              }
          }
          return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
      }
  }
  
  ```

  - 上述代码中，核心方法是 `beanclass.getDeclaredfields` ，用于获取类中的所有属性。看到这里的同学，必须要把反射的常用语法熟记于心了
  - `field.setAccessible(true)` 的功能是设置字段的访问权限为可访问状态，即使该字段是私有的。这允许通过反射机制直接操作类的私有字段，而不会受到 Java 访问控制的限制
  - `field.set(bean, proxy)` ：将 proxy 对象设置为 bean 中对应字段的值



##### 4. 注册已编写的启动类

- 最后，在 Spring 中加载已经编写好的启动类

  - 如何加载呢?

    - 需求是，仅在用户使用 `@EnableRpc` 注解时，才启动 RPC 框架。所以，可以通过给 `@EnableRpc` 增加 `@Import` 注解，来注册我们自定义的启动类，实现灵活的可选加载

    ```java
    package com.lhk.kkrpcspringbootstarter.annotation;
    
    import com.lhk.kkrpcspringbootstarter.bootstrap.RpcConsumerBootstrap;
    import com.lhk.kkrpcspringbootstarter.bootstrap.RpcInitBootstrap;
    import com.lhk.kkrpcspringbootstarter.bootstrap.RpcProviderBootstrap;
    import org.springframework.context.annotation.Import;
    
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
    
    /**
     * 启用 rpc 注解
     */
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Import({RpcInitBootstrap.class, RpcProviderBootstrap.class, RpcConsumerBootstrap.class})
    public @interface EnableRpc {
    
        /**
         * 默认需要启动服务端
         * @return
         */
        boolean needServer() default true;
    }
    
    ```

- 这样，一个基于注解驱动的 RPC 框架 Starter 开发完成



## 四、测试 starter

- 新建 2 个使用 Spring Boot 2 框架的项目

  - Spring Boot 消费者: `example-springboot-consumer`
  - Spring Boot 提供者:`example-springboot-provider`

- 都引入 rpc 框架依赖

  ```java
  
          <dependency>
              <groupId>com.lhk</groupId>
              <artifactId>kk-rpc-spring-boot-starter</artifactId>
              <version>0.0.1-SNAPSHOT</version>
          </dependency>
          <dependency>
              <groupId>com.lhk</groupId>
              <artifactId>example-common</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
  
  ```

- 在服务提供者项目的入口类加上`@EnableRpc` 注解

  ```java
  package com.lhk.examplespringbootprovider;
  
  import com.lhk.kkrpcspringbootstarter.annotation.EnableRpc;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @SpringBootApplication
  @EnableRpc
  public class ExampleSpringbootProviderApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(ExampleSpringbootProviderApplication.class, args);
      }
  
  }
  ```

- 在服务提供者项目提供一个示例服务，并加上`@RpcService`注解

  ```java
  package com.lhk.examplespringbootprovider;
  
  import com.lhk.example.common.model.User;
  import com.lhk.example.common.service.UserService;
  import com.lhk.kkrpcspringbootstarter.annotation.RpcService;
  import org.springframework.stereotype.Service;
  
  /**
   * 示例服务实现类
   */
  @Service
  @RpcService
  public class UserServiceImpl implements UserService {
      @Override
      public User getUser(User user) {
          System.out.println("用户名：" + user.getName());
          return user;
      }
  }
  
  ```

- 在服务消费者的入口类加上 `@EnableRpc(needserver = false)` 注解，标识启动 RPC 框架，但不启动服务器

  ```java
  package com.lhk.examplespringbootconsumer;
  
  import com.lhk.kkrpcspringbootstarter.annotation.EnableRpc;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @SpringBootApplication
  @EnableRpc(needServer = false)
  public class ExampleSpringbootConsumerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(ExampleSpringbootConsumerApplication.class, args);
      }
  
  }
  
  ```

- 服务消费者编写一个 Spring 的 Bean，引入 `UserService` 属性并打上 `@RpcReference` 注解，表示需要使用远程服务提供者的服 务

  ```java
  package com.lhk.examplespringbootconsumer;
  
  import com.lhk.example.common.model.User;
  import com.lhk.example.common.service.UserService;
  import com.lhk.kkrpcspringbootstarter.annotation.RpcReference;
  import org.springframework.stereotype.Service;
  
  /**
   * 远程调用示例
   */
  @Service
  public class ExampleConsumer {
  
      @RpcReference
      private static UserService userService;
  
      public  void consumer() {
          User user = new User();
          user.setName("lhk");
          System.out.println("远程调用后用户名："+userService.getUser(user));
      }
  }
  
  ```

- 服务消费者编写单元测试，验证能否调用远程服务

  ```java
  package com.lhk.examplespringbootconsumer;
  
  import org.junit.jupiter.api.Test;
  import org.springframework.boot.test.context.SpringBootTest;
  
  import javax.annotation.Resource;
  
  @SpringBootTest
  class ExampleSpringbootConsumerApplicationTests {
  
      @Resource
      private ExampleConsumer exampleConsumer;
  
      @Test
      void consumer() {
          exampleConsumer.consumer();
      }
  
  }
  
  ```

## 五、扩展

- Spring Boot Starter 项目支持读取 yml/yaml 配置文件来启动 RPC 框架。
  - 参考思路: 像读取 properties 文件一样，提供一个工具类来读取 yml 配置。
- 服务提供者启动逻辑也可以改 bean 后置执行为“使用组件扫描”