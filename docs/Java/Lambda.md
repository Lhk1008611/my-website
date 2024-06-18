---
sidebar_position: 21
---
# Lambda 表达式

## Lambda 语法
- Lambda 表达式:
  - 在Java8 语言中引入的一种新的语法元素和操作符。这个操作符为 `->`,该操作符被称为 Lambda 操作符或箭头操作符。
  - 它将 Lambda 分为两个部分:
    - 左侧:指定了 Lambda 表达式需要的参数列表
    - 右侧:指定了 Lambda 体，是抽象方法的实现逻辑，也即 Lambda 表达式要执行的功能。

- 语法格式一: 无参，无返回值

    ```java
            Runnable runnable = ()->{
                System.out.println("hello");
            };
    ```
  
- 语法格式二: Lambda 需要一个参数，但是没有返回值

    ```java
            Consumer<String> consumer = (String s) -> {
                System.out.println(s);
            };
    ```
  
- 语法格式三: 数据类型可以省略，因为可由编译器推断得出，称为“类型推断“

    ```java
            Consumer<String> consumer = (s) -> {
                System.out.println(s);
            };
    ```
  
- 语法格式四: Lambda 若只需要一个参数时，参数的小括号可以省略

    ```java
            Consumer<String> consumer = s -> { System.out.println(s); };
    ```
- 语法格式五: Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值

    ```java
            Comparator<Integer> comparator = (x,y)->{
                System.out.println("hello");
                return Integer.compare(x,y);
            };
    ```
  
- 语法格式六:当 Lambda 体只有一条语句时 return 与大括号若有，都可以省略

    ```java
            Comparator<Integer> comparator = (x,y)-> Integer.compare(x,y);
    ```
  
## 类型推断
- Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。
  - Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推断“

## Lambda 表达式的使用

```java
/**
* Lambda表达式的使用（6中情况）
* 格式：->：Lambda操作符 或 箭头操作符
* ->左边：Lambda形参列表（接口中的抽象方法的形参列表）
* ->右边：Lambda体（重写的抽象方法体）
*
* Lambda 表达式的本质：作为接口（函数式接口）的实例
*
* @author Lhk
*/
public class LambdaTest1 {
    //语法格式一：无参，无返回值
    @Test
    public void test1(){
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("theMutents");
            }
        };
        runnable.run();
        System.out.println("*******使用Lambda表达式*******");
        Runnable r2 = () -> System.out.println("Lhk");
        r2.run();
    }

    //格式二：Lambda 需要一个参数，但是没有返回值
    @Test
    public void test2(){
        Consumer<String > consumer=new Consumer<String>() {
            @Override
            public void accept(String s) {
            System.out.println(s);
            }
        };
        consumer.accept("TheMutents");
        System.out.println("*******使用Lambda表达式*******");
        Consumer<String > consumer1=(String s)->{
        	System.out.println(s);
        };
        consumer1.accept("不鸣则已，一鸣惊人");
    }

    //语法格式三：数据类型可以省略，因为可由编译器推断得出，称为“类型推断”
    @Test
    public void test3(){
        Consumer<String > consumer1=(s)->{//参数s的数据类型可由前面的类型推断出来
        	System.out.println(s);
        };
        consumer1.accept("不鸣则已，一鸣惊人");
        // ArrayList<String> arrayList=new ArrayList<>();//类型推断
        // int[] ints={1,2,3};//类型推断
    }

    //语法格式四：Lambda 若只需要一个参数时，参数的小括号可以省略
    @Test
    public void test4(){
        Consumer<String > consumer1= s ->{
        	System.out.println(s);
        };
        consumer1.accept("不鸣则已，一鸣惊人");
    }

    //语法格式五：Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值
    @Test
    public void test5(){
        Comparator<Integer> com1=new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                System.out.println(o1);
                System.out.println(o2);
                return o1.compareTo(o2);
            }
        };
        System.out.println(com1.compare(11,22));
        System.out.println("----------------------");
        Comparator<Integer> com2=(o1, o2) ->{
            System.out.println(o1);
            System.out.println(o2);
            return o1.compareTo(o2);
        };
        System.out.println(com2.compare(22,12));
    }

    //语法格式六：当 Lambda 体只有一条语句时，return 与大括号若有，都可以省略
    @Test
    public void test6(){
        Comparator<Integer> com1=(o1, o2) ->{
        	return o1.compareTo(o2);
        };
        System.out.println(com1.compare(22,12));
        System.out.println("-----------------------");
        Comparator<Integer> com2=(o1, o2) -> o1.compareTo(o2);
        System.out.println(com2.compare(10,20));
    }
}
```

