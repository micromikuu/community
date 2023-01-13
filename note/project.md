# IDEA问题
1. SpringBoot Iinitial项目导入后，源代码目录冲突，即不同模块引用了同一源代码目录
   - 删除不同模块里的源代码目录还是不起作用，要删除掉整个模块

# SpringBoot配置
## 基础配置
1. SpringBoot因为内置了Tomcat，所以可以依靠Java Application 的Main入口直接启动Web项目
2. application.yml
    ```
   spring: //先是spring
      datasource:
        driver-class-name: //这个可以通过Libraries找到mysql的依赖，从而在连接包里找到那个具体的Driver类名
        url: jdbc:mysql//localhost:3306/databasename?serverTimezone=UTC&CharacterEncodeing=UTF-8&useUnicode=true //协议是jdbc:mysql，地址是主机加数据库
        username:
        password: 
    ```
3. pom里spring-boot-maven-plugin报红无法找到
   - 添加版本号
4. 如果我们使用了@SpringBootApplication注解的话，系统会去入口类的同级包以及下级包中去扫描实体类，因此我们建议入口类的位置在groupId+arctifactID组合的包名下
5. Spring Boot使用一个全局的配置文件application.properties
   - server.context-path=/index
   - server.port=8080
6. JdbcTemplate使用BeanPropertyRowMapper设置实体类映射的时候，必须保证实体类有Getter/Setter
7. 报错`java.sql.SQLException: Zero date value prohibited`
   - MySQL数据库在面对0000-00-00 00:00:00日期的处理时，如果没有设置对应的对策，就会产生异常。
   - 在配置文件jdbc-url参数中加上&zeroDateTimeBehavior=convertToNull，作用是将0000-00-00 00:00:00转化为null。

## 配置访问
1. 使用`@Value`读取注解
   - `@Value`注解是在实例化的时候执行的，因此静态变量不能被注入
