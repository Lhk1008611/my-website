---
sidebar_position: 2
---

# Spring MVC

## 1. 什么是 MVC

- MVC 是一种软件架构思想，将软件按照模型、视图、控制器来划分
- M：model，模型层，指工程中的 Java bean，作用是处理数据
  - Java Bean 分两类
    1. 实体类 Bean：专门存储业务数据的
    2. 业务处理 Bean：指 service 或 Dao 对象，专门用于处理业务逻辑和数据访问
- V：view，视图层，指工程中的 html 或 jsp 等页面
  - 作用：与用户进行交互，展示数据
- C：controller，控制层，指工程中的 servlet
  - 作用：接收请求和响应浏览器



## 2. 什么是 Spring MVC

- Spring MVC 是 Spring 为**表述层**开发提供的一整套完备的解决方案
- **三层式架构**：表述层（表示层）、业务逻辑层、数据访问层
  
  - 表述层：前台页面 + 后台 servlet
- 特点：
  - 无缝对接 IOC 容器
  - 基于原生 servlet 实现了强大的前端控制器 DispatcherServlet，对请求和响应进行统一处理
  - 全方位覆盖表述层各细分领域，提供全面解决方案
  - 提升开发效率
  - 内部组件化程度高，配置相应组件即可使用对应的功能
  - 性能卓著，适合大型互联网项目

- 依赖

  ```xml
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.0.5</version>
      </dependency>
  ```

## 3. 配置前端控制器

