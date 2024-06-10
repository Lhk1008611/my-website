---
sidebar_position: 19
---

# JDBC 核心技术

## 一、获取数据库连接
```java
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

/**
 * @author Lhk
 */
public class JDBC_connection {
    //连接方式1
    @Test
    public void connectiontest1() throws SQLException {
        //1.提供java.sql.Driver接口实现类的对象
        Driver driver = new com.mysql.jdbc.Driver();
        
        //2.提供url，指明具体操作的数据库
        //  url:http://localhost:8080/gmall/keyboard.jpg
        //  jdbc:mysql  :  协议:子协议
        //  localhost : ip地址
        //  3306 : 默认mysql的端口号
        //  test : test数据库
        String url = "jdbc:mysql://localhost:3306/test";

        //3.提供Properties的对象，指明用户名和密码
        Properties info = new Properties();
        info.setProperty("user", "root");
        info.setProperty("password","123456");

        //4.调用driver的connect()，获取连接
        Connection connection = driver.connect(url,info);
        System.out.println(connection);
    }

    //连接方式2：使用反射获取Driver对象，不会出现第三方的api，有更好的移植性
    @Test
    public void connectionTest2() throws ClassNotFoundException, InstantiationException, IllegalAccessException, SQLException {
        //1.使用java反射机制来提供java.sql.Driver接口实现类的对象
        Class clazz = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) clazz.newInstance();

        //2.提供url，指明具体操作的数据库
        String url = "jdbc:mysql://localhost:3306/test";

        //3.提供Properties的对象，指明用户名和密码
        Properties info = new Properties();
        info.setProperty("user","root");
        info.setProperty("password","123456");

        //4.调用driver的connect()，获取连接
        Connection connect = driver.connect(url, info);
        System.out.println(connect);
    }

    //连接方式3：使用DriverManager代替Driver（常用方式）
    @Test
    public void connectionTest3() throws Exception {
        //1.使用java反射机制来提供java.sql.Driver接口实现类的对象
        Class clazz = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) clazz.newInstance();

        //2.获取连接信息：url user pasaword
        String url="jdbc:mysql://localhost:3306/test";
        String user="root";
        String password="123456";

        //3.注册驱动
        DriverManager.registerDriver(driver);
        //4.获取连接
        Connection conn = DriverManager.getConnection(url, user, password);
        System.out.println(conn);
    }

    //连接方式4：只加载驱动，不用显示的注册驱动
    @Test
    public void connectionTest4() throws Exception{
        //1.获取连接信息：url  user  password
        String url="jdbc:mysql://localhost:3306/test";
        String user="root";
        String password="123456";

        //2.加载 Driver 类，会执行类中的静态代码块，即注册驱动
        /*
        * 在 com.mysql.jdbc.Driver 实现类中，声明了如下操作
        * static {
        *   try {
        *     DriverManager.registerDriver(new Driver());
        *   } catch (SQLException var1) {
        *     throw new RuntimeException("Can't register driver!");
        *   }
        * }
        */
        //在mysql中该句可省略，因为引入的包包含了该操作，但是不建议省略，原因是连接其他数据库时不能省略
        Class.forName("com.mysql.jdbc.Driver");

        //3.获取连接
        Connection conn = DriverManager.getConnection(url, user, password);
        System.out.println(conn);
    }

    //连接方式5(最终版)：将数据库连接需要的4个基本信息声明在配置文件中，通过读取配置文件的方式获取连接
    /*
     * 该方式的好处：
     * 1.实现了数据与代码的分离，实现了解耦
     * 2.如果需要修改配置文件信息，可以避免程序重新打包
     */
    @Test
    public void ConnectionTest5() throws IOException, ClassNotFoundException, SQLException {
        //1.读取配置文件中的4个基本信息
        InputStream resource = JDBC_connection.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros=new Properties();
        pros.load(resource);

        String url = pros.getProperty("url");
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String driverClass = pros.getProperty("driverClass");

        //2.加载 Driver 类
        Class.forName(driverClass);

        //3.获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println(connection);
    }
}
```

## 二、使用 PreparedStatement 实现 CRUD 操作
```java
import com.JDBC.lhk.Connection.JDBC_connection;
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

/**
 * 获取数据库连接和关闭资源
 * @author Lhk
 */
public class jdbcUtils {
    /**
     * 获取数据库连接
     * @return Connection
     * @throws Exception
     */
    public static Connection getConnection() throws Exception {
        //1.读取配置文件中的4个基本信息
        InputStream resource = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros=new Properties();
        pros.load(resource);
        String url = pros.getProperty("url");
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String driverClass = pros.getProperty("driverClass");

        //2.加载Driver类
        Class.forName(driverClass);

        //3.获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        return connection;
    }

    /**
     * 关闭Connection和PreparedStatement
     * @param connection
     * @param ps
     */
    public static void closeResource(Connection connection, Statement ps){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    public static void closeResource(Connection connection, Statement ps, ResultSet rs){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (rs!=null)
                rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    //通用的增删改方法
    public static int update(String sql, Object ...args)  {
        Connection coon = null;
        PreparedStatement ps = null;
        try {
            //1.获取数据库连接
            coon = jdbcUtils.getConnection();
            //2.预编译sql语句
            ps = coon.prepareStatement(sql);
            //3.填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //4.执行
            /**
             * ps.execute():
             * 如果执行的是查询操作,有返回结果，则此方法返回true;
             * 如果执行的是增、删、改操作，没有返回结果，则此方法返回false.
             */
            //ps.execute();
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            jdbcUtils.closeResource(coon,ps);
        }
        return 0;
    }
}
```

