---
sidebar_position: 10
---

# File 类
- java.io.File 类:代表文件和目录，在开发中，读取文件、生成文件、删除文件、修改文件的属性时经常会用到本类
- File 类的常见构造方法:`public File(String pathname)`以 pathname 为路径创建 File 对象，如果 pathname 是相对路径，则默认的当前路径
- 常用方法:

  | 方法                         | 说明                                           |
  | ---------------------------- | ----------------------------------------------- |
  | public boolean exists()      | 判断 File 是否存在                             |
  | publlc boolean lsDlrectory() | 判断 File 是否是目录                           |
  | public boolean isFlle()      | 判断 File 是否是文件                           |
  | public long lastModified()   | 返団 File 最后修改时间                         |
  | public long length()         | 返回 File 大小                                 |
  | publlc String getName()      | 返回文件名                                     |
  | public String getPath()      | 返回文件的相对路径                             |
  | public String getAbsolutePath() | 返回 File 的绝对路径                          |
  | isHidden()                   | 判断 File 是否是隐藏文件                       |
  | createNewFile()              | 创建新的 File                                  |
  | delete()                     | 删除 File 对应的文件                           |
  | mkdir()                      | 创建一个目录;中间某个目录缺失，则创建失败       |
  | mkdirs()                     | 创建多个目录;中间某个目录缺失，则创建该缺失目录 |
  | isDirectory()                | 判断当前路径是否为目录                           |
  | getParentFile()              | 获取当前目录的父级目录                           |
  | list()                       | 返回一个字符串数组，包含目录中的文件和目录的路径名 |
  | listFiles()                  | 返回一个 File 数组，包含当前目录下的文件和目录 |

  ```java
      /**
      * 测试File类
      */
      import java.io.File;
      import java.io.IOException;
      import java.util.Date;

      public class File_Test01 {
      public static void main(String[] args) throws IOException {

            File f1=new File("d:/movies/刺杀小说家.mp4");//代表一个mp4文件
            File f2=new File("d:/movies");//代表文件目录
		
            System.out.println(System.getProperty("user.dir"));//项目的路径
            File f3=new File(System.getProperty("user.dir"));//f3代表项目文件目录
		
            f1.createNewFile();//创建空文件（需要创建好文件路径）
		
            System.out.println(f2.exists());//判断File对象是否存在
            System.out.println(f2.isDirectory());//判断File对象是否是目录
            System.out.println(f1.isFile());//判断File对象是否是文件
            System.out.println(new Date(f3.lastModified()));//File对象最后修改时间
            System.out.println(f3.length());//File对象的大小
            System.out.println(f3.getName());//获取File对象的文件名
            System.out.println(f1.getPath());//获取文件的目录路径

            File f4 = new File("d:/movies/java学习资料/java_se");
            System.out.println(f4.mkdir());//创建失败，返回false
            System.out.println(f4.mkdirs());//创建成功，返回true
        }
      }
  ```

  ```java
  /**
   * 测试文件操作方法
   */
  import java.io.File;
  import java.io.IOException;
  
  public class File_Demo1 {
      public static void main(String[] args) throws IOException {
          //创建File对象
          File file=new File("d:/movies/aa.txt");
          //创建文件
          System.out.println(file.createNewFile());
          //获取文件名字
          System.out.println(file.getName());
          //获取文件的相对路径
          System.out.println(file.getPath());
          //获取文件的绝对路径
          System.out.println(file.getAbsolutePath());
          //判断是否为文件
          System.out.println(file.isFile());
          //判断是否为隐藏文件
          System.out.println(file.isHidden());
          //删除文件
          System.out.println(file.delete());
          //判断file对象文件是否存在
          System.out.println(file.exists());
      }
  }
  ```
  
  ```java
  /**
   * 测试操作目录的方法
   */
  import java.io.File;
  
  public class Directory_Demo {
      public static void main(String[] args) {
          //创建File对象
          File file1=new File("d:/movies/demo1");
          //创建单级目录
          System.out.println(file1.mkdir());
          File file2=new File("d:/movies/demo2/a");
          //创建多级目录
          System.out.println(file2.mkdirs());
          //判断文件是否存在
          System.out.println(file2.exists());
          //判断路径是否为目录
          System.out.println(file2.isDirectory());
          //获取父级目录
          System.out.println(file2.getParent());
          System.out.println(file2.getParentFile().getName());
          //获取路径下所有的目录和文件
          File file=new File("d:/movies");
          String[] arr=file.list();//获取文件名和目录名
          for(String str:arr){
              System.out.println(str);
          }
  
          System.out.println("----------------");
          File[] arr2=file.listFiles();//获取文件和路径的路径
          for(File f:arr2){
              System.out.println(f);
          }
      }
  }
  ```
