---
sidebar_position: 6
---



# 负载均衡

## 一、需求分析

- 目前这个 rpc 框架实现了服务消费者从 etcd 注册中心获取服务提供者注册的信息，当然同一个服务可能会有多个服务提供者，但是目前我们消费者始终读取了第一个服务提供者节点发起调用，不仅会增大单个节点的压力，而且没有利用好其他节点的资源。
- 因此我们完全可以从服务提供者节点中，选择一个服务提供者发起请求，而不是每次都请求同一个服务提供者，这个操作就叫做 **负载均衡**。



## 二、负载均衡

### 什么是负载均衡

- 何为**负载**?可以把负载理解为要处理的工作和压力，比如网络请求、事务、数据处理任务等
- 何为**均衡**?把工作和压力平均地分配给多个工作者，从而分摊每个工作者的压力，保证大家正常工作
- 所以，负载均衡是一种用来**分配网络或计算负载到多个资源上**的技术。它的目的是确保每个资源都能够有效地处理负载、增加系统的并发量、避免某些资源过载而导致性能下降或服务不可用的情况
- 常用的负载均衡实现技术有 Nginx(七层负载均衡)、LVS(四层负载均衡)等

### 常见的负载均衡算法

#### 1. 轮询

- 轮询(Round Robin): 按照循环的顺序将请求分配给每个服务器，适用于各服务器性能相近的情况

#### 2. 随机

- 随机(Random): 随机选择一个服务器来处理请求，适用于服务器性能相近且负载均匀的情况

#### 3. 加权轮询

- 3)加权轮询(Weighted Round Robin): 适用于服务器性能不均的情况。根据服务器的性能或权重分配请求，性能更好的服务器会获得更多的请求

#### 4. 加权随机

- 加权随机(Weighted Random): 根据服务器的权重随机选择一个服务器处理请求，适用于服务器性能不均的情况

#### 5. 最小连接数

- 最小连接数(Least Connections): 选择当前连接数最少的服务器来处理请求，适用于长连接场景

#### 6. ip hash

- IP Hash: 根据客户端 IP 地址的哈希值选择服务器处理请求，确保同一客户端的请求始终被分配到同一台服务器上，适用于需要保持会话一致性的场景，当然，也可以根据请求中的其他参数进行 Hash，比如根据请求接口的地址路由到不同的服务器节点

### 一致性 hash

- 一致性哈希(Consistent Hashing)是一种经典的哈希算法，用于**将请求分配到多个节点或服务器上**，所以非常适用于负载均衡。
  - 它的核心思想是将整个哈希值空间划分成一个环状结构，每个节点或服务器在环上占据一个位置，每个请求根据其哈希值映射到环上的一个点，然后顺时针寻找第一个大于或等于该哈希值的节点，将请求路由到该节点上
- 一致性哈希还解决了 **节点下线** 和 **倾斜问题**
  - **节点下线**: 当某个节点下线时，该节点上的负载（请求）会被平均分摊到其他节点上，而不会影响到整个系统的稳定性，因为只有该节点的部分请求会受到影响
    - 如果是轮询取模算法，只要节点数变了，很有可能大多数服务器处理的请求都要发生节点的变化，对系统的影响巨大
  - **倾斜问题**: 通过虚拟节点的引入，将每个物理节点映射到多个虚拟节点上，使得节点在哈希环上的分布更加均匀，减少了节点间的负载差异。

## 三、开发实现

### 多种负载均衡器的实现

- 对于负载均衡器的实现，可以参考 Nginx 的负载均衡器的实现

#### 1. 通用接口编写