```java
import com.JDBC.lhk.JDBCutil.jdbcUtils;
import org.junit.Test;
import java.io.InputStream;
import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Properties;

/**
 * 使用PrepareStatement代替Statement，实现对数据库的增删改操作
 * prepareStatement可以解决拼串和sql注入问题
 * prepareStatement还可以操作Blob数据
 * 可以实现更高效的批量操作
 * @author Lhk
 *
 */
public class PrepareStatementUpdateTest {

    /**
     * 向account表插入数据
     */
    @Test
    public void insertTest() {
        Connection connection = null;
        PreparedStatement ps = null;
        try {
            //1.读取配置文件中的4个基本信息
            InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
            Properties pos = new Properties();
            pos.load(is);
            String url = pos.getProperty("url");
            String user = pos.getProperty("user");
            String password = pos.getProperty("password");
            String driverClass = pos.getProperty("driverClass");

            //2.加载Driver类
            Class<?> aClass = Class.forName(driverClass);

            //3.获取连接
           connection = DriverManager.getConnection(url, user, password);

            //4.预编译sql语句，返回PrepareStatement实例
            String sql="insert into account(username,balance,birth) values(?,?,?)";//?:占位符
            ps = connection.prepareStatement(sql);

            //5.填充占位符
            ps.setString(1,"lhk");
            ps.setDouble(2,15800);
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd");
            java.util.Date date = simpleDateFormat.parse("2000-04-22");
            ps.setDate(3,new Date(date.getTime()));

            //6.执行sql
            ps.execute();//execute:执行
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
        //7.资源关闭
            try {
                if (ps!=null)
                ps.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            try {
                if (connection!=null)
                connection.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }

    //修改account表的一条记录
    @Test
    public void modifyTest() {
        Connection coon = null;
        PreparedStatement ps = null;
        try {
            //1.获取数据库连接
            coon = jdbcUtils.getConnection();

            //2.预编译sql语句，返回PrepareStatement实例
            String sql="update account set username=?,balance=? where id=3";
            ps = coon.prepareStatement(sql);

            //3.填充占位符
            ps.setString(1,"LHK");
            ps.setDouble(2,16800);

            //4.执行
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //5.关闭资源
            jdbcUtils.closeResource(coon,ps);
        }
    }

    //通用的增删改方法
    public void update(String sql, Object ...args)  {
        Connection coon = null;
        PreparedStatement ps = null;
        try {
            //1.获取数据库连接
            coon = jdbcUtils.getConnection();

            //2.预编译sql语句
            ps = coon.prepareStatement(sql);

            //3.填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }

            //4.执行
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            jdbcUtils.closeResource(coon,ps);
        }
    }

    //测试通用的增删改方法
    @Test
    public void testCommonUpdate(){
        //删除
        //String sql="delete from account where id = ?";
        //update(sql,3);

        //修改
        //String sql="update stuinfo set sex=? where id=?";
        //update(sql,"f",1);

        //增加
        String sql="insert into stuinfo(id,name,sex) values(?,?,?)";
        update(sql,2,"Themutents","m");
    }
}
```

```java
import com.JDBC.lhk.JDBCutil.jdbcUtils;
import com.JDBC.lhk.bean.Dept2;
import org.junit.Test;
import java.lang.reflect.Field;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;

/**
 * 针对dept2表进行查询操作
 * @author Lhk
 */
public class dept2ForQuery {

    @Test
    public void test(){
        String sql="select department_id departmentId,department_name departmentName,manager_id managerId,location_id locationId from dept2 where department_id=?";
        Dept2 dept2 = dept2ForQuery(sql, 20);
        System.out.println(dept2);

        sql="select department_id departmentId,department_name departmentName from dept2 where department_id=?";
        Dept2 dept2_1 = dept2ForQuery(sql, 60);
        System.out.println(dept2_1);
    }
    
    /**
     * 针对dept2表的通用的查询方法
     *
     * 针对于表的字段名与类的属性名不相同的情况:
     * 1．必须声明sql时，使用类的属性名来命名字段的别名
     * 2．使用ResultSetMetaData时，需要使用getColumnLabel()来替换getColumnName( ),
     * 获取列的别名。
     * 说明:如果sql中没有给字段其别名，getColumnLabel()获取的就是列名
     */
    public Dept2 dept2ForQuery(String sql,Object ...args)  {
        Connection coon = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            coon = jdbcUtils.getConnection();

            //2.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);

            //3.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }

            //4.执行并返回结果集
            rs = ps.executeQuery();

            //5.处理结果集
            //5.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数

            //5.2处理结果集每一行数据的各个列
            if (rs.next()){
                Dept2 dept2 = new Dept2();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);

                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);

                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = Dept2.class.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(dept2,columnValue);
                }
                return dept2;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6.关闭资源
            jdbcUtils.closeResource(coon,ps,rs);
        }
        return null;
    }

    @Test
    public void testQuery() {
        Connection coon = null;
        PreparedStatement ps = null;
        ResultSet resultSet = null;
        try {
            //1.获取连接
            coon = jdbcUtils.getConnection();

            //2.预编译sql语句,并获取PrepareStatement实例
            String sql="select * from dept2 where department_id=?";
            ps = coon.prepareStatement(sql);

            //3.填充占位符
            ps.setObject(1,80);

            //4.执行并返回结果集
            resultSet = ps.executeQuery();

            //5.处理结果集
            //next():判断结果集的下一条是否有数据，有则返回true，且指针下移，否则返回false
            if (resultSet.next()) {
                int dept_id = resultSet.getInt(1);
                String dept_name = resultSet.getString(2);
                int manager_id = resultSet.getInt(3);
                int location_id = resultSet.getInt(4);
                Dept2 dept2 = new Dept2(dept_id, dept_name, manager_id, location_id);
                System.out.println(dept2);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6.关闭资源
            jdbcUtils.closeResource(coon,ps,resultSet);
        }
    }
}
```