- 在 web.xml 文件中配置

  ```xml
      <!--配置SpringMVC的前端控制器-->
      <servlet>
          <servlet-name>DispatcherServlet</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:spring-mvc.xml</param-value>
          </init-param>
          <!--服务器启动时就加载Servlet对象-->
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>DispatcherServlet</servlet-name>
          <!--任何请求都会经过该Servlet,不会匹配 *.jsp 请求-->
          <url-pattern>/</url-pattern>
      </servlet-mapping>
  ```

  - `url-pattern` 中配置 / 和 /* 的区别

    - /：匹配浏览器向服务器发送的所有请求（不包括 .jsp 的请求）
    - /*：匹配浏览器向服务器发送的所有请求（包括 .jsp 的请求）
    - 由于.jsp 的请求是需要 Tomcat 中配置的 JspServlet 去处理的，DispatcherServlet 是无法对 jsp 请求进行处理，所以在配置 DispatcherServlet 时使用 / 即可

  - `init-param`：配置加载配置文件的路径

    ```xml
    <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:spring-mvc.xml</param-value>
            </init-param>
    ```

  - `load-on-startup`：设置 DispatcherServlet 的初始化时机

    - 默认初始化是在请求第一次访问 DispatcherServlet  时初始化，会造成第一次访问的请求花费的时间比较长
    - 设置为 1 表示将 DispatcherServlet  初始化时机提前到服务器启动时

## 4. 配置视图解析器

- 在 spring-mvc 配置文件中进行配置

  ```xml
          <!--配置视图解析器内部资源-->
          <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                  <!-- /jsp/xxx.jsp  -->
                  <property name="prefix" value="/jsp/"></property>
                  <property name="suffix" value=".jsp"></property>
          </bean>
  ```

## 5. `@RequestMapping` 注解

1. `@RequestMapping` 的作用：

   - 将请求和处理请求的控制器关联，建立映射关系

2. `@RequestMapping`注解可以标识的位置

   1. 类上：设置映射请求的请求路径的初始信息

      ```java
      @Controller
      @RequestMapping("/user")
      public class UserController {}
      ```

   2. 方法上：设置映射请求的请求路径的具体信息

      ```java
          //通过return进行转发和重定向
          @RequestMapping(value="/quick",method = RequestMethod.GET,params = {"username"})
          public String save(){
              System.out.println("Controller save running....");
              return "success";
          }
      ```

3. `@RequestMapping` 的 value 属性

   - 设置多个 value 可以映射多个请求路径

     ```java
     @RequestMapping({"/test1","/test2"})
     ```

4. `@RequestMapping` 的 method 属性

   - method  为数组类型，用于匹配请求的请求方式（GET、POST......）

     ```java
     @RequestMapping(value = "/test1",method = {RequestMethod.GET,RequestMethod.POST})
     ```

   - 请求方式不匹配时会报 405 

5. `@RequestMapping` 的一些派生注解

   - 在`@RequestMapping` 的基础上结合请求方式而成的注解
   - `@GetMapping` `@PutMapping` `@PostMapping` `@DeleteMapping` 

6. `@RequestMapping` 的 params 属性

   - 功能：浏览器发送的请求的请求参数必须满足 params 属性的设置（所有设置都需要满足才能请求成功）

     ```java
         @RequestMapping(
                 value = "/test1",
                 method = {RequestMethod.GET,RequestMethod.POST},
                 params = {"username","!user","age=21","sex!=1"}
         )
     ```

   - params 不匹配报 400

7. `@RequestMapping` 的 headers 属性

   - 匹配请求头信息

   - 用法与 params 一致
   - headers  不匹配报 404，即找不到资源（请求头不匹配）

8. `@RequestMapping`注解可以使用 ant 风格的路径

   - 在注解的 value 属性中设置路径时可以往路径中加入一些特殊字符

     1. ?：表示任意单个字符（不包括?）

        ```java
            @RequestMapping(
                    value = "/a?b/test1",
                    method = {RequestMethod.GET,RequestMethod.POST},
                    params = {"username","!user","age=21","sex!=1"}
            )
        ```

        

     2. *：表示任意个数的任意字符（不包括?和/）

        ```java
            @RequestMapping(
                    value = "/a*b/test1",
                    method = {RequestMethod.GET,RequestMethod.POST},
                    params = {"username","!user","age=21","sex!=1"}
            )
        ```

        

     3. **：表示任意层数的任意目录

        ```java
            @RequestMapping(
                    value = "/**/test1",
                    method = {RequestMethod.GET,RequestMethod.POST},
                    params = {"username","!user","age=21","sex!=1"}
            )
        ```

9. spring MVC 路径中的占位符

   - `@PathVariable` 注解进行占位符的匹配获取工作

   - ```java
         /**
          * 获得Restful风格的参数
          * Restful风格：http://localhost:8080/Spring_mvc/user/quick16/lhk
          * 在业务方法中使用@PathVariable注解进行占位符的匹配获取工作
          * @param username
          * @throws IOException
          */
         @RequestMapping(value="/quick16/{username}")
         @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
         public void save16(@PathVariable(value = "username") String username) throws IOException {
             System.out.println(username);
         }
     ```

10. spring MVC 获取请求参数

    1. 通过 servlet API 获取请求参数

       - 在控制器方法中设置 `HttpServlet`相关的形参，即可使用 request、response 和 session 相关对象

         ```java
             /**
              * SpringMVC中获取Servlet的相关API
              * @param request
              * @param response
              * @param session
              * @throws IOException
              */
             @RequestMapping(value="quick18")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save18(HttpServletRequest request, HttpServletResponse response, HttpSession session) throws IOException {
                 System.out.println(request);
                 System.out.println(response);
                 System.out.println(session);
             }
         ```

    2. 通过控制器的形参获取请求参数

       - Controller 中的业务方法参数名称与请求参数的名称一致，参数值会自动映射匹配

         ```java
             /**
              * 获得基本类型参数
              * @param username
              * @param age
              * @throws IOException
              */
             //Controller中的业务方法参数名称与请求参数的name一致，参数值会自动映射匹配
             //http://localhost:8080/Spring_mvc/user/quick10?username=lhk&age=21
             @RequestMapping(value="/quick10")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save10(String username,int age) throws IOException {
                 System.out.println(username);
                 System.out.println(age);
             }
         ```

       - 当请求的参数名称与 Controller 的业务方法参数名称不一致时，可以使用`@RequestParam`注解将请求参数和方法中的形参显示的进行绑定

         - value 设置请求参数的名称
         - required = true 表示必须传一个请求参数，否则报 400 的错误
         - defaultValue 设置形参的默认值

         ```java
             /**
              * 当请求的参数名称与Controller的业务方法参数名称不一致时，
              * 就需要使用@RequestParam注解显示的绑定
              * @param username
              * @throws IOException
              */
             @RequestMapping(value="/quick15")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save15(@RequestParam(value = "name" , required = false,defaultValue = "lhk") String username) throws IOException {
                 System.out.println(username);
             }
         ```

       - `@RequestHeader`：将请求头信息和控制器方法的形参进行绑定

         ```java
             /**
              * 获取请求头的信息
              * @param user_agent
              * @throws IOException
              */
             @RequestMapping(value="quick19")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save19(@RequestHeader(value = "User-Agent") String user_agent) throws IOException {
                 System.out.println(user_agent);
             }
         ```

       - `@CookieValue`：将 cookie 数据和控制器方法的形参进行绑定

         ```java
             /**
              * 通过注解@CookieValue获取cookie的值
              * @param jsessionid
              * @throws IOException
              */
             @RequestMapping(value="quick20")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save20(@CookieValue(value = "JSESSIONID") String jsessionid) throws IOException {
                 System.out.println(jsessionid);
             }
         ```

       - 通过 pojo 获取请求参数

         - 控制器方法的实体类类型形参的属性名和请求参数名保持一致，即可将请求参数映射到实体类类型的形参中

         ```java
             /**
              * 获得pojo类型参数
              * @param user
              * @throws IOException
              */
             //Controller中的业务方法中的pojo参数属性名称与请求参数的name一致，参数值会自动映射匹配
             //http://localhost:8080/Spring_mvc/user/quick11?name=lhk&age=21
             @RequestMapping(value="/quick11")
             @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
             public void save11(User user) throws IOException {
                 System.out.println(user);
             }
         ```

    3. 解决获取请求参数乱码问题

       - 在设置编码执行之前，不能获取任何的请求参数，如果先获取了请求参数，再设置编码将不会生效

       - spring MVC 处理编码的过滤器一定要设置在其他过滤器之前，否则无效

       - 在 web.xml 文件中配置全局过滤器

         ```xml
             <!--配置全局过滤的filter 解决post请求乱码问题-->
             <filter>
                 <filter-name>CharacterEncodingFilter</filter-name>
                 <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
                 <!-- 设置请求的编码 -->
                 <init-param>
                     <param-name>encoding</param-name>
                     <param-value>utf-8</param-value>
                 </init-param>
                 <!-- 设置响应的编码 -->
                 <init-param>
                     <param-name>forceEncoding</param-name>
                     <param-value>true</param-value>
                 </init-param>
             </filter>
             <filter-mapping>
                 <filter-name>CharacterEncodingFilter</filter-name>
                 <url-pattern>/*</url-pattern>
             </filter-mapping>
         ```

## 6. 往域对象中共享数据

1. 使用 servletAPI 往 request 域对象中共享数据

   ```java
       @RequestMapping(value="/quick5")
       public String save5(HttpServletRequest request){
           request.setAttribute("username","大时代");
           return "success";
       }
   ```

2. 使用 modelAndView 往 request 域对象中共享数据

   - model：向请求域中共享数据

   - view：设置逻辑视图

     ```java
         //方法中传入SpringMVC提供的ModelAndView对象参数，进行转发和重定向
         @RequestMapping(value="/quick3")
         public ModelAndView save3(ModelAndView modelAndView){
             //设置模型数据
             modelAndView.addObject("username","LHK");
             //设置视图名称(ViewName)
             modelAndView.setViewName("success");
             return modelAndView;
         }
     
     
         //通过返回ModelAndView对象进行转发和重定向
         @RequestMapping(value="/quick2")
         public ModelAndView save2(){
             ModelAndView modelAndView = new ModelAndView();
             //设置模型数据
             modelAndView.addObject("username","TheMetents");
             //设置视图名称(ViewName)
             modelAndView.setViewName("success");
             return modelAndView;
         }
     ```

   - 使用 model 往 request 域中共享数据

     ```java
         @RequestMapping(value="/quick4")
         public String save4(Model model){
             model.addAttribute("username","lhk1008611");
             return "success";
         }
     ```

   - 使用 ModelMap 往 request 域中共享数据

     ```java
         // modelMap 的使用
         @RequestMapping(value="/modelMap")
         public String testModelMap(ModelMap modelMap){
             modelMap.addAttribute("username","lhk1008611");
             return "success";
         } 
     ```

   - 使用 map 往 request 域中共享数据

     ```java
         // map 的使用
         @RequestMapping(value="/map")
         public String testModelMap(Map<String,Object> map){
             map.put("username","lhk1008611");
             return "success";
         }
     ```

   - model 、ModelMap 、map 的关系

     - 底层都是都是通过 `BindingAwareModelMap` 创建，本质上都是 `BindingAwareModelMap`  类型

       - ```java
         public class BindingAwareModelMap extends ExtendedModelMap {}
         public class ExtendedModelMap extends ModelMap implements Model {}
         public class ModelMap extends LinkedHashMap<String, Object> {}
         ```

3. 往 session 与中共享数据

   - 使用 servletAPI 往 session 域中共享数据

     ```java
         //往session域中共享数据
         @RequestMapping("/testSession")
         public String testSession(HttpSession session){
             session.setAttribute("key","value");
             return null;
         }
     ```

4. 往 application 域中共享数据

   ```java
       //往应用域中共享数据
       @RequestMapping("/testApplication")
       public String testApplication(HttpSession session){
           ServletContext application = session.getServletContext();
           application.setAttribute("key","value");
           return null;
       }
   ```


## 7. springMVC 的视图

- springMVC 中的视图是 View 接口
  - 视图的作用是渲染数据，将 model 的数据展示给用户
  - springMVC 中默认的转发视图（internalResourceView）和重定向视图（redirectView）
    - 当工程中引入 jstl 的依赖，转发视图会自动转换为 jstlView
    - 若在springMVC的配置文件中配置了 Thymeleaf 的视图解析器，则此视图解析后的视图为 ThymeleafView

- ThymeleafView
  - 配置 Thymeleaf 的视图解析器

    ```xml
    <!-- 配置Thymeleaf视图解析器 -->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
        
                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>
        
                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
    ```

  - 配置了 ThymeleafViewResolver 后

    - 在 controller 中直接 `return "视图名称"` 会解析成 ThymeleafView 进行视图渲染
    - 在 controller 中直接 `return "forward:/xxx/xxx"`  会解析成 internalResourceView 实现路由转发
    - 在 controller 中直接 `return "redirect:/xxx/xxx"`  会解析成 redirectView实现重定向
      - 转发和重定向的区别：
        - 转发直接跳转到页面不会改变页面 url
        - 重定向直接跳转到页面会将页面 url 变为跳转后页面的 url

- 视图控制器

  - 配置视图控制器

    ```xml
    <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
    ```

  - 开启注解驱动

    - 若不开启 mvc 的注解驱动，则只会处理视图控制器所对应的请求,其他请求 404；

    - 开启 mvc 的注解驱动后可以处理视图控制器和 controller 中的所有请求

      ```xml
      <mvc:annotation-driven /">
      ```

## 8. RESTFul 风格

- REST : Representational State Transfer,表现层资源状态转移

- | HTTP method | restful | 动作 | 获取url中的参数的注解 | 占位符url |
  | :---------: | :-----: | :--: | :-------------------: | :-------: |
  |     get     | /user/1 | 查询 |    `@PathVariable`    | /url/{id} |
  |    post     |  /user  | 新增 |                       |           |
  |     put     | /user/1 | 修改 |    `@PathVariable`    | /url/{id} |
  |   delete    | /user/1 | 删除 |    `@PathVariable`    | /url/{id} |

  - 通过超链接发送的是 get 请求

  - 浏览器只能发送 get 或 post 请求

  - 默认情况下 form 表单只能发送 get 和 post 请求

    - 后端通过配置处理 http method 的过滤器可以实现发送 put、delete 请求的效果

      ```xml
          <filter>
              <filter-name >HiddenHttpMethodFilter</filter-name>
              <filter-class >org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
          </filter>
          <filter-mapping>
              <filter-name>HiddenHttpMethodFilter</filter-name>
              <url-pattern>/*</url-pattern>
          </filter-mapping>
      ```

    - 前端 form 表单需要将 method 属性设置为 post，并在表单传送一个 _method 参数到 controller 方法

      ```html
      <form action="/Spring_mvc/user/quick13" method="post">
          <input type="hidden" name="_method" value="put">
          .......
          <input type="submit" value="提交">
      </form>
      ```

  - 通过查看 HiddenHttpMethodFilter 这个过滤器源码中的 doFilterInternal 方法可以了解底层是如何对 http method 进行处理

## 9. springMVC 处理 ajax 请求

```java
    /**
     * 使用ajax提交时，可以指定contentType为json形式
     * 那么在 controller 方法参数位置使用 @RequestBody 可以直接接收集合数据而无需对集合进行pojo包装
     * @param userlist
     * @throws IOException
     */
    @RequestMapping(value="/quick14")
    @ResponseBody  //该注解告知SpringMVC框架，不进行视图跳转，直接进行数据响应
    public void save14(@RequestBody List<User> userlist) throws IOException {
        System.out.println(userlist);
    }
