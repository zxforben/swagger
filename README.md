# springboot 集成 swagger #
`MyBatis` 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射，而实际开发中，我们都会选择使用 `MyBatisPlus`,它是对 `MyBatis` 框架的进一步增强，能够极大地简化我们持久层的代码，下面一起来看看 `MyBatisPlus` 中的一些奇淫巧技吧。

> 说明：本篇文章需要一定的 `MyBatis` 与 `MyBatisPlus` 基础

MyBatis-Plus 官网地址 ：https://baomidou.com/ 。

## CRUD

使用 `MyBatisPlus` 实现业务的增删改查非常简单，一起来看看吧。

**1.首先新建一个SpringBoot 工程，然后引入依赖：**

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.4.2</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```
**2.配置一下数据源：**

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    url: jdbc:mysql:///mybatisplus?serverTimezone=UTC
    password: 123456
```

**3.创建一下数据表：**

```sql
CREATE DATABASE `mybatisplus`;

USE `mybatisplus`;

DROP TABLE IF EXISTS `tbl_employee`;

CREATE TABLE `tbl_employee` (
  `id` bigint(20) NOT NULL,
  `last_name` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` char(1) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=gbk;

insert  into `tbl_employee`(`id`,`last_name`,`email`,`gender`,`age`) values (1,'jack','jack@qq.com','1',35),(2,'tom','tom@qq.com','1',30),(3,'jerry','jerry@qq.com','1',40);
```

**4.创建对应的实体类**

```java
@Data
public class Employee {

    private Long id;
    private String lastName;
    private String email;
    private Integer age;
}
```

**5.编写Mapper接口**

```java
public interface EmployeeMapper extends BaseMapper<Employee> {
}
```

我们只需要继承 `MyBatisPlus` 提供的 `BaseMapper` 接口即可，现在我们就拥有了对 `Employee` 进行增删改查的API,比如：

```java
@SpringBootTest
@MapperScan("com.wwj.mybatisplusdemo.mapper")
class MybatisplusDemoApplicationTests {

    @Autowired
    private EmployeeMapper employeeMapper;

    @Test
    void contextLoads() {
        List<Employee> employees = employeeMapper.selectList(null);
        employees.forEach(System.out::println);
    }
}
```

运行结果：

```java
org.springframework.jdbc.BadSqlGrammarException:
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Table 'mybatisplus.employee' doesn't exist
```

程序报错了，原因是不存在 `employee` 表，这是因为我们的实体类名为 `Employee`，`MyBatisPlus` 默认是以类名作为表名进行操作的，可如果类名和表名不相同（实际开发中也确实可能不同），就需要在实体类中使用 `@TableName` 注解来声明表的名称：

```java
@Data
@TableName("tbl_employee") // 声明表名称
public class Employee {

    private Long id;
    private String lastName;
    private String email;
    private Integer age;
}
```

重新执行测试代码，结果如下：

```java
Employee(id=1, lastName=jack, email=jack@qq.com, age=35)
Employee(id=2, lastName=tom, email=tom@qq.com, age=30)
Employee(id=3, lastName=jerry, email=jerry@qq.com, age=40)
```

`BaseMapper` 提供了常用的一些增删改查方法：

[![20210519084059865.png](https://i.postimg.cc/3wc0s4fv/20210519084059865.png)](https://postimg.cc/xkLdHC2f)

具体细节可以查阅其源码自行体会，注释都是中文的，非常容易理解。

在开发过程中，我们通常会使用 `Service` 层来调用 `Mapper` 层的方法，而 `MyBatisPlus` 也为我们提供了通用的 `Service`：

```java
public interface EmployeeService extends IService<Employee> {
}