```java
/**
 * 使用preparedStatement实现针对不同类的通用查询方法
 * @author Lhk
 */
public class PreparedSatementQuery {

    @Test
    public void test(){
        String sql="select department_id departmentId,department_name departmentName from dept2 where department_id=?";
        Dept2 d = query(Dept2.class, sql, 50);
        System.out.println(d);

        sql="select * from account where id=?";
        Account a = query(Account.class, sql, 1);
        System.out.println(a);

        System.out.println("----------------------------");
        sql="select * from account";
        List<Account> list1 = getQuery(Account.class, sql);
        list1.forEach(System.out::println);

        sql="select department_id departmentId,department_name departmentName from dept2 where department_id<?";
        List<Dept2> list2 = getQuery(Dept2.class, sql, 100);
        list2.forEach(System.out::println);
    }

    /**
     * 查询一条记录的通用方法
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
    public <T>T query(Class<T> clazz,String sql,Object ...args){
        Connection coon = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            coon = jdbcUtils.getConnection();

            //2.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);

            //3.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }

            //4.执行并返回结果集
            rs = ps.executeQuery();

            //5.处理结果集
            //5.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数

            //5.2处理结果集每一行数据的各个列
            if (rs.next()){
                T t = clazz.newInstance();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);

                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);

                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t,columnValue);
                }
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6.关闭资源
            jdbcUtils.closeResource(coon,ps,rs);
        }
        return null;
    }

    /**
     * 查询多条记录的方法
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
    public <T> List<T> getQuery(Class<T> clazz, String sql, Object ...args) {
        Connection coon = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            coon = jdbcUtils.getConnection();
            //2.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);
            //3.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //4.执行并返回结果集
            rs = ps.executeQuery();
            
            //5.处理结果集
            //5.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数
            //创建集合对象
            ArrayList<T> list = new ArrayList<>();
            //5.2处理结果集每一行数据的各个列，给指定的每个t对象赋值
            while (rs.next()){
                T t = clazz.newInstance();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t,columnValue);
                }
                list.add(t);
            }
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6.关闭资源
            jdbcUtils.closeResource(coon,ps,rs);
        }
        return null;
    }
}
```

### 操作 Blob 类型数据
- 如果在指定了相关的 Blob 类型以后，还报错：`xxx too large`
  - 那么在 mysql 的安装目录下，找 my.ini 文件加上如下的配置参数： `max_allowed_packet=16M`
  - 同时注意：修改了 my.ini 文件之后，需要重新启动 mysql 服务

```java
import com.JDBC.lhk.JDBCutil.jdbcUtils;
import org.junit.Test;
import java.io.*;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

/**
 * 使用PrepareStatement操作Blob数据
 * @author Lhk
 */
public class BlobTest {
    /**
     * 向beauty表中插入blob数据
     * 修改mysql中的全局变量：SET GLOBAL max_allowed_packet=16777216;
     * @throws Exception
     */
    @Test
    public void test()  {
        Connection connection = null;
        PreparedStatement ps = null;
        FileInputStream is=null;
        try {
            connection = jdbcUtils.getConnection();
            String sql="insert into beauty(name,sex,borndate,phone,photo,boyfriend_id) values(?,?,?,?,?,?) ";
            ps = connection.prepareStatement(sql);
            ps.setObject(1,"MM");
            ps.setObject(2,"f");
            ps.setObject(3,"1998-05-06");
            ps.setObject(4,"15677775555");
            is=new FileInputStream(new File("xc.png"));
            ps.setBlob(5,is);
            ps.setInt(6,3);
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (is!=null){
                is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            jdbcUtils.closeResource(connection,ps);
        }
    }

    /**
     * 查询beauty表的blob字段
     */
    @Test
    public void test1()  {
        Connection connection = null;
        PreparedStatement ps = null;
        InputStream is=null;
        FileOutputStream fos=null;
        try {
            connection = jdbcUtils.getConnection();
            String sql="select photo from beauty where id=?";
            ps = connection.prepareStatement(sql);
            ps.setInt(1,27);
            ResultSet rs = ps.executeQuery();
            if (rs.next()){
                is = rs.getAsciiStream("photo");
                fos = new FileOutputStream(new File("xiaochou.png"));
                int i=0;
                byte[] buff=new byte[1024];
                while((i= is.read(buff))!=-1){
                   fos.write(buff,0, i);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (is!=null){
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (fos!=null){
                fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        jdbcUtils.closeResource(connection,ps);
    }
}
```