```

1. `@RequestBody`注解：获取请求体信息

   - 导入 jackson 包配合 `@RequestBody` 注解可将请求体数据解析为 Java 类型数据

2. `@ResponseBody`注解：将 controller 方法返回数据以响应报文的响应体形式返回给客户端

   - 导入 jackson 包配合 `@ResponseBody`注解可将 controller 方法返回的 Java 类型数据解析为 json 格式数据以响应报文的响应体形式返回给客户端

3. 需要开启 springmvc 的注解驱动

   ```xml
   <mvc:annotation-driven /">
   ```

4. `@RestController`  = `@Controller` +`@ResponseBody`

   - 该注解一般标识在 controller 类的前面

## 10. 文件上传和下载

1. 文件上传：将文件从浏览器复制到服务器

2. 文件下载：将文件从服务器复制到浏览器

3. Java 原生文件上传下载实现：通过 IO 流实现文件上传下载，输入流读取文件，输出流见文件输出到指定位置

4. spring MVC 中的文件上传下载

   1. 文件下载

      1. 使用`ResponseEntity`**响应报文对象**作为 controller 方法的返回值类型，可以实现文件下载功能

         ```java
             /**
              * 文件下载
              * @param httpSession
              * @return ResponseEntity
              * @throws IOException
              */
             @RequestMapping("/download")
             public ResponseEntity<byte[]> download(
                     HttpSession httpSession
             ) throws IOException {
                 //1. 获取需要下载的文件路径
                 ServletContext servletContext = httpSession.getServletContext();
                 String imagePath = servletContext.getRealPath("image");
                 String filePath = imagePath + File.separator + "appreciate_afdian.png";
                 //2. 创建输入流读取文件字节
                 FileInputStream fileInputStream = new FileInputStream(filePath);
                 byte[] bytes = new byte[fileInputStream.available()];
                 //将文件流读入字节数组
                 fileInputStream.read(bytes);
                 //3. 创建 HttpHeaders 对象，设置头信息
                 MultiValueMap<String,String> httpHeaders = new HttpHeaders();
                 httpHeaders.add("Content-Disposition","attachment;filename=appreciate_afdian.png");
                 //4. 返回 ResponseEntity 对象，作为响应报文响应到浏览器
                 ResponseEntity<byte[]> responseEntity = new ResponseEntity<byte[]>(bytes,httpHeaders,HttpStatus.OK);
                 fileInputStream.close();
                 return responseEntity;
             }
         ```

         

   2. 文件上传

      1. 使用`MultipartFile`对象接收文件上传参数，调用`transferTo`方法实现文件上传

         ```html
         <h2>
             <span>文件上传</span>
         </h2>
         <form action="${pageContext.request.contextPath}/user/upload" method="post" enctype="multipart/form-data">
             文件1：<input type="file" name="uploadFile"><br>
             文件2：<input type="file" name="uploadFile"><br>
             <input type="submit" value="上传"><br>
         </form>
         ```

         ```java
             /**
              * 多文件上传,表单中文件名不同
              * 单文件上传，传入一个MultipartFile参数即可
              * @param uploadFile
              * @throws IOException
              */
             @RequestMapping(value="upload")
             public String uploadFile(
                     MultipartFile[] uploadFile,
                     HttpSession httpSession
             ) throws IOException {
                 ServletContext servletContext = httpSession.getServletContext();
                 String imagePath = servletContext.getRealPath("image");
                 File file = new File(imagePath);
                 System.out.println(uploadFile.length);
                 if(!file.exists()){
                     file.mkdir();
                 }
                 //将文件保存到磁盘
                 for (MultipartFile multipartFile : uploadFile) {
                     if (!multipartFile.isEmpty()){
                         multipartFile.transferTo(new File(imagePath+File.separator+multipartFile.getOriginalFilename()));
                     }
                 }
                 return "success";
             }
         ```

      2. 需要配置文件上传解析器

         ```xml
                 <!--配置文件上传解析器-->
                 <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
                         <property name="defaultEncoding" value="UTF-8"></property>
                         <property name="maxUploadSize" value="500000"></property>
         
                 </bean>
         ```

      3. 需要导入文件上传包

         ```xml
             <!--文件上传包-->
             <dependency>
               <groupId>commons-fileupload</groupId>
               <artifactId>commons-fileupload</artifactId>
               <version>1.3.1</version>
             </dependency>
         ```

      4. 问题： 上传同名文件会将文件的原内容覆盖

         - 解决：将文件名设置为不重复的文件名即可
           1. 文件名添加 uuid 或使用 uuid 作为文件名
           2. 文件名添加`时间戳`或使用`时间戳`作为文件名

## 11. 拦截器

1. filter 与 拦截器的区别

   1. filter：会在请求达到 dispatchSeverlet 之前执行，先对请求进行过滤

   2. 拦截器：是在 dispatchSeverlet 匹配到对应的处理器（controller 或 handler）后，在该处理器前后对请求进行拦截

      - 三个方法

        ```java
            @Override
            //preHandle()方法是 在处理器方法执行前 执行
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        		return false;
            }
        
            @Override
            //postHandle()方法是 在处理器方法执行后 视图返回之前 执行
            public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        
            }
        
            //afterCompletion()方法是在 整个访问资源流程执行完毕后执行
            @Override
            public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        
            }
        ```

2. 拦截器的使用

   1. 创建拦截器类，实现接口 `HandlerInterceptor`

      ```java
      public class MyInterceptor1 implements HandlerInterceptor {
      
          @Override
          //preHandle()方法是 在目标方法执行前 执行
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
              System.out.println("preHandle().......");
              String param = request.getParameter("param");
              if ("yes".equals(param)){
                  return true;//返回true代表放行，返回false则不会执行目标方法
              }else {
                  request.getRequestDispatcher("/error.jsp").forward(request,response);
                  return false;
              }
          }
      
          @Override
          //postHandle()方法是 在目标方法执行后 视图返回之前 执行
          public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
              System.out.println("postHandle()............");
          }
      
          //afterCompletion()方法是在 整个访问资源流程执行完毕后执行
          @Override
          public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
              System.out.println("afterCompletion().....");
          }
      }
      ```

   2. 配置拦截器，使拦截器生效

      - `/**` 表示对所有请求都使用该拦截器进行拦截

        1. 配置方式1

           ```xml
               <!--配置拦截器-->
               <mvc:interceptors>
                   <mvc:interceptor>
                       <!--配置需要进行拦截的资源-->
                       <mvc:mapping path="/**"/>
                       <bean class="com.lhk.interceptor.MyInterceptor1"></bean>
                   </mvc:interceptor>
                   <mvc:interceptor>
                       <!--配置需要进行拦截的资源-->
                       <mvc:mapping path="/**"/>
                       <bean class="com.lhk.interceptor.MyInterceptor2"></bean>
                   </mvc:interceptor>
               </mvc:interceptors>
           ```

        2. 使用 `@Component` 注解和注解扫描配置拦截器

           ```xml
               <!--组件扫描-->
               <context:component-scan base-package="com.lhk.interceptor"></context:component-scan>
           
               <!--配置拦截器-->
               <mvc:interceptors>
                   <mvc:interceptor>
                       <!--配置需要进行拦截的请求-->
                       <mvc:mapping path="/**"/>
                       <bean class="com.lhk.interceptor.MyInterceptor1"></bean>
                   </mvc:interceptor>
                   <mvc:interceptor>
                       <!--配置需要进行拦截的请求-->
                       <mvc:mapping path="/**"/>
                       <ref bean="myInterceptor2"></ref>
                   </mvc:interceptor>
               </mvc:interceptors>
           ```

           ```java
           @Component
           public class MyInterceptor2 implements HandlerInterceptor {
           	//....
           }
           ```

3. 多个拦截器的执行顺序

   - ```text
     preHandle().......111111
     preHandle().......222222
     controller 执行....
     postHandle()........222222
     postHandle().......111111
     afterCompletion().....222222
     afterCompletion().....111111
     ```

   - `preHandle`方法按拦截器配置的顺序依次执行，`postHandle`和`afterCompletion`按拦截器配置的逆序依次执行

## 12.异常解析器

1. `HandlerExceptionResolver`接口

2. `DefaultHandlerExceptionResolver`：默认的异常解析器，内部实现了对异常的默认处理方式

3. `SimpleMappingExceptionResolver`：可以配置异常的处理方式，可以配置异常出现时跳转至哪个页面

   ```xml
       <!--配置SpringMVC的异常处理器-->
      <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
           <property name="defaultErrorView" value="error"></property>
   
           <property name="exceptionMappings">
               <map>
                   <entry key="java.lang.ClassCastException" value="error1"></entry>
                   <entry key="com.lhk.exception.MyException" value="error2"></entry>
               </map>
           </property>
       </bean>
   ```

4. 自定义异常解析器，可实现异常处理的定制化

   - 创建异常解析类实现`HandlerExceptionResolver`接口，即可

     ```java
     public class MyExceptionResolver implements HandlerExceptionResolver {
         /**
          *
          * @param httpServletRequest
          * @param httpServletResponse
          * @param e 异常对象
          * @return ModelAndView 跳转到错误视图信息页面
          */
         @Override
         public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
             ModelAndView modelAndView = new ModelAndView();
             if (e instanceof ClassCastException){
                 modelAndView.addObject("info","类转换异常");
             }else if (e instanceof MyException){
                 modelAndView.addObject("info","自定义异常");
             }
             modelAndView.setViewName("error");
             return modelAndView;
         }
     }
     ```

     ```xml
          <!--配置自定义的异常处理器-->
         <bean class="com.lhk.resolver.MyExceptionResolver"></bean>
     ```

5. 自定义异常

   - 创建异常类继承`Exception`，即可得到自定义异常

     ```java
     public class MyException extends Exception{
     
     }
     ```

6. 注解实现自定义异常解析器

   1. `@ControllerAdvice`注解可将类标识为异常解析器组件

   2.  `@ExceptionHandler`注解表示遇到哪些异常时需要进行异常解析

      ```java
      @ControllerAdvice
      public class DemoController_1 {
      
          @Autowired
          private DemoService demoService;
      
          @ExceptionHandler(ClassCastException.class)
          public String Show(Throwable ex, Model model) {
              System.out.println("遇到异常时进行一些处理.....");
              demoService.show1();
              model.addAttribute("ex", ex);//将异常信息存入请求域
              return "error";//出现 ClassCastException 异常时跳转到 error 页面
          }
      }
      ```

## 13、注解配置 Spring MVC

1. 使用配置类和注解来替代 web.xml 和 SpringMVC 的配置文件

2. 配置类替代 web.xml

   ```java
   /**
    * 代替 web.xml 文件的配置类
    */
   public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
       @Override
       //设置配置类代替 Spring 的配置文件
       protected Class<?>[] getRootConfigClasses() {
           return new Class[]{SpringConfig.class};
       }
   
       @Override
       //设置配置类代替 SpringMVC 的配置文件
       protected Class<?>[] getServletConfigClasses() {
           return new Class[]{WebConfig.class};
       }
   
       @Override
       //设置 SpringMVC 的前端控制器 DispatcherServlet 的url-pattern
       protected String[] getServletMappings() {
           return new String[]{"/"};
       }
   
       @Override
       //设置过滤器
       protected Filter[] getServletFilters() {
           //编码过滤器
           CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
           characterEncodingFilter.setEncoding("utf-8");
           characterEncodingFilter.setForceEncoding(true);
           //处理请求方式过滤器
           HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
           return new Filter[]{characterEncodingFilter,hiddenHttpMethodFilter};
       }
   }
   
   ```

3. 配置类代替 Spring 配置文件

   - 使用 `@Configuration` 注解将类标识为配置类

4. 配置类代替 Spring 配置文件

   - 使用 `@Configuration` 注解将类标识为配置类

   - 实现 `WebMvcConfigurer`类

   - 配置 **组件扫描**、**视图解析器**、**默认的 Servlet 处理静态资源**   、**mvc 注解驱动**、**视图控制器**、**文件上传解析器**、**拦截器**、**异常解析器**

     ```java
     /**
      * 配置类代替 springMVC 配置文件
      */
     @Configuration
     @ComponentScan("com.lhk.controller") //组件扫描
     @EnableWebMvc //开启 mvc 的注解驱动
     public class WebConfig implements WebMvcConfigurer {
         @Override
         //配置默认 Servlet 处理静态资源
         public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
             configurer.enable();
         }
     
         @Override
         //配置视图控制器
         public void addViewControllers(ViewControllerRegistry registry) {
             registry.addViewController("/").setViewName("index");
         }
     
         @Bean //可以将方法的返回值作为 bean 交给容器进行管理，bean 的 id 为注解标识的方法名
         //配置文件上传解析器
         public CommonsMultipartResolver multipartResolver() {
             return new CommonsMultipartResolver();
         }
     
         @Override
         //配置拦截器
         public void addInterceptors(InterceptorRegistry registry) {
             registry.addInterceptor(new MyInterceptor1()).addPathPatterns("/**").excludePathPatterns("/");
         }
     
         @Override
         //配置异常解析器
         public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
             SimpleMappingExceptionResolver simpleMappingExceptionResolver = new SimpleMappingExceptionResolver();
             Properties properties = new Properties();
             properties.setProperty("java.lang.ClassCastException", "error");
             simpleMappingExceptionResolver.setExceptionMappings(properties);
             simpleMappingExceptionResolver.setExceptionAttribute("ex");
         }
     
         //以下为配置视图解析器
         //1.配置生成模板解析器
         @Bean
         public ITemplateResolver templateResolver() {
             WebApplicationContext currentWebApplicationContext = ContextLoader.getCurrentWebApplicationContext();
             ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(currentWebApplicationContext.getServletContext());
             templateResolver.setPrefix("/templates/");
             templateResolver.setSuffix(".html");
             templateResolver.setCharacterEncoding("utf-8");
             templateResolver.setTemplateMode(TemplateMode.HTML);
             return templateResolver;
         }
     
         //2.配置生成模板引擎，并注入模板解析器
         @Bean
         public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
             SpringTemplateEngine springTemplateEngine = new SpringTemplateEngine();
             springTemplateEngine.setTemplateResolver(templateResolver);
             return springTemplateEngine;
         }
     
         //3.配置生成视图解析器，并注入模板引擎
         @Bean
         public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
             ThymeleafViewResolver thymeleafViewResolver = new ThymeleafViewResolver();
             thymeleafViewResolver.setCharacterEncoding("UTF-8");
             thymeleafViewResolver.setTemplateEngine(templateEngine);
             return thymeleafViewResolver;
         }
     }
     
     ```

     

##  14、spring MVC 的整个流程

 ### 1. 常用组件

- `DispatcherServlet `：前端控制器，由框架提供
  - 统一处理请求和响应
- `HandlerMapping`：处理器映射器，由框架提供
  - 根据请求的 url、method 等信息映射到对应的处理器（controller 方法）
- ``Handler`：处理器（Controller），需要工程师开发
  - 在 `DispatcherServlet`的控制下，Handler 对请求进行具体的处理