- 建立 `loadbalancer` 包，编写一个负载均衡器的通用接口

  - 提供一个`select()` 选择服务方法，接受请求参数和可用服务列表，可以根据这些信息进行节点的选择

  ```java
  package com.lhk.kkrpc.loadbalancer;
  
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  
  import java.util.List;
  import java.util.Map;
  
  /**
   * 负载均衡器（消费端使用）
   */
  public interface LoadBalancer {
  
      /**
       * 选择服务调用
       *
       * @param requestParams       请求参数
       * @param serviceMetaInfoList 可用服务列表
       * @return
       */
      ServiceMetaInfo select(Map<String, Object> requestParams, List<ServiceMetaInfo> serviceMetaInfoList);
  }
  ```

  

#### 2. 轮询负载均衡器的实现

- 使用 JUC 包的 `AtomicInteger` 实现原子计数器，防止并发冲突问题

  ```java
  package com.lhk.kkrpc.loadbalancer;
  
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  
  import java.util.List;
  import java.util.Map;
  import java.util.concurrent.atomic.AtomicInteger;
  
  /**
   * 轮询负载均衡器
   */
  public class RoundRobinLoadBalancer implements LoadBalancer {
  
      /**
       * 当前轮询的下标
       * 使用 JUC 包的 AtomicInteger 实现原子计数器，防止并发冲突问题
       */
      private final AtomicInteger currentIndex = new AtomicInteger(0);
  
      @Override
      public ServiceMetaInfo select(Map<String, Object> requestParams, List<ServiceMetaInfo> serviceMetaInfoList) {
          if (serviceMetaInfoList.isEmpty()) {
              return null;
          }
          // 只有一个服务，无需轮询
          int size = serviceMetaInfoList.size();
          if (size == 1) {
              return serviceMetaInfoList.get(0);
          }
          // 取模算法轮询
          int index = currentIndex.getAndIncrement() % size; // 加一取模
          return serviceMetaInfoList.get(index);
      }
  }
  ```

#### 3. 随机负载均衡器的实现

- 使用 Java 自带的 Random 类实现随机选取即可

  ```java
  package com.lhk.kkrpc.loadbalancer;
  
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  
  import java.util.List;
  import java.util.Map;
  import java.util.Random;
  
  /**
   * 随机负载均衡器
   */
  public class RandomLoadBalancer implements LoadBalancer {
  
      private final Random random = new Random();
  
      @Override
      public ServiceMetaInfo select(Map<String, Object> requestParams, List<ServiceMetaInfo> serviceMetaInfoList) {
          int size = serviceMetaInfoList.size();
          if (size == 0) {
              return null;
          }
          // 只有 1 个服务，不用随机
          if (size == 1) {
              return serviceMetaInfoList.get(0);
          }
          return serviceMetaInfoList.get(random.nextInt(size));
      }
  }
  ```

#### 4. 一致性 hash 负载均衡器的实现

