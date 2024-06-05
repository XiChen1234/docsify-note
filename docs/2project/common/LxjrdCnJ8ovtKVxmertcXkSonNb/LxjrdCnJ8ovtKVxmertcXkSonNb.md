# MyBatis Plus 引入及其使用

# 什么是 MyBatis Plus？

MyBatis 和 MyBatis Plus 都是 Java 语言中常用的 ORM 框架，尤其是 MyBatis Plus 可以扩展了 MyBatis 的数据库操作功能，能够通过简单的方法很方便的完成对数据的操作，在敏捷开发的项目中非常有用。

> MyBatis 和 MyBatis Plus 的区别：  
> 
> MyBatis 是一款优秀的持久化框架，它简化了 jdbc 的代码，可以使用简单的 xml 或注解来配置来映射；MyBatis-plus 是 MyBatis 的增强工具，它在 MyBatis 的基础上又添加了许多的功能，如自动映射、通用 CRUD 操作等，是的开发效率大大提高。  
> 在技术选型方面，对于需要高度定制化 SQL 和精细控制数据库操作的项目，可以选择 MyBatis；而对于追求快速开发、注重效率的项目，MyBatis Plus 则是一个更好的选择。

# 导入项目

## **Spring Boot 集成 MyBatis Plus**

目前所接触到的大多数项目都是通过 Spring Boot 快速搭建，敏捷开发的，并且 Maven 导入的方式也非常方便，所以这个方法是最常用的方法

### **版本信息**

- JDK: `1.8`
- Spring Boot: `2.6.13`
- mysql-connector-jave: `8.0.31`
- **MyBatis Plus**: `3.5.6`

### **引入 Maven 依赖**

在 Spring Boot 项目中的 pom.xml 文件中添加 MyBatis Plus 及 MySQL 数据库的依赖，为了后面简化 DO 对象的编写，也引入 lombok 的依赖

```xml
<!-- mysql -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!-- mybatis-plus -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.6</version>
</dependency>


<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### **数据库初始化**

在 MySQL 中创建新的数据库，并导入部分示例数据：

| id | name   | age | email                                       |
| -- | ------ | --- | ------------------------------------------- |
| 1  | Jone   | 18  | test1@baomidou.commailto:test1@baomidou.com |
| 2  | Jack   | 20  | test2@baomidou.commailto:test2@baomidou.com |
| 3  | Tom    | 28  | test3@baomidou.commailto:test3@baomidou.com |
| 4  | Sandy  | 21  | test4@baomidou.commailto:test4@baomidou.com |
| 5  | Billie | 24  | test5@baomidou.commailto:test5@baomidou.com |

sql 语句如下：

```sql
-- 数据库表结构
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user`
(
    id BIGINT NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);

-- 数据库数据
DELETE FROM `user`;

INSERT INTO `user` (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

### **项目配置**

在项目中的 application.yml 配置文件中设置数据库连接配置

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://127.0.0.1:3306/demo
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### **创建数据库 DO 对象**

创建数据库 DO 对象，注意，该对象为业务层与数据库之间进行数据传输的对象，基本要与数据库中的字段内容一致

```java
/**
 * @Description 用户，数据库DO对象，与数据库表一一对应
 */
@Data
@TableName("user")
public class User {
    @TableId("id") // 表id的注解，映射不一致时写入
    private Integer user_id;
    @TableField("name") // 字段value，映射不一致时写入
    private String username;
    private Integer age;
    private String email;
}
```

### **添加 Mapper 层**

添加 Mapper 层，mapper 接口直接集成 BaseMapper 并对应一个 DO 对象。BaseMapper 接口提供了多种操作数据库的方法，可以直接在上层依赖注入后调用接口方法。也可以在其中自定义操作语句，本文较为基础，因此该扩展略。

```java
/**
 * @Description 数据库接口层，用于操作user表的接口类
 */
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

如果不在 Mapper 接口上面添加 @Mapper，则需要在启动类中添加 Mapper 的扫描文件地址才能生效：`@MapperScan("com.example.项目名.mapper")`，每个项目具体地址不一样

### **添加 Service 层与 Controller 层**

根据业务逻辑，添加 Service 层和 Controller 层即可，这里以条件查询用户为例子

service 层：

```java
/**
 * @Description 用户业务接口
 */
public interface UserService {
    public User getUserByUsernameAndEmail(String username, String email);
}

/**
 * @Description 用户业务接口的具体实现
 */
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserMapper userMapper;

    @Override
    public User getUserByUsernameAndEmail(String username, String email) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.lambda()
                .eq(User::getUsername, username)
                .eq(User::getEmail, email);
        return userMapper.selectOne(queryWrapper);
    }
}
```

controller 层

```java
/**
 * @Description 用户接口层，包含用户相关操作
 */
@RestController
public class UserController {
    @Resource
    private UserService userService;

    @GetMapping("/get")
    public User getUser(@RequestParam String username, @RequestParam String email) {
        return userService.getUserByUsernameAndEmail(username, email);
    }
}
```

随后访问：http://localhost:8080/get?username=Jack&email=test2@baomidou.com，若浏览器出现数据，则证明引入过程成功！

```json
{
  "user_id": null,
  "username": "Jack",
  "age": 20,
  "email": "test2@baomidou.com"
}
```

## **总结**

通过以上几个简单的步骤，我们就实现了 User 表的 CRUD 功能，甚至连 XML 文件都不用编写！

从以上步骤中，我们可以看到集成 MyBatis-Plus 非常的简单，只需要引入 starter 依赖，简单进行配置即可使用。

## 其他导入方式：如 jar 包等，等未来用到了再进行补充和记录

# 基本使用

详见官网：[MyBatis-Plus](https://baomidou.com/)