- `HandlerAdapter`：处理器适配器，由框架提供
  - 通过`HandlerAdapter`执行处理器（controller 方法）
- `ViewResolver`：视图解析器，由框架提供
  - 进行视图解析，得到相应的视图，如：`ThymeleafView `,`internalResourceView`
- `View`：视图
  - 将模型数据通过页面展示给用户

### 2. 具体执行流程

1. 浏览器发送请求	
2. 检查请求地址是否与 web.xml 文件中设置的 `url-pattern` 相匹配，匹配成功则将该请求交由 DispatcherServlet  进行处理
3. 前端控制器 DispatcherServlet  读取 SpringMvc 的核心配置文件，通过组件扫描找到 controller (控制器)，检查请求地址是否与控制器中的 `@RequestMapping` 注解中的 value 值相匹配，匹配成功则该注解所标识的 controller (控制器)方法会对该请求进行处理
4. controller 方法需要返回一个字符串的类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，并对该视图对应的页面进行渲染，最终将请求转发到视图所对应的页面

## 15. 整合SSM

1. springMVC 的 IOC 容器是在初始化 `dispatcherServlet` 时进行的创建的，由于 controller 层会依赖 service 层的组件，故 spring 的 IOC 容器应该在 springMVC IOC 容器创建之前进行创建

   - 故：在整合 spring 和 springMVC 的时候，项目启动时应先读取 spring 的配置文件创建 spring 的 IOC 容器，再读取SpringMVC的配置文件，创建它的 IOC 容器

