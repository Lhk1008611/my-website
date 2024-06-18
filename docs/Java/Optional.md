---
sidebar_position: 23
---
# Optional 类

- 臭名昭著的空指针异常是导致 Java 应用程序失败的最常见原因，以前，为了解决空指针异常，Google 公司著名的 Guava 项目引入了 Optional 类，Guava 通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional 类已经成为 Java 8 类库的一部分。
- `Optional<T>` 类( `java.util.Optional`)是一个容器类，它可以保存类型 T 的值，代表这个值存在。或者仅仅保存 null，表示这个值不存在。
  - 原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。
- Optional 类的 Javadoc 描述如下:这是一个可以为 null 的容器对象。如果值存在则 `isPresent()` 方法会返回true，调用 `get()` 方法会返回该对象。
- Optional 提供很多有用的方法，这样我们就不用显式进行空值检测
  - 创建 Optional 类对象的方法:
    - `Optional.of(T t)`: 创建一个 Optional 实例，t必须非空;
    - `Optional.empty()`: 创建一个空的 Optional 实例 
    - `Optional.ofNullable(T t)`: t 可以为null
  - 判断 Optional 容器中是否包含对象:
    - `boolean isPresent()`: 判断是否包含对象
    - `void ifPresent(Consumer<?super T>consumer)`: 如果有值，就执行 Consumer 接口的实现代码，并且该值会作为参数传给它。
  - 获取Optional容器的对象:
    - `T get()`: 如果调用对象包含值，返回该值，否则抛异常
    - `T orElse(T other)`: 如果有值则将其返回，否则返回指定的 other 对象。
    - `T orElseGet(Supplier<?extends T> other)`: 如果有值则将其返回，否则返回由 Supplier 接口实现提供的对象 
    - `T orElseThrow(Supplier<?extends X>exceptionSupplier)`:如果有值则将其返回，否则抛出由 Supplier 接口实现提供的异常。

```java
/**
* Optional类：为了在程序中避免出现空指针异常而创建的
*
* 常用方法：
* ofNullable(T t)
* orElse(T other)
*
* @author Lhk
*/
public class OptionalTest {
    /*
    创建Optional类对象的方法：
    Optional.of(T t) : 创建一个 Optional 实例，t必须非空
    Optional.empty() : 创建一个空的 Optional 实例
    Optional.ofNullable(T t)：t可以为null
    */
    @Test
    public void test1(){
        Girl girl = new Girl();
        //of(T t):保证t非空
        Optional<Girl> optionalGirl = Optional.of(girl);
    }
    
    @Test
    public void test2(){
        Girl girl = new Girl();
        girl=null;
        //ofNullable(T t):t可以为null
        Optional<Girl> optionalGirl = Optional.ofNullable(girl);
        // orElse(T other) ：如果当前的Optional内封装有值则将其返回，如果为null返回指定的other对象。
        Girl girl1 = optionalGirl.orElse(new Girl("小爱"));
        System.out.println(girl1);
    }

    //该方法存在空指针异常
    public String getGirlName(Boy boy){
    	return boy.getGirl().getName();
    }

    @Test
    public void test3(){
        Boy boy = new Boy();
        getGirlName(boy);//java.lang.NullPointerException
    }

    //优化后的getGirlName
    public String getGirlName1(Boy boy){
        if (boy!=null){
        	Girl girl = boy.getGirl();
        	if (girl!=null){
        		return girl.getName();
        	}
        }
        return null;
    }

    @Test
    public void test4(){
        Boy boy = new Boy();
        boy=null;
        String girlName1 = getGirlName1(boy);
        System.out.println(girlName1);
    }

    //使用Optional类的getGirlName
    public String getGirlName2(Boy boy){
        Optional<Boy> boyOptional = Optional.ofNullable(boy);
        Boy boy1 = boyOptional.orElse(new Boy(new Girl("小花")));
        Girl girl = boy1.getGirl();
        Optional<Girl> optionalGirl = Optional.ofNullable(girl);
        Girl girl1 = optionalGirl.orElse(new Girl("小小"));
        return girl1.getName();
    }

    @Test
    public void test5(){
        Boy boy = new Boy();
        boy =null;
        boy=new Boy(new Girl("小霞"));
        String name = getGirlName2(boy);
        System.out.println(name);
    }
}
```

```java
/**
* 常用方法测试
* @author Lhk
*/
public class OptionalTest1 {

    @Test
    public void test1(){
        //实例化Optional对象
        Optional<Object> emptyop = Optional.empty();
        if (!emptyop.isPresent()){//判断Optional对象中是否包含数据,有数据为true
        	System.out.println("数据为空");
        }
        System.out.println(emptyop.isPresent());//false
        System.out.println(emptyop);
        //get():返回Optional封装的数据，必须保证Optional对象内有数据，否则报错
        // System.out.println(emptyop.get());
	}

    @Test
    public void test2(){
        String str="lhk";
        //of(T t):封装数据t实例化Optional对象，参数t必须非空，否则报错
        Optional<String> op = Optional.of(str);
        //通常of(T t)与get()搭配使用，用于获取Optional对象内封装的数据value
        String s = op.get();
        System.out.println(s);
    }

    @Test
    public void test3(){
        // String str="hello";
        String str=null;
        //ofNullable(T t):封装数据t实例化Optional对象内部的value，不要求t非空
        Optional<String> stringOptional = Optional.ofNullable(str);
        //orElse(T t1):若Optional对象内部的value值为空，则返回t1,否则返回Optional对象内部的value
        String str1 = stringOptional.orElse("hi");
        System.out.println(str1);
    }
}
```