- 可以使用 `TreeMap` 实现有序的一致性 Hash 环，该数据结构提供了 `ceilingEntry()` 和 `firstEntry()` 两个方法，便于获取符合算法要求的节点

  ```java
  package com.lhk.kkrpc.loadbalancer;
  
  import com.lhk.kkrpc.model.ServiceMetaInfo;
  
  import java.util.List;
  import java.util.Map;
  import java.util.TreeMap;
  
  /**
   * 一致性 hash 负载均衡器
   */
  public class ConsistentHashLoadBalancer implements LoadBalancer {
  
      /**
       * 一致性 hash 环，存放虚拟节点
       * TreeMap 保证节点的有序性，会根据节点的 hash 值进行排序
       */
      private final TreeMap<Integer, ServiceMetaInfo> virtualNodes= new TreeMap<>();
  
      /**
       * 单个节点的虚拟节点数量，默认 100
       */
      private final int VIRTUAL_NODE_COUNT = 100;
  
      @Override
      public ServiceMetaInfo select(Map<String, Object> requestParams, List<ServiceMetaInfo> serviceMetaInfoList) {
          if (serviceMetaInfoList.isEmpty()) {
              return null;
          }
  
          // 构建一致性 hash 环
          for (ServiceMetaInfo serviceMetaInfo : serviceMetaInfoList) {
              for (int i = 0; i < VIRTUAL_NODE_COUNT; i++) {
                  Integer virtualNodeHash = getHash(serviceMetaInfo.getServiceKey() + "#" + i);
                  virtualNodes.put(virtualNodeHash, serviceMetaInfo);
              }
          }
  
          // 根据请求参数的 hash 值，在虚拟节点的 hash 值范围内进行查找，找到对应的虚拟节点，返回对应的真实节点
          Integer requestHash = getHash(requestParams);
          // 选择最接近且大于等于调用请求 hash 值的虚拟节点
          Map.Entry<Integer, ServiceMetaInfo> entry = virtualNodes.ceilingEntry(requestHash);
          if (entry == null){
              // 如果没有找到，则选择第一个虚拟节点
              return virtualNodes.firstEntry().getValue();
          }
          return entry.getValue();
      }
  
      /**
       * 计算 key 的 hash 值
       * @param key
       * @return
       */
      private Integer getHash(Object key) {
          // 可以实现更好的哈希算法
          return key.hashCode();
      }
  }
  
  ```

  - 注意：每次调用负载均衡器时，都会重新构造 Hash 环，这是为了能够即时处理节点的变化，重新分配下线节点的请求



### 实现支持配置和扩展负载均衡器（工厂模式 + SPI）

- 一个成熟的 RPC 框架可能会**支持多个负载均衡器**，像序列化器和注册中心一样，能够让开发者能够填写配置来指定负载均衡器，并且支持自定义负载均衡器，让框架更易用、更利于扩展
  - 要实现这点，开发方式和序列化器、注册中心都是一样的，都可以使用**工厂模式**创建对象、使用 **SPI 机制**动态加载自定义的注册中心

#### 1. 负载均衡器常量

- 列举出框架支持的所有负载均衡器键名

  ```java
  package com.lhk.kkrpc.loadbalancer;
  
  /**
   * 负载均衡器键名常量
   */
  public interface LoadBalancerKeys {
  
      /**
       * 轮询
       */
      String ROUND_ROBIN = "roundRobin";
  
      /**
       * 随机
       */
      String RANDOM = "random";
  
      /**
       * 一致性哈希
       */
      String CONSISTENT_HASH = "consistentHash";
  
  }
  ```



#### 2. 使用工厂模式，实现根据 key 从 SPI 获取指定负载均衡器对象实例

```java
package com.lhk.kkrpc.loadbalancer;

import com.lhk.kkrpc.spi.SpiLoader;

/**
 * 负载均衡器工厂（工厂模式，用于获取负载均衡器对象）
 */
public class LoadBalancerFactory {

    static {
        SpiLoader.load(LoadBalancer.class);
    }

    /**
     * 默认负载均衡器
     */
    private static final LoadBalancer DEFAULT_LOAD_BALANCER = new RoundRobinLoadBalancer();

    /**
     * 获取实例
     *
     * @param key
     * @return
     */
    public static LoadBalancer getInstance(String key) {
        return SpiLoader.getInstance(LoadBalancer.class, key);
    }
}
```

#### 3. 编写负载均衡器的 SPI 配置文件

- 在 `META-INF` 的 `rpc/system` 目录下编写负载均衡器接口的 SPI 配置文件，文件名称为 `com.lhk.kkrpc.loadbalance
  r.LoadBalancer`

  ```java
  roundRobin=com.lhk.kkrpc.loadbalancer.RoundRobinLoadBalancer
  random=com.lhk.kkrpc.loadbalancer.RandomLoadBalancer
  consistentHash=com.lhk.kkrpc.loadbalancer.ConsistentHashLoadBalancer
  ```

#### 4. 新增负载均衡器的全局配置

- 在 `RpcConfig` 全局配置中新增负载均衡器的配置

  ```java
  package com.lhk.kkrpc.config;
  
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
  }
  ```