2. 在SpringBoot的jar包外读取配置文件：
   - 使用启动命令来指定配置文件：`java -jar springBootDemo-0.0.1-SNAPSHOT.jar --spring.config.location=D:\Jayson\config\application.properties`
        - **注意：该命令指定的配置文件会替换默认的配置文件**
   - 约定访问外部配置文件的优先级：
      1. 第一优先级：*.jar config/*.yml
      2. 第二优先级：*.jar *.yml 
      3. 第三优先级：*.jar/config/*.yml
      4. 第四优先级：*.jar/*.yml
      5. 所有位置的配置文件都会被读取，如果有重复的配置项，高优先级的配置项会覆盖低优先级的，如果没有重复，那么会共同存在
## 多种配置
1. 首先如果启动的时候，没有指定配置文件，或者指定的配置文件没有对应的项，则会从默认的配置文件application.yml中读取
2. SpringBoot提供profile实现多环境配置 配置命名文件规则:
    - 前缀统一为application，后缀为自定义名称，格式为application-{profile}.properties
    - dev环境下的配置在application-dev.yml
    - prod环境下的配置在application-dev.yml
4. 可以分为多个配置文件application-dev.yml、application-dev.yml以及application.yml，先加载默认的application.yml，并设置活跃配置文件。也可以在命令行执行时指定。
   - 设置活跃配置文件:`spring.profiles.active=dev`
   - 添加多配置文件：`spring.profiles.include:dev,prod`

# spring-boot-devtools热启动工具注意事项
1. **必须要注意这是热启动，不是热部署。热部署是当应用还在运行时，修改后的源代码可以不重启应用的运行，重载到应用中，并立即生效。这种重载类的方式只能修改方法体内的代码，不能像热启动那样可以修改类结构，比如增删成员变量、函数以及类，注解、配置也不会生效。**
2. 因为 spring-boot-devtools 一般只使用于开发环境，在生产环境是需要禁用的，所以设置optional=true
3. 默认配置
   - Spring Boot 为了支持某些库使用缓存来提高性能，提供了一些设置，如：spring.thymeleaf.cache，它将会缓存编译模板，避免因模板的重复解析而降低系统性能，这在生产环境中是重要的，但是在本地开发时，这会导致每次修改模板都要重启服务，降低了开发速度，为了避免这个问题，我们需要手动禁用 thymeleaf 缓存，但是如果项目中添加了 devtools ，那就不需要去一一设置这些选项了，devtools 为此提供了非常多的默认设置。
   - 如果你需要禁用 devtools 提供的默认属性，请设置 spring.devtools.add-properties=false
4. 自动重启 
   - 1. 当项目 classpath 下的文件发生了变动，devtools 将会帮我们自动重启服务。 
     2. devtools 将会监控应用的 classpath 下的资源，因此当 classpath 下的资源发生了变更时，应用就会被重启。在 Eclipse 中，保存修改后的文件将会触发重启；在 Idea 中，构建项目（Build -> Build Project）将会导致应用重启。
   - 1. 某些资源的修改其实不需要重启服务，比如前端使用的静态资源文件，默认情况下，Spring Boot 在以下这些目录中的资源变更时，不会触发重启（但是会触发 live reload）:`/META-INF/maven`，`/META-INF/resource`，`/resource`，`/static`，`/public`，`/templates`
     2. 可以手动指定这些路径:`spring.devtools.restart.exclude=static/**,public/**`
     3. 只是项额外增加路径而不覆盖 Spring Boot 提供的默认设置时，你可以设置这个属性:`spring.devtools.restart.additional-exclude = /custom-path/**`
   - 当想设置的触发重启的文件不在 classpath 下的时候，还可以使用以下设置作为触发应用重启的条件:`spring.devtools.restart.additional-paths`
   - 禁用自动重启:`spring.devtools.restart.enabled=false`
   - 自定义重启使用的类加载器（热加载可能会出现的严重问题）：往往是类型转换错误，变化的项目与稳定的库用不同的类加载器加载了同一个类
     1. devtools 将会使用两个类加载器用于重启应用，我们可以通过以下属性指定 classpath 下面的某个包使用哪个类加载器：
     ```
     restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
     restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
     ```
     2. 以上的属性需要添加到`classpath: META-INF/spring-devtools.properties`路径下。
# 静态资源问题
## 关于url与路径的基本问题
1. `/`是根路径，也即根目录
2. `/ed`是根路径下的文件
3. `/ed/`是根路径下的ed目录
4. `./`是正在访问路径的对象的当前路径
- 这就是所有问题的答案
- 有`/`的路径都是**绝对路径**。在url里，是域名的根目录，在文件系统路径里，是所在盘的根目录
- 若以绝对路径定位静态资源，浏览器访问该静态资源时不会再拼接任何路径
- 相对路径可以不加`./`，若网页使用了相对路径来定位静态资源，意味着浏览器以请求资源网页的url为基本路径，将相对路径拼接上该路径得到资源的完整url
- 因为文件可以没有后缀名，所以`/ed`不会表示一个目录。在增加url映射的时候，若以一个文件为映射的url，那么该url的路径为文件所在路径，即`/`
- 以文件`/ed`为location，或者base等映射后的定位，可能会产生定位失败。一定要在最后加上文件分隔符`/`，以表示为一个目录。
## 本地获取资源
1. 获取classpath下的静态资源，可以使用ClassPathResource类
   - 可以拿到文件的路径以及输入流：`ClassPathResource classPathResource = new ClassPathResource("/static/xxx/xxx.png");`
2. 本地相对路径问题
   - 在Config包里的WebConfig类里的addResourceHandlers里输出'/'与'./'的绝对路径，分别是盘符根目录与项目根目录
   - 配置文件的spring.web.resources.static-locations或WebMvcConfiguration配置类可以使用classpath、file进行匹配，如果使用file，这个时候的相对路径为项目地址（打包为.jar后，相对路径就是.jar运行地址）
## 请求获取资源
1. 在`spring.mvc.static-path-pattern`会覆盖掉Spring默认的
2. Spring默认的静态资源解析似乎会过滤掉.ico文件
3. 脑子真的瓦塔了，设想是没有错的pathPattern就是一个字符串匹配的模式，用来匹配URL，匹配成功就映射到相应的location上请求资源。但是！但是！location不是模式！！！！不能写通配符！！！！
   - Soring是完全按照pathPattern去匹配URL，`/*`及`/**`后的`*`，指代通配符匹配到的资源的相对路径，将会直接拼接到映射的location路径上，不会在location上做任何匹配的。
4. 多个spring.mvc.static-path-pattern配置项
5. 可以使用`addResourceLocations`方法包含多个位置，从而为同一个申请URL映射多个路径。程序将按顺序搜索位置列表，直到找到资源

# SpringBoot的Windows部署
## Winsw工具
### 方法引导
1. 下载winsw
2. 将winsw可执行文件与打包好的jar包放在同一个目录下
3. 建立一个xml文档，并将以上一共三个文件设置为一样的名字
4. xml文档用于配置生成的Windows服务信息：
    ```
    <service>
    <id>energy</id>
    <name>energy</name>
    <description>This service runs myapp project.</description>
    <executable>java</executable>
    <arguments>-jar "D:\springboot-service\demo-0.0.1-SNAPSHOT.jar"</arguments>
    <startmode>Automatic</startmode>
    <logmode>rotate</logmode>
    </service>
    ```
   - id:id是安装成windows服务后的服务名，id必须是唯一的。
   - name：name是服务的简写名字，name也必须是唯一的，这里我设为和id相同。
   - description:服务的文字说明。
   - executable:执行的命令，因为启动springboot应用的命令是java -jar app.jar
   - arguments：命令执行参数， 如果端口号要在这里设置，可以在后面添上：--server.port=8080
   - startmode：可以设置为开机自启动
   - logpath：日志路径
   - 基本命令： `uninstall`：删除服务 `start`：启动服务 `stop`：停止服务 `restart`：重启服务 `status`：输出当前服务的状态
5. 在服务管理中，找到新增加的服务，设置启动类型为自动，则该服务会随着Windows的启动而启动

## nssm
- nssm是一个服务封装程序，它可以将普通exe程序封装成服务，实现开机自启动、崩溃重启
###方法引导
1. 管理员权限打开命令行工具，切换到nssm.exe所在路径，运行 nssm install，打开程序配置界面
    - Path：运行应用程序的程序
    - Startup directory：应用程序所在的目录 
    - Arguments：应用运行的参数 
    - Service name：生成服务的名称
2. 最后点击install service，完成windows服务安装，在windows服务列表就能看到创建的服务了
3. 常用命令
   - nssm install servername //创建servername服务，弹出配置界面
   - nssm start servername //启动服务
   - nssm stop servername //暂停服务 
   - nssm restart servername //重新启动服务 
   - nssm remove servername //删除创建的servername服务 
   - nssm edit servername//更改servername服务，弹出修改界面 
   - nssm set servername parameter-name parameter-value //设置服务参数值 
   - sc delete servername//windows删除服务命令

## alwaysup
收费应用，但功能强大