2. 在 Tomcat 中，listener、filter、servlet 的执行顺序是 `listener——>filter——>servlet`

   - 故：可以在监听器（listener）执行时加载 spring 配置文件，创建 spring 的 IOC 容器，即可实现 spring 与 springMVC 的整合

3. 监听器（`listener`）

   1. `ServletContextListener`：用于监听 Web 应用程序的启动和关闭事件，即 Servlet 上下文的创建和销毁事件
   2. `ServletRequestListener`：用于监听 HTTP 请求的开始和结束事件，即 Servlet 请求的创建和销毁事件
   3. `HttpSessionListener`：用于监听 HTTP 会话的创建和销毁事件，即会话的创建和销毁事件。
   4. `ServletContextAttributeListener`：用于监听 Servlet 上下文属性的添加、移除和替换事件。
   5. `ServletRequestAttributeListener`：用于监听 Servlet 请求属性的添加、移除和替换事件。
   6. `HttpSessionAttributeListener`：用于监听会话属性的添加、移除和替换事件。

   这些监听器可以通过在 Web 应用程序的配置文件（web.xml）中进行注册，或通过注解的方式进行配置

4. 在 spring中提供了 `ContextLoaderListener`实现了`ServletContextListener`，用于在 Web 应用程序启动时加载和初始化 Spring 应用程序上下文

   - 该类会在 Web 服务器启动时，读取 spring 的配置文件，创建 spring 的 IOC 容器

   - 在 web.xml 中配置

     ```xml
       <!--Spring的监听器 默认读取 WEB-INF/application.xml-->
       <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
       </listener>
       <!--全局的初始化参数,可自定义 spring 配置文件的位置和名称-->
       <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:applicationContext.xml</param-value>
       </context-param>
     ```