### 应用负载均衡器

- 修改 `ServiceProxy` 的代码，将获取服务节点的逻辑改为调用负载均衡器获取一个服务节点

  ```java
  package com.lhk.kkrpc.proxy;
  
  import cn.hutool.core.collection.CollUtil;
  import com.lhk.kkrpc.RpcApplication;
  import com.lhk.kkrpc.config.RpcConfig;
  import com.lhk.kkrpc.constant.RpcConstant;
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
              RpcResponse rpcResponse = VertxTcpClient.doRequest(rpcRequest, selectedServiceMetaInfo);
              return rpcResponse.getData();
          }catch (Exception e){
              throw new RuntimeException("rpc 服务调用失败");
          }
      }
  }
  ```

- 上述代码中，我们给负载均衡器传入了一个 `requestParams` ，并且将请求方法名作为参数放到了 HashMap 中。如果使用的是一致性 Hash 算法，那么会根据 `requestParams` 计算 Hash 值，调用相同方法的请求 Hash 值肯定相同，所以总会请求到同一个服务器节点上

## 四、测试

1. 单元测试类

   ```java
   package com.lhk.kkrpc.loadbalancer;
   
   import com.lhk.kkrpc.model.ServiceMetaInfo;
   import org.junit.Assert;
   import org.junit.Test;
   
   import java.util.Arrays;
   import java.util.HashMap;
   import java.util.List;
   import java.util.Map;
   
   import static org.junit.Assert.*;
   
   /**
    * 负载均衡器测试
    */
   public class LoadBalancerTest {
   
       final LoadBalancer loadBalancer = new ConsistentHashLoadBalancer();
   
       @Test
       public void select() {
           // 请求参数
           Map<String, Object> requestParams = new HashMap<>();
           requestParams.put("methodName", "apple");
           // 服务列表
           ServiceMetaInfo serviceMetaInfo1 = new ServiceMetaInfo();
           serviceMetaInfo1.setServiceName("myService");
           serviceMetaInfo1.setServiceVersion("1.0");
           serviceMetaInfo1.setServiceHost("localhost");
           serviceMetaInfo1.setServicePort(1234);
           ServiceMetaInfo serviceMetaInfo2 = new ServiceMetaInfo();
           serviceMetaInfo2.setServiceName("myService");
           serviceMetaInfo2.setServiceVersion("1.0");
           serviceMetaInfo2.setServiceHost("lhk.rpc");
           serviceMetaInfo2.setServicePort(80);
           List<ServiceMetaInfo> serviceMetaInfoList = Arrays.asList(serviceMetaInfo1, serviceMetaInfo2);
           // 连续调用 3 次
           ServiceMetaInfo serviceMetaInfo = loadBalancer.select(requestParams, serviceMetaInfoList);
           System.out.println(serviceMetaInfo);
           Assert.assertNotNull(serviceMetaInfo);
           serviceMetaInfo = loadBalancer.select(requestParams, serviceMetaInfoList);
           System.out.println(serviceMetaInfo);
           Assert.assertNotNull(serviceMetaInfo);
           serviceMetaInfo = loadBalancer.select(requestParams, serviceMetaInfoList);
           System.out.println(serviceMetaInfo);
           Assert.assertNotNull(serviceMetaInfo);
       }
   }
   ```

2. 首先在不同的端口启动 2 个服务提供者，然后启动服务消费者项目，通过 Debug 或者控制台输出来观察每次请求的节点地址

## 五、 扩展

1. 实现更多不同算法的负载均衡器
   - 参考思路: 比如**最少活跃数负载均衡器**，选择当前正在处理请求的数量最少的服务提供者
2. 自定义一致性 Hash 算法中的 Hash 算法
   - 参考思路: 比如**根据请求客户端的 IP 地址来计算 Hash 值**，保证同 IP 的请求发送给相同的服务提供者