## 函数式接口
- 什么是函数式(Functional)接口 
  - 只包含一个抽象方法的接口，称为函数式接口。
  - 可以通过 Lambda 表达式来创建该接口的对象。(若 Lambda 表达式抛出一个受检异常(即:非运行时异常)，那么该异常需要在目标接口的抽象方法上进行声明)。
  - 可以在一个接口上使用 `@FunctionalInterface` 注解，这样做可以检查它是否是一个函数式接口。同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。
  - 在 `java.util.function` 包下定义了 Java 8 的丰富的函数式接口
- Java 从诞生日起就是一直倡导“一切皆对象”，在 Java 里面面向对象(OOP)编程是一切。但是随着 python、scala 等语言的兴起和新技术的挑战，Java 不得不做出调整以便支持更加广泛的技术要求，也即 java 不但可以支持 OOP 还可以支持 OOF(面向函数编程)
  
- 在函数式编程语言当中，函数被当做一等公民对待。
  - 在将函数作为一等公民的编程语言中，Lambda 表达式的类型是函数。
  - 但是在 Java8 中，有所不同。在Java8中，Lambda 表达式是对象，而不是函数，它们必须依附于一类特别的对象类型——函数式接口。
  - 简单的说，在 Java8 中，Lambda 表达式就是一个函数式接口的实例。这就是 Lambda 表达式和函数式接口的关系。
    - 也就是说，只要一个对象是函数式接口的实例，那么该对象就可以用Lambda表达式来表示。 
    - 所以以前用匿名实现类表示的现在都可以用Lambda表达式来写

  ```java
  /**
  * 自定义函数式接口
  * @author Lhk
  */
  @FunctionalInterface
  public interface FunctionInterface {
  	void method1();
  }
  ```
  
### Java 内置四大核心函数式接口

| 函数式接口                   | 参数类型 | 返回类型 | 用途                                                         |
| ---------------------------- | -------- | -------- | ------------------------------------------------------------ |
| `Consumer<T>`  消费型接口    | T        | void     | 对类型为T的对象应用操作，包含方法: `void accept(T t)`        |
| `Supplier<T>`  供给型接口    | 无       | T        | 返回类型为T的对象，包含方法: `T get()`                       |
| `Function<T, R>`  函数型接口 | T        | R        | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。包含方法:`R apply(T t)` |
| `Predicate<T>`  断定型接口   | T        | boolean  | 确定类型为T的对象是否满足某约束，并返回 boolean 值。包含方法: `boolean test(T t)` |

- 总结：
  1. 当需要对一个函数式接口实例化的时候，可以使用 Lambda 表达式
  2. 如果在开发中需要定义一个函数式接口，首先看看在已有的 JDK 提供的函数式接口是否提供了能满足的函数式接口，如果有，则直接调用即可，不需要再次定义

```java
/**
* java内置4大核心函数式接口
* 消费型接口：Consumer<T> void accept(T t)
* 供给型接口：Supplier<T> T get()
* 函数型接口：Function<T,R> R apply(T t)
* 断定型接口：Predicate<T> boolean test(T t)
*
* @author Lhk
*/
public class LambdaTest2 {

    @Test
    public void test1(){
        happyTime(500, new Consumer<Double>() {
            @Override
            public void accept(Double aDouble) {
                System.out.println("消费"+aDouble+"元");
            }
        });
        System.out.println("----------------------------");
        happyTime(5000, money->System.out.println("消费"+money+"元"));
    }

    public void happyTime(double money, Consumer<Double> consumer){
        consumer.accept(money);
    }

    @Test
    public void test2(){
        List<String> list= Arrays.asList("北京","南京","天津","东京","西京","普京");
        List<String> stringList = filterString(list, new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.contains("京");
            }
        });
        System.out.println(stringList);
        System.out.println("-----------------");
        List<String> stringList1 =filterString(list, s-> s.contains("京")) ;
        System.out.println(stringList1);
    }
    //根据给定的规则过滤集合中的字符串，该规则由Predicate的方法决定
    public List<String > filterString(List<String> list, Predicate<String> predicate){
        ArrayList<String> arrayList=new ArrayList<>();
        for (String str:list) {
        	if (predicate.test(str)){
        		arrayList.add(str);
        	}
        }
        return arrayList ;
    }
}
```