## 三、批量插入
```java
import com.JDBC.lhk.JDBCutil.jdbcUtils;
import org.junit.Test;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.Statement;

/**
 * 实现批量插入操作,使用PreparedStatement实现更高效的批量插入
 * update、 delete本身就具有批量操作的效果。
 * 此时的批量操作，主要指的是批量插入。
 * 向goods表中插入20000条数据
 * @author Lhk
 */
public class InsertTest {
    //方式1：使用Statement实现批量插入操作
    @Test
    public void test1() throws Exception {
        long start = System.currentTimeMillis();
        Connection connection = jdbcUtils.getConnection();
        Statement sm = connection.createStatement();
        for (int i=1;i<=20000;i++){
            String sql="insert into goods(name) values('name_+"+i+"')";
            sm.execute(sql);
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间："+(end-start)+"ms");//花费时间：19844ms
        jdbcUtils.closeResource(connection,sm);
    }

    //方式2：使用PrepareStatement实现批量插入操作
    @Test
    public void test2() throws Exception {
        long start = System.currentTimeMillis();
        Connection connection = jdbcUtils.getConnection();
        String sql="insert into goods(name) values(?)";
        PreparedStatement ps = connection.prepareStatement(sql);
        for (int i=1;i<=20000;i++){
            ps.setString(1,"name_"+i);
            ps.executeUpdate();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间："+(end-start)+"ms");//花费时间：19802ms
        jdbcUtils.closeResource(connection,ps);
    }

    //方式3
    /*
     * 修改1：使用 addBatch() / executeBatch() / clearBatch()
     * 修改2：mysql服务器默认是关闭批处理的，我们需要通过一个参数，让mysql开启批处理的支持。
     * 		 ?rewriteBatchedStatements=true 写在配置文件jdbc.properties的url后面
     * 修改3：使用更新的mysql 驱动：mysql-connector-java-5.1.37-bin.jar
     *
     */
    @Test
    public void test3()  {
        Connection connection = null;
        PreparedStatement ps = null;
        try {
            long start = System.currentTimeMillis();
            connection = jdbcUtils.getConnection();
            String sql="insert into goods(name) values(?)";
            ps = connection.prepareStatement(sql);
            for (int i=1;i<=20000;i++){
                ps.setString(1,"name_"+i);
                //1.攒sql
                ps.addBatch();
                if(i%500==0){//攒500条sql
                    //2.执行
                    ps.executeBatch();
                    //3.清空Batch
                    ps.clearBatch();
                }
            }
            long end = System.currentTimeMillis();
            System.out.println("花费时间："+(end-start)+"ms");//花费时间：682ms
        } catch (Exception e) {
            e.printStackTrace();
        }
        jdbcUtils.closeResource(connection,ps);
    }

    //方式4
    /*
     * 在方式3的基础上操作
     * 使用 Connection 的 setAutoCommit(false) / commit() 取消自动提交
     */
    @Test
    public void test4() throws Exception {
        long start = System.currentTimeMillis();
        Connection connection = jdbcUtils.getConnection();

        //取消自动提交
        connection.setAutoCommit(false);

        String sql="insert into goods(name) values(?)";
        PreparedStatement ps = connection.prepareStatement(sql);
        for (int i=1;i<=20000;i++){
            ps.setString(1,"name_"+i);
            //1.攒sql
            ps.addBatch();
            if (i%500==0){
                //2.执行
                ps.executeBatch();
                //3.清空
                ps.clearBatch();
            }
        }
        //提交数据
        connection.commit();
        long end = System.currentTimeMillis();
        System.out.println("花费时间："+(end-start)+"ms");//花费时间：623ms
        jdbcUtils.closeResource(connection,ps);
    }
}
```

## 四、数据库事务
```java
import com.jdbc.lhk.Bean.Account;
import com.jdbc.lhk.utils.JdbcUtils;
import org.junit.Test;
import java.lang.reflect.Field;
import java.sql.*;

/**
 * 事务：一组逻辑操作单元,使数据从一种状态变换到另一种状态。
 *
 * 事务处理（事务操作）：保证所有事务都作为一个工作单元来执行，即使出现了故障，都不能改变这种执行方式。
 * 当在一个事务中执行多个操作时，要么所有的事务都被提交(commit),那么这些修改就永久地保存下来；要么数据
 * 库管理系统将放弃所作的所有修改，整个事务回滚(rollback)到最初状态
 *
 * 数据一旦提交，就不可回滚。
 *
 *4.哪些操作会导致数据的自动提交?
 * DDL操作一旦执行,都会自动提交。
 *      set autocommit = false对DDL操作失效
 * DML默认情况下，—旦执行,就会自动提交。
 *      >我们可以通过set autocommit = false的方式取消DML操作的自动提交。
 * 默认在关闭连接时，会自动的提交数据
 *
 * @author Lhk
 */
public class TransactionTest {
    /**
     * 不考虑事务的转账操作
     * 针对account表：
     * 张无忌给赵敏转账100
     *
     */
    @Test
    public void test(){
        String sql="update account set balance=balance-100 where username='?'";
        update(sql,"张无忌");

        //遇到异常（网络异常等等）
        System.out.println(10/0);

        String sql2="update account set balance=balance+100 where username='?'";
        update(sql2,"赵敏");
        System.out.println("转账成功");
    }

    //通用的增删改方法-----1.0
    public int update(String sql, Object ...args)  {
        Connection coon = null;
        PreparedStatement ps = null;
        try {
            //1.获取数据库连接
            coon = JdbcUtils.getConnection();
            //2.预编译sql语句
            ps = coon.prepareStatement(sql);
            //3.填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //4.执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            JdbcUtils.closeResource(coon,ps);
        }
        return 0;
    }

    /**
     * 考虑事务后的转账操作
     * 针对account表：
     * 张无忌给赵敏转账100
     */
    @Test
    public void test1()  {
        Connection coon = null;
        try {
            coon = JdbcUtils.getConnection();
            //1.取消自动提交
            coon.setAutoCommit(false);
            
            String sql="update account set balance=balance-100 where username=?";
            update(coon,sql,"张无忌");

            //遇到异常（网络异常等等）
            System.out.println(10/0);

            String sql2="update account set balance=balance+100 where username=?";
            update(coon,sql2,"赵敏");
            System.out.println("转账成功");

            //2.提交数据
            coon.commit();
        } catch (Exception e) {
            e.printStackTrace();
            //3.遇到异常回滚数据
            try {
                coon.rollback();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        } finally {
            //恢复连接自动提交
            try {
                coon.setAutoCommit(true);
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            JdbcUtils.closeResource(coon,null);
        }
    }

    //通用的增删改方法-----2.0（考虑事务）
    public int update(Connection coon,String sql, Object ...args)  {
        PreparedStatement ps = null;
        try {
            //1.预编译sql语句
            ps = coon.prepareStatement(sql);
            //2.填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //3.执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //4.关闭资源,这里不关闭连接；默认在关闭连接时，会自动的提交数据
            JdbcUtils.closeResource(null,ps);
        }
        return 0;
    }

//***********************************************************************

    /**
     * 设置数据库的隔离级别
     * 查询记录
     */
    @Test
    public void test3() throws Exception {
        Connection coon= JdbcUtils.getConnection();
        //获取当前的连接的隔离级别（全局的隔离级别）
        System.out.println(coon.getTransactionIsolation());
        //设置当前数据库的隔离级别
        //coon.setTransactionIsolation();
        //取消自动提交
        coon.setAutoCommit(false);
        String sql="select id,username,balance from account where id=?";
        Account a1 = query(coon, Account.class, sql, 1);
        System.out.println(a1);
    }

    /*修改记录*/
    @Test
    public void test4() throws Exception {
        Connection coon = JdbcUtils.getConnection();
        //取消自动提交
        coon.setAutoCommit(false);
        String sql="update account set balance=? where id= ?";
        update(coon, sql, 1500, 1);
        Thread.sleep(15000);
        System.out.println("修改结束");
    }

    /**
     * 查询一条记录的通用方法---2.0  考虑了事务
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
    public <T>T query(Connection coon,Class<T> clazz,String sql,Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);
            //2.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //3.执行并返回结果集
            rs = ps.executeQuery();
            //4.处理结果集
            //4.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数
            //4.2处理结果集每一行数据的各个列
            if (rs.next()){
                T t = clazz.newInstance();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t,columnValue);
                }
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源；默认在关闭连接时，会自动的提交数据
            JdbcUtils.closeResource(null,ps,rs);
        }
        return null;
    }
}
```