5. 在 pom.xml 文件中可以在 `properties`  标签中统一管理依赖版本号

   ```xml
     <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <maven.compiler.source>1.7</maven.compiler.source>
       <maven.compiler.target>1.7</maven.compiler.target>
       <spring.version>5.0.5.RELEASE</spring.version>
     </properties>
     
     <dependencies>
     	<dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-context</artifactId>
         <version>${spring.version}</version>
       </dependency>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-test</artifactId>
         <version>${spring.version}</version>
       </dependency>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-web</artifactId>
         <version>${spring.version}</version>
       </dependency>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>${spring.version}</version>
       </dependency>
     </dependencies>
   ```

6. 有 mybatis 为何还需导入 `spring-jdbc`

   - 因为 `spring-jdbc` 中有事务管理器，目的是为了导入事务管理器，更好的进行实物相关操作

7. 在 web.xml 中配置 **spring 的编码过滤器**、配置**处理请求方式的过滤器**、配置 **spring 的前端控制器 `dispatcherServlet`** 、配置 **spring 的监听器**用于加载 spring 的配置文件、配置 **spring 配置文件的自定义位置和名称**

   1. 在 springMVC.xml 中配置**组件扫描（controller）**、**视图解析器**、**默认 servlet 处理静态资源**、**mvc 注解驱动**、**视图控制器**、**文件上传解析器**，**拦截器**和**异常处理器**按需配置即可

