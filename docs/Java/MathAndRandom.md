---
sidebar_position: 9
---

# Math and Random 类

## Math 类
- java.lang.Math 提供了一系列静态方法用于科学计算;其方法的参数和返回值类型般为 double 型。如果需要更加强大的数学运算能力，计算高等数学中的相关内容，可以使用 apache commons 下面的 Math 类库
- Math 类的常用方法:

  | 方法名 | 含义 |
  | -------------------------- | ------------------------------------------- |
  | abs                        | 绝对值                                      |
  | acos,asin,atan,cos,sin,tan | 三角函数                                    |
  | sqrt                       | 平方根                                      |
  | pow(double a, double b)    | a的b 次幂                                   |
  | max(double a, double b)    | 取大值                                      |
  | min(double a, double b)    | 取小值                                      |
  | ceil(double a)             | 大于a的最小整数                             |
  | floor(double a)            | 小于a的最大整数                             |
  | random()                   | 返回 0.0 到 1.0 的随机数                    |
  | long round(double a)       | double 型的数据 a 转换为 long 型 (四舍五入) |
  | toDegrees(double angrad)   | 弧度->角度                                  |
  | toRadians(double angdeg)   | 角度->弧度                                  |

  ```java
  package Math_Test;
  import java.lang.Math;
  
  public class Test_01 {
  	public static void main(String[] args) {
  		test();
  	}

	public static void test(){
		System.out.println(Math.abs(-3.5));	
		System.out.println(Math.cos(0)+" "+Math.sin(0)+" "+Math.tan(0)+" "+Math.acos(0)+" "+Math.asin(0)+" "+Math.atan(0));	
		System.out.println(Math.sqrt(4));//开方
		System.out.println(Math.pow(5,2));	//5的2次方
		System.out.println(Math.max(3.5, 3.6));//两个数的最大值
		System.out.println(Math.min(3.5, 3.6));//两个数的最小值
		System.out.println(Math.ceil(9.9));//大于9.9的最小整数
		System.out.println(Math.floor(9.9));//小于9.9的最大整数
		System.out.println(Math.random());//[0.0,1.0)的随机数
		System.out.println(Math.round(12.55623));//将double转成long（四舍五入）
		System.out.println(Math.toDegrees(Math.PI));//弧度转成角度
		System.out.println(Math.toRadians(180));//角度转成弧度
	}
  }
  ```

## Random 类
- Math 类中虽然为我们提供了产生随机数的方法 `Math.random()`，但是通常我们需要的随机数范围并不是[0,1)之间的 double 类型的数据，这就需要对其进行一些复杂的运算来得到我们需要的随机数
- 如果使用 `Math.random()` 计算过于复杂的话，我们可以使用例外一种方式得到随机数，即 Random 类，这个类是专门用来生成随机数的，并且`Math.random()`底层调用的就是 Random 的 `nextDouble()`方法
```java
import java.util.Random;

public class Test_01 {
	public static void main(String[] args) {
		Random r=new Random();
		System.out.println(r.nextBoolean());//随机产生true或者false
		System.out.println(r.nextDouble());//产生[0,1)的double类型随机数
		System.out.println(r.nextFloat());//产生[0,1)的float类型随机数
		System.out.println(r.nextInt());//产生在int类型范围内的随机数
		System.out.println(r.nextInt(20));//产生[0,20)的int类型随机数
		System.out.println(10+r.nextInt(10));//产生[10,20)的int类型随机数
		System.out.println(10+(int)(10*r.nextDouble()));//产生[10,20)的int类型随机数
	}

}

```
