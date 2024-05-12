---
sidebar_position: 2
---

# 抽象类与接口


## 一、普通类，抽象类，接口的区别
- 声明：普通类，抽象类和接口都是类
- 区别
  - 普通类：具体实现
  - 抽象类：具体实现、规范（抽象方法）
  - 接口：规范

## 二、抽象类的相关要点
1. 有抽象方法的类只能定义为抽象类
2. 抽象类只能被继承
3. 抽象类不能实例化，既不能使用 `new` 创建对象
4. 抽象类可以包含属性、方法、构造方法，但是构造方法不能进行实例化，只能被子类调用
5. 抽象方法必须被子类实现

## 三、接口的相关要点
1. 接口里面只能定义常量和抽象方法（限制修饰符会自动赋予为 public ）
   - 接口可以继承多个接口，但只能继承一个抽象类

2. 要实现接口，就必须实现接口的所有抽象方法

3. 某个类实现多个接口时，创建的对象的引用为某一个接口类型时，则该对象只能调用实现此接口的方法，不能调用实现其他接口的方法，如需调用实现其他接口的方法，则需改变对象引用类型。


## 四、接口中定义静态方法和默认方法
- 扩展知识
  - 此知识点在开发过程中用的少，一般在一些底层源码中使用，一些开源软件会用到）
### 1.默认方法 
- java 8之后，允许给接口添加一个非抽象的方法具体实现，只需要使用 default 关键字即可，这个特征被称为默认方法（或扩展方法）
- java 8之前，接口中只能定义抽象方法

- 默认方法与抽象方法的区别
  - 抽象方法必须被子类实现，抽象方法不能有具体实现
  - 默认方法可以继承给实现该接口的子类，并且子类可以覆盖默认方法，覆盖默认方法的时候，必须使用 `override` 关键字
- 例：
    ```java
    public interface Shape {
        default void test(){
            System.out.println("接口的默认方法");//接口中的默认方法，会继承给实现该接口的子类
        }
    double size();
    }
    
    public class Shape_interface_test {
        public static void main(String args[]){
            Shape a=new Rectangle(10.0,10.0);
            a.test();//父类引用子类对象调用继承接口的默认方法
            Rectangle b=new Rectangle(12.0,10.0);
            b.test();//子类对象调用继承接口的默认方法
        }
    }
    ```


### 2.静态方法
- java 8之后，允许给接口添加一个静态方法，只需要使用 static 关键字即可，这个特征被称为静态方法
- 该方法从属于接口，可用接口名来调用，因为接口的本质是类
- 
- 例：
   ```java
     public class Test_03_interface_staticfangfa {
        
        public static void main(String args[]){
            A.staticmethod();  // 输出：接口A中的默认方法
            //类和接口中定义同名静态方法，这两个方法分别从属于A和B
            B.staticmethod();  //输出：类B中的静态方法
            B b=new B();
            b.defaultmethed(); //输出：接口A中的默认方法
        }
    }
    
    interface A{
        static void staticmethod(){
            System.out.println("接口A中的静态方法");//在静态方法中不能调用默认方法
        }
        
        default void defaultmethed(){
            staticmethod();   //在默认方法中可以调用静态方法
            System.out.println("接口A中的默认方法");
        }
    }
    
    
    class B implements A{
        static void staticmethod(){
            System.out.println("类B中的静态方法");
        }
    }
   ```
## 五、接口的多继承
- 普通类继承只能单继承
- 接口可以继承多个父接口

```java
public class Test_04_interface_duojicheng implements C {

	public void testA(){}   //实现接口C必须实现C中的所有方法
	public void testB(){}
	public void testC(){}
}


interface A{
	void testA();
}

interface B{
	void testB();
}

//接口C继承接口A，完成多继承
interface C extends A,B{
	void testC();
}
```