## 五、DAO 及其相关实现类
- DAO: 封装了针对数据表的通用操作

### BaseDao
```java
import com.jdbc.lhk.utils.JdbcUtils;
import java.lang.reflect.Field;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**
 * DAO:
 * 封装了针对数据表的通用操作
 * @author Lhk
 */
public abstract class BaseDao {

    //通用的增删改方法-----2.0（考虑事务）
    public int update(Connection coon, String sql, Object ...args)  {
        PreparedStatement ps = null;
        try {
            //1.预编译sql语句
            ps = coon.prepareStatement(sql);
            //2.填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //3.执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //4.关闭资源,这里不关闭连接
            JdbcUtils.closeResource(null,ps);
        }
        return 0;
    }

    /**
     * 查询一条记录的通用方法---2.0  考虑了事务
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
    public <T>T query(Connection coon,Class<T> clazz,String sql,Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);
            //2.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //3.执行并返回结果集
            rs = ps.executeQuery();
            //4.处理结果集
            //4.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数
            //4.2处理结果集每一行数据的各个列
            if (rs.next()){
                T t = clazz.newInstance();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t,columnValue);
                }
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            JdbcUtils.closeResource(null,ps,rs);
        }
        return null;
    }

    /**
     * 查询多条记录的方法----2.0(考虑上事务)
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
    public <T> List<T> getQuery(Connection coon,Class<T> clazz, String sql, Object ...args) {
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取PrepareStatement实例
            ps = coon.prepareStatement(sql);
            //2.填充在占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            //3.执行并返回结果集
            rs = ps.executeQuery();

            //4.处理结果集
            //4.1获取结果集的列数
            ResultSetMetaData rsmd = rs.getMetaData();//获取结果集的元数据：即解释结果集的相关数据
            int columnCount = rsmd.getColumnCount();//获取结果集的列数
            //创建集合对象
            ArrayList<T> list = new ArrayList<>();
            //4.2处理结果集每一行数据的各个列，给指定的每个t对象赋值
            while (rs.next()){
                T t = clazz.newInstance();
                for (int i=0;i<columnCount;i++){
                    //获取每一列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //获取结果集每一列的列名
                    //String columnName = rsmd.getColumnName(i + 1);
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //通过反射机制给dept2对象指定的columnName属性，赋值为columnValue
                    //使用getColumnLabel()替换getColumnName(),来获取列的别名
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t,columnValue);
                }
                list.add(t);
            }
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6.关闭资源
            JdbcUtils.closeResource(null,ps,rs);
        }
        return null;
    }

    //用于查询特殊值的通用方法
    public <E> E getValue(Connection coon,String sql,Object ...args) {
        PreparedStatement ps=null;
        ResultSet rs=null;
        try {
             ps = coon.prepareStatement(sql);
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            if(rs.next()){
                return (E) rs.getObject(1);
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(null,ps,rs);
        }
        return null;
    }
}
```

### 实体类
```java
  /**
   * @author Lhk
   */
  public class Account {
    private int id ;
    private String username;
    private double balance;
  
    public Account() {
    }
  
    public Account(int id, String username, double balance) {
      this.id = id;
      this.username = username;
      this.balance = balance;
    }
  
    public int getId() {
      return id;
    }
  
    public void setId(int id) {
      this.id = id;
    }
  
    public String getUsername() {
      return username;
    }
  
    public void setUsername(String username) {
      this.username = username;
    }
  
    public double getBalance() {
      return balance;
    }
  
    public void setBalance(double balance) {
      this.balance = balance;
    }
  
    @Override
    public String toString() {
      return "Account{" +
      "id=" + id +
      ", username='" + username + '\'' +
      ", balance=" + balance +
      '}';
    }
  }
```

### 工具类
```java
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

/**
 * 获取数据库连接和关闭资源
 * @author Lhk
 */
public class JdbcUtils {
    /**
     * 获取数据库连接
     * @return Connection
     * @throws Exception
     */
    public static Connection getConnection() throws Exception {
        //1.读取配置文件中的4个基本信息
        InputStream resource = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros=new Properties();
        pros.load(resource);
        String url = pros.getProperty("url");
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String driverClass = pros.getProperty("driverClass");
        //2.加载Driver类
        Class.forName(driverClass);
        //3.获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        return connection;
    }

    /**
     * 关闭Connection和PreparedStatement
     * @param connection
     * @param ps
     */
    public static void closeResource(Connection connection, Statement ps){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    public static void closeResource(Connection connection, Statement ps, ResultSet rs){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (rs!=null)
                rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
```