### 其他接口

| 函数式接口                                                   | 参数类型                | 返回类型                | 用途                                                         |
| ------------------------------------------------------------ | ----------------------- | ----------------------- | ------------------------------------------------------------ |
| `BiFunction<T, U, R>`                                        | T, U                    | R                       | 对类型为 T, U 参数应用操作，返回 R 类型的结果。包含方法为: `R apply(T t,U u);` |
| `UnaryOperator<T>`   Function 的子接口                       | T                       | T                       | 对类型为T的对象进行一元运算，并返回T类型的结果。包含方法为: `T apply(T t);` |
| `BinaryOperator<T> `  BiFunction 的子接口                    | T, T                    | T                       | 对类型为T的对象进行二元运算，并返回T类型的结果。包含方法为:`T apply(T t1,T t2);` |
| `BiConsumer<T, U>`                                           | T, U                    | void                    | 对类型为T. U 参数应用操作。包含方法为: `void accept(T t, U u)` |
| `BiPredicate<T,U>`                                           | T, U                    | boolean                 | 包含方法为: `boolean test(T t,U u)`                          |
| `TolntFunction<T>`<br/>`ToLongFunction<T>`<br/>`ToDoubleFunction<T>` | T                       | int<br/>long<br/>double | 分别计算 int、long、double 值的函数                          |
| `IntFunction<R>`<br/>`LongFunction<R>`<br/>`DoubleFunction<R>` | int<br/>long<br/>double | R                       | 参数分别为 int、long、double 类型的函数                      |


## 方法引用与构造器引用（基于 Lambda 表达式）
- 方法引用(Method References)
  - 当要传递给 Lambda 体的操作，已经有实现的方法了，可以使用方法引用!
  - 方法引用可以看做是 Lambda 表达式深层次的表达。换句话说，方法引用就是 Lambda 表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是 Lambda 表达式的一个语法糖。
  - 要求: 
    - 实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致!、
  - 格式: 使用操作符`::`将类(或对象)与方法名分隔开来。 
    - 如下三种主要使用情况:
      - `对象::实例方法名`
      - `类::静态方法名`
      - `类::实例方法名`

