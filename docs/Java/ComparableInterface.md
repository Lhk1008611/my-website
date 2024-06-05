---
sidebar_position: 6
---

# Comparable 接口
- 想对某个类的对象之间做比较，就需要实现 `Comparable` 接口。
- 按口中只有一个方法 `compareTo`，这个方法定义了对象之间的比较规则。
  - 依据这个“比较规则”，我们就能对所有对象实现排序。
  - 事实上，java 中排序算法的底层也依赖 `Comparable` 接口
  - `Commparable` 接口中只有一个方法: `public int compareTa(0bject obj)`
    - `obj` 为要比较的对象
    - 方法中，将当的对象和 `obj` 这个对象进行比较，如果大于返回1，等于返回0，小于返回-1.(此处的1也可以是正整数，-1也可以是负整数)。
    - `compareTo` 方法的代码也比较固定，案例如下:
    
    ```java
    //按折扣进行比较
    public int compareTo(Object o) {
            Commodity good=(Commodity)o;
            if(this.discount>good.discount){
                return 1;
            }else if(this.discount<good.discount){
                return -1;
            }else
                return 0;
        }
    ```

# 常见算法
- 推荐学习算法网站：
  - https://visualgo.net/en
  - https://leetcode.cn/

## 一、冒泡排序算法
> 冒泡排序算法：重复地遍历要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来，这样越大的元素会经由交换慢慢“浮”到数列的顶端，
> 冒泡排序算法的运作如下:
> 1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个
> 2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
> 3. 针对所有的元家重复以上的步骤，除了最后一个
> 4. 持续每次对越来越少的元素重复上面的步照，直到没有任何一对数字需要比较

- 代码实现
```java
//冒泡排序算法实现（从小到大排序）
	public static void bubbleSort1(int[] a){
		int temp;
		for(int i=0;i<a.length;i++)
			for(int j=0;j<a.length-1;j++){
				if(a[j]>a[j+1]){
					temp=a[j];
					a[j]=a[j+1];
					a[j+1]=temp;
				}
			}
	}
```
- 算法优化
  1. 整个数列分成两部分:前面是无序数列，后面是有序数列
  2. 初始状态下，整个数列都是无序的，有序数列是空
  3. 每一趟循环可以让无序数列中最大数排到最后，(也就是说有序数列的元素个数增加1)，也就是不用再去顾及有序序列
  4. 每一趟循环都从数列的第一个元素开始进行比比较，依次比较相邻的两个元素，比较到无序数列的未尾即可(而不是数列的未尾);如果前一个大于后一个，交换
  5. 判断每一趟是否发生了数组元系的交换，如果没有发生，则说明此时数组已经有序
- 代码实现
  ```java
      public static void bubbleSort2(int[] a){
          int temp;//中间变量
          
          //外层for循环：n个元素排序至少需要n-1趟循环
          for(int i=0;i<a.length-1;i++){
              //定义一个布尔型变量，标记数组是否已达到有序状态
               boolean flag = true;
               
               //内层循环：相邻元素两两进行比较，比较到无序数组的最后
               //一次外层循环会把无序数组的最大值交换到无序数组的最后
              for(int j=0;j<a.length-1;j++){
                  //对于前面的无序数组执行if语句后会把flag赋值为false
                  //对于后面的有序数组将不会执行该if语句
                  if(a[j]>a[j+1]){//交换元素位置
                      temp=a[j];
                      a[j]=a[j+1];
                      a[j+1]=temp;
                      flag=false;//标记数组处于无序状态
                  }	
              }
              //flag为false是继续循环，直到flag为ture
              //当此处flag为true时，表明整个数组处于有序状态，则执行以下if语句，退出循环
              if(flag){
                  break;
              }
          }
      }
  ```

## 二、二分查找（折半查找）
> - 二分法检索(binary search)又称折半检索,这是一种高效的检索方式
> - 二分法检索的基本思想是设数组中的元素从小到大有序地存放在数组(array)中
> - 首先将给定值key与数组中间位置上元素的关建码(key)比较，如果相等，则检索成功
> - 否则，若 key 小，则在数组前半部分中继续进行二分法检索:
> - 若 key 大，则在数组后半部分中继续进行二分法检索。
> - 这样，经过一次比较就缩小一半的检素区间,如此进行下去，直到检索成功或检索失败
> - 注意：折半查找是在有序数组中查找的，所以必须先对数组进行排序


