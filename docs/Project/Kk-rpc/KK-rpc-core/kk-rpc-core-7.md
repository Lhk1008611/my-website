---
sidebar_position: 7
---

# 重试机制

## 一、需求分析

- 目前，使用 RPC 框架的服务消费者调用接口失败时，就会直接报错
- 但是由于调用接口失败可能有很多原因
  - 有时可能是服务提供者返回了错误
  - 有时可能只是网络不稳定
  - 或服务提供者重启等临时性问题。
- 考虑这些情况，因此需要设计服务消费者拥有自动重试的能力，提高系统的可用性



## 二、设计方案

### 重试机制

- 如何设计重试机制？
  - 重试机制的核心是**重试策略**，一般来说，包含以下几个考虑点:
    1. 什么时候、什么条件下重试?
    2. 重试时间(确定下一次的重试时间)
    3. 什么时候、什么条件下停止重试?
    4. 重试后要做什么?

#### 1. 重试条件

- 首先是什么时候、什么条件下重试?
  - 我们希望提高系统的可用性，当由于**网络等异常**情况发生时，触发重试

#### 2. 重试时间

- 重试时间(也叫**重试等待**)的策略就比较丰富了，可能会用到一些算法，主流的重试时间算法有
  1. **固定重试间隔** (Fixed Retry Interval): 在每次重试之间使用固定的时间间隔
  2. **指数退避重试** (Exponential Backof Retny): 在每次失败后，重试的时间间隔会以指数级增加，以避免请求过于密集
  3. **随机延迟重试** (Random Delay Retry): 在每次重试之间使用随机的时间间隔，以避免请求的同时发生
  4. **可变延迟重试** (Variable Delay Retry): 这种策略更"高级"了，根据先前重试的成功或失败情况，动态调整下一次重试的延迟时间。比如，根据前一次的响应时间调整下一次重试的等待时间
- 以上的策略是可以**组合使用**的，一定要根据具体情况和需求灵活调整。比如可以先使用指数退避重试策略，如果连续多次重试失败，则切换到固定重试间隔策略

#### 3. 停止重试

- 一般来说，**重试次数是有上限的**，否则随着报错的增多，系统同时发生的重试也会越来越多，造成雪崩。
- 主流的停止重试策略有:
  1. **最大尝试次数**: 一般重试当达到最大次数时不再重试
  2. **超时停止**: 重试达到最大时间的时候，停止重试

#### 4. 重试工作

- 最后一点是重试后要做什么事情?
  - 一般来说就是重复执行原本要做的操作，比如发送请求失败了，那就再发一次请求。需要注意的是，当重试次数超过上限时，往往还要进行其他的操作，比如:
    1. **通知告警**: 让开发者人工介入
    2. **降级容错**: 改为调用其他接口、或者执行其他操作

### 重试方案设计

- 将 `vertxTcpclient.doRequest()` 封装为一个可重试的任务，如果请求失败(重试条件)，系统就会自动按照重试策略再次发起请求，不用开发者关心
- 对于重试算法，我们就选择主流的重试算法好了，Java 中可以使用 `Guava-Retrying` 库轻松实现多种不同的重试算法
  - 和序列化器、注册中心、负载均衡器一样，重试策略本身也可以使用 **SPI+ 工厂**的方式，允许开发者动态配置和扩展自 己的重试策略
- 最后，如果重试超过一定次数，停止重试，并且抛出异常

## 三、开发实现

### 多种重试策略的实现

- 在 RPC 项目中新建 `fault.retry` 包，将所有重试相关的代码放到该包下

#### 1. 编写通用接口

- 先编写重试策略通用接口。提供一个重试方法，接受一个具体的任务参数，可以使用 `Callable`类代表一个任务

  ```
  package com.lhk.kkrpc.fault.retry;
  
  import com.lhk.kkrpc.model.RpcResponse;
  import java.util.concurrent.Callable;
  
  /**
   * 重试策略
   */
  public interface RetryStrategy {
  
      /**
       * 重试
       *
       * @param callable
       * @return
       * @throws Exception
       */
      RpcResponse doRetry(Callable<RpcResponse> callable) throws Exception;
  }
  
  ```

#### 2. 引入 Guava-Retrying 重试库

```
<!-- https://github.com/rholder/guava-retrying -->
<dependency>
    <groupId>com.github.rholder</groupId>
    <artifactId>guava-retrying</artifactId>
    <version>2.0.0</version>
</dependency>
```



#### 3. 不重试策略

- 直接返回异常，不进行重试

  ```
  package com.lhk.kkrpc.fault.retry;
  
  import com.lhk.kkrpc.model.RpcResponse;
  import lombok.extern.slf4j.Slf4j;
  
  import java.util.concurrent.Callable;
  
  /**
   * 不重试 - 重试策略
   */
  @Slf4j
  public class NoRetryStrategy implements RetryStrategy {
  
      /**
       * 重试
       *
       * @param callable
       * @return
       * @throws Exception
       */
      public RpcResponse doRetry(Callable<RpcResponse> callable) throws Exception {
          return callable.call();
      }
  }
  ```



#### 4. 固定重试间隔策略实现

