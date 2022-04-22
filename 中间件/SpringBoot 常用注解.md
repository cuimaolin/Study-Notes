#### 1. `@SpringBootApplication`

`@SpringBootApplication` 为Spring Boot项目的基石，创建SpringBoot项目之后会默认在主类加上。

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

我们可以将@SpringBootApplication看作为以下三个注解作用的集合：

- `@EnableAutoConfiguration`：抵用SpringBoot的自动配置机制
- `@ComponentScan`：扫描被`@Component`(`@Service`, `@Controller`)注解的`bean`，注解默认会扫描该类所在的包下所有的类；
- `@Configuration`：允许在Spring上下文中注册额外的bean或导入其他配置类

#### 2. Spring Bean 相关

##### 2.1 `@Autowired`

自动导入到对象到类中，被注入进得类同样要被Spring容器管理，比如：Service类注入到Controoler类中。

Spring 2.5 引入了`@Autowired`注释，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过`@Autowired`的使用来消除set ，get方法。

```java 
@Service
public class UserService {
  ......
}

@RestController
@RequestMapping("/users")
public class UserController {
   @Autowired
   private UserService userService;
   ......
}
```

##### 2.2 `@Component`, `@Repository`, `@Service`, `@Controller`

我们一般使用`@Autowired`注解让Spring容器帮我们自动装配bean。要想把标识成可用于`@Autowired`注解自动装配的bean的类，可以采用以下注解实现：

- `@Component`：通用注解，可标注任意类为`Spring`组件。如果一个Bean不知道属于哪个层，可以使用`@Component`注解标注
- `@Repository`：对应持久层即Dao层，主要用于数据库相关操作
- `@Service`：对应服务层，主要涉及一些复杂的逻辑，需要用到Dao层
- `@Controller`：对应Spring MVC控制层，主要用户接受用户请求并调用Service层返回数据给前端页面

##### 2.3 `@RestController`

`@RestController`注解是`@Controller`和`@ResponseBody`的合集，表示这是个控制器bean，并且将函数的返回值直接填入HTTP响应体中，是REST风格的控制器。

单独使用`@Controller`不加`@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVA的应用，对应于前后端不分离的情况。`@Controller`+`@ResponseBody`返回JSON或XML形式的数据

##### 2.4 `@Scope`

声明Spring Bean的作用域，使用方法：
```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

四种常见的Spring Bean的作用域：

- singleton：唯一bean实例，Spring中的bean默认都是单例的；
- prototype：每次请求都会创建一个新的bean实例；
- request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效；
- session：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP session内有效

##### 2.5 `@Configuration`

一般用来声明配置类，可以使用`@Component`注解替代，不过使用`Configuration`声明配置类更加语义化

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
#### 3. 处理常见的HTTP请求类型

- GET：请求从服务器过去特定资源。举个例子：`GET/user`(获取所有学生)；
- POST：在服务器上创建一个新的资源。举个例子：`POST/user`(创建学生)；
- PUT：更新服务器上的资源(客户端提供更新后的整个资源)。举个例子：`PUT/user/12`(更新编号为12的学生)；
- DELETE：从服务器删除特定的资源。举个例子：`DELETA/user/12`(删除编号为12的学生)；
- PATCH：更新服务器上的资源(客户端提供更改的属性，可以看作是部分更新)

#### 4. 前后端传值

##### 4.1 `@PathVariable`用于获取路径参数，`@RequestParam`用于获取查询参数

举个简单的例子

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```
如果我们请求的url是：`/klasses/{123456}/teachers?type=web`，那么我们服务获取得到的数据就是：`klassId=123456,type=web`

##### 4.2 `@RequestBody`

用于读取Request请求(可能是POST, PUT, DELETE, GET请求)的body部分并且Content-Type为application/json格式的数据，接受到数据之后会自动将数据绑定到Java对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的body中的json字符串转换为java对象。

假设我们有一个注册的接口：

```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```
`UserRegisterRequest`对象：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserRegisterRequest {
    @NotBlank
    private String userName;
    @NotBlank
    private String password;
    @FullName
    @NotBlank
    private String fullName;
}
```
我们发送这个post请求到这个接口，并且body携带JSON数据：