- 算法代码
  ```java
  import java.util.Arrays;
  
  public class Binary_Search {
      public static void main(String args[]){
          int[] a={18,5,36,22,23,18,11,12,24};
          Arrays.sort(a);
          System.out.println(Arrays.toString(a));
          System.out.println(binarySearch(a,23));//返回排序后23在数组中的索引位置
          
      }
  
      //折半查找
      public static int binarySearch(int[] array,int value){
          int left=0;
          int right=array.length-1;
          
          while (left<=right){
              int middle=(left+right)/2;
              
              if(value==array[middle]){
                  return middle;  //返回value的索引位置
              }
              if(value>array[middle]){
                  left=middle+1;
              }
              if(value<array[middle]){
                  right=middle-1;
              }
          }
          return -1;   //while循环完毕，说明为查找到value，返回-1
      }
  }
  ```
  
## 三、递归
- 递归是一种常见的解决问题的方法，即把问题逐渐简单化。递归的基本思想就是“自己调用自己”，一个使用递归技术的方法将会直接或者间接的调用自己
- 利用递归可以用简单的程序来解决一些复杂的问题。比如:斐波那契数列的计算、汉诺塔、快排等问题。
- 递归结构包括两个部分:
  1. 定义递归头
     - 解答:什么时候不调用自身方法。如果没有头，将陷入死循环，也就是递归的结束条件。
  2. 递归体
     - 解答:什么时候需要调用自身方法
- 递归的缺陷
  - 简单的程序是递归的优点之一。但是递归调用会占用大量的系统堆栈，内存耗用多，在递归调用层次多时速度要比循环慢的多，所以在使用递归时要慎重
  - 任何能用递归解决的问题也能使用选代解决。当递归方法可以更加自然地反映问题，并且易于理解和调试，并且不强调效率问题时，可以采用递归
  - 在要求高性能的情况下尽量避免使用递归，递归调用既花时间又耗内存
```java
/**
 * 使用递归求 n！和斐波那契数列
 * @author Lhk
 *
 */
public class DiGui_Test {
	public static void main(String[] args) {
		long d1=System.currentTimeMillis();
		System.out.println("20的阶乘："+factorial(20));
		long d2=System.currentTimeMillis();
		System.out.println("递归消耗时间："+(d2-d1));
		
		//循环求n！
		long d3=System.currentTimeMillis();
		long result=1;
		int n=20;
		while(n>1){
			result*=n;
			n--;
		}
		long d4=System.currentTimeMillis();
		System.out.println("20的阶乘："+result);
		System.out.println("循环消耗的时间："+(d4-d3));	
	
		long d5=System.currentTimeMillis();
		System.out.println("15的斐波那契数："+fibonacci(15));
		long d6=System.currentTimeMillis();
		System.out.println("fibonacci消耗的时间："+(d6-d5));	

	}
	
	
	//递归求n！
	public static long factorial(long n){
		if(n==1){//递归头
			return 1;
		}else{//递归体
			return n*factorial(n-1);
		}
	}
	
	//递归求斐波那契数列
	public static long fibonacci(int n){
		if(n==1){
			return 1;
		}
		else if(n==2){
			return 2;
		}
		else{
			return fibonacci(n-1)+fibonacci(n-2);
		}
		
	}

}
```

```java
import java.io.File;

/**
 * 使用递归算法打印目录树状结构
 * @author Lhk
 *
 */
public class DiGui_Test02 {
	public static void main(String[] args) {
		File f=new File("d:/movies");
		printFile(f,0);
	}
	
	/**
	 * 打印文件目录树状结构
	 * @param f
	 * @param level
	 */
	private static void printFile(File f, int level) {
		// TODO Auto-generated method stub
		//输出层次数
		for(int i=0;i<level;i++){
			System.out.print("-");//第几层就输出几个“-”
		}
		//输出文件名
		System.out.println(f.getName());
		//如果f是目录，则获取子文件列表，并对子文件进行相同操作
		if(f.isDirectory()){
			File[] files=f.listFiles();//listFiles()方法列出子文件和子目录
			
			for(File temp:files){
				//递归，对子目录和子文件进行输出，此处注意level+1
				printFile(temp,level+1);
			}
			
		}
		
	}

}
```