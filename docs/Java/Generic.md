---
sidebar_position: 13
---

# Java 泛型
## 泛型简介
- 泛型是 JDK1.5 以后增加的，它可以帮助我们建立类型安全的集合
- 泛型的本质就是“数据类型的参数化”，处理的数据类型不是固定的，而是可以作为数传入
- 我们可以把“泛型”理解为数据类型的一个占位符(类似:形式参数)，即告诉译器，在调用泛型时必须传入实际类型
  - 这种参数类型可以用在类、接口和方法中，分别称为**泛型类**、**泛型接口**、**泛型方法**
  - 参数化类型，白话说就是:
    1. 把类型当作是参数一样传递
    2. `<数据类型>`只能是引用类型
## 泛型好处
- 在不使用泛型的情况下，我们可以使用 Object 类型来实现任意的参数类型，
  - 但是在使用时需要我们强制进行类型转换,这就要求程序员明确知道实际类型，不然可能引起类型转换错误:但是，在编译期我们无法识别这种错误，只能在运行期发现这种错误
  - 使用泛型的好处就是可以在编译期就识别出这种错误，有了更好的安全性
  - 同时，所有类型转换由编译器完成，在程序员看来都是自动转换的，提高了代码的可读性。
- 总结一下，就是使用泛型主要是两个好处:
  - 代码可读性更好 【不用强制转换】
  - 程序更加安全 【只要编译时期没有警告，运行时期就不会出现 `ClassCastException` 异常】

## 类型擦除
- 编码时采用泛型写的类型参数，编译器会在编译时去掉，这称之为“类型擦除”
- 泛型主要用于编译阶段，编译后生成的字节码 class 文件不包含泛型中的类型信息，**涉及类型转换仍然是普通的强制类型转换**
  - **类型参数在编译后会被替换成 object**，运行时虚拟机并不知道泛型
- 泛型主要是方便了程序员的代码编写，以及更好的安全性检测

## 泛型的使用
### 泛型字符
- 泛型字符可以是任何标识符，一般采用几个标记:`E`、`T`、`K`、`V`、`N`、`?`

  | 泛型标记 | 对应单词 | 说明                           |
  | -------- | -------- | ------------------------------ |
  | E        | Element  | 在容器中使用，表示容器中的元素 |
  | T        | Type     | 表示普通的  JAVA  类           |
  | K        | Key      | 表示键，例如: Map中的键        |
  | V        | Value    | 表示值                         |
  | N        | Number   | 表示数值类型                   |
  | ?        |          | 表示不确定的 JAVA 类型         |

### 泛型类
- 泛型类就是把泛型定义在类上，用户使用该类的时候，才把类型明确下来。
- 泛型类的具体使用方法是在类的名称后添加一个或多个类型参数声明，
  - 如:`<T>`、`<T,K,V>`
- 语法结构
  ```java
    public class 类名<泛型表示符号>{
    }
  ```
  
- 例
  ```java
     /**
      * 测试泛型
      * @author Lhk
      *
      */

     /**
      * 如果一个类不使用泛型，则在该类中所定义的成员变量和方法参数类型、返回值类型都得自己去定义所需要类型
      * 如果使用泛型，则该类中的成员变量，方法参数或者返回值类型都可以使用该类定义的泛型类型
      */
      public class Generic <T> {
         private T flag;

         public T getFlag() {
            return flag;
         }

         public void setFlag(T flag) {
            this.flag = flag;
         }

	     public static void main(String[] args) {
           // 创建对象时不使用泛型的话，falg 为 Object 类型，且方法中的参数和返回值类型都为 Object 类型
		     // 成员属性类型和方法参数类型、返回值类型还得根据所需要类型来进行类型强制转换
		     Generic<String> str=new Generic<>();
		     str.setFlag("泛型");
		     System.out.println(str.getFlag());
		     //此处同上，泛型类型可变
		     Generic<Integer> i=new Generic<>();
		     i.setFlag(1000);
		     System.out.println(i.getFlag());
	       }
      }
  ```
  
### 泛型接口
- 泛型接口和泛型类的声明方式一致
- 泛型接口的具体类型需要在实现类中进行声明
  ```java
    public interface 接口名<泛型表示符号>{
    }
  ```

```java
/**
 * 定义一个泛型接口
 * @author Lhk
 * @param <T>
 */
public interface Igeneric <T> {
	T getName(T name);
}
```