```java
{"userName":"coder","fullName":"shuangkou","password":"123456"}
```

这样我们的后端就可以直接把json格式的数据映射到我们的`UserRegisterRequest`类上

需要注意的是：一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PatrhVariable`。

#### 5. 读取配置信息

##### 5.1 `@value`常用

使用`@Value("${property}")`读取比较简单的配置信息：

```java
@Value("${wuhan2020}")
String wuhan2020;
```

##### 5.2 `@ConfigurationProperties`(常用)

通过`@ConfigurationProperties`读取配置信息并与bean绑定

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

##### 5.3 `@PropertySource`(不常用)

`@PropertySource`读取指定properties文件

```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  省略getter/setter
  ......
}
```

#### 6. 参数校验

即使在前端对数据进行校验的情况下，我们还是对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些HTTP工具直接向后端请求一些违法数据。

JSR(Java Specification Requests)是一套JavaBean参数校验标准，它定义了很多常用的校验注解，我们可以直接将这些注解加载我们的JavaBean属性上，这也就可以在需要校验的时候进行校验了。

校验的时候我们实际用的是Hibernate Vaidator框架。

需要注意的是：所有的注解，推荐使用JSR注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`

##### 6.1 一些常用的字段验证注解

- `@NotEmpty`：被注释的字符串的不能为 null 也不能为空
- `@NotBlank`：被注释的字符串非 null，并且必须包含一个非空白字符
- `@Null`：被注释的元素必须为 null
- `@NotNull`： 被注释的元素必须不为 null
- `@AssertTrue`：被注释的元素必须为 true
- `@AssertFalse`：被注释的元素必须为 false
- `@Pattern(regex=, flag=)`：备注是的元素v必须符合指定的正则表达式
- `@Email`：被注释的元素必须是Email格式
- `@Min(value)`：被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)`：被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)`：被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)`：被注释的元素必须是一个数字，其值必须小于等于指定的最小值
- `@Size(max=, min=)`：被注释的元素的大小必须在指定范围内
- `@Digits(integer, fraction)`：被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past`：被注释的元素必须是一个过去的日期
- `@Future`：被注释的元素必须是一个将来的日期
- ......

##### 6.2 验证请求体(RequestBody)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;

}
```

我们在需要验证的参数上加上`@Vaild`注解，如果验证失败，它将抛出`MethodArgumentNotValidException`

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

##### 6.3 验证请求参数(Path Variables和Request Parameters)

要在类上加上`@Validated`注解，这个参数可以告诉Spring去校验方法参数

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }
}
```

#### 7. 全局处理Controller异常

相关注解

- `@ControllerAdvice`：注解定义全局异常处理类
- `@ExceptionHandler`：注解声明异常处理方法

在6.2 如果方法参数不对则会跑出`MethodArgumentNotValidException`异常，我们来处理这个异常。

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```

#### 8. JPA相关

##### 8.1 创建表

`@Entity`声明一个类对应一个数据库实体。

`@Table`设置表名

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    省略getter/setter......
}
```

##### 8.2 创建主键

`@Id`：声明一个字段为主键

使用`@Id`声明之后，我们还需要定义主键的生成策略。我们可以使用`@GenerateVaule`指定主键生成策略

1. 通过`@GeneratedVaule`直接使用JPA内置提供的四种主键生成策略来指定生成策略

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

JPA使用枚举定义了4个常见的主键生成策略，如下：

```java
public enum GenerationType {

    /**
     * 使用一个特定的数据库表格来保存主键
     * 持久化引擎通过关系数据库的一张特定的表格来生成主键,
     */
    TABLE,

    /**
     *在某些数据库中,不支持主键自增长,比如Oracle、PostgreSQL其提供了一种叫做"序列(sequence)"的机制生成主键
     */
    SEQUENCE,

    /**
     * 主键自增长
     */
    IDENTITY,

    /**
     *把主键生成策略交给持久化引擎(persistence engine),
     *持久化引擎会根据数据库在以上三种主键生成 策略中选择其中一种
     */
    AUTO
}
```

