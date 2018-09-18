# 善用 Maven profile 和 Spring profile 管理配置文件

> 吾尝终日而思矣，不如须臾之所学也。吾尝跂而望矣，不如登高之博见也。登高而招，臂非加长也，而见者远；顺风而呼，声非加疾也，而闻者彰。假舆马者，非利足也，而致千里；假舟楫者，非能水也，而绝江河。君子生非异也，善假于物也。—— 《荀子》

以前参与的很多软件开发团队，因为不懂得「善假于物」，浪费了很多时间，工作效率也不高。就拿 Java 项目构建来说，用 Maven 和不用 Maven 区别是很大的，而且现在 Maven 基本上是标配。针对不同的项目环境，如果你还在手工改配置文件，那真是石器时代的做法了，不仅效率低，还易出错。还记得以前在移动做项目时，为了安全起见，生产环境的数据库密码每三个月改一次，改密码那天每个人都必须留下来加班，手工一个文件一个文件的改密码，重启 Tomcat，测试……现在想想真是惨不忍睹。

下面介绍如何结合 Maven 和 Spring 的 profile 特性，针对不同的环境和场景，对配置文件进行无缝切换。
## 上下文 Context

我们在做开发的过程中，任何模块或者功能都有其特定的上下文环境，比如 ASP.NET 里面 HttpContext，封装了在一次 HTTP 请求过程中的上下文环境。

对于 Java Spring 项目来说，也有上下文环境，也就是各种参数配置信息，一般都会写在配置文件里面。Spring 就提供了读取这种上下文环境的类  `Environment`，你可以这么做：

配置文件：

```
connection.username=root
```

Java 代码：

``` java
@Autowired
Environment env;
...
String uname = env.getProperty("connection.username");
```

这是 Spring 推荐的做法，当然你还可以用 `@PropertySource("classpath:application.properties")` 引入配置文件，然后利用 `@Value` 注解。

``` java
@Value("${connection.username}")
private String username;
```

那么问题来了，除了我们本地的开发环境，还有生产环境，测试环境，测试又分为集成测试环境，单元测试环境，这么多不同的环境，如果每个环境都都要使用不同的 `connection.username`，在为不同环境部署打包的时候又不想配置文件改来改去的该怎么办呢？

Maven 的 profile 功能可以很好的解决这个问题。为了简单起见，这里我假定我们只有两种环境：**开发环境和生产环境**。
## Maven Resources Plugin

要想使用 Maven的 profile 功能，首先介绍 Maven Resources Plugin， Maven 对配置文件的管理主要是通过 Maven Resources Plugin 实现的。

上面用 `${Variable}` 来引用系统配置文件中的配置项的做法，熟悉 Spring 的朋友肯定都不陌生。**Maven 也可以把 `${Variable}` 用在资源文件中**，其中的 `Variable` 可以来源于这么几个地方：
- 系统属性
- Maven 项目的属性
- Maven 过滤器资源文件
- 命令行

比如，我有一个资源文件 `src/main/resources/hello.txt`：

```
Hello ${name}
```

在 pom 文件里面，你是这么写的

```
<project>
  ...
  <name>world</name>
  ...
  <build>
    ...
    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      ...
    </resources>
    ...
  </build>
  ...
</project>
```

当你运行 `mvn resources:resources` 输出资源文件到 `target/classes/hello.txt` 目录，打开文件，发现里面还是 `Hello ${name}` 。重点来了，这时候你就要用到资源文件的 filtering 属性了：

```
...
<resource>
    <directory>src/main/resources</directory>
    <filtering>true</filtering>
</resource>
...
```

之后再运行 `mvn resources:resources` 你就会得到想要的结果 `Hello world`。还可以用 `-D` 参数从命令行传值。

```
mvn resources:resources -Dname="world"
```

当然，你可以把配置都写在 pom 文件里，就像这样：

```
<project>
  ...
  <properties>
    <your.name>world</your.name>
  </properties>
  ...
</project>
```