```java
/**
 * 实现Igeneric接口
 * @author Lhk
 *
 */
public class IplemIgeneric implements Igeneric<String> {//此处将泛型定义为String类型
	
	@Override
	//实现接口方法，此时返回值类型和参数类型都为String类型
	public String getName(String name) {
		return name;
	}
	
	
	public static void main(String[] args) {
		//创建IplemIgeneric类型对象，此时接口里面泛型都变为String类型
		IplemIgeneric name=new IplemIgeneric();
		System.out.println(name.getName("TheMutents"));
		
		/*Igneric类型引用IplemIgrneric对象，此时需要设置泛型的类型，
		 * 不然对象属性和方法参数、返回值类型都属于Object类型
		 */
		Igeneric<String> name1=new IplemIgeneric();
		System.out.println(name1.getName("TheMutents"));
	}
}
```

### 泛型方法
- 泛型类中所定义的泛型，在方法中也可以使用。
  - 但是，我们经常需要仅仅在某一个方法上使用泛型，这时候可以使用泛型方法
- 泛型方法是指将方法的参数类型定义成泛型，以便在调用时接收不同类型的参数
  - 类型参数可以有多个，用逗号隔开，如:`<K,V>`
  - 定义时，类型参数一般放到返回值前面
  - 调用泛型方法时，不需要像泛型类那样告诉编译器是什么类型，编译器可以自动推断出类型来

#### 非静态方法使用泛型
-  两种使用泛型的方式
   1. 使用泛型类的泛型作为方法的参数和返回值
   2. 使用如下语法结构在方法中使用泛型
      ```java
        public <泛型表示符号> void getName(泛型表示符号 name){
        }
      ```
      ```java
        public <泛型表示符号> 泛型表示符号 getName(泛型表示符号 name){
        }
      ```

```java
/**
 * 非静态方法使用泛型类泛型
 * @author Lhk
 * @param <T>
 */
public class MethodGeneric_1<T> {
	
	public void getValue(T value){
		 System.out.println(value); 
	}
	
	public T setValue(T value ){
		return value;
	}
	
	public static void main(String[] args) {
		MethodGeneric_1<String> m1=new MethodGeneric_1<>();
		m1.getValue("TheMutents");
		System.out.println(m1.setValue("themutents"));
	}
}

```

```java
/**
 * 普通类中的非静态泛型方法
 * @author Lhk
 *
 */
public class MethodGeneric_02 {

	public<T> void getValue(T value){
		System.out.println(value);
	}
	
	public<T> T setValue(T value){
		return value;
	}
	
	public static void main(String[] args) {
		MethodGeneric_02 m2=new MethodGeneric_02();
		m2.getValue("TheMutens");//泛型方法会自动识别参数类型
		System.out.println(m2.setValue(212792772));//此处识别value为Integer类型
	}
}
```

#### 静态方法使用泛型
- 静态方法中使用泛型需注意：静态方法无法访问泛型类上的泛型
- 当静态方法操作的引用数据类型不确定的时候，必须将泛型定义在方法上
- 语法结构
  ```java
  public static <泛型表示符号> void getName(泛型表示符号 name){
  }
  ```
  ```java
  public static <泛型表示符号> 泛型表示符号 getName(泛型表示符号 name){
  }
  ```

```java
/**
 * 测试静态泛型方法
 * @author Lhk
 *
 */
public class MethodGeneric_3 {

	public static<T> T setValue(T value){
		return value;
	}
	
	public static<T> void getValue(T value){
		System.out.println(value);
	}
	
	public static void main(String[] args) {
		MethodGeneric_3.getValue("TheMutents");//自动识别参数类型
		System.out.println(MethodGeneric_3.setValue(212792772));
	}
}

```

#### 泛型方法和可变参数
- 在泛型方法中，泛型也可以定义为可变参数

```java
/**
 * 测试泛型方法与可变参数
 * @author Lhk
 *
 */
public class MethodGeneric_04 {
	
	@SafeVarargs
	public static<T> void print(T...obj){
		for(T obj1:obj){
			System.out.println(obj1);
		}
	}
	
	public static void main(String[] args) {
		print("TheMutents","QQ",212792772);
	}
}
```

## 泛型通配符和上下限定
### 无界通配符
- `?` 表示类型通配符，用于代替具体的类型。它只能在`<>`中使用。可以解决当具体类型不确定的问题