### 针对于实体类对应的常用操作的 DAO 接口
```java
import com.jdbc.lhk.Bean.Account;
import java.sql.Connection;
import java.sql.Date;
import java.util.List;

/**
 * 此接口用于规范针对于account表的常常用操作
 * @author Lhk
 */
public interface AccountDao {

    /**
     * 将account对象添加到数据表中
     * @param coon
     * @param account
     */
    void insert(Connection coon, Account account);

    /**
     * 根据id删除表中的一条记录
     * @param coon
     * @param id
     */
    void deleteById(Connection coon,int id);

    /**
     * 根据内存中的account的对象修改表中的指定的记录
     * @param coon
     * @param account
     */
    void updateById(Connection coon,Account account);

    /**
     * 根据指定的id从查询得到对应的Account对象
     * @param coon
     * @param id
     */
    Account getAccountById(Connection coon,int id);

    /**
     * 查询表中的所有记录构成的集合
     * @param coon
     * @return
     */
    List<Account> getAll(Connection coon);

    /**
     * 返回表中的记录条数
     * @param coon
     * @return
     */
    Long getCount(Connection coon);
}
```

### DAO 接口实现类
```java
import com.jdbc.lhk.Bean.Account;
import java.sql.Connection;
import java.util.List;

/**
 * @author Lhk
 */
public class AccountDaoImpl extends BaseDao implements AccountDao{
    @Override
    public void insert(Connection coon, Account account) {
        String sql ="insert into account(username,balance) values(?,?)";
        update(coon,sql,account.getUsername(),account.getBalance());
    }

    @Override
    public void deleteById(Connection coon, int id) {
        String sql="delete from account where id=?";
        update(coon,sql,id);
    }

    @Override
    public void updateById(Connection coon, Account account) {
        String sql="update account set username=?,balance=? where id=?";
        update(coon,sql,account.getUsername(),account.getBalance(),account.getId());
    }

    @Override
    public Account getAccountById(Connection coon, int id) {
        String sql="select id,username,balance from account where id=?";
        Account query = query(coon, Account.class, sql, id);
        return query;
    }

    @Override
    public List<Account> getAll(Connection coon) {
        String sql="select id,username,balance from account";
        List<Account> query = getQuery(coon, Account.class, sql);
        return query;
    }

    @Override
    public Long getCount(Connection coon) {
        String sql="select count(*) from account";
        return getValue(coon, sql);
    }
}
```

### 测试
```java
import com.jdbc.lhk.Bean.Account;
import com.jdbc.lhk.DAO.AccountDaoImpl;
import com.jdbc.lhk.utils.JdbcUtils;
import org.junit.jupiter.api.Test;
import java.sql.Connection;
import java.util.List;

/**
 * @author Lhk
 */
class AccountDaoImplTest {

    AccountDaoImpl accountDaoImpl=new  AccountDaoImpl();

    @Test
    void insert(){
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            Account account = new Account(3,"lhk",15000);
            accountDaoImpl.insert(connection,account);
            System.out.println("添加成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }

    @Test
    void deleteById() {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            accountDaoImpl.deleteById(connection,3);
            System.out.println("删除成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }

    @Test
    void updateById() {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            Account account = new Account(1,"无忌",15000);
            accountDaoImpl.updateById(connection,account);
            System.out.println("修改成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }

    @Test
    void getAccountById() {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            Account account = accountDaoImpl.getAccountById(connection, 1);
            System.out.println(account);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }

    @Test
    void getAll() {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            List<Account> all = accountDaoImpl.getAll(connection);
            all.forEach(System.out::println);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }

    @Test
    void getCount() {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            Long count = accountDaoImpl.getCount(connection);
            System.out.println("account表中共有"+count+"条记录");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(connection,null);
        }
    }
}
```

## 六、数据库连接池
### C3P0 数据库连接池技术
- C3P0 配置文件 `c3p0-config.xml`
  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <c3p0-config>
  
      <named-config name="MyC3P0">
          <!-- 提供获取连接的4个基本信息  -->
          <property name="driverClass">com.mysql.jdbc.Driver</property>
          <property name="jdbcUrl">jdbc:mysql://localhost:3306/test</property>
          <property name="user">root</property>
          <property name="password">123456</property>
  
          <!--  进行数据库管理的基本信息    -->
          <!-- acquireIncrement：当数据库连接池连接数不够时，c3p0一次性向数据库服务器申请的连接数     -->
          <property name="acquireIncrement">5</property>
          <!--  initialPoolSize：c3p0数据库连接池初始化时的连接数    -->
          <property name="initialPoolSize">10</property>
          <!-- minPoolSize：c3p0数据库连接池维护的最少的连接数    -->
          <property name="minPoolSize">10</property>
          <!-- maxPoolSize：c3p0数据库连接池维护的最多的连接数     -->
          <property name="maxPoolSize">100</property>
  
          <!-- maxStatements：c3p0数据库连接池维护的最多的Statement的个数     -->
          <property name="maxStatements">50</property>
          <!-- maxStatementsPerConnection：每个连接中可以最多使用的Statement的个数     -->
          <property name="maxStatementsPerConnection">2</property>
  
      </named-config>
  </c3p0-config>
  ```

```java
import com.mchange.v2.c3p0.ComboPooledDataSource;
import com.mchange.v2.c3p0.DataSources;
import org.junit.Test;
import java.beans.PropertyVetoException;
import java.sql.Connection;
import java.sql.SQLException;

/**
 * 通过 C3P0 数据库连接池获取数据库连接
 * @author Lhk
 */