```java
/**
* 方法引用的使用
*
* 1.使用情景： 当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用
*
* 2.方法引用本质上就是Lambda表达式，而Lambda表达式作为函数式接口的实例，所以方法引用也是函数式接口的实例
*
* 3.使用格式：类（对象）::方法名
* 具体有如下三种情况：
* 对象::非静态方法
* 类::静态方法
* 类::非静态方法 当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时
*
* 4.方法引用的使用要求：
* 要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同(针对第一种和第二种情况)
* 当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时:ClassName::methodName（针对第三种情况）
*/
public class MethodRefTest {

    // 情况一：对象 :: 实例方法
    //Consumer中的void accept(T t)
    //PrintStream中的void println(T t)
    @Test
    public void test1() {
        Consumer<String > consumer1=str -> System.out.println(str);
        consumer1.accept("Lhk");
        System.out.println("********************");
        PrintStream ps = System.out;
        Consumer<String > consumer=ps::println;
        System.out.println("湖南");
    }

    //Supplier中的T get()
    //Employee中的String getName()
    @Test
    public void test2() {
        Employee employee=new Employee(1001,"lhk",21,5000);
        Supplier<String> supplier=()-> employee.getName();
        System.out.println(supplier.get());
        System.out.println("********************");
        Supplier<String> supplier1= employee :: getName;
        System.out.println(supplier1.get());
    }

    // 情况二：类 :: 静态方法
    //Comparator中的int compare(T t1,T t2)
    //Integer中的int compare(T t1,T t2)
    @Test
    public void test3() {
        Comparator<Integer> comparator1=(t1,t2)->Integer.compare(t1,t2);
        System.out.println(comparator1.compare(10,20));
        System.out.println("------------------");
        Comparator<Integer> comparator2=Integer::compare;
        System.out.println(comparator2.compare(20,10));
    }

    //Function中的R apply(T t)
    //Math中的Long round(Double d)
    @Test
    public void test4() {
        Function<Double,Long> function=d->Math.round(d);
        System.out.println(function.apply(2000.056));
        System.out.println("-----------------");
        Function<Double,Long> function1=Math::round;
        System.out.println(function1.apply(2056.2056));
    }

    // 情况三：类 :: 实例方法
    // Comparator中的int comapre(T t1,T t2)
    // String中的int t1.compareTo(t2)
    @Test
    public void test5() {
        Comparator<String> comparator1=(t1,t2)->t1.compareTo(t2);
        System.out.println(comparator1.compare("acc","acb"));
        System.out.println("----------------------");
        Comparator<String> comparator2=String::compareTo;
        System.out.println(comparator2.compare("abc","abj"));
    }

    //BiPredicate中的boolean test(T t1, T t2);
    //String中的boolean t1.equals(t2)
    @Test
    public void test6() {
        BiPredicate<String,String > biPredicate1=(s1,s2)->s1.equals(s2);
        System.out.println(biPredicate1.test("abc","abb"));
        System.out.println("--------------------");
        BiPredicate<String,String > biPredicate2=String::equals;
        System.out.println(biPredicate2.test("abc","abc"));
    }

    // Function中的R apply(T t)
    // Employee中的String getName();
    @Test
    public void test7() {
        Function<Employee,String> function=e -> e.getName();
        System.out.println(function.apply(new Employee(1001,"TheMutents",21,5000)));
        System.out.println("-----------------------");
        Function<Employee,String> function1=Employee::getName;
        System.out.println(function.apply(new Employee(1002,"Lhk",21,5000)));
    }
}
```

- 构造器引用
  - 格式: `ClassName::new`
  - 与函数式接口相结合，自动与函数式接口中方法兼容。
  - 可以把构造器引用赋值给定义的方法，要求构造器参数列表要与接口中抽象方法的参数列表一致!且方法的返回值即为构造器对应类的对象

- 数组引用
  - 格式: `type::new[]`

```java
/**
* 一、构造器引用
* 与方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致
* 抽象方法的返回值类型即为构造器所属的类型
*
* 二、数组引用
* 可以把数组看做成一个特殊的类，则写法与构造器引用一致
*
*/
public class ConstructorRefTest {
    //构造器引用
    //Supplier中的T get()
    //Employee的空参构造器：Employee()
    @Test
    public void test1(){
        Supplier<Employee> supplier=new Supplier<Employee>() {
            @Override
            public Employee get() {
            return new Employee();
            }
        };
        supplier.get();
        System.out.println("--------------------------------");
        Supplier<Employee> supplier1=() -> new Employee();
        supplier1.get();
        System.out.println("--------------------------------");
        //构造器引用
        Supplier<Employee> supplier2=Employee::new;
        supplier2.get();
    }

    //Function中的R apply(T t)
    @Test
    public void test2(){
        Function<Integer,Employee> function=id ->new Employee(id);
        System.out.println(function.apply(1002));
        System.out.println("--------------------------------");
        Function<Integer,Employee> function1=Employee::new;
        System.out.println(function1.apply(1003));
    }

    //BiFunction中的R apply(T t,U u)
    @Test
    public void test3(){
        BiFunction<Integer,String,Employee> biFunction=(id,name)->new Employee(id,name);
        System.out.println(biFunction.apply(1004,"Lhk"));
        System.out.println("--------------------------------");
        BiFunction<Integer,String,Employee> biFunction2=Employee::new;
        System.out.println(biFunction2.apply(1005,"小明"));
    }

    //数组引用
    //Function中的R apply(T t)
    @Test
    public void test4(){
        Function<Integer,String[]> function=lenght -> new String[lenght];
        String[] strings = function.apply(5);
        System.out.println(Arrays.toString(strings));
        System.out.println("-----------------------");
        Function<Integer,String[]> function1=String[]::new;
        String[] strings1 = function1.apply(8);
        System.out.println(Arrays.toString(strings1));
    }
}
```