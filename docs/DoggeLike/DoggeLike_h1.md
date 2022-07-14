# 将玩家数据添加到数据库

******************

<br>
为玩家建立一个专门的数据库，以更好得储存玩家在游玩过程中的各种数据，方便后来的分析。

该数据库中拥有多个表格，在此将其在数据库中建立：：

- 玩家表 Player
  - 包含玩家ID（主键）
  - 对应用户创建时间
  - 用户名
  - 密码

<br>

- 操作记录表 Operate
  - 玩家ID
  - 游玩次数
  - 操作记录

<br>

- 成绩表 Mark
  - 玩家ID
  - 单次游玩成绩
  - 根据上表得出的玩家综合成绩
    
<br>

在这里是使用MySQL作为数据库管理软件，所以需要先在计算机中安装该软件才能实现接下来的功能（然而使用JDBC API不需要mysql客户端）。

在这里以较新版本的MySQL8.0.29版本为例。在安装好后创建一个专门用于JAVA开发的数据库管理员JAVAdev。

具体的实现逻辑为：

首先由JAVA程序绑定到我们为其建立的JAVAdev管理员，然后借其手对我们的数据库进行一系列添加、删除、修改等操作。
逻辑上还是比较简单的。但是如何让程序与该管理员建立联系是关键问题，毕竟只要建立联系，后续数据的维护工作将全部交由我们的MySQL负责。
还好我们的MySQL官方为开发者准备了用于建立此联系的API: __mysql-connector-java__。

如果你是直接从空文档建立的项目，则需要去官网根据自己的数据库版本下载对应的API。
如果你是使用MAVEN建立的项目，只需要在 __pom.xml__ 中添加如下依赖：(记得同步项目依赖)

```xml
<dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.29</version>
        </dependency>
    </dependencies>
```

![logic](./Database_logic.png)

## 创建你的第一个连接

********************

该API的调用与我们所熟知的直接导入JAR包不同，在创建连接之前需要通过提供的驱动管理在主类中进行注册。

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
// Notice, do not import com.mysql.cj.jdbc.*
// or you will have problems!
public class LoadDriver {
 public static void main(String[] args) {
 try {
 // The newInstance() call is a work around for some
 // broken Java implementations
 Class.forName("com.mysql.cj.jdbc.Driver").newInstance();
 } catch (Exception ex) {
 // handle the error
 }
 }
}
```

然后才能够创建连接：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class  CreatConnection{
    public void Creat_connection(){
    Connection conn = null;
    try {
        //在这里通过访问MySQL URL的方式实现的连接
      conn = DriverManager.getConnection("jdbc:mysql://localhost/test?" + "user=minty&password=greatsqldb");
      // Do something with the Connection
    } catch (SQLException ex) {
      // handle any errors
      System.out.println("SQLException: " + ex.getMessage());
      System.out.println("SQLState: " + ex.getSQLState());
      System.out.println("VendorError: " + ex.getErrorCode());
    }
  }
}
```

用于连接MySQL的语法结构为：

*     protocol//[hosts:prots][/database][?properties]
注意：这里的URL格式比较严格，任何特殊符号（除上面示例之外的字符）需要按照直链格式转码

<br>

********************

## 建立连接后的操作

建立连接后，可以通过 *Statement* 对象来执行数据库操作，其返回的结果将会储存在 *ResultSet* 类中。

```java
public void test{
    Connection con = null;
        con = DriverManager.getConnection("jdbc:mysql://localhost/test?" + "user=minty&password=greatsqldb");
        //通过.creatStatement()方法返回一个Statement类。
        Statement sta = con.creatStatement();
        }
```

```xml
    然后通过execute(String SQL)方法进行更新，嵌入，删除操作；

    其中如果进行SELECT请求操作，可通过调用getReslktSet()方法获取数据,或者通过新建类ResultSet来接收；

    其他操作可通过调用getUpdateCount()方法获取操作成功的行数；

    记得操作完成后通过.close()方法释放Connection对象和Statement对象。
```