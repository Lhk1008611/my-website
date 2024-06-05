---
sidebar_position: 7
---

# Java 包装类
> 为什么需要包装类(Wrapper class)?
> - Java 并不是纯面向对象的语言。Java 语言是一个面向对象的语言，但是 Java 中的基本数据类型却是不面向对象的。但是我们在实际使用中经常需要将基本数据转化成对象，便于操作。比如:集合的操作中。这时，我们就需要将基本类型数据转化成对象!
> - 包装类的作用:
>   - 提供:字符串、基本类型数据与对象之间互相转化的方式
>   - 包含每种基本数据类型的相关属性如最大值、最小值等

- 包装类均位于 `java.lang` 包下，对应关系如下：

  | 基本数据类型 | 包装类    |
  | :----------- | --------- |
  | byte         | Byte      |
  | boolean      | Boolean   |
  | short        | Short     |
  | char         | Character |
  | int          | Integer   |
  | long         | Long      |
  | float        | Float     |
  | double       | Double    |
- 在上述八个包装类中，除了 Character 和 Boolean 以外，其他的都是**数字型**，**数字型**都是 `java.lang.Number` 的子类
- 例：
  ```java
   public static void main(String[] args) {
			Integer i= new Integer(30);//从java9开始被废弃，可以使用但不建议
			Integer a= Integer.valueOf(100);//官方建议使用这种方法创建Integer对象，将int类型转化为Integer对象

			int b=i.intValue();//将Integer对象i转化为int类型
			double d=i.doubleValue();
			float f=a.floatValue();
			short s=a.shortValue();
			long l=a.longValue();
			byte bb=a.byteValue();
			
			System.out.println(a+" "+b+" "+d+" "+f+" "+s+" "+l+" "+bb);
			
			//将字符串数字转成Integer对象
			Integer a1=Integer.valueOf("555");
			Integer a2=Integer.parseInt("123");
			
			System.out.println(a1+" "+a2);
			
			//将包装类对象转成字符串
			String str1=a1.toString();
			System.out.println(str1);
			
			//常用常量
			System.out.println(Integer.MAX_VALUE+" "+Integer.MIN_VALUE+" "+Integer.BYTES+" "+Integer.SIZE);
   }
  ```
  
## 自动装箱和自动拆箱
- 自动装箱和拆箱就是将基本数据类型和包装类之间进行自动的互相转换。JDK1.5 后 Java 引入了自动装箱(autoboxing)/拆箱(unboxing)
- 自动装箱-autoboxing
  - 基本类型就自动地封装到与它相同类型的包装中，如
    - `Integeri= 100;` 本质上是，编译器编译时为我们添加了 `Integeri=IntegervalueOf(100);`
- 自动拆箱-autounboxing
  - 包装类对象自动转换成基本类型数据。如
    - `int a= new Integer(100);` 本质上，编译器编译时为我们添加了`int a= new Integer(100).intValue();`
- 示例:
  ```java
			//自动装箱
			Integer a3=500;//编译器改为Integer a3=Integer.valueOf(500);
			Double b1=3000.000;
			//自动拆箱
			int a4=a3;  //编译器改为int a4=a3.intValue();
			double b2=b1;
  ```
  
- 注意：
  ```java
			//空指针异常
			Integer c = null;
			int c1 = c;   
			//此处编译器会改为 int c1=c.intValue(); 编译能通过，运行会有空指针异常
			//Exception in thread "main" java.lang.NullPointerException
			//因为对象c为空，调用了对象的方法 intValue() 导致报错
  ```

## 包装类的缓存问题
- 整型、char 类型所对应的包装类，在自动装箱时，对于-128~127之间的值会进行缓存处理，其目的是提高效率。
- 缓存处理的原理为:如果数据在 -128~127 这个区间，那么在类加载时就已经为该区间的每个数值创建了对象，并将这256个对象存放到一个名为 cache 的数组中。每当自动装箱过程发生时(或者手动调用 `value0f()` 时)，就会先判断数据是否在该区间，如果在则直接获取数组中对应的包装类对象的引用，如果不在该区间，则会通过 new 调用包装类的构造方创建对象
- `valueOf()`源代码
  ```java
      public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
      }
  ```
  
- 测试代码：
  ```java
			//包装类的缓存问题
			Integer e1=1000;
			Integer e2=1000;
			//数字在[-127,128]的时候，放回缓冲数组中的某个元素，是同一个对象
			Integer e3=120;
			Integer e4=120;
			System.out.println(e1==e2);//false 两个不同的对象
			System.out.println(e3==e4);//true
			System.out.println(e1.equals(e2));//true
			System.out.println(e3.equals(e4));//true
   ```

## 自定义一个包装类
```java
package com.lhk.WrapperClass;
/**
 * 自定义一个简单的包装类
 * @author Lhk
 *
 */
public class MyInteger {
	private int value;
	private static MyInteger[] cache=new MyInteger[256];
	final static int low=-128;
	final static int high=127;
	
	
	static{
		for(int i=low;i<=high;i++){
		//数组索引从0开始，故索引从i-low开始到256
			cache[i-low]=new MyInteger(i);
		}
		
	}

	
	 private MyInteger(int i) {
		this.setValue(i);
	}

	
	public static MyInteger valueOf(int i) {
		if(i>=low&&i<=high){
			return cache[i-low];
		}
		return new MyInteger(i);
	}
	

	public int getValue() {
		return value;
	}


	public void setValue(int value) {
		this.value = value;
	}
	
	@Override
	public String toString() {
		return  value+"" ;
	}
	
	

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + value;
		return result;
	}


	public boolean equals(Object a) {
		if (a == null)
			return false;
		
		if (getClass() != a.getClass())
			return false;
		
		MyInteger other = (MyInteger) a;
		if (value != other.value)
			return false;
		return true;
	}
	
	public int intValue(){
		return this.value;
	}


	public static void main(String[] args) {
		MyInteger m=MyInteger.valueOf(300);
		int m1=m.intValue();
		MyInteger a1=MyInteger.valueOf(123);
		MyInteger a2=MyInteger.valueOf(123);
		MyInteger b1=MyInteger.valueOf(277);
		MyInteger b2=MyInteger.valueOf(277);
		System.out.println(m+" "+m1);
		System.out.println(a1+" "+a2+" "+(a1==a2)+" "+a1.equals(a2));
		System.out.println(b1+" "+b2+" "+(b1==b2)+" "+b1.equals(b2));
	}
}
```

