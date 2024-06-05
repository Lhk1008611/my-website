---
sidebar_position: 11
---

# Java Enum
## 枚举
- JDK1.5 引入了枚举类型，枚举类型的定义包括枚举声明和枚举体，格式如下:
    ```java
      enum 枚举名{
        枚举体(常量列表)
      }
    ```
  
- 所有的枚举类型隐性地继承自 java.lang.Enum
- 枚举实质上还是类!而每个被枚举的成员实质就是一个枚举类型的实例，他们默认都是 `public static final` 修饰的。可以直接通过枚举类型名使用它们
- 当你需要定义一组常量时，可以使用枚举类型
- 尽量不要使用枚举的高级特性，事实上高级特性都可以使用普通类来实现，没有必要引入枚举，增加程序的复杂性

```java
import java.util.Random;

/**
 * 测试枚举类型
 * @author Lhk
 *
 */
public class Test01 {
	
	public static void main(String[] args) {
		System.out.println(Week.星期一);
		
		//枚举遍历
		for(Week k:Week.values())//增强for循环，Week.values()返回一个Week[],里面包含了所有枚举元素
		{
			System.out.println(k);
		}
		
		//switch中使用枚举
		int a = new Random().nextInt(4);//生成[0,4)的随机整数
		switch(Season.values()[a]){
		case 春天:
				System.out.println("春天");
		break;
		case 夏天:
			System.out.println("夏天");
		break;
		case 秋天:
			System.out.println("秋天");
		break;
		case 冬天:
			System.out.println("冬天");
		break;	
		}
	}

}

/*星期*/
enum Week{
	星期一,星期二,星期三,星期四,星期五,星期六,星期天
}

/*季节*/
enum Season{
	春天,夏天,秋天,冬天
}
```