8. 在 spring.xml 中配置 **组件扫描（除 controller）**，配置**数据源**

9. 整合 mybatis ，可以将一些卸载 mybatis.xml 文件中的配置写在 spring.xml 中

   - 需要配置`SqlSessionFactory`交给 IOC 容器管理
     1. 在 spring,xml 中配置 mybatis 的 `SqlSessionFactory`

        ```xml
            <!--
            配置mybatis的SqlSessionFactory
            其中SqlSessionFactoryBean是mybatis-spring包提供的SqlSessionFactory接口的一个实现
            -->
            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="dataSource"></property><!--加载数据源-->
                <property name="configLocation" value="classpath:mybatis-config-spring.xml"></property><!--加载mybatis核心配置文件-->
                <property name="mapperLocations" value="classpath:com.lhk.mapper/*.xml"></property><!--加载映射文件-->
            </bean>
        ```

     2. 在 spring,xml 中配置 mybatis 的`SqlSessionFactoryBean`，可以直接在 spring 的 IOC 容器中获取 `SqlSessionFactory`

        ```xml
            <bean class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="dataSource"></property><!--加载数据源-->
                <property name="configLocation" value="classpath:mybatis-config-spring.xml"></property><!--加载mybatis核心配置文件-->
                <property name="mapperLocations" value="classpath:com.lhk.mapper/*.xml"></property><!--加载映射文件,只有 mapper 接口的包和映射文件的包不一致时需要设置-->
            </bean>
        ```

   - 在 spring,xml 中配置 mapper 接口扫描，可以将指定包下所有的 mapper 接口通过 `SqlSession` 创建代理实现类对象，并交给 IOC 容器管理，这样可以直接在 service 层中注入 mapper 组件

     ```xml
         <!--扫描mapper相关接口所在的包 为mapper创建实现类-->
         <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
             <property name="basePackage" value="com.lhk.mapper"></property>
         </bean>
     ```

   - 配置**事务管理器**

     ```xml
         <!--声明式事式控制-->
         <!--平台事务管理器-->
         <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
             <property name="dataSource" ref="dataSource"></property><!--注入数据元-->
         </bean>
     
         <!-- 开启事务的注解驱动 -->
         <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
     ```

     