`@GeneratedVaule`注解默认使用的策略是`GenerationType,AUTO`：

```java
public @interface GeneratedValue {

    GenerationType strategy() default AUTO;
    String generator() default "";
}
```

一般使用MySQL数据库的话，使用`GenerationType.IDENTITY`策略比较普遍一点，分布式系统的话需要另外考虑使用分布式ID

2. 通过`@GenericGenerator`声明一个主键策略，然后`@GeneratedVaule`使用这个策略

```java
@Id
@GeneratedValue(generator = "IdentityIdGenerator")
@GenericGenerator(name = "IdentityIdGenerator", strategy = "identity")
private Long id;
```

等价于

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

jpa提供的主键生成策略有如下几种：

```java
public class DefaultIdentifierGeneratorFactory
  implements MutableIdentifierGeneratorFactory, Serializable, ServiceRegistryAwareService {

 @SuppressWarnings("deprecation")
 public DefaultIdentifierGeneratorFactory() {
  register( "uuid2", UUIDGenerator.class );
  register( "guid", GUIDGenerator.class );   // can be done with UUIDGenerator + strategy
  register( "uuid", UUIDHexGenerator.class );   // "deprecated" for new use
  register( "uuid.hex", UUIDHexGenerator.class );  // uuid.hex is deprecated
  register( "assigned", Assigned.class );
  register( "identity", IdentityGenerator.class );
  register( "select", SelectGenerator.class );
  register( "sequence", SequenceStyleGenerator.class );
  register( "seqhilo", SequenceHiLoGenerator.class );
  register( "increment", IncrementGenerator.class );
  register( "foreign", ForeignGenerator.class );
  register( "sequence-identity", SequenceIdentityGenerator.class );
  register( "enhanced-sequence", SequenceStyleGenerator.class );
  register( "enhanced-table", TableGenerator.class );
 }

 public void register(String strategy, Class generatorClass) {
  LOG.debugf( "Registering IdentifierGenerator strategy [%s] -> [%s]", strategy, generatorClass.getName() );
  final Class previous = generatorStrategyToClassNameMap.put( strategy, generatorClass );
  if ( previous != null ) {
   LOG.debugf( "    - overriding [%s]", previous.getName() );
  }
 }

}
```

##### 8.3 设置字段类型

`@Column`声明字段

设置属性userName对应的数据库字段名为user_name，长度为32，非空

```java
@Column(name = "user_name", nullable = false, length=32)
private String userName;
```

设置字段类型并且加默认值

```java
Column(columnDefinition = "tinyint(1) default 1")
private Boolean enabled;
```

##### 8.4 指定不持久化特定字段

`@Transient`：声明不需要与数据库映射的字段，在保存的时候不需要保存进数据库。

如果我们想让`secrect`这个字段不被持久化，可以使用`@Transient`关键字声明。

```java
Entity(name="USER")
public class User {

    ......
    @Transient
    private String secrect; // not persistent because of @Transient

}
```

除了`@Transient`关键字声明，还可以采用下面几种方法

```java
static String secrect; // not persistent because of static
final String secrect = “Satish”; // not persistent because of final
transient String secrect; // not persistent because of transient
```

一般使用注解的方法比较多

##### 8.5 声明大字段

`@Lob`：声明某个字段为大字段

```java
@Lob
private String content;
````

更详细的声明

```java
@Lob
//指定 Lob 类型数据的获取策略， FetchType.EAGER 表示非延迟 加载，而 FetchType. LAZY 表示延迟加载 ；
@Basic(fetch = FetchType.EAGER)
//columnDefinition 属性指定数据表对应的 Lob 字段类型
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```

##### 8.6 创建枚举类型字段

可以使用枚举类型的字段，不过枚举字段要用`@Enumerated`来修饰

```java
public enum Gender {
    MALE("男性"),
    FEMALE("女性");

    private String value;
    Gender(String str){
        value=str;
    }
}

