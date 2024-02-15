## JDBC自学讲义

### 初识JDBC

JDBC：Java DataBase Connectivity（Java语言连接数据库）

本质：SUN公司制定的一套接口（interface）

接口的调用者是程序员，实现者是数据库，实现的方式是**驱动**

驱动内附有实现接口的类，在数据库官网下载相关的jar包即可，因为各数据库实现方式不同，驱动也不同。



为什么要面向接口编程？

解耦合：降低程序耦合度，提高程序扩展力。

例子：多态机制就是典型的面向抽象编程。



### IDEA编程和记事本编程导入驱动的方法

* 记事本编程

创建环境变量classPath，指向驱动jar文件地址

* IDEA编程

[Intellij IDEA——配置MySQL的驱动程序_idea配置mysql驱动-CSDN博客](https://blog.csdn.net/fox372/article/details/109527407)

从第五步添加驱动开始看即可



### JDBC编程六步

* 注册驱动（告诉Java程序，要连接哪个品牌的数据库）

* 获取连接（表示JVM进程和数据库进程之间的通道打开了，这属于进程之间的通信，重量级的，使用完之后一定要关闭）

* 获取数据库操作对象（专门执行sql语句的对象）
* 执行sql语句
* 处理查询数据集
* 释放资源（Java和数据库属于进程间的通信）

```java
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Connection;
import java.sql.Statement;

public class JDBCTest01 {
	public static void main(String[] args) {
		Connection conn = null;
		Statement stmt = null;
		try{
			// 1、注册驱动
			Driver driver = new com.mysql.cj.jdbc.Driver(); // 多态，父类型引用指向子类型对象
			DriverManager.registerDriver(driver);
			// 2、获取连接,jdbc:mysql://IP:port/databasename?服务器时区&useSSL
			String url = "jdbc:mysql://127.0.0.1:3306/learn?serverTimezone=UTC&useSSL=false";
			String user = "root";
			String password = "111";
			conn = DriverManager.getConnection(url, user, password);
			System.out.println("数据库连接对象 = " + conn);
			// 3、获取数据库操作对象
			stmt = conn.createStatement();
			// 4、执行sql
			String sql = "insert into dept(deptno, dname, loc) values(50, '人事部', '北京')";
			// 专门执行DML语句中的(insert delete update)
			// 返回值是"影响数据库中的记录条数"
			int count = stmt.executeUpdate(sql);
			System.out.println(count == 1 ? "保存成功" : "保存失败");
			// 5、处理查询结果集
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			// 6、释放资源
			// 为了保证资源一定释放，在finally语句块中关闭资源
			// 并且要遵循从小到大依次关闭
			// 分别对其try..catch
			try{
				if(stmt != null) {
					stmt.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			try{
				if(conn != null) {
					conn.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}
```

参考文献：

[java.sql.SQLException: The server time zone value 的解决办法-CSDN博客](https://blog.csdn.net/qq_43322436/article/details/121389983)

[解决错误：javax.net.ssl.SSLException MESSAGE: closing inbound before receiving peer's close_notify和The server time zone value ‘?й???’ is unrecognized - 江上酒，故人倾 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zscwb/p/12515965.html)



### 运用反射机制注册驱动

```java
/*
	注册驱动的另一种方式（这种方式常用）
*/
import java.sql.*;

public class JDBCTest02 {
	public static void main (String[] args) {
		try{
			// 1、注册驱动
			// 这是注册驱动的第一种写法
			// DriverManager.registerDriver(new com.mysql.jdbc.Driver());
			// 注册驱动的第二种方式，更常用的
			// 为什么这种方式常用？因为参数是一个字符串，字符串可以写到xxx.properties文件中
			// 以下方法不需要接收返回值，因为我们只想用它的类加载动作
			Class.forName("com.mysql.cj.jdbc.Driver");
			// 2、获取连接
			Connection conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?serverTimezone=UTC&useSSL=false", "root", "111");
			System.out.println(conn);
		} catch (SQLException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}
```



### 从properties配置文件中获取资源

properties文件应和java文件处于同一目录下

```java
// 将连接数据库的所有信息配置到配置文件中
/*
	实际开发中不建议把连接数据库的信息写死到java程序中
*/
import java.sql.*;
import java.util.*;

public class JDBCTest03 {
	public static void main(String[] args) {

		// 使用资源绑定器绑定属性配置文件
		ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
		String driver = bundle.getString("driver");
		String url = bundle.getString("url");
		String user = bundle.getString("user");
		String password = bundle.getString("password");

		Connection conn = null;
		Statement stmt = null;
		try{
			// 1、注册驱动
			Class.forName("com.mysql.cj.jdbc.Driver");
			// 2、获取连接
			conn = DriverManager.getConnection(url, user, password);
			System.out.println("数据库连接对象 = " + conn);
			// 3、获取数据库操作对象
			stmt = conn.createStatement();
			// 4、执行sql
			String sql = "insert into dept(deptno, dname, loc) values(60, '秘书部', '北京')";
			// 专门执行DML语句中的(insert delete update)
			// 返回值是"影响数据库中的记录条数"
			int count = stmt.executeUpdate(sql);
			System.out.println(count == 1 ? "保存成功" : "保存失败");
			// 5、处理查询结果集
		} catch (SQLException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} finally {
			// 6、释放资源
			// 为了保证资源一定释放，在finally语句块中关闭资源
			// 并且要遵循从小到大依次关闭
			// 分别对其try..catch
			try{
				if(stmt != null) {
					stmt.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			try{
				if(conn != null) {
					conn.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}
```



### JDBC实现查询

```java
/*
	处理查询结果集（遍历结果集）
*/
import java.sql.*;
import java.util.*;

public class JDBCTest04 {
	public static void main(String[] args) {

		// 使用资源绑定器绑定属性配置文件
		ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
		String driver = bundle.getString("driver");
		String url = bundle.getString("url");
		String user = bundle.getString("user");
		String password = bundle.getString("password");

		Connection conn = null;
		Statement stmt = null;
		ResultSet rs = null;
		try{
			// 1、注册驱动
			Class.forName("com.mysql.cj.jdbc.Driver");
			// 2、获取连接
			conn = DriverManager.getConnection(url, user, password);
			System.out.println("数据库连接对象 = " + conn);
			// 3、获取数据库操作对象
			stmt = conn.createStatement();
			// 4、执行sql
			String sql = "select deptno as n, dname, loc from dept";
			// executeUpdate(insert/delete/update)返回int值
			// executeQuery(select)返回ResultSet值
			rs = stmt.executeQuery(sql);
			// 5、处理查询结果集
			// rs.next()使光标指向新一行，若新一行存在则返回true，否则返回false
			while(rs.next()) {
				// 以列的名字获取
				int deptno = rs.getInt("n");
				String name = rs.getString("dname");
				String loc = rs.getString("loc");
				System.out.println(deptno + ", " + name + ", " + loc);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} finally {
			// 6、释放资源
			// 为了保证资源一定释放，在finally语句块中关闭资源
			// 并且要遵循从小到大依次关闭
			// 分别对其try..catch
			try{
				if(rs != null) {
					rs.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			try{
				if(stmt != null) {
					stmt.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			try{
				if(conn != null) {
					conn.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}
```



### JDBC模拟用户登录界面及SQL注入问题

```java
import java.sql.*;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

/**
 1、需求：
    模拟用户登录界面功能的实现
 2、业务描述
    程序运行时，提供窗口填写用户名和密码
    用户输入后，提交，Java收集信息
    连接数据库检验是否合法
    合法，显示登录成功
    不合法，显示登录失败
 3、数据的准备
    PowerDesigner
    4、当前程序存在的问题
    用户名：
    fdsa
    密码：
    fdsa' or '1'='1
    登录成功
    这种现象被称为SQL注入，黑客经常使用
 5、导致这种SQL注入的根本原因是什么？
    用户输入的信息含有sql语句的关键字，并且这些关键字参与sql语句的编译过程
    导致sql语句的原意被扭曲，进而达到sql注入
    sql语句 + 4中的输入会变成：
    select * from t_user where loginName = 'fdsa' and loginPwd = 'fdsa' or '1'='1'
    where子句变成了恒成立，从而检索出数据库中所有的值
    rs.next()=true从而使得登录成功

 */
public class JDBCTest06 {
    public static void main(String[] args) {
        // 初始化一个界面
        Map<String, String> userLoginInfo = initUI();
        // 验证用户名和密码
        boolean loginSuccess = login(userLoginInfo);
        // 最后输出结果
        System.out.println(loginSuccess ? "登录成功" : "登录失败");
    }

    /**
     * 用户登录
     * @param userLoginInfo 用户登录信息
     * @return false表示失败，true表示成功
     */
    private static boolean login(Map<String, String> userLoginInfo) {
        // 打标记的意识
        boolean loginSuccess = false;
        // 单独定义变量
        String loginName = userLoginInfo.get("loginName");
        String loginPwd = userLoginInfo.get("loginPwd");
        // JDBC代码
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        try {
            // 1、注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 2、获取连接
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                    "serverTimezone=UTC&useSSL=false", "root", "111");
            // 3、获取数据库操作对象
            stmt = conn.createStatement();
            // 4、执行sql
            String sql = "select * from t_user where loginName = '"+loginName+"' and loginPwd = '"+loginPwd+"'";
            rs = stmt.executeQuery(sql);
            // 5、处理结果集
            if(rs.next()) {
                // 登录成功
                loginSuccess = true;
            }
        } catch (ClassNotFoundException | SQLException e) {
            throw new RuntimeException(e);
        } finally {
            try{
                if(rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try{
                if(stmt != null) {
                    stmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try{
                if(conn != null) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        // 6、释放资源
        return loginSuccess;
    }

    /**
     * 初始化用户界面
     * @return 用户输入的用户名和密码等登录信息
     */
    private static Map<String, String> initUI() {
        Scanner s = new Scanner(System.in);

        System.out.println("用户名：");
        String loginName = s.nextLine();

        System.out.println("密码：");
        String loginPwd = s.nextLine();
        Map<String, String> userLoginInfo = new HashMap<>();
        userLoginInfo.put("loginName", loginName);
        userLoginInfo.put("loginPwd", loginPwd);
        return userLoginInfo;
    }
}
```



### 解决SQL注入问题

```java
import java.sql.*;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

/**
 * 1、解决SQL注入问题
 *      只要用户提供的信息不参与SQL语句的编译过程，问题就解决了
 *      即使用户提供的信息含有SQL语句的关键字，但是没有参与编译，不起作用
 *      要想用户信息不参与SQL语句的编译，那么必须使用java.sql.PreparedStatement
 *      PreparedStatement接口继承了java.sql.Statement
 *      PreparedStatement是属于预编译的数据库操作对象
 *      PreparedStatement的原理是，预先对SQL语句的框架进行编译，然后再对SQL语句传值
 * 2、测试结果
 *      用户名：
 *      fdsa
 *      密码：
 *      fdsa' or '1'='1
 *      登录失败
 * 3、解决SQL注入的关键是什么
 *      用户提供的信息中即使含有sql语句的关键字，但是这些关键字没有参与编译，不起作用
 * 4、对比一下Statement和PreparedStatement
 *      Statement存在sql注入问题，PreparedStatement解决了SQL注入问题
 *      Statement是编译一次执行一次。PreparedStatement是编译一次，可执行N次
 *      PreparedStatement会在编译阶段做类型的安全检查
 *      综上所述：PreparedStatement使用较多，只有极少数情况下需要使用Statement
 * 5、什么情况下必须使用Statement
 *      业务方面要求必须支持SQL注入的时候
 *      Statement支持SQL注入，凡是业务方面要求是需要进行SQL语句拼接的，必须使用Statement
 */
public class JDBCTest07 {
    public static void main(String[] args) {
        // 初始化一个界面
        Map<String, String> userLoginInfo = initUI();
        // 验证用户名和密码
        boolean loginSuccess = login(userLoginInfo);
        // 最后输出结果
        System.out.println(loginSuccess ? "登录成功" : "登录失败");
    }

    /**
     * 用户登录
     *
     * @param userLoginInfo 用户登录信息
     * @return false表示失败，true表示成功
     */
    private static boolean login(Map<String, String> userLoginInfo) {
        // 打标记的意识
        boolean loginSuccess = false;
        // 单独定义变量
        String loginName = userLoginInfo.get("loginName");
        String loginPwd = userLoginInfo.get("loginPwd");
        // JDBC代码
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            // 1、注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 2、获取连接
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                    "serverTimezone=UTC&useSSL=false", "root", "111");
            // 一个?代表占位符，一个?接收一个值，占位符不能用单引号括起来
            String sql = "select * from t_user where loginName = ? and loginPwd = ?";
            // 3、获取预编译的数据库操作对象
            ps = conn.prepareStatement(sql);
            // 给占位符传值，第一个?下标为1，第二个占位符下标为2
            ps.setString(1, loginName);
            ps.setString(2, loginPwd);
            // 4、执行sql
            rs = ps.executeQuery();
            // 5、处理结果集
            if (rs.next()) {
                // 登录成功
                loginSuccess = true;
            }
        } catch (ClassNotFoundException | SQLException e) {
            throw new RuntimeException(e);
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (ps != null) {
                    ps.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn != null) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        // 6、释放资源
        return loginSuccess;
    }

    /**
     * 初始化用户界面
     *
     * @return 用户输入的用户名和密码等登录信息
     */
    private static Map<String, String> initUI() {
        Scanner s = new Scanner(System.in);

        System.out.println("用户名：");
        String loginName = s.nextLine();

        System.out.println("密码：");
        String loginPwd = s.nextLine();
        Map<String, String> userLoginInfo = new HashMap<>();
        userLoginInfo.put("loginName", loginName);
        userLoginInfo.put("loginPwd", loginPwd);
        return userLoginInfo;
    }
}

```



### Statement的使用场景

```java
import java.sql.*;
import java.util.Scanner;

/**
 * 演示必须使用Statement的情况
 * PreparedStatement在setString的时候会用单引号括住输入整个替代占位符。
 * 如：ps.setString(desc);   select * from t_user order by 'desc'
 * 这是不合法的，所以要用Statement
 */
public class JDBCTest08 {
    public static void main(String[] args) {
        // 用户在控制台输入desc就是降序，输入asc就是升序
        Scanner s = new Scanner(System.in);
        System.out.println("输入desc或asc，desc表示降序，asc表示升序");
        System.out.println("请输入：");
        String keyWords = s.nextLine();

        // 执行SQL
        Connection conn = null;
        Statement  stmt = null;
        ResultSet rs = null;
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                    "serverTimezone=UTC&useSSL=false", "root", "111");
            stmt = conn.createStatement();
            String sql = "select * from t_user order by loginName " + keyWords;
            rs = stmt.executeQuery(sql);
            while(rs.next()) {
                System.out.println(rs.getString("loginName"));
            }
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            if(rs != null) {
                try {
                    rs.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
            if(stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
            if(conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```



### PreparedStatement的插入操作

```java
import java.sql.*;
import java.util.Scanner;

public class JDBCTest09 {
    public static void main(String[] args) {
        // 执行SQL
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                    "serverTimezone=UTC&useSSL=false", "root", "111");
            String sql = "insert into dept(deptno,dname,loc) values(?,?,?)";
            ps = conn.prepareStatement(sql);
            ps.setInt(1,70);
            ps.setString(2, "销售部");
            ps.setString(3, "武汉");
            int count = ps.executeUpdate();
            System.out.println(count);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            if(ps != null) {
                try {
                    ps.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
            if(conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```



### JDBC事务机制

JDBC事务是自动提交的

只要执行一条DML语句，则自动提交一次。这是JDBC默认的事务行为。但是在实际的业务中，通常都是N条DML语句共同联合才能完成的，必须保证他们这些DML语句在同一个事物中同时成功或者同时失败。

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * sql脚本
 * drop table if exists t_act;
 * create table t_act(
 * actno int,
 * balance double(7,2) // 注意：7表示有效数字，2表示小数位的个数
 * );
 * insert into t_act(actno,balance) values(111,20000);
 * insert into t_act(actno,balance) values(222,0);
 * commit;
 * select * from t_act;
 */
public class JDBCTest10 {
    public static void main(String[] args) {
        // 执行SQL
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                    "serverTimezone=UTC&useSSL=false", "root", "111");
            // 将自动提交模式改为手动提交
            conn.setAutoCommit(false);
            String sql = "update t_act set balance = ? where actno = ?";
            ps = conn.prepareStatement(sql);

            // 用户222收到转账10000
            ps.setDouble(1, 10000);
            ps.setInt(2, 222);
            int count = ps.executeUpdate();

//            String s = null;
//            s.toString();

            // 用户111转出10000
            ps.setDouble(1, 10000);
            ps.setInt(2, 111);
            count += ps.executeUpdate();
            System.out.println(count == 2 ? "转账成功" : "转账失败");

            // 程序能够走到这里说明以上程序没有异常，事务结束，手动提交数据
            conn.commit(); // 提交事务
        } catch (Exception e) {
            // 回滚事务
            if(conn != null) {
                try {
                    conn.rollback();
                } catch (SQLException ex) {
                    throw new RuntimeException(ex);
                }
            }
            throw new RuntimeException(e);
        }  finally {
            if (ps != null) {
                try {
                    ps.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```



### 工具类DButil的开发

```java
package utils;

import java.sql.*;

/**
 * JDBC工具类，简化JDBC编程
 */
public class DBUtil {

    /**
     * 工具类中的构造方法都是私有的
     * 工具类当中的方法都是静态的，不需要new对象，直接采用类名调用
     */
    private DBUtil(){}

    // 静态代码块在类加载时执行，且只执行一次
    // 而该驱动程序也只需加载一次即可
    static {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 获取数据库连接对象
     * @return 连接对象
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/learn?" +
                "serverTimezone=UTC&useSSL=false", "root", "111");
    }

    /**
     * 关闭资源
     * @param conn 连接对象
     * @param ps 数据库操作对象
     * @param rs 结果集
     */
    public static void close(Connection conn, Statement ps, ResultSet rs) {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
        if(ps != null) {
            try {
                ps.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
        if(conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
    }
}

```

```java
import utils.DBUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 运用工具类DBUtil实现模糊查询
 */
public class JDBCTest12 {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            // 获取连接
            conn = DBUtil.getConnection();
            String sql = "select loginName from t_user where loginName like ?";
            ps = conn.prepareStatement(sql);
            ps.setString(1, "_a%");
            rs = ps.executeQuery();
            while(rs.next()) {
                System.out.println(rs.getString("loginName"));
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
    }
}

```



### 悲观锁和乐观锁的机制

悲观锁（行级锁）：事务必须排队执行。数据锁住了，不允许并发。（select语句最后加for update）

乐观锁：支持并发，事务不需要排队，但数据表每行数据都有一个版本号



乐观锁运行流程：

其中两个线程都同时试图修改数据，此时数据版本号为1.1

事务1先完成了修改，修改后的数据版本号变为1.2

事务2完成修改准备提交时，发现数据版本已经变为了1.2，和最初读到的版本不一致，于是回滚

```java
import utils.DBUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 这个程序开启一个事务，这个事务专门进行查询，并且使用行级锁/悲观锁，锁住相关记录
 */
public class JDBCTest13 {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            conn = DBUtil.getConnection();
            // 开启事务
            conn.setAutoCommit(false);

            String sql = "select loginName,loginPwd,realName from t_user where loginPwd = ? for update";
            ps = conn.prepareStatement(sql);
            ps.setString(1, "123");

            rs = ps.executeQuery();
            while (rs.next()) {
                System.out.println(rs.getString("loginName") + ", " + rs.getString("loginPwd")
                        + ", " + rs.getString("realName"));
            }

            // 提交事务(事务结束)
            conn.commit();
        } catch (SQLException e) {
            if(conn != null) {
                try {
                    conn.rollback();
                } catch (SQLException ex) {
                    throw new RuntimeException(ex);
                }
            }
            throw new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
    }
}

```