public class C3P0 {
        //方式一：
        @Test
        public void testGetConnection() throws PropertyVetoException, SQLException {
                //获取C3P0数据库连接池
                ComboPooledDataSource cpds = new ComboPooledDataSource();
                cpds.setDriverClass( "com.mysql.jdbc.Driver" ); //loads the jdbc driver
                cpds.setJdbcUrl("jdbc:mysql://localhost:3306/test");
                cpds.setUser("root");
                cpds.setPassword("123456");
                //通过设置相关参数，对数据库连接池进行管理  
                //设置初始的数据库连接池中的连接数
                cpds.setInitialPoolSize(10);
                Connection coon = cpds.getConnection();
                System.out.println(coon);
                //销毁c3p0连接池
                DataSources.destroy(cpds);
        }

        //方式二：使用配置文件获取数据库连接池
        @Test
        public void testGetConnection1() throws SQLException {
                ComboPooledDataSource cpds = new ComboPooledDataSource("MyC3P0");
                Connection conn = cpds.getConnection();
                System.out.println(conn);
                DataSources.destroy( cpds );
        }
}
```

### DBCP 数据库连接池技术
- DBCP 配置文件 `dbcp.properties`
  ```properties
  #基本配置信息
  driverClassName=com.mysql.jdbc.Driver
  url=jdbc:mysql:///test
  username=root
  password=123456
  
  #设置其他管理数据库连接池的相关属性
  initialSize=10
  ```

```java
import org.apache.commons.dbcp.BasicDataSource;
import org.apache.commons.dbcp.BasicDataSourceFactory;
import org.junit.Test;
import javax.sql.DataSource;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 通过 DBCP 数据库连接池技术获取数据库连接
 * @author Lhk
 */
public class DBCP {

    /**
     * 获取DBCP连接池方式一（不推荐）
     * @throws SQLException
     */
    @Test
    public void testGetConnection() throws SQLException {
        //创建DBCP的数据库连接池
        BasicDataSource source=new BasicDataSource();
        //基本配置信息
        source.setDriverClassName("com.mysql.jdbc.Driver");
        source.setUrl("jdbc:mysql:///test");
        source.setUsername("root");
        source.setPassword("123456");
        //设置其他管理数据库连接池的相关属性
        source.setInitialSize(10);
        source.setMaxActive(10);
        Connection coon = source.getConnection();
        System.out.println(coon);
    }