@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    @Enumerated(EnumType.STRING)
    private Gender gender;
    省略getter/setter......
}
```

数据库里面对应存储的是MALE/FEMALE。

##### 8.7 增加审计功能

只要继承了`@AbstractAuditBase`的类都会默认加上下面四个字段

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}
```

我们对应的审计功能对应地配置类可能是下面这也的：

```java
@Configuration
@EnableJpaAuditing
public class AuditSecurityConfiguration {
    @Bean
    AuditorAware<String> auditorAware() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName);
    }
}
```

简单介绍一下上面设计到的一些注解：

- `@CreatedDate`：表示该字段为创建时间字段，在这个字段被insert的时候，会设置值
- `@CreatedBy`：表示该字段为创建人，在这个实体被insert的时候，会设置值`@LastModifiedDade`和`@LastModifiedBy`同理
- `@EnableJpaAuditing`：开启JPA审计功能

##### 8.8 删除/修改数据

`@Modifying`注解提示该JPA操作是修改操作，注意还要配合`@Transactional`注解使用。

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserName(String userName);
}
```

##### 8.9 关联关系

- `@OneToOne`：一对一关系
- `@OntToMany`：一对多关系
- `@ManyToOne`：多对一关系
- `@ManyToMany`：多对多关系

#### 9. 事务 `@Transactional`

在要开启事务的方法使用`@Transactional`注解即可

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}
```

我们知道Exception分为运行时异常RuntimeException和非运行时异常。在`@Transactional`注解中如果不配置`rollbackFor`属性，那么事务只会在遇到`RuntimeException`的时候才会回滚，加上`rollbackFor=Exception.class`后，可以在事务遇到非运行时异常时 也回滚

`@Transactional`注解一般用在可以作用在`类`或者`方法`上

- 作用于类：当把`@Transactional`注解放在类上时，表示该类的pubulic方法都配置相同的事务属性信息
- 作用于方法：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息

#### 10 json数据处理

##### 10.1 过滤json数据

`@JsonIgnoreProperties`作用在类上用于过滤掉特定字符不返回或者不解析

```java
//生成json时将userRoles属性过滤
@JsonIgnoreProperties({"userRoles"})
public class User {

    private String userName;
    private String fullName;
    private String password;
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```

`@JsonIngore`一般用于类的属性上，作用和上面的`@JsonIgnoreProperties`一样

```java
public class User {

    private String userName;
    private String fullName;
    private String password;
   //生成json时将userRoles属性过滤
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```

##### 10.2 格式化json数据

`@JsonFormat` 一般用来格式化json数据

```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone="GMT")
private Date date;
```

##### 10.3 扁平化对象

```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;

  @Getter
  @Setter
  @ToString
  public static class Location {
     private String provinceName;
     private String countyName;
  }
  @Getter
  @Setter
  @ToString
  public static class PersonInfo {
    private String userName;
    private String fullName;
  }
}
```

未扁平化之前：

```java
{
    "location": {
        "provinceName":"湖北",
        "countyName":"武汉"
    },
    "personInfo": {
        "userName": "coder1234",
        "fullName": "shaungkou"
    }
}
```

使用`@JsonUnwarpped`扁平对象之后：

```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;
    ......
}

{
  "provinceName":"湖北",
  "countyName":"武汉",
  "userName": "coder1234",
  "fullName": "shaungkou"
}
```

#### 11. 测试相关

`@ActiveProfiles`一般作用于测试类上，用于声明生效的Spring配置文件

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@ActiveProfiles("test")
@Slf4j
public abstract class TestBase {
  ......
}
```

- `@test`：声明一个方法为测试方法
- `@Transactional`：被声明的测试方法的数据会回滚，避免污染测试数据
- `@WithMochUser`：Spring Security提供的，用来模拟一个真实用户，并且可以赋予权限

```java
@Test
@Transactional
@WithMockUser(username = "user-id-18163138155", authorities = "ROLE_TEACHER")
void should_import_student_success() throws Exception {
    ......
}

```