@Service
public class EmployeeServiceImpl extends ServiceImpl<EmployeeMapper, Employee> implements EmployeeService {
}
```

事实上，我们只需让 `EmployeeServiceImpl` 继承 `ServiceImpl` 即可获得 `Service` 层的方法，那么为什么还需要实现 `EmployeeService` 接口呢？

这是因为实现 `EmployeeService` 接口能够更方便地对业务进行扩展，一些复杂场景下的数据处理，`MyBatisPlus` 提供的 `Service` 方法可能无法处理，此时我们就需要自己编写代码，这时候只需在 `EmployeeService` 中定义自己的方法，并在 `EmployeeServiceImpl` 中实现即可。

先来测试一下 `MyBatisPlus` 提供的 `Service` 方法：

```java
@SpringBootTest
@MapperScan("com.wwj.mybatisplusdemo.mapper")
class MybatisplusDemoApplicationTests {

    @Autowired
    private EmployeeService employeeService;

    @Test
    void contextLoads() {
        List<Employee> list = employeeService.list();
        list.forEach(System.out::println);
    }
}
```

运行结果：

```java
Employee(id=1, lastName=jack, email=jack@qq.com, age=35)
Employee(id=2, lastName=tom, email=tom@qq.com, age=30)
Employee(id=3, lastName=jerry, email=jerry@qq.com, age=40)
```

接下来模拟一个自定义的场景，我们来编写自定义的操作方法，首先在 `EmployeeMapper` 中进行声明：

```java
public interface EmployeeMapper extends BaseMapper<Employee> {

    List<Employee> selectAllByLastName(@Param("lastName") String lastName);
}
```

此时我们需要自己编写配置文件实现该方法，在 `resource` 目录下新建一个 `mapper` 文件夹，然后在该文件夹下创建 `EmployeeMapper.xml` 文件：

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.wwj.mybatisplusdemo.mapper.EmployeeMapper">

    <sql id="Base_Column">
        id, last_name, email, gender, age
    </sql>

    <select id="selectAllByLastName" resultType="com.wwj.mybatisplusdemo.bean.Employee">
        select <include refid="Base_Column"/>
        from tbl_employee
        where last_name = #{lastName}
    </select>
</mapper>
```

`MyBatisPlus` 默认扫描的是类路径下的 `mapper` 目录，这可以从源码中得到体现：