- 使用 `Guava-Retnying` 提供的 `RetryerBuilder` 能够很方便地指定重试条件、重试等待策略、重试停止策略、重试工作

  ```
  package com.lhk.kkrpc.fault.retry;
  
  import com.github.rholder.retry.*;
  import com.lhk.kkrpc.model.RpcResponse;
  import lombok.extern.slf4j.Slf4j;
  import java.util.concurrent.Callable;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.TimeUnit;
  
  /**
   * 固定时间间隔 - 重试策略
   */
  @Slf4j
  public class FixedIntervalRetryStrategy implements RetryStrategy {
  
      /**
       * 重试
       *
       * @param callable
       * @return
       * @throws ExecutionException
       * @throws RetryException
       */
      public RpcResponse doRetry(Callable<RpcResponse> callable) throws ExecutionException, RetryException {
          Retryer<RpcResponse> retryer = RetryerBuilder.<RpcResponse>newBuilder()
                  .retryIfExceptionOfType(Exception.class)
                  .withWaitStrategy(WaitStrategies.fixedWait(3L, TimeUnit.SECONDS))  //固定时间间隔重試策略
                  .withStopStrategy(StopStrategies.stopAfterAttempt(3)) //重试次数
                  .withRetryListener(new RetryListener() {
                      @Override
                      public <V> void onRetry(Attempt<V> attempt) {
                          log.info("重试次数 {}", attempt.getAttemptNumber());
                      }
                  })
                  .build();
          return retryer.call(callable);
      }
  }
  ```

- 重试条件: 使用 `retrylfExceptionOfType()` 方法指定当出现 `Exception` 异常时重试。

- 重试等待策略: 使用 `withWaitStrategy()` 方法指定策略，选择 `fixedWait()` 固定时间间隔策略。

- 重试停止策略: 使用 `withStopStrategy()` 方法指定策略，选择 `stopAfterAttempt()` 超过最大重试次数停止。

- 重试工作: 使用 `withRetryListener()` 监听重试，每次重试时，除了再次执行任务外，还能够打印当前的重试次数

#### 5. 单元测试

```
package com.yupi.yurpc.fault.retry;

import com.yupi.yurpc.model.RpcResponse;
import org.junit.Test;

/**
 * 重试策略测试
 */
public class RetryStrategyTest {

    RetryStrategy retryStrategy = new NoRetryStrategy();

    @Test
    public void doRetry() {
        try {
            RpcResponse rpcResponse = retryStrategy.doRetry(() -> {
                System.out.println("测试重试");
                throw new RuntimeException("模拟重试失败");
            });
            System.out.println(rpcResponse);
        } catch (Exception e) {
            System.out.println("重试多次失败");
            e.printStackTrace();
        }
    }
}
```

### 实现支持配置和扩展重试策略（工厂模式 + SPI）

- 一个成熟的 RPC 框架可能会支持多种不同的重试策略，像序列化器、注册中心、负载均衡器一样，让开发者能够填写配置来指定使用的重试策略，并且支持自定义重试策略，让框架更易用、更利于扩展
- 要实现这点，开发方式和序列化器、注册中心、负载均衡器都是一样的，都可以使用**工厂**创建对象、使用 **SPI 动态加载**自定义的注册中心

#### 1. 重试策略常量

- 新建 `RetrystrategyKeys` 类，列举所有支持的重试策略键名

  ```
  package com.lhk.kkrpc.fault.retry;
  
  /**
   * 重试策略键名常量
   */
  public interface RetryStrategyKeys {
  
      /**
       * 不重试
       */
      String NO = "no";
  
      /**
       * 固定时间间隔
       */
      String FIXED_INTERVAL = "fixedInterval";
  
  }
  ```

#### 2. 使用工厂模式，实现根据 key 从 SPI 获取重试策略对象实例

- 新建 `RetryStrategyFactory` 类 

  ```
  package com.lhk.kkrpc.fault.retry;
  
  
  import com.lhk.kkrpc.spi.SpiLoader;
  
  /**
   * 重试策略工厂（用于获取重试器对象）
   */
  public class RetryStrategyFactory {
  
      static {
          SpiLoader.load(RetryStrategy.class);
      }
  
      /**
       * 默认重试器
       */
      private static final RetryStrategy DEFAULT_RETRY_STRATEGY = new NoRetryStrategy();
  
      /**
       * 获取实例
       *
       * @param key
       * @return
       */
      public static RetryStrategy getInstance(String key) {
          return SpiLoader.getInstance(RetryStrategy.class, key);
      }
  
  }
  ```



#### 3. 编写重试策略的 SPI 配置文件

- 在 `META-INF` 的 `rpc/system` 目录下编写重试策略接口的 SPI 配置文件，文件名称为 `com.lhk.kkrpc.fault.retry.RetryStrategy`

  ```
  no=com.lhk.kkrpc.fault.retry.NoRetryStrategy
  fixedInterval=com.lhk.kkrpc.fault.retry.FixedIntervalRetryStrategy
  ```

  

#### 4. 新增重试策略的全局配置

