# Mybatis

## 1、Mybatis 获取参数值的两种方式

>两种获取参数值的方式：**${}** 和 **#{}**
>
>${}的本质是字符串拼接，会造成SQL注入问题，在使用时需要注意添加单引号，例如：'${userName}'
>
>#{}的本质是占位符赋值，避免SQL注入问题，在填充占位符 (?) 时会自动带上单引号

### Mybatis 获取参数的情况

1. 若 mapper 接口方法的参数为单个的字面量类型，此时可以通过 #{} 和 ${} 以任意内容获取参数值（mybatis3.5版本之后）

2. 若 mapper 接口方法的参数为多个的字面量类型，此时 mybatis 会将多个参数放入按以下两种方式存储在map集合中

   1. 以 arg0，arg1...为 key ，以参数为 value 存储
   2. 以 param1，param2...为 key，以参数为 value 存储 

   - **因此只需通过 #{} 和 ${} 访问map集合的 key ，就可以获取相应的参数值**

3. 若 mapper 接口方法的参数为map集合类型的参数，**只通过 #{} 和 ${} 访问map集合的 key ，就可以获取相应的参数值**

4. 若 mapper 接口方法的参数为实体类类型的参数，**只需要通过 #{} 和 ${} 访问实体类中的属性名，就可以获取相应属性值**

   - 实体类中的属性名与实体类中的成员变量无关，只与实体类中的 getter 和 setter 方法有关
   - 当实体类没有成员变量，只有 getter 和 setter 方法，也是可以访问到实体类的属性

5. 可以在 mapper 接口方法的参数上设置 @Param 注解，此时 mybatis 会将这些参数按以下两种方式存储在map集合中

   1. 以 @Param 注解的 value 属性值为 key ，以参数为 value 存储
   2. 以 param1，param2...为 key，以参数为 value 存储 

   - **只需通过 #{} 和 ${} 访问map集合的 key ，就可以获取相应的参数值**

## 2、Mybatis的各种查询

1. 查询单个实体类对象
   - 若 SQL 语句查询的结果为多条时，则不能以实体类类型作为 mapper 接口方法的返回值类型，否则会抛出异常 TooManyResultsException
   - 若 SQL 语句查询的结果为一条时，则可以使用实体类类型或者 List 集合类型作为mapper接口方法方返回值类型
2. 查询一个 List 集合
3. 查询单个数据值（例：count(*) ....）
   - ResultType 可以使用类型别名
4. 查询一条数据为 map 集合
   - ResultType = map
   - 查询出来的 map 集合会以字段名为 key ，以字段值为value
5. 查询多条数据为 map 集合
   - ResultType = map
   - 一个 map 只能对应一条查询出来的记录，若需要查询多条数据为 map 集合可以通过以下两种方式
     1. 将 mapper 接口方法的返回值类型设置为泛型是 map 的 list 集合，例：List<Map<String,Object>>
     2. 在 mapper 接口方法中使用 @MapKey 注解，可以将查询的多条 map 集合数据放入一个 map 集合中，当时需要将查询的某个字段的值作为该大 map 的 key，例如：@MapKey('id')，则会以字段 id 的值作为大 map 的 key

## 3、MyBatis 中自带的常用的类型别名