[![20210519084046472.png](https://i.postimg.cc/MK7RnGHC/20210519084046472.png)](https://postimg.cc/PPqNcTmy)

所以我们直接将 `Mapper` 配置文件放在该目录下就没有任何问题，可如果不是这个目录，我们就需要进行配置，比如：

```yaml
mybatis-plus:
  mapper-locations: classpath:xml/*.xml
```

编写好 `Mapper` 接口后，我们就需要定义 `Service` 方法了：

```java
public interface EmployeeService extends IService<Employee> {

    List<Employee> listAllByLastName(String lastName);
}

@Service
public class EmployeeServiceImpl extends ServiceImpl<EmployeeMapper, Employee> implements EmployeeService {

    @Override
    public List<Employee> listAllByLastName(String lastName) {
        return baseMapper.selectAllByLastName(lastName);
    }
}
```

在 `EmployeeServiceImpl` 中我们无需将 `EmployeeMapper` 注入进来，而是使用 `BaseMapper`，查看 `ServiceImpl` 的源码：

[![1.png](https://i.postimg.cc/sXM7x9Gm/1.png)](https://postimg.cc/S2pJt9rY)

可以看到它为我们注入了一个 `BaseMapper` 对象，而它是第一个泛型类型，也就是 `EmployeeMapper` 类型，所以我们可以直接使用这个 `baseMapper` 来调用 `Mapper` 中的方法，此时编写测试代码：

```java
@SpringBootTest
@MapperScan("com.wwj.mybatisplusdemo.mapper")
class MybatisplusDemoApplicationTests {

    @Autowired
    private EmployeeService employeeService;

    @Test
    void contextLoads() {
        List<Employee> list = employeeService.listAllByLastName("tom");
        list.forEach(System.out::println);
    }
}
```

运行结果：

```java
Employee(id=2, lastName=tom, email=tom@qq.com, age=30)
```

### ID 策略

在创建表的时候我故意没有设置主键的增长策略，现在我们来插入一条数据，看看主键是如何增长的：

```java
@Test
void contextLoads() {
    Employee employee = new Employee();
    employee.setLastName("lisa");
    employee.setEmail("lisa@qq.com");
    employee.setAge(20);
    employeeService.save(employee);
}
```

插入成功后查询一下数据表：

```sql
mysql> select * from tbl_employee;

+---------------------+-----------+--------------+--------+------+
| id                  | last_name | email        | gender | age  |
+---------------------+-----------+--------------+--------+------+
|                   1 | jack      | jack@qq.com  | 1      |   35 |
|                   2 | tom       | tom@qq.com   | 1      |   30 |
|                   3 | jerry     | jerry@qq.com | 1      |   40 |
| 1385934720849584129 | lisa      | lisa@qq.com  | NULL   |   20 |
+---------------------+-----------+--------------+--------+------+
4 rows in set (0.00 sec)
```

可以看到 id 是一串相当长的数字，这是什么意思呢？提前剧透一下，这其实是分布式 id，那又何为分布式 id 呢？

我们知道，对于一个大型应用，其访问量是非常巨大的，就比如说一个网站每天都有人进行注册，注册的用户信息就需要存入数据表，随着日子一天天过去，数据表中的用户越来越多，此时数据库的查询速度就会受到影响，所以一般情况下，当数据量足够庞大时，数据都会做分库分表的处理。

然而，一旦分表，问题就产生了，很显然这些分表的数据都是属于同一张表的数据，只是因为数据量过大而分成若干张表，那么这几张表的主键 id 该怎么管理呢？每张表维护自己的 id？那数据将会有很多的 id 重复，这当然是不被允许的，其实，我们可以使用算法来生成一个绝对不会重复的 id，这样问题就迎刃而解了，事实上，分布式 id 的解决方案有很多：

1. UUID
2. SnowFlake
3. TinyID
4. Uidgenerator
5. Leaf
6. Tinyid
7. ....

以 UUID 为例，它生成的是一串由数字和字母组成的字符串，显然并不适合作为数据表的 id，而且 id 保持递增有序会加快表的查询效率，基于此，`MyBatisPlus` 使用的就是 `SnowFlake`（雪花算法）。


Snowflake 是 Twitter 开源的分布式 ID 生成算法。`Snowflake` 由 64 bit 的二进制数字组成，这 64bit 的二进制被分成了几部分，每一部分存储的数据都有特定的含义：

- **第 0 位:** 符号位（标识正负），始终为 0，没有用，不用管。
- **第 1~41 位 ：** 一共 41 位，用来表示时间戳，单位是毫秒，可以支撑 2 ^41 毫秒（约 69 年）
- **第 42~52 位 ：** 一共 10 位，一般来说，前 5 位表示机房 ID，后 5 位表示机器 ID（实际项目中可以根据实际情况调整）。这样就可以区分不同集群/机房的节点。
- **第 53~64 位 ：** 一共 12 位，用来表示序列号。 序列号为自增值，代表单台机器每毫秒能够产生的最大 ID 数(2^12 = 4096),也就是说单台机器每毫秒最多可以生成 4096 个 唯一 ID。

[![up-a7e54a77b5ab1d9fa16d5ae3a3c50c5aee9.png](https://i.postimg.cc/1zgLK2kZ/up-a7e54a77b5ab1d9fa16d5ae3a3c50c5aee9.png)](https://postimg.cc/zLrtztDd)

这也就是为什么插入数据后新的数据 `id` 是一长串数字的原因了，我们可以在实体类中使用 `@TableId` 来设置主键的策略：

```java
@Data
@TableName("tbl_employee")
public class Employee {

    @TableId(type = IdType.AUTO) // 设置主键策略
    private Long id;
    private String lastName;
    private String email;
    private Integer age;
}
```

MyBatisPlus 提供了几种主键的策略：

[![334.png](https://i.postimg.cc/85YgGDLc/334.png)](https://postimg.cc/CBbQC3Pp)

其中 `AUTO` 表示数据库自增策略，该策略下需要数据库实现主键的自增（auto_increment)，`ASSIGN_ID` 是雪花算法，默认使用的是该策略，`ASSIGN_UUID` 是 `UUID` 策略，一般不会使用该策略。


这里多说一点， 当实体类的主键名为 id，并且数据表的主键名也为 id 时，此时 `MyBatisPlus` 会自动判定该属性为主键 id，倘若名字不是 id 时，就需要标注 `@TableId` 注解，若是实体类中主键名与数据表的主键名不一致，则可以进行声明：

```java
@TableId(value = "uid",type = IdType.AUTO) // 设置主键策略
private Long id;
```

还可以在配置文件中配置全局的主键策略：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
```

这样能够避免在每个实体类中重复设置主键策略。

### 属性自动填充

翻阅《阿里巴巴Java开发手册》，在第 5 章 MySQL 数据库可以看到这样一条规范：

[![image.png](https://i.postimg.cc/V6ZgHknG/image.png)](https://postimg.cc/r0W5Dchx)

对于一张数据表，它必须具备三个字段：

- `id`:  唯一ID
- `gmt_create`: 保存的是当前数据创建的时间
- `gmt_modified `: 保存的是更新时间

我们改造一下数据表：

```sql
alter table tbl_employee add column gmt_create datetime not null;
alter table tbl_employee add column gmt_modified datetime not null;
```

然后改造一下实体类：

```java
@Data
@TableName("tbl_employee")
public class Employee {

    @TableId(type = IdType.AUTO) // 设置主键策略
    private Long id;
    private String lastName;
    private String email;
    private Integer age;
    private LocalDateTime gmtCreate;
    private LocalDateTime gmtModified;
}
```

此时我们在插入数据和更新数据的时候就需要手动去维护这两个属性：

```java
@Test
void contextLoads() {
    Employee employee = new Employee();
    employee.setLastName("lisa");
    employee.setEmail("lisa@qq.com");
    employee.setAge(20);
    // 设置创建时间
    employee.setGmtCreate(LocalDateTime.now());
    employee.setGmtModified(LocalDateTime.now());
    employeeService.save(employee);
}

@Test
void contextLoads() {
    Employee employee = new Employee();
    employee.setId(1385934720849584130L);
    employee.setAge(50);
    // 设置创建时间
    employee.setGmtModified(LocalDateTime.now());
    employeeService.updateById(employee);
}
```

每次都需要维护这两个属性未免过于麻烦，好在 `MyBatisPlus` 提供了字段自动填充功能来帮助我们进行管理，需要使用到的是 `@TableField` 注解：

```java
@Data
@TableName("tbl_employee")
public class Employee {

    @TableId(type = IdType.AUTO)
    private Long id;
    private String lastName;
    private String email;
    private Integer age;
    @TableField(fill = FieldFill.INSERT) // 插入的时候自动填充
    private LocalDateTime gmtCreate;
    @TableField(fill = FieldFill.INSERT_UPDATE) // 插入和更新的时候自动填充
    private LocalDateTime gmtModified;
}
```

然后编写一个类实现 MetaObjectHandler 接口：

```java
@Component
@Slf4j
public class MyMetaObjectHandler implements MetaObjectHandler {

    /**
     * 实现插入时的自动填充
     * @param metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("insert开始属性填充");
        this.strictInsertFill(metaObject,"gmtCreate", LocalDateTime.class,LocalDateTime.now());
        this.strictInsertFill(metaObject,"gmtModified", LocalDateTime.class,LocalDateTime.now());
    }

    /**
     * 实现更新时的自动填充
     * @param metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("update开始属性填充");
        this.strictInsertFill(metaObject,"gmtModified", LocalDateTime.class,LocalDateTime.now());
    }
}
```

该接口中有两个未实现的方法，分别为插入和更新时的填充方法，在方法中调用 `strictInsertFill()` 方法 即可实现属性的填充，它需要四个参数：

1. `metaObject`:元对象，就是方法的入参
2. `fieldName`:为哪个属性进行自动填充
3. `fieldType`:属性的类型
4. `fieldVal`:需要填充的属性值

此时在插入和更新数据之前，这两个方法会先被执行，以实现属性的自动填充，通过日志我们可以进行验证：

```java
@Test
void contextLoads() {
    Employee employee = new Employee();
    employee.setId(1385934720849584130L);
    employee.setAge(15);
    employeeService.updateById(employee);
}
```

运行结果：

```java
15584 --- [           main] c.w.m.handler.MyMetaObjectHandler        : update开始属性填充
2021-04-24 21:32:19.788  INFO 15584 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-04-24 21:32:21.244  INFO 15584 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
```

属性填充其实可以进行一些优化，考虑一些特殊情况，对于一些不存在的属性，就不需要进行属性填充，对于一些设置了值的属性，也不需要进行属性填充，这样可以提高程序的整体运行效率：

```java
@Component
@Slf4j
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        boolean hasGmtCreate = metaObject.hasSetter("gmtCreate");
        boolean hasGmtModified = metaObject.hasSetter("gmtModified");
        if (hasGmtCreate) {
            Object gmtCreate = this.getFieldValByName("gmtCreate", metaObject);
            if (gmtCreate == null) {
                this.strictInsertFill(metaObject, "gmtCreate", LocalDateTime.class, LocalDateTime.now());
            }
        }
        if (hasGmtModified) {
            Object gmtModified = this.getFieldValByName("gmtModified", metaObject);
            if (gmtModified == null) {
                this.strictInsertFill(metaObject, "gmtModified", LocalDateTime.class, LocalDateTime.now());
            }
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        boolean hasGmtModified = metaObject.hasSetter("gmtModified");
        if (hasGmtModified) {
            Object gmtModified = this.getFieldValByName("gmtModified", metaObject);
            if (gmtModified == null) {
                this.strictInsertFill(metaObject, "gmtModified", LocalDateTime.class, LocalDateTime.now());
            }
        }
    }
}
```

### 逻辑删除

逻辑删除对应的是物理删除，分别介绍一下这两个概念：

1. **物理删除**: 指的是真正的删除，即：当执行删除操作时，将数据表中的数据进行删除，之后将无法再查询到该数据
2. **逻辑删除**: 并不是真正意义上的删除，只是对于用户不可见了，它仍然存在与数据表中

在这个数据为王的时代，数据就是财富，所以一般并不会有哪个系统在删除某些重要数据时真正删掉了数据，通常都是在数据库中建立一个状态列，让其默认为 0，当为 0 时，用户可见；当执行了删除操作，就将状态列改为 1，此时用户不可见，但数据还是在表中的。

[![image.png](https://i.postimg.cc/B6BptkGv/image.png)](https://postimg.cc/jwCPF82V)

按照《阿里巴巴Java开发手册》第 5 章 MySQL 数据库相关的建议，我们来为数据表新增一个 `is_deleted` 字段：

```sql
alter table tbl_employee add column is_deleted tinyint not null;
```

在实体类中也要添加这一属性：

```java
@Data
@TableName("tbl_employee")
public class Employee {

    @TableId(type = IdType.AUTO)
    private Long id;
    private String lastName;
    private String email;
    private Integer age;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime gmtCreate;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime gmtModified;
    /**
     * 逻辑删除属性
     */
    @TableLogic
    @TableField("is_deleted")
    private Boolean deleted;
}
```

[![image.png](https://i.postimg.cc/qBCnmWF6/image.png)](https://postimg.cc/jCtL23tR)

还是参照《阿里巴巴Java开发手册》第 5 章 MySQL 数据库相关的建议，对于布尔类型变量，不能加 `is` 前缀，所以我们的属性被命名为 `deleted`，但此时就无法与数据表的字段进行对应了，所以我们需要使用 `@TableField` 注解来声明一下数据表的字段名，而` @TableLogic` 注解用于设置逻辑删除属性；此时我们执行删除操作：

```java
@Test
void contextLoads() {
    employeeService.removeById(3);
}
```

查询数据表：

```sql
mysql> select * from tbl_employee;
+---------------------+-----------+--------------+--------+------+---------------------+---------------------+------------+
| id                  | last_name | email        | gender | age  | gmt_create          | gmt_modified        | is_deleted |
+---------------------+-----------+--------------+--------+------+---------------------+---------------------+------------+
|                   1 | jack      | jack@qq.com  | 1      |   35 | 0000-00-00 00:00:00 | 0000-00-00 00:00:00 |          0 |
|                   2 | tom       | tom@qq.com   | 1      |   30 | 0000-00-00 00:00:00 | 0000-00-00 00:00:00 |          0 |
|                   3 | jerry     | jerry@qq.com | 1      |   40 | 0000-00-00 00:00:00 | 0000-00-00 00:00:00 |          1 |
| 1385934720849584129 | lisa      | lisa@qq.com  | NULL   |   20 | 0000-00-00 00:00:00 | 0000-00-00 00:00:00 |          0 |
| 1385934720849584130 | lisa      | lisa@qq.com  | NULL   |   15 | 2021-04-24 21:14:18 | 2021-04-24 21:32:19 |          0 |
+---------------------+-----------+--------------+--------+------+---------------------+---------------------+------------+
5 rows in set (0.00 sec)
```

可以看到数据并没有被删除，只是 `is_deleted` 字段的属性值被更新成了 1，此时我们再来执行查询操作：

```java
@Test
void contextLoads() {
    List<Employee> list = employeeService.list();
    list.forEach(System.out::println);
}
```

执行结果：

```java
Employee(id=1, lastName=jack, email=jack@qq.com, age=35, gmtCreate=2021-04-24T21:14:18, gmtModified=2021-04-24T21:14:18, deleted=false)
Employee(id=2, lastName=tom, email=tom@qq.com, age=30, gmtCreate=2021-04-24T21:14:18, gmtModified=2021-04-24T21:14:18, deleted=false)
Employee(id=1385934720849584129, lastName=lisa, email=lisa@qq.com, age=20, gmtCreate=2021-04-24T21:14:18, gmtModified=2021-04-24T21:14:18, deleted=false)
Employee(id=1385934720849584130, lastName=lisa, email=lisa@qq.com, age=15, gmtCreate=2021-04-24T21:14:18, gmtModified=2021-04-24T21:32:19, deleted=false)
```

会发现第三条数据并没有被查询出来，它是如何实现的呢？我们可以输出 `MyBatisPlus` 生成的 SQL 来分析一下，在配置文件中进行配置：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 输出SQL日志
```

运行结果：

```java
==>  Preparing: SELECT id,last_name,email,age,gmt_create,gmt_modified,is_deleted AS deleted FROM tbl_employee WHERE is_deleted=0
==> Parameters:
<==    Columns: id, last_name, email, age, gmt_create, gmt_modified, deleted
<==        Row: 1, jack, jack@qq.com, 35, 2021-04-24 21:14:18, 2021-04-24 21:14:18, 0
<==        Row: 2, tom, tom@qq.com, 30, 2021-04-24 21:14:18, 2021-04-24 21:14:18, 0
<==        Row: 1385934720849584129, lisa, lisa@qq.com, 20, 2021-04-24 21:14:18, 2021-04-24 21:14:18, 0
<==        Row: 1385934720849584130, lisa, lisa@qq.com, 15, 2021-04-24 21:14:18, 2021-04-24 21:32:19, 0
<==      Total: 4
```

原来它在查询时携带了一个条件： `is_deleted=0` ，这也说明了 `MyBatisPlus` 默认 0 为不删除，1 为删除。 若是你想修改这个规定，比如设置-1 为删除，1 为不删除，也可以进行配置：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted # 逻辑删除属性名
      logic-delete-value: -1 # 删除值
      logic-not-delete-value: 1 # 不删除值
```

但建议使用默认的配置，阿里巴巴开发手册也规定 1 表示删除，0 表示未删除。