    /**
     * 获取DBCP连接池方式二
     */
    private static Properties properties=null;
    static{
        try {
            //创建一个Properties对象
            properties = new Properties();
            //读取 dbcp.properties 文件中的配置信息
            //方式一
            //InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("dbcp.properties");
            //方式二
            FileInputStream is = new FileInputStream(new File("src/dbcp.properties"));
            //将流对象加载到Properties对象中
            properties.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void testGetConnection1() throws Exception {
        //创建DBCP的数据库连接池
        DataSource source = BasicDataSourceFactory.createDataSource(properties);
        Connection coon = source.getConnection();
        System.out.println(coon);
    }
}
```

### Druid 数据库连接池技术
- Druid 配置文件 `druid.properties`
  ```properties
  #基本配置信息
  driverClassName=com.mysql.jdbc.Driver
  url=jdbc:mysql:///test
  username=root
  password=123456
  
  #设置其他管理数据库连接池的相关属性
  initialSize=10
  maxActive=10
  ```

```java
import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.pool.DruidDataSourceFactory;
import org.junit.Test;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 通过 Druid 数据库连接池技术获取数据库连接
 * @author Lhk
 */
public class Druid {
    @Test
    public void getConnection() throws Exception {
        Properties properties=new Properties();
        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");
        properties.load(is);
        DataSource source = DruidDataSourceFactory.createDataSource(properties);
        Connection conn = source.getConnection();
        System.out.println(conn);
    }
}
```

### 数据库连接池工具类
```java
import com.alibaba.druid.pool.DruidDataSourceFactory;
import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.commons.dbcp.BasicDataSourceFactory;
import javax.sql.DataSource;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

/**
 * 获取数据库连接和关闭资源
 * @author Lhk
 */
public class JdbcUtils {

    //C3P0，数据库连接池只需提供一个即可
    private static ComboPooledDataSource cpds = new ComboPooledDataSource("MyC3P0");
    /**
     * 使用C3PO的数据库连接技术
     * @return
     * @throws SQLException
     */
    public static Connection getConnection1() throws SQLException {
        Connection conn = cpds.getConnection();
        return conn;
    }

    /**
     * 使用DBCP的数据库连接技术
     * @return
     * @throws Exception
     */
    private static Properties properties=null;
    private static DataSource source=null;
    static {
        try {
            properties = new Properties();
            FileInputStream is = new FileInputStream(new File("src/dbcp.properties"));
            //将流对象加载到Properties对象中
            properties.load(is);
            source = BasicDataSourceFactory.createDataSource(properties);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection2() throws Exception {
        Connection coon = source.getConnection();
        return coon;
    }

    /**
     * 使用 Druid 的数据库连接技术
     * @return
     */
    private static DataSource source1;
    static {
        try {
            Properties properties=new Properties();
            InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");
            properties.load(is);
            source1 = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Connection getConnection3() throws SQLException {
        Connection conn = source1.getConnection();
        return conn;
    }

    /**
     * 关闭Connection和PreparedStatement
     * @param connection
     * @param ps
     */
    public static void closeResource(Connection connection, Statement ps){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    public static void closeResource(Connection connection, Statement ps, ResultSet rs){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (rs!=null)
                rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
```

## 七、Apache-DBUtils 实现 CRUD 操作
- commons-dbutils 是 Apache 组织提供的一个开源 JDBC 工具类库
  - 它封装了针对数据库的增删改查操作，学习成本极低，并且使用 dbutils 能极大简化 jdbc 编码的工作量，同时也不会影响程序的性能

### 工具类
```java
import com.alibaba.druid.pool.DruidDataSourceFactory;
import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.commons.dbcp.BasicDataSourceFactory;
import org.apache.commons.dbutils.DbUtils;
import javax.sql.DataSource;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

/**
 * DbUtils
 * @author Lhk
 */
public class JdbcUtils {
    /**
     * 使用C3PO的数据库连接技术
     * @return
     * @throws SQLException
     */
    //数据库连接池只需提供一个即可
    private static ComboPooledDataSource cpds = new ComboPooledDataSource("MyC3P0");
    public static Connection getConnection1() throws SQLException {
        Connection conn = cpds.getConnection();
        return conn;
    }

    /**
     * 使用DBCP的数据库连接技术
     * @return
     * @throws Exception
     */
    private static Properties properties=null;
    private static DataSource source=null;
    static {
        try {
            properties = new Properties();
            FileInputStream is = new FileInputStream(new File("src/dbcp.properties"));
            //将流对象加载到Properties对象中
            properties.load(is);
            source = BasicDataSourceFactory.createDataSource(properties);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection2() throws Exception {
        Connection coon = source.getConnection();
        return coon;
    }

    /**
     * 使用Druid的数据库连接技术
     * @return
     */
    private static DataSource source1;
    static {
        try {
            Properties properties=new Properties();
            InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");
            properties.load(is);
            source1 = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Connection getConnection3() throws SQLException {
        Connection conn = source1.getConnection();
        return conn;
    }

    /**
     * 关闭Connection和PreparedStatement
     * @param connection
     * @param ps
     */
    public static void closeResource(Connection connection, Statement ps){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    public static void closeResource(Connection connection, Statement ps, ResultSet rs){
        //资源关闭
        try {
            if (ps!=null)
                ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (rs!=null)
                rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 使用dbutils.jar中的DbUtils工具类，实现资源的关闭
     * @param connection
     * @param ps
     * @param rs
     */
    public static void closeResource1(Connection connection, Statement ps, ResultSet rs){
        //try {
        //    DbUtils.close(connection);
        //    DbUtils.close(rs);
        //    DbUtils.close(ps);
        //} catch (SQLException throwables) {
        //    throwables.printStackTrace();
        //}
        DbUtils.closeQuietly(rs);
        DbUtils.closeQuietly(ps);
        DbUtils.closeQuietly(connection);
    }
}
```

### 测试
```java
import com.jdbc.lhk.Bean.Account;
import com.jdbc.lhk2.utils.JdbcUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.ResultSetHandler;
import org.apache.commons.dbutils.handlers.*;
import org.junit.Test;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import java.util.Map;

/**
 * commons-dbutils 的使用
 * @author Lhk
 */
public class QueryRunnerTest {

    //使用dbutils测试插入操作
    @Test
    public void testInsert()  {
        Connection conn = null;
        try {
            QueryRunner queryRunner = new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="insert into account(username,balance) values(?,?)";
            int insertCount = queryRunner.update(conn, sql, "lhk", 12000);
            System.out.println("成功添加了"+insertCount+"条数据");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * BeanHandler:是ResultSetHandler接口的实现类，用于封装表中的一条记录
     */
    @Test
    public void query1() {
        Connection conn=null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select id,username,balance from account where id=?";
            BeanHandler<Account> handler = new BeanHandler<>(Account.class);
            Account account = queryRunner.query(conn, sql, handler, 4);
            System.out.println(account);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * BeanListHandler:是ResultSetHandler接口的实现类，用于封装表中的多条记录构成的集合。
     */
    @Test
    public void query2() {
        Connection conn = null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select id,username,balance from account where id<?";
            BeanListHandler<Account> handler = new BeanListHandler<>(Account.class);
            List<Account> accounts = queryRunner.query(conn, sql, handler, 3);
            accounts.forEach(System.out::println);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * MapHandler:是ResultSetHandler接口的实现类，对应表中的一条记录
     * 将字段及相应字段的值作为map中的key和value
     */
    @Test
    public void query3() {
        Connection conn = null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select id,username,balance from account where id=?";
            MapHandler handler = new MapHandler();
            Map<String, Object> map = queryRunner.query(conn, sql, handler, 4);
            System.out.println(map);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * MapListHandler:是ResultSetHandler接口的实现类，对应表中的多条记录，封装为集合
     * 将字段及相应字段的值作为map中的key和value，封装到list集合中
     */
    @Test
    public void query4() {
        Connection conn = null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select id,username,balance from account where id<?";
            MapListHandler handler = new MapListHandler();
            List<Map<String, Object>> list = queryRunner.query(conn, sql, handler, 3);
            list.forEach(System.out::println);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * ScalarHandler:用于查询特殊值
     */
    @Test
    public void query5() {
        Connection conn = null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select count(*) from account";
            ScalarHandler handler = new ScalarHandler();
            var count = queryRunner.query(conn, sql, handler);
            System.out.println(count);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }

    /**
     * 自定义ResultSetHandler的实现类
     */
    @Test
    public void query6() {
        Connection conn = null;
        try {
            QueryRunner queryRunner=new QueryRunner();
            conn = JdbcUtils.getConnection3();
            String sql="select id,username,balance from account where id=?";

            //匿名类实现ResultSetHandler，查询一条记录
            ResultSetHandler<Account> handler = new ResultSetHandler<>() {
                @Override
                public Account handle(ResultSet resultSet) throws SQLException {
                    if (resultSet.next()){
                        int id=resultSet.getInt("id");
                        String username=resultSet.getString("username");
                        Double balance=resultSet.getDouble("balance");
                        Account account = new Account(id, username, balance);
                        return account;
                    }
                    return null;
                }
            };
            Account account = queryRunner.query(conn, sql, handler, 4);
            System.out.println(account);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.closeResource(conn,null);
        }
    }
}
```