查看地址：[配置_MyBatis中文网](https://mybatis.net.cn/configuration.html#typeAliases)

## 4、 特殊 SQL 的执行

1. MyBatis 实现模糊查询，使用 mybatis 的三种 SQL 写法

   1. 使用 ${} 进行字串拼接，实现模糊查询

      ```sql
      select * from tableName where columnName like '%${query}%'
      ```

   2. 使用 concat() 函数拼接 #{} 占位符，也可实现模糊查询

      ```sql
      select * from tableName where columnName like concat('%',#{query},'%')
      ```

   3. 使用 "" 拼接 #{} 实现模糊查询

      ```sql
      select * from tableName where columnName like "%"#{query}"%"
      ```

2. MyBatis 实现批量删除

   ```sql
   delete from tableName where id in(${ids})  #ids 的形式为 1,2,3,.. 的字串
   ```

3. MyBatis 动态设置表名，使用 ${}

   ```sql
   select * from ${tableName}
   ```

4. 添加功能——获取添加成功后自增的主键

   ```xml
       <!--
           添加用户 并获取自增的主键
           useGeneratedKeys: 表示当前添加功能是否使用自增的主键
           keyProperty: 将添加成功后自增的主键赋值给实体类参数的属性
       -->
       <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
           insert into user1 values(null,#{name},#{pwd})
       </insert>
   ```

## 5、自定义映射 resultMap

1. 字段名与实体类属性名不一致的情况，处理映射的方式如下

   1. 可以在 SQL 中给字段设置和属性名一致的别名

   2. 若字段名使用 _ 命名方式，属性使用驼峰命名方式，则可以在 MyBatis 核心配置文件中添加以下配置，可以实现自动将 _ 映射为驼峰

      ```xml
          <settings>
              <setting name="mapUnderscoreToCamelCase" value="true"/>
          </settings>
      ```

   3. 使用 resultMap 实现自定义映射 

      - id：设置唯一标识
      - type ：处理映射关系对应的实体类类型
      - id 标签：处理主键和实体类中属性的映射关系
      - result 标签：处理普通字段和实体类中属性的映射关系
      - association: 处理多对一或一对一映射关系（处理实体类类型属性）
      - column：字段名
   - property：实体类的属性名
     
      ```xml
          <select id="getUserById" resultMap="userMap">
              select * from user1 where id=#{userId}
          </select>
          <resultMap id="userMap" type="user">
              <id column="id" property="id"></id>
              <result column="name" property="name"></result>
              <result column="pwd" property="password"></result>
          </resultMap>
      ```

2. 多对一映射处理

   1. 级联方式处理

      ```xml
        <resultMap id="orderMap" type="order">
              <result column="ordertime" property="orderTime"></result>
              <result column="oid" property="id"></result>
              <result column="total" property="total"></result>
              
              <result column="uid" property="user.id"></result>
              <result column="name" property="user.name"></result>
              <result column="pwd" property="user.pwd"></result>
          </resultMap>
          
          <select id="queryOrder" resultMap="orderMap">
              SELECT *,o.`id` oid FROM user1 u,orders o WHERE o.`uid`=u.`id`
          </select>
      ```

   2. 使用 association 标签处理映射

      ```xml
          <resultMap id="orderMap" type="order">
              <id column="oid" property="id"></id>
              <result column="ordertime" property="orderTime"></result>
              <result column="total" property="total"></result>
              <!--
                  association: 处理多对一或一对一映射关系（处理实体类类型属性）
               -->
              <association property="user" javaType="user" >
                  <id column="uid" property="id"></id>
                  <result column="name" property="name"></result>
                  <result column="pwd" property="pwd"></result>
              </association>
          </resultMap>
      ```

   3. 分步查询

      ```xml
          <!--  第三种：分步查询  -->
          <resultMap id="queryOrderAndUserStepOneMap" type="order">
              <id column="id" property="id"></id>
              <result column="ordertime" property="orderTime"></result>
              <result column="total" property="total"></result>
              <!--
                  association: 处理多对一或一对一映射关系（处理实体类类型属性）
                  property: 设置需要映射的实体类
                  select：设置下一步 SQL 查询语句的唯一标识
                  column：将本步查询出来的字段作为下一步 SQL 查询的条件
               -->
              <association
                      property="user"
                      select="com.lhk.mapper.UserMapper.queryOrderAndUserStepTwo"
                      column="uid" >
              </association>
          </resultMap>
      
          <select id="queryOrderAndUserStepOne" resultMap="queryOrderAndUserStepOneMap">
              select * from orders
          </select>
      
      ```

      - 分步查询的优势：可以实现**延迟加载**（懒加载），（用到哪些数据，就只执行对应的SQL语句，按需加载）

        1. 全局配置延迟加载

           ```xml
                   <!-- 开启延迟加载 -->
                   <setting name="lazyLoadingEnabled" value="true"/>
                   <!-- 开启按需加载,默认为false -->
                   <setting name="aggressiveLazyLoading" value="false"/>
           ```

        2. 配置某个分布查询延迟加载

           - fetchType="lazy"（本分步查询开启延迟加载）
           - fetchType="eager"  （本分步查询开启立即加载）

           ```xml
           <association
           	property="user"
           	fetchType="lazy"
               select="com.lhk.mapper.UserMapper.queryOrderAndUserStepTwo"
           	column="uid" >
           </association>
           ```

3. 一对多映射处理

   1.  使用 collection 标签一步查询
   
      ```xml
          <resultMap id="userMap" type="user">
              <id column="uid" property="id"></id>
              <result column="name" property="name"></result>
              <result column="pwd" property="pwd"></result>
      
              <!--
                  一个user对应多个order
                  collection: 处理一对多或多对多映射关系
                  ofType： 设置集合中存储的数据的类型
              -->
              <collection property="orderList" ofType="order">
                  <id column="oid" property="id"></id>
                  <result column="ordertime" property="orderTime"></result>
                  <result column="total" property="total"></result>
              </collection>
          </resultMap>
      
          <select id="queryUser" resultMap="userMap">
              SELECT *,o.`id` oid FROM user1 u,orders o WHERE o.`uid`=u.`id`
          </select>
      ```
   
   2. 分布查询
   
      ```xml
          <resultMap id="queryUserAndOrderStepOneMap" type="user">
              <id column="uid" property="id"></id>
              <result column="name" property="name"></result>
              <result column="pwd" property="pwd"></result>
      
              <collection 
      			property="orderList"
                  select="com.lhk.mapper.OrderMapper#queryUserAndOrderStepTwo"
                  column="id"
              ></collection>
          </resultMap>
      
          <select id="queryUserAndOrderStepOne" resultType="queryUserAndOrderStepOneMap">
              select * from user1 where id = #{userId}
          </select>
      ```

## 6、动态SQL

1. if 标签

   - test 属性：通过属性 test 中的表达式判断标签中的内容是否可以进行 SQL 拼接

   - 只使用 if 标签会出现的问题：当所有条件不满足时或第一个条件不满足时，就会出现 SQL 语法错误，如下的代码就会出现上述问题

     ```xml
     <select id="findActiveBlogLike"
          resultType="Blog">
       SELECT * FROM BLOG
       WHERE
       <if test="state != null">
         state = #{state}
       </if>
       <if test="title != null">
         AND title like #{title}
       </if>
       <if test="author != null and author.name != null">
         AND author_name like #{author.name}
       </if>
     </select>
     ```

     - 解决方案：

       1. 可以在 where 关键字后面添加上一个恒成立的条件，并给第一个条件加上 AND，如下

          ```xml
          <select id="findActiveBlogLike"
               resultType="Blog">
            SELECT * FROM BLOG
            WHERE 1=1
            <if test="state != null">
              AND state = #{state}
            </if>
            <if test="title != null">
              AND title like #{title}
            </if>
            <if test="author != null and author.name != null">
              AND author_name like #{author.name}
            </if>
          </select>
          ```

       2. 使用 where 标签

          ```xml
          <select id="findActiveBlogLike"
               resultType="Blog">
            SELECT * FROM BLOG
            <where>
              <if test="state != null">
                   state = #{state}
              </if>
              <if test="title != null">
                  AND title like #{title}
              </if>
              <if test="author != null and author.name != null">
                  AND author_name like #{author.name}
              </if>
            </where>
          </select>
          ```

2. where 标签

   - *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。（子句末尾的 “AND” 或 “OR” 不能自动去除，可以使用trim 标签自定义 where 标签的功能）

3. trim 标签

   - 属性 prefix、suffix ：在内容前面或后面添加指定内容

   - 属性 prefixOverrides、suffixOverrides：在内容前面或后面移除指定内容

     ```xml
     <trim prefix="WHERE" prefixOverrides="AND |OR ">
       ...
     </trim>
     ```

4. choose、when、otherwise 标签

   - 类似于 switch case 语句

     ```xml
     <select id="findActiveBlogLike"
          resultType="Blog">
       SELECT * FROM BLOG WHERE state = ‘ACTIVE’
       <choose>
         <when test="title != null">
           AND title like #{title}
         </when>
         <when test="author != null and author.name != null">
           AND author_name like #{author.name}
         </when>
         <otherwise>
           AND featured = 1
         </otherwise>
       </choose>
     </select>
     ```

5. foreach 标签

   - 使用 foreach 标签可以实现一些批量操作
   - collection ：设置要循环的数组或集合
   - item ：表示数组中的某一条数据
   - separator ： 设置每次循环的数据之间的分隔符
   - open ： 循环的所有内容以什么符号开始
   - close ： 循环的所有内容以什么符号结束

   ```xml
   <select id="selectPostIn" resultType="domain.blog.Post">
     SELECT *
     FROM POST P
     WHERE ID in
     <foreach item="item" index="index" collection="list"
         open="(" separator="," close=")">
           #{item}
     </foreach>
   </select>
   ```

6. SQL 标签

   - 用于记录 SQL 片段

     ```xml
         <!-- sql片段 -->
         <sql id="if-title-author">
             <if test="title !=null">
                 title=#{title}
             </if>
             <if test="author != null">
                 and author=#{author}
             </if>
         </sql>
     
     ```

   - 使用 include 标签引入 SQL 片段

     ```xml
         <select id="queryBlogIf" parameterType="map" resultType="blog">
             select * from mybatisdb.blog
             <where>
                 /*包含sql片段*/
                 <include refid="if-title-author"></include>
             </where>
     
         </select>
     ```

## 7、MyBatis 缓存

1. 一级缓存

   - mybatis 的一级缓存是默认开启的

   - 一级缓存是 SqlSession 级别的 ，通过同一个 SqlSession 查询数据数据会被缓存，下次再次通过该SqlSession 查询相同的数据则直接从缓存中获取

   - 一级缓存失效情况

     1. 不同的 SqlSession  对应不同的一级缓存

     2. 同一个 SqlSession 但是查询条件不同

     3. 同一个 SqlSession 两次查询期间执行了任何一次增删改操作（增删改会改变数据库的数据）

     4. 同一个 SqlSession 两次查询期间手动清空缓存

        ```Java
                SqlSession sqlSession = MybatisUtils.getSqlSession();
                sqlSession.clearCache();//手动清空缓存
        ```

        

2. 二级缓存

   - 二级缓存是 SqlSessionFactory 级别的 ，通过同一个 SqlSessionFactory 创建的SqlSession 查询数据数据会被缓存，下次再次通过该SqlSession 查询相同的数据则直接从缓存中获取

   - 开启二级缓存

     ````xml
     # 只需要在 SQL 映射文件中添加 cache 标签
     <cache/>
     ````

     - 全局配置属性：```cacheEnabled```
       - 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存

   - 二级缓存是事务性的，所以二级缓存必须在 SqlSession 关闭或提交后才会生效

   - 二级缓存失效

     - 两次查询期间执行了任何一次增删改操作

   - 二级缓存配置

     ```xml
     <cache
       eviction="FIFO"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
     ```

     - eviction：缓存清除策略
       1. `LRU` – 最近最少使用：移除最长时间不被使用的对象。
       2. `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
       3. `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
       4. `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。
     - flushInterval：缓存刷新间隔，毫秒数，默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新
     - size：引用数目，默认值是 1024
     - readOnly：
       - true：只读，只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改。这就提供了可观的性能提升。
       - false：可读写，而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

3. mybatis 的缓存查询顺序

   - 二级缓存 ----> 一级缓存 ----> 数据库
     1. 先查询二级缓存，因为可能会有其他程序查询的数据存在二级缓存中，可以直接使用
     2. 二级缓存未命中，再查询一级缓存
     3. 一级缓存未命中，则查询数据库
     4. SqlSession 关闭或提交后，一级缓存中的数据会写入二级缓存

4. 使用自定义缓存或第三方缓存方案

   ```xml
   <cache type="com.domain.something.MyCustomCache"/>
   ```

   - 可以通过实现自定义的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为
   - 第三方缓存方案：ehcache ....

## 8、Mybatis 的逆向工程

- 根据数据库表，反向生成以下资源
  1. Java实体类
  2. mapper接口
  3. mapper映射文件
- 参考：[058-MyBatis逆向工程之清晰简洁版_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ya411S7aT?p=58&spm_id_from=pageDriver)

## 9、分页插件

- 参考：[060-分页功能分析_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ya411S7aT?p=60&spm_id_from=pageDriver)

- 逻辑：

  - pageSize：每页显示条数

  - pageNum：当前页的页码

  - index：当前页的起始索引，从 0 开始

    - index = (pageNum - 1) * pageSize

  - count：总记录数

  - totalPage：总页数

    ````java
    totalPage = count / pageSize;
    if( count % pageSize !=0 ){
        totalPage  +=1;
    }
    ````

- 分页插件：PageHelper