- 为 `RpcConfig` 全局配置新增重试策略的配置

  ```
  package com.lhk.kkrpc.config;
  
  import com.lhk.kkrpc.fault.retry.RetryStrategyKeys;
  import com.lhk.kkrpc.loadbalancer.LoadBalancerKeys;
  import com.lhk.kkrpc.serializer.SerializerKeys;
  import lombok.Data;
  
  /**
   * RPC 框架配置
   */
  @Data
  public class RpcConfig {
  
      /**
       * 名称
       */
      private String name = "kk-rpc";
  
      /**
       * 版本号
       */
      private String version = "1.0";
  
      /**
       * 服务器主机名
       */
      private String serverHost = "localhost";
      
      /**
       * 服务器端口号
       */
      private Integer serverPort = 8888;
  
      /**
       * 模拟调用
       */
      private boolean mock = false;
  
      /**
       * 注册中心配置
       */
      private RegistryConfig registryConfig = new RegistryConfig();
  
      /**
       * 序列化器
       */
      private String serializer = SerializerKeys.JDK;
  
      /**
       * 负载均衡器
       */
      private String loadBalancer = LoadBalancerKeys.ROUND_ROBIN;
  
      /**
       * 重试策略
       */
      private String retryStrategy = RetryStrategyKeys.NO;
  }
  ```



### 应用重试策略

- 修改 `ServiceProxy` 的代码，从工厂中获取重试器，并且将请求代码封装为个 `Callable` 接口，作为重试器的参数，调用重试器即可

  ```
  package com.lhk.kkrpc.proxy;
  
  import cn.hutool.core.collection.CollUtil;
  import com.lhk.kkrpc.RpcApplication;
  import com.lhk.kkrpc.config.RpcConfig;
  import com.lhk.kkrpc.constant.RpcConstant;
  import com.lhk.kkrpc.fault.retry.RetryStrategy;
  import com.lhk.kkrpc.fault.retry.RetryStrategyFactory;
  import com.lhk.kkrpc.loadbalancer.LoadBalancer;
  import com.lhk.kkrpc.loadbalancer.LoadBalancerFactory;
  import com.lhk.kkrpc.model.RpcRequest;
  import com.lhk.kkrpc.model.RpcResponse;
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  import com.lhk.kkrpc.registry.Registry;
  import com.lhk.kkrpc.registry.RegistryFactory;
  import com.lhk.kkrpc.server.tcp.VertxTcpClient;
  
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  import java.util.HashMap;
  import java.util.List;
  
  /**
   * 服务代理（JDK 动态代理）（客户端发送请求时对请求进行处理后再发送）
   */
  public class ServiceProxy implements InvocationHandler {
  
      /**
       * 调用代理发送请求
       *
       * @return
       * @throws Throwable
       */
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) {
          // 构造请求
          String serviceName = method.getDeclaringClass().getName();
          RpcRequest rpcRequest = RpcRequest.builder()
                  .serviceName(serviceName)
                  .methodName(method.getName())
                  .parameterTypes(method.getParameterTypes())
                  .args(args)
                  .build();
          try {
              // 从注册中心获取服务提供者请求地址
              RpcConfig rpcConfig = RpcApplication.getRpcConfig();
              Registry registry = RegistryFactory.getInstance(rpcConfig.getRegistryConfig().getRegistry());
              ServiceMetaInfo serviceMetaInfo = new ServiceMetaInfo();
              serviceMetaInfo.setServiceName(serviceName);
              serviceMetaInfo.setServiceVersion(RpcConstant.DEFAULT_SERVICE_VERSION);
              List<ServiceMetaInfo> serviceMetaInfoList = registry.serviceDiscovery(serviceMetaInfo.getServiceKey());
              if (CollUtil.isEmpty(serviceMetaInfoList)) {
                  throw new RuntimeException("暂无服务地址");
              }
              // 负载均衡
              LoadBalancer loadBalancer = LoadBalancerFactory.getInstance(rpcConfig.getLoadBalancer());
              HashMap<String, Object> requestParams = new HashMap<>();
              requestParams.put("methodName", rpcRequest.getMethodName());
              ServiceMetaInfo selectedServiceMetaInfo = loadBalancer.select(requestParams, serviceMetaInfoList);
  
              // 发送 tcp 请求
              // 使用重试机制
              RetryStrategy retryStrategy = RetryStrategyFactory.getInstance(rpcConfig.getRetryStrategy());
              RpcResponse rpcResponse = retryStrategy.doRetry(() ->
                      VertxTcpClient.doRequest(rpcRequest, selectedServiceMetaInfo)
              );
              return rpcResponse.getData();
          }catch (Exception e){
              throw new RuntimeException("rpc 服务调用失败");
          }
      }
  }
  
  ```

  - 上述代码中，使用 Lambda 表达式将 `VertxTcpclient.doRequest()` 封装为了一个匿名函数，简化了代码

## 四、测试

- 首先启动服务提供者，然后使用 Debug 模式启动服务消费者，当服务消费者发起调用时，立刻停止服务提供者，就会看 到调用失败后重试的情况



## 五、扩展

- 新增更多不同类型的重试器
  - 参考思路: 比如指数退避算法的重试器