```java
public class GenericTest_05<T> {
	private T t;

	public T getT() {
		return t;
	}

	public void setT(T t) {
		this.t = t;
	}
	
}

//该类实现获取类GenericTest_05类的泛型属性t
class Show_t{
	//方法参数是使用泛型的GenericTest_05对象，且泛型设置为通配符<?>
	//这样当传入该方法参数GenericTest_05对象的泛型设置成任意类型时都可以使用该方法
	public void showT(GenericTest_05<?> generic){
		System.out.println(generic.getT());
	}
}

public class Test_05 {
	
		public static void main(String[] args) {
			Show_t show_t=new Show_t();
			//泛型类型为Integer类型的GenericTest_05对象
			GenericTest_05<Integer> i1=new GenericTest_05<>();
			i1.setT(212792772);
			show_t.showT(i1);
			
			//Number是Integer的父类
			//泛型类型为Number类型的GenericTest_05对象
			GenericTest_05<Number> i2=new GenericTest_05<>();
			i2.setT(212792772);
			show_t.showT(i2);
			
			//泛型类型为String类型的GenericTest_05对象
			GenericTest_05<String> s1=new GenericTest_05<>();
			s1.setT("TheMutents");
			show_t.showT(s1);
		}
	}
```

### 通配符的上下限定
- 因为通配符`<?>`表示的是任意类型，当我们不需要任意类型的时候（觉得任意类型太广了），我们可以对通配符进行限定（上限限定和下限限定）

#### 上限限定（extends）
- 语法结构
  ```java
    public void showT(GenericTest_05<? extends Number> generic){}
  ```
  或
  ```java
  public class GenericTest_05<T extends Number> {}
  ```
  
- 表示`<?>`的类型只能是在`extends`关键字后面的类型其本身或其子类，这样就对`<?>`进行上限限定
- 当然也可以对`<T>`按照同样语法进行上限限定

```java
public class GenericTest_05<T> {
	private T t;

	public T getT() {
		return t;
	}

	public void setT(T t) {
		this.t = t;
	}
	
}

//该类实现获取类GenericTest_05类的泛型属性t
class Show_t{
	//方法参数是使用泛型的GenericTest_05对象，且泛型设置为通配符<?>
	//这样当传入该方法参数GenericTest_05对象的泛型设置成任意类型时都可以使用该方法
	public  void showT(GenericTest_05<? extends Number>  generic){
		System.out.println(generic.getT());
	}
}

public class Test_05 {
		public static void main(String[] args) {
			Show_t show_t=new Show_t();
			//泛型类型为Integer类型的GenericTest_05对象
			GenericTest_05<Integer> i1=new GenericTest_05<>();
			i1.setT(212792772);
			show_t.showT(i1);
			
			//Number是Integer的父类
			//泛型类型为Number类型的GenericTest_05对象
			GenericTest_05<Number> i2=new GenericTest_05<>();
			i2.setT(212792772);
			show_t.showT(i2);
		}
}
```

#### 下限限定（super）
- 下限限定表示通配符的类型是`T`类以及`T`类的父类或者`T`接口以及`T`接口的父接口
- 注意:该方法不适用泛型类
- 语法结构
    ```java
    public void showT(GenericTest_05<? super Integer> generic){}
   ```

```java
public class GenericTest_05<T> {
	private T t;

	public T getT() {
		return t;
	}

	public void setT(T t) {
		this.t = t;
	}
	
}

//该类实现获取类GenericTest_05类的泛型属性t
class Show_t{
	//方法参数是使用泛型的GenericTest_05对象，且泛型设置为通配符<?>
	//这样当传入该方法参数GenericTest_05对象的泛型设置成任意类型时都可以使用该方法
	public  void showT(GenericTest_05<? super Integer>  generic){
		System.out.println(generic.getT());
	}
}

public class Test_05 {
		public static void main(String[] args) {
			Show_t show_t=new Show_t();
			//泛型类型为Integer类型的GenericTest_05对象
			GenericTest_05<Integer> i1=new GenericTest_05<>();
			i1.setT(212792772);
			show_t.showT(i1);
			
			//Number是Integer的父类
			//泛型类型为Number类型的GenericTest_05对象
			GenericTest_05<Number> i2=new GenericTest_05<>();
			i2.setT(212792772);
			show_t.showT(i2);	
		}
}
```

## 泛型总结
- 泛型主要用于编译阶段，编译后生成的字节码 class 文件不包含泛型中的类型信息
- 类型参数在编译后会被替换成 Object，运行时虚拟机并不知道泛型。
  - 因此，使用泛型时，如下几种情况是错误的:
    1. 基本类型不能用于泛型 
       `Test<int> t;` 这样写法是错误，我们可以使用对应的包装类:`Test<Integer> t;`
    2. 不能通过类型参数创建对象。
       `T elm=new T();`运行时类型参数T会被替换成 Object，无法创建T类型的对象，容易引起误解，所以在Java 中不支持这种写法