然后在资源文件里，用 `${your.name}` 来引用这个值，但是如果需要配置的参数过多，这么做不太方便了，这时候你可以把这些参数都放在一个属性文件里，然后配置 filter：

```
<build>
    ...
    <filters>
        <filter>my-filter-values.properties</filter>
    </filters>
    ...
</build>
```

那么 filtering 如何跟 profile 结合呢，我们接着往下看。
## Maven profile

根据上面的假定，我们配置两个 profile，开发环境（dev）和生产环境（prod）。

```
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <build.profile.id>dev</build.profile.id>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <build.profile.id>prod</build.profile.id>
        </properties>
    </profile>
</profiles>

<build>
    ...
    <filters>
        <filter>src/main/resources/profiles/${build.profile.id}.properties</filter>
    </filters>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ...
</build>
```

注意我们还配置了一个属性 build.profile.id，并且在 filter 中我们引用了这个属性 `${build.profile.id}`，这时，我们的目录是这个样子的：

```
├── src/main/resources
│   └── profile
│       └── dev.properties
│       └── prod.properties
│   └── application.properties
```

上面我们说了，Maven 可以用 `${Variable}` 用在资源文件中，那么 application.properties 我们可以这么写：

```
connection.username=${connection.username}
```

dev.properties 文件：

```
connection.username=dev_user
```

prod.properties 文件：

```
connection.username=prod_user
```

因为我们配置了 filter，对于 dev 环境，Maven 会自动去 dev.properties 里面取值 dev_user，prod 环境同理。这样我们把通用的配置放在 application.properties 里面，每个环境下特有的配置分别放在各自的配置文件下面就可以了。
## Spring profile

> 本人不喜欢基于 XML 的 Spring 配置文件，因此这里介绍的都是基于 Java 类的配置方法。

bean profile 功能是在 Spring 3.1 版本中引入的。可以使用 `@Profile` 注解指定某个 bean 属于哪一个 profile。

和 Maven 的 profile 类似，我们也定义两个 profile，分别是 `@Profile("dev")` 和 `@Profile("prod")`，不过在这里我用注解稍微改进一下：

``` java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Profile("dev")
public @interface Development {

}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Profile("prod")
public @interface Production {

}
```

使用自定义注解来定义不同的场景的好处是，简化了代码，同时可以利用 IDE 的自动补全功能。为了演示 profile 功能，我创建了两个 DataSource：

``` java
@Autowired
Environment env;

@Bean
@Production
public DataSource prodDataSource() {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://" + env.getProperty("connection.url"));
    dataSource.setUsername(env.getProperty("connection.username"));
    dataSource.setPassword(env.getProperty("connection.password"));
    ...
    dataSource.setInitialSize(env.getProperty("druid.initialSize", Integer.class));
    dataSource.setMinIdle(env.getProperty("druid.minIdle", Integer.class));
    dataSource.setMaxActive(env.getProperty("druid.maxActive", Integer.class));
    ...
    return dataSource;
}

@Bean
@Development
public DataSource devDataSource() {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://" + env.getProperty("connection.url"));
    dataSource.setUsername(env.getProperty("connection.username"));
    dataSource.setPassword(env.getProperty("connection.password"));
    return dataSource;
}
```

生产环境用到的 DataSource 你可以配置更丰富的功能，相反，开发环境没必要搞这么复杂，简单一些。

那么如何激活相应的 profile 呢，把 spring.profiles.active 放入系统属性文件即可：

```
spring.profiles.active=${build.profile.id}
```

dev.properties 文件：

```
build.profile.id=dev
connection.username=dev_user
```

prod.properties 文件：

```
build.profile.id=prod
connection.username=prod_user
```
## 场景切换

终于来到最关键的地方了，按照上面的步骤进行了配置，那么在打包的时候怎么确定启动场景呢，答案是在 Maven 编译时通过 `-P` 参数指定 Maven profile 即可。

```
mvn clean install -P prod
```

打完包，去 target 目录验证一下，看看配置文件是不是都被生产环境的变量替换了呢？
