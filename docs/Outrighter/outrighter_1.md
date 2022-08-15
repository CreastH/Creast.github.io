# 登录认证

******

## 具体步骤：

![login_logic.png](login_logic.png)

### 登录页面

***********

可以直接百度登录页面模板，选择一个自己喜欢的，但是精美的登录
页面模板通常会有` .css `文件，该文件在像素层面定义了页面各个元素的布局和色彩
可以通过适当修改得到自己想要的结果。

![img.png](login_page.png)

### 登录信息

****************

从用户页面获得数据的方式有很多种，可以通过网页URL参数获取，也可以通过JavaScript脚本执行各种指令
这里使用的是通过页面渲染引擎 thymeleaf 动态获取数据，想要使用这一功能，需要导入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

并且在头元素中加入如下语句，开启功能：
```html
<html xmlns:th="http://www.thymeleaf.org">
```

- #### 提交表单信息

这一步是在网页中实现的，编辑模式打开网页模板，会看到 form 元素
```html
<form method="post">
    <label color=x000f th:text="${error_name}">pp</label>
    <input type="text" th:name="userid" th:value="用户名" onfocus="this.value = '';" onblur="if (this.value == '') {this.value = '用户名';}">
    <input type="password" th:name="psw" th:value="PASSWORD" onfocus="this.value = '';" onblur="if (this.value == '') {this.value = 'PASSWORD';}">
    <div class="button-row">
        <input type="submit" class="sign-in" value="登录">
        <input type="button" class="register" onclick="location.href='/register.html" value="注册">
        <div class="clear"></div>
    </div>
</form>
```

    只有在 templates 文件夹中的页面才会调用 thymeleaf 引擎

这里使用` post `方法来提交表单，好处是该请求是隐式的，只是用户的账户和密码不会以明文方式显示在URL中。

在属性变量前面添加 `th: ` 是为了获取，或者改变该属性内的值。具体内容可参考 thymeleaf 官网。


<br>

- ##### 后台接收表单信息

由于第一次访问，在不经过 springboot 应用的情况下是访问的静态资源目录。
因此不需要特地去接收 `"/"` 路径下的访问，他会默认访问 index.html 页面。

在这里，提交表单后使用` @PostMapping` 来接收表单，而不是通过 @RestMapping 注解，
这是因为后者默认处理 `get` 类型请求，因此会处理我们第一次访问页面（”/“）时的数据，
此时的账户和密码都为空，会提示用户输入登录信息，不够友好。

使用` redirect: `是为了重定向页.面，如果不使用，那么登录失败后再次登录，
提交表单的路径依然时（"/"）。重定向可以避免登录失败时无法提示正确的错误信息。

```java
@Controller
public class AccLogin {
    //第一次登录的为静态页面，没有被thymeleaf渲染
    @PostMapping(value = {"/","/login"})
    public String loginjudge(@RequestParam("userid") String name,
                             @RequestParam("psw") String pswd,
                             Model module,
                             HttpSession session,
                             Map<String ,Object> mapList){
        //查看页面的输入信息是否为空
        if(name.equals("用户名") || pswd.equals("PASSWORD")) {
            if (name.equals("用户名") ) {
                module.addAttribute("error_name", "请输入用户名！");
            }
            if (pswd.equals("PASSWORD")) {
                module.addAttribute("error_pswd", "请输入密码！");
            } else{
                module.addAttribute("error_pswd", "请输入用户名和密码！");
            }
            return "redirect:/login";
        }
        //登录正确返回TRUE 否则返回到根目录的由thymeleaf渲染的登录页面
        //DATASOURCE是在下面语句执行之前就注入的，因此未查询到数据会报错
        if(dataAcc.ConnectTest_name(name, mapList)){
            session.setAttribute("token","testtrue");
            return "redirect:/acc";
        }else {
            module.addAttribute("error_name", "账户或密码错误！");
            //硬重定向
            //重定向属于二次请求
            return "redirect:/login";
        }
    }

    //二次重定向请求处理
    @GetMapping("/login")
    public String login_page(){
        return "/login";
    }
}
```
    可以不使用 @RequstParam 注解来获取表单中数值，添加只是提高了代码可读性。

可以注意到，代码中关于 **“查看页面的输入信息是否为空”** 的地方并没有进行空值判断。

这是因为默认情况下， **用户名** 和 **密码** 的值就是 _用户名_ 和 _PASSWORD_ .

- ##### 登录认证

这里需要访问数据库，是要调用其他外部依赖的，Spring 贴心的提供了自带的 **JdbcTemplate** 
来解决该问题，但是他并不能完美兼容各种复杂操作。（注意mysql5.x使用的驱动和mysql8.x不同）
反而是 **Mybaits** 等第三方库可以提供更好的解决方案。

然而这些工具其实都是基于官方提供的 **JDBC** 实现的。

##### 连结数据库

使用官方正统的 mysql 操作工具需要加入依赖：

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
<!--            <version>自己的mysql版本</version>-->
        </dependency>
```

然后加载驱动：

```java
try {
            // The newInstance() call is a work around for some
            // broken Java implementations
            Class.forName("com.mysql.cj.jdbc.Driver").newInstance();
            System.out.println("Driver actived!!!!");
        } catch (Exception ex) {
            // handle the error
        }
```
    这个驱动需要在实现数据库连结之前加载完毕，否则提示找不到合适驱动。

在配置类中实现连结，并将该连结注入到IOC容器：

```java
//初始化连
@Configuration(proxyBeanMethods = false)
public class initDataCon{

    //将Map注入到容器中
    @Bean
    public Map<String,Object> initCon() {
        Connection connection=null;
        Map<String,Object> map = new HashMap<>();

        String URL = "jdbc:mysql://yourhost:3306/Yourdatabase?" + "user=yourname&password=yourpswd";
        try {
            //在这里通过访问MySQL URL的方式实现的连接
            connection = DriverManager.getConnection(URL);
            map.put("connection",connection);
        } catch (SQLException ex) {
            // handle any errors
            System.out.println("SQLException: " + ex.getMessage());
            System.out.println("SQLState: " + ex.getSQLState());
            System.out.println("VendorError: " + ex.getErrorCode());
        }
        return map;
    }
}
```
    不直接返回 Connection 类型的原因是该类型为接口类，详情可参考排雷手册

#### 登录信息校验

往IOC容器中注入一个其他类型组件，该组件会拿到 配置类 中的 **map**
并提取 **Connection** 执行查询语句后返回判断结果，并将其注入到IOC容器中
可以直接通过 **connecttest_name** 直接获取到返回值。
```java
@Component
public class DataAcc {
    private ResultSet resultSet;
    //拿到自动配置中的 Map
    @Bean
    public boolean ConnectTest_name(String name, Map<String,Object> mapList){
        Connection connection;
        String sql4name = "select Soldier_name from Soldier where Soldier_name='LAOMOB'";
        try {
            connection = (Connection) mapList.get("connection");
            Statement state = connection.createStatement();
            resultSet = state.executeQuery(sql4name);
        }catch (SQLException e){
            System.out.println("Coon error");
        }
        //如果为空，表明未查询到结果
        return !resultSet.toString().isEmpty();
    }
}
```
    如果由于网络或其他原因得不到连结，那么在执行sql指令的时候会报错。
    可以在执行前判断 connection 是否为空。

    

