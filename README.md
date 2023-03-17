## 启动bootstrap和Springboot组合的项目
访问https://v3.bootcss.com/css/ 下载BootStrap
1.解压，并将其中的css,fonts,js文件夹复制到项目的/resources/static文件夹下
2.resources文件夹下新建一个templates文件夹
3.在thymeleaf官网下载一个index.html的使用模板，放在templates文件夹下；网址https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html
# 第一步：
```html
    <!DOCTYPE html>

  <html xmlns:th="http://www.thymeleaf.org">

  <head>
      <title>亮亮社区</title>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
      <!--三个资源文件的引入-->
      <link rel="stylesheet" href="css/bootstrap.min.css">
      <link rel="stylesheet" href="css/bootstrap-theme.min.css">
      <script src="js/bootstrap.min.js" type="application/javascript"></script>

  </head>
  <body>
  </body>
  </html>
```


# 第二步：
 在bootstrap官网找一个导航栏样式，放入index.html的body中去

# 第三步：写一个controller
    注意这里的注解用的是@Controller,我使@RestController的时候，访问浏览器返回的是字符串：index，原因还没查：
```java
    package com.community.commucity;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.GetMapping;
    
    /**
     * Created by 舒先亮 on 2019/8/20.
     */
    @Controller
    public class CommucityController {
    
        @GetMapping("/")
        public String index(){
            return "index";
        }
    }

```

    此时我以为能够返回"index.html",但是由于没有在pom.xml文件中加入thymeleaf的依赖：
```xml
            <dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-thymeleaf</artifactId>
    		</dependency>
```
             

# 第四步：看一下github主页的api,主页拉到最下面有一个API接口
    找到:Building a GitHub App or an OAuth App     创建一个GitHub的APP，或者授权APP

# 第五步：git操作
    在这之前，由于没有操作过git的提交，所以尝试先把我的workspace“springboot”初始化为管理库repository，
    windows下，先进入springboot文件夹，按住Ctrl+鼠标右键，点击进入git bash
    命令：git init             https://www.liaoxuefeng.com/wiki/896043488029600/896827951938304
    添加：git add .
    提交：git commit -m "add master of workspace"

    设置身份认证：我是谁  git config --global user.email "1610700471@qq.com"
                        git config --global user.name "Jackshufu"

# 第六步：git初始化并提交远程仓库
    在新建master的时候，发现其实只要进入到某个project文件夹，就可以使用git init，然后初始化好后就能git add . ,最后再提交
    当git push之前，还需要配置远程仓库的目的地
    1.先在github上创建一个仓库，然后获得一个访问地址，再然后使用命令git remote add origincommunity https://github.com/Jackshufu/community.git
    在本地关联远程仓库
    2.使用 git push -u origincommunity master  上传至github，蛋疼的是需要输入github密码，用户名

## 工具
   [visual-paradigm（画UML图工具）](https://www.visual-paradigm.com/cn/)
   [json在线解析工具](https://jsoneditoronline.org)
   [Postman](https://chrome.google.com/webstore/detail/tabbed-postman-rest-clien/coohjcphdfgbiolnekdpbcijmhambjff)
   
# 第七步：申请github的APP，用于github授权APP登录
        进入github->settings->Developer settings->New Github App
        设置APP name（Community），网址http://community.shu.com
        webhook,我写的是本地关联远程仓库的网址：https://github.com/Jackshufu/community.git
```
        Owned by: @Jackshufu
        
        App ID: 39020
        
        Client ID: Iv1.c83b663ef84a8086
        
        Client secret: 8c41ac8a3ab868d25fbb36f8977172cbb45ad2b7
```
        
    
    GitHub授权登录的时序：
    1.用户访问我的社区，在社区上处理自己的登录逻辑，然后调用authorize接口去访问github，然后github回调我们注册APP时填写的
    redirect-uri并携带code，当社区接收到这个信息后，会再次调用githubde access_token接口，并携带code，这步为了获取github
    的token，然后当github验证成功后，返回access_token给社区，社区可以通过access_token来调用GitHub的userAPI，github返回user信息
    此时可以更新登录状态，把获取到的user信息保存到数据库，最后返回用户登录成功

# 步骤八：添加fastjson依赖
[2019009看到说fastJson出现漏洞，最好修复到1.2.60版本](https://mp.weixin.qq.com/s/yVzZTTR5R6QbspTg5WI5hg)
```xml
    <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.59</version>
    </dependency>
```


    添加okhttp依赖
```xml
        <dependency>
			<groupId>com.squareup.okhttp3</groupId>
			<artifactId>okhttp</artifactId>
			<version>3.14.1</version>
		</dependency>
```
		
    json格式的数据转换成对象

# 步骤九：当github通过验证并发放认证令牌后，社区如何通过access_token发起请求访问userAPI拿到user信息？
    首先在github的Personal access tokens新建一个私人访问token，在官网API的描述下，模拟一下
     Google打开匿名窗口：win   Ctrl+shift+N
     拼接如下：https://api.github.com/user?access_token=f47c9b74247c9342f4183c0543c704733ab5441e
     能够看到用户相关信息
        其中我可以关注id ,name , bio  
    
# 步骤十：整个代码写完了，诸如client_id,client_secret,redirect_uri这些参数写在代码中不合适，我将他配置在application.properties
 中，如下(注意配置文件值的末端不能添加;号)：
 ```properties
            github.client.id = Iv1.c83b663ef84a8086
            github.client.secret = 8c41ac8a3ab868d25fbb36f8977172cbb45ad2b7
            github.redirect.uri = http://localhost:8080/callback
```
            
 
    代码中依赖注入用@Value，其中${}中填的是配置文件中的key
 ```java
            @Value("${github.client.id}")
            private String clientId;
            @Value("${github.client.secret}")
            private String clientSecret;
            @Value("${github.redirect.uri}")
            private String redirectUri;
 ```
           
# 步骤十一：什么是session，什么是cookie？
    session就相当于银行账户，cookie就相当于银行卡，拿来银行卡，才能知道银行账户
    HTTP是无状态的，怎样让服务器记住我的状态，每次带一个cookie来就好了
     1.在CallBackController的形参中加入
     
     
     菜鸟教程：https://www.runoob.com
# 步骤十二：
    1.在application.properties中配置MySQL数据库：
```properties
        spring.datasource.url=jdbc:mysql://localhost:3306/store?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
        spring.datasource.username=root
        spring.datasource.password=root
```
    2.添加JPA、mysql、jdbc依赖
```xml
        <dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
```
        
        
 1. 思考关于cookie的问题，之所以能够在浏览器能找到服务器端，是因为页面发送那个了一个cookie给服务器，服务器端通过框架的解释将cookie路由到
 了session，我们不使用框架为我们提供的session值，我们需要自己设置session值,登录成功之后，手动将session值设置到cookie里，这个session就是一个
 token，访问数据库的时候，我们可以拿着这个token去数据库里查。
 2. 将自己设置的session(token)存入数据库后，我们可以手动模拟cookie和session之间的交互，保证服务器宕机或者重启服务器的时候，
    都会使得用户重新登录，并保有登录态->登录成功后，通过java代码往前端写cookie，以token为依据，来绑定前端和后端的登录状态。
 3. 把java代码中的UUID.randomUUID().toString()抽取出来，来代替曾经的session
```java
     String token = UUID.randomUUID().toString()
```

    
    
    int     对应java中的int
    bigint  对应java中的long类型
    gmt     格林威治时间
    
# 步骤十三：集成mybatis,百度搜索Springboot mybatis，http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/
    1.添加MyBatis依赖：(这样才能够在Mapper类中使用@Mapper注解)
```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
```
        
     2.@Insert注解和里面的sql就相当于xml中的映射配置了，注解写在哪，就相当于于映射谁
     
    
     备注：
        1.类类之间，对象在网络中传输使用DTO;
        2.对象在数据库中的话，我们叫他model
        3.service层需要抛出异常，controller层可以同一个异常，多处try-catch
        
# 步骤十四：还是要手动下的热部署
    1. 在application.properties文件中配置 ：spring.thymeleaf.cache=false
    此步骤先清除thymeleaf的缓存
    1.1：重新启动一下主程序(run一下)，然后关闭主程序（否则执行1.2报错占用端口） 之后修改html，js，css等相关文件无需重启，按 Ctrl+f9 手动加载一下资源后直接刷新页面就可以看到可以修改，这样热部署就算成功了
    1.2:使用maven方式启动：mvn spring-boot:run
    2. 使用devtools方式
    增加maven的devtools依赖
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
```
    3.java代码热加载
```xml
    <plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<fork>true</fork>
				</configuration>
			</plugin>
```

# 步骤十五：引用jQuery和bootstrap.min.js顺序
    如果bootstrap.min.js在jQuery的前面，则浏览器报错
    Uncaught Error: Bootstrap's JavaScript requires jQuery at bootstrap.min.js:6
    百度找到一个jQuery文件，问题解决：
```html
    <script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
```

# 步骤十六：提交问题到数据库
    1. 建表
```mysql
      CREATE TABLE `community`.`question` (
      `id` INT NOT NULL AUTO_INCREMENT COMMENT '主键自增',
      `title` VARCHAR(50) NULL COMMENT '问题标题',
      `description` TEXT NULL COMMENT '问题的详细描述，放在文本编辑框，文本类型是TEXT类型',
      `gmt_create` BIGINT NULL,
      `gmt_modified` BIGINT NULL,
      `creator` INT NULL,
      `comment_count` INT NULL DEFAULT 0,
      `view_count` INT NULL DEFAULT 0,
      `like_count` INT NULL DEFAULT 0,
      `tag` VARCHAR(256) NULL,
      PRIMARY KEY (`id`))
    COMMENT = '用于提交问题的表';
```

    2. 写一个QuestionMapper
```java
    package com.community.mapper;

import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;

/**
 * Created by 舒先亮 on 2019/8/24.
 */
@Mapper
public interface QuestionMapper {

    @Insert("insert int question (title,description,gmt_create,gmt_modified,creator,tag) values (#{title}),#{description},#{gmt_create},#{gmt_modified},#{creator},#{tag}")
    void insertQuestion();
}

```

    3.后台通过Model传值给thymeleaf模板中的<input>输入框的时候，用的是th:value = "${Model的属性}"
      <textarea>中用的是th:text ;其中要注意name的值要和Model的属性值一致
          
# 步骤十七：当用户登录返回一个头像
```mysql
  ALTER TABLE `community`.`user` 
ADD COLUMN `avatar_url` VARCHAR(100) NULL AFTER `gmt_modified`;
```
    1.一个新知识点，如何使用Lombok？去官网查询，添加依赖,IDEA还需要下载一个插件
```xml
    <dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.8</version>
		<scope>provided</scope>
	</dependency>
```
    2.将后台获取的头像URL，放在首页亮亮的位置(还未完成)

# 步骤十八：遍历数据库question表，展示在首页
    1.需要根据question表获取展示问题所需要的关键字，还需要根据用户的信息获取用户的头像，这些的实现需要我新建一个
    对象类QuestionDTO，用来实现数据在网络中传输。
    2.为了将数据库表question和user的属性查找出来，我们创建了一个QuestionDTO的类，这两个整合的model对象，我们使用
    service层对其进行处理
    3.将查出来的两个集合对象揉合到一起，使用方法BeanUtils方法，将对象属性（第一个参数）转换称第二个对象属性（其实这里可以用级联查询）
    4.遍历用th:each   th:each="question : ${questions}"；传后台图片用th:src  th:src="${question.user.avatar_url}"
    将文本展示在页面上用th:text = "${model里传过来值.属性}"  进行展示。
    5.mybatis驼峰命名法则在application.properties配置：mybatis.configuration.map-underscore-to-camel-case=true
    
# 步骤十九：thymeleaf+pagehelper实现分页查询
[Pagehelper 官网](https://pagehelper.github.io/docs/howtouse/)
    1.先引入依赖
```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.12</version>
</dependency>

```
    2.分页的时候前端发起请求，进入index页面的时候，会携带pageNum(不是必须的)和pageSize（无要求）,后端判断两个参数的实际值，并设置默认
    然后调用startPage方法开始分页，注意紧跟着这个方法后面就是一个数据库查询语句的方法（XXXMapper.queryXXX），然后new PageInfo<XXX>
    就能够进行分页，最后设置到model里面去。
    
    2.1 问题：主要就是startPage后面紧跟的必须是一条查询语句，但是我们视频里的是先查出question的list，再遍历list，通过question表中的
    creator字段去User表再查一遍找头像，导致，pageHelper分页出来后total=pageSize。
    解决：但是，当我的查询语句中还包含一个关联查询的时候，这个时候我查询问题返回的List集合的泛型可以设置为QuestionDTO，查出来的数据包含
    空user，可以接受，然后再增强for循环遍历List，找到每一个question，通过creator值去查找User类中的头像的URL设置到questionDTO中，这样
    每个questionDTO对象就会都找到自己的用户头像了，再将这整个questionDTO添加到questionDTOList中去，set到pageInfo中去，并返回pageInfo
    对象，这样查出来的total就不会等于pageSize了
    参考博文：https://blog.csdn.net/zs40122/article/details/82620781
    
# 步骤二十：之前引入了JQuery，此时如何证明JQuery引入成功了呢？
在浏览器的控制台输入$符，会出来一串信息ƒ (e,t){return new k.fn.init(e,t)}，说明引入成功了
我们操作DOM的时候都是用JQuery去做
比如说我们用document.getElementsByClassName("main")来获取修饰css的class，可以这么操作，也可以用如下的JQuery去操作：$(".main")

# 如何让样式跟着选中的走？
```html
    <a href="/profile/questions"  th:class="(${action} == 'questions') ? ('list-group-item active'):('list-group-item')" >
```
    * 其中的小括号可以去掉
    
# 步骤二十一 ：在个人中心的“我的问题”栏位展示只属于我的问题
    为了能够在创建question的时候记录下唯一创建人，用user表的自增id肯定不行，因为id是变的，但是里面的accountId使用的是github账户用户的id
    是唯一的，因此在设置question的creator的时候，可以用user表中的accountId;
        小插曲：在创建字段的时候，user表中的accountId使用的是account_id，由于配置文件设置的映射是驼峰命名规则，导致项目启动后插入问题的时候
        account_id映射不到，报错“java.lang.NumberFormatException: null”
        解决办法：将user表中的account_id字段改成accountId,并把相应的引用位置改变即可
        
# 问题二十二： 配置拦截器
[spring配置拦截器官方文档地址](https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/web.html#mvc-handlermapping-interceptor)
点击实例创建拦截器
    注意  
        1.当拦截器interceptor中注入了UserMapper后，webConfig文件中就不能通过new去实例化一个对象，也要通过注入的方式。
        2.@EnableWebMvc使用后，代表你全面接管springMVC，之前的默认配置都没了，样式就很丑了，此时应该讲WebConfig中的@EnableWebMvc注解去除，就会有样式了
        3.每个controller怎么获取拦截器里的user？
  ```java
        //userFindByToken是拦截器里set的值
        User foundUserByToken = (User) request.getSession().getAttribute("userFindByToken");
  ```
 P30源码分析，再过一遍   
 
# 问题二十三：点击问题标题进入问题页面
    1.疑问：由于和匠哥之前通过question查找user是通过question的creator和user表中的id一致，我改成通过question查到user之后，在question中将creator赋值user
    表的account_id,现在都改成String类型了，在页面再判断就能判断出来，我想知道用这种String会不会影响速度
    2.为了整改登录验证是否已经存在user{若存在，则update，若不存在，则insert}，测试的时候，我把原始user数据删除了后，访问index页面的时候，发现页面报错，
    原因是我把user都删除了，之前首页是需要展示question的（所有user），当我删除之后，展现question的前端就找不到user了，近而找不到需要展示页面的参数，因此就报错了
    解决办法，将question表的数据也删除掉，这也证明了我的程序还是有bug的，后期再看看，先记录下
    3.当对登录进行验证了之后，我自己创建的将user表中的account_id赋值给question表的creator是没有必要的了
    4.当我修改account_id后，我的浏览器还有cookie的时候，还是能够登录的
    
# 问题二十四：退出登录
    1.删除cookie，退出服务端的session
   
# 编辑页面的传参
```html
    <input type="hidden" id="id" name = "id" th:value="${id}">
```
    这个传参的时候涉及PublishController的methoddoPublish获取值放到model中去，把这些属性set值，其中id的获取是在html中写上面代码获取的
   注意 name = "id"很重要，不然后台获取不到id，则不能根据id去寻找question，则不能判断是新增还是修改
   
# 问题二十五：集成Mybatis Generator
    Mybatis Generator寻求的主要影响是数据库增删改查这些简单的操作，例如join查询和存储过程的SQL代码和对象还需要自己编写
    Mapper中给参数加注解用@Param(value = "传入参数的形参名")
    
    1.在添加generatorConfig.xml的时候报红http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd，发现没有引入maven插件、依赖
    报错：URI is not registered (Settings | Languages & Frameworks | Schemas and DTD
    统一资源定位符没有注册
    操作：file-->settings...-->languages & frameworks --> Schemas and DTDs,进去添加注册一下就OK了 
       生成mapper文件和对应的Model等命令：
       mvn -Dmybatis.generator.overwrite=true mybatis-generator:generate
    2.章节34分页查询使用了MyBatisGenerator中的插件RowBounds
    3.我的MybatisGenerator不能生成诸如selectByPrimaryKey这样的工具，在标签jdbcConnection中添加如下代码段就生成了
```xml
    <property  name = "useInformationSchema" value = "true" />
```

# 问题二十六: 增加页面友好度p35  
    将报错做的更可视化一些，友好一些，把错误都包裹起来
    处理Whitelabel Error page：
        找springboot的官方文档：
[springboot的官方文档"Error Handling"](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-error-handling)
    1.先创建一个error.html页面;th:text="${message}"可将后台错误信息传值到前端去
    2.当创建一个advice的时候，再次debug的时候，就会发现页面并不会返回报错信息了，这个时候我们就需要通过Model把错误信息传值给前端
    3.在debug调试的过程中，我们发现导致查不到问题的原因是我们查到的缺少婷婷为空，那么怎么样才能将空这个原因传到前端呢，而不是通通
    报错服务端error，这个时候我们就需要自定义异常；
    4.自定义异常继承RuntimeException原因：在抛出异常的时候我不想在当前层try catch,只需要在ControllerAdvice进行处理try catch即可
    5.自定义异常后，在每一个可能抛出异常的地方 throw new XXXException("Exception description"),然后在ControllerAdvice中判断如果
    异常对象 instance XXXException,则将ex.getMessage()传给前端
    
# 问题二十七：使用枚举列出所有异常
    1.涉及改动的类有CustomErrorCodeEnum、CustomErrorCodeEnumImp、CustomException、QuestionService(修改抛出异常)
    2.处理在8080端口后随便输入参数的情况：报错No message available
    在IDEA按住Ctrl+Shift+F查找抛出异常在什么地方功能名称Find in path
    
    3.如何处理诸如输入错误抛出来的No message available异常呢？
        3.1 这个异常，我们了解到它是源码内部提供的，我们可以 以实现ErrorController来处理这种异常
        3.2 BasicErrorController中BasicErrorController就是在ErrorMvcAutoConfiguration中实现的，里面的errorHtml方法就是实现处理异常的
        方法，发现方法中需要使用getStatus方法，粘贴到自己的ErrorController方法中去
        3.3 将getStatus方法中的protected改成private;
   代码：
```java
        @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
            public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
                HttpStatus status = getStatus(request);
                Map<String, Object> model = Collections
                        .unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
                response.setStatus(status.value());
                ModelAndView modelAndView = resolveErrorView(request, response, status, model);
                return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
            }

```
    删除后：
```java
        @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
        public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
            HttpStatus status = getStatus(request);
            
            return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
        }
        
 //getStatus方法中的代码如下
        private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        try {
            return HttpStatus.valueOf(statusCode);
        }
        catch (Exception ex) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
    }
```
通用的处理异常我们可以实现自定义的ErrorController来处理异常，同时通过ControllerAdvice配合ExceptionHandler去处理上下文中的业务
异常的透传。但是要注意这个异常要继承RuntimeException，这样不会影响我们其他的代码，只需要在CustomExceptionHandler去拦截就好了，
但是这个拦截，他拦截的是所有我们SpringMVC可以handler的异常；不能handler包括404我们应该怎么办呢，我们需要做一个通用的Controller->
CustomErrorController去处理，当没有拦截住呢，4xx的请求我们需要告诉你：你没有拦截住，直接失误了，根本没有路由到任何mapper中去
所以根本不会去ControllerAdvice中去

1. 分一异常，与业务相关的异常或者与持久化数据相关的异常肯定要回滚，否则会产生数据错误，就是生产事故
2. 异常就直接回滚了，这波操作流程做不下去的   
3. 如果是那种网络异常，或者缓存处出现异常，对持久化数据没有影响的，可以先让用户通过，最终出现的问题也
是在缓存中，缓存是不对业务主数据造成影响的，除非你将缓存的数据持久化了;一般缓存的数据不会持久化
4. 举个例子，比如微博中的热搜假如是根据缓存来排的，正好你在用微博的时候，出现缓存一场，没有计算出来热搜排行版第一，或者说第一的热搜
计算错误，这时候针对这个计算错误的异常，处理就是不给你显示这条热搜信息，但是其他热搜还是给你看;
影响不到人家主数据，就可以不报错，但是如果你在发表微博的时候，这是后服务端出现了错误，肯定会提示你发布失败;
换句话说，你在浏览微博的时候，你浏览的记录因为缓存异常，没有将你的浏览的东西记录到缓存，这时候系统不会给你报错，因为你个人异常不会改
变什么，顶多会给你浏览的记录上热搜减少的一点机会而已，对系统丝毫没有任何变化;
5. 持久化过程的效率就看sql优化,什么框架都是要看sql写的好不好

# 问题二十八：增加阅读数
    1.我们这个做的是只要进入页面成功就会累加阅读数,从数据库获取viewCount，然后进行累加
    2.第一种方法其实在面对大量并发的时候就会拉闸，以为很有可能在同一时刻就会有很多用户去数据库取viewCount，当大家都拿到一个值的时候
    在此值的基础上加1，会丢失很多数据，因此，我们需要使用数据库自己的viewCount+1来表示查询到的viewCount
    3.实现这个功能，自己扩展mapper.xml，因为自动生成的mapper.xml，在每次改变表的时候会被覆盖，所以自己扩展一个，mapper.xml
    
# 问题二十九：添加评论功能
1. ajax局部刷新，使用异步处理方式把请求发到服务器端，得到响应之后直接处理
2. 查看FormDate->post_hash
3. 流程：
    + 进入问题提问的时候，提问的下方加载出来最新的所有的回复，可以按票数回复，时间倒序回复，我们实现的是时间倒序回复，
    + 同时下面有输入框，可以提交问题，点击回复的时候，异步调用服务器的请求，请求成功以后局部刷新，把内容追加上来，并且修改回复数
    + 前后端交互的接口是怎么实现的：
        * 设计表，先加载列表页面，再加载列表页面里面的子评论
        * 前端传过来请求的时候，我们要拿到一个json，服务端拿到这个json之后，反序列化成自己的对象，再做操作，然后回给前端也是返回一个java的
        object，让spring把object转换成json
        * 我们可以直接拿到一个自动封装成CommentDTO的请求体RequestBody,在controller中如下使用RequestBody，对前端传过来的json数据
        进行反序列化，序列化成CommentDTO对象：
        * 报错：template might not exist or might not be accessible by any of the configured Template Resolvers
            * 加注解：@ResponseBody ,指定返回的格式是一个json格式
```java
        public Object postComment(@RequestBody CommentDTO commentDTO){

             return false;
        }
```

# 问题三十： 关于评论模块的异常处理
1. 直观的我们可以在添加了评论之后，将处理成功的状态放在hashMap中返回给前台一个json格式的状态
```java
        package com.community.controller;

        import com.community.dto.CommentCreateDTO;
        import com.community.mapper.CommentMapper;
        import com.community.model.Comment;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.stereotype.Controller;
        import org.springframework.web.bind.annotation.*;
        
        import java.util.HashMap;
        
        /**
         * Created by 舒先亮 on 2019/9/9.
         */
        @Controller
        public class CommentController {
        
            @Autowired
            private CommentMapper commentMapper;
        
            @ResponseBody
            @RequestMapping(value = "/comment",method = RequestMethod.POST)
            public Object postComment(@RequestBody CommentCreateDTO commentCreateDTO){
                Comment comment = new Comment();
                comment.setParentId(commentCreateDTO.getParentId());
                comment.setContent(commentCreateDTO.getContent());
                comment.setType(commentCreateDTO.getType());
                comment.setGmtCreate(System.currentTimeMillis());
                comment.setGmtModified(comment.getGmtCreate());
                comment.setCommentator(1);
                comment.setLikeCount(0L);
                commentMapper.insert(comment);
                HashMap<Object, Object> objectObjectHashMap = new HashMap<>();
                objectObjectHashMap.put("message", "成功");
                return objectObjectHashMap;
            }
        }

```
2. 我们自己定义一个返回状态结果的DTO
```java
        package com.community.dto;

        import lombok.Data;
        
        /**
         * Created by 舒先亮 on 2019/9/9.
         */
        @Data
        public class ResultDTO {
        //    code是像code码一样，用来告诉前端当前是此状态码的状态
            private Integer code;
        //    message是用来提示的
            private String message;
        
            public static ResultDTO errorOf(Integer code,String message){
                ResultDTO resultDTO = new ResultDTO();
                resultDTO.setCode(code);
                resultDTO.setMessage(message);
                return resultDTO;
        
            }
        
        }
```
3. CommentController中添加判断用户是否存在的判断，如果用户不存在则用ResultDTO进行判断
```java
            package com.community.controller;
            
            import com.community.dto.CommentCreateDTO;
            import com.community.dto.ResultDTO;
            import com.community.mapper.CommentMapper;
            import com.community.model.Comment;
            import com.community.model.User;
            import org.springframework.beans.factory.annotation.Autowired;
            import org.springframework.stereotype.Controller;
            import org.springframework.web.bind.annotation.*;
            
            import javax.servlet.http.HttpServletRequest;
            import java.util.HashMap;
            
            /**
             * Created by 舒先亮 on 2019/9/9.
             */
            @Controller
            public class CommentController {
            
                @Autowired
                private CommentMapper commentMapper;
            
                @ResponseBody
                @RequestMapping(value = "/comment",method = RequestMethod.POST)
                public Object postComment(@RequestBody CommentCreateDTO commentCreateDTO,
                                          HttpServletRequest request){
            
                    User user = (User) request.getSession().getAttribute("userFindByToken");
                    if(user == null){
                        return ResultDTO.errorOf(2002, "当前用户未登录，不能进行评论，请先登录");
                    }
                    Comment comment = new Comment();
                    comment.setParentId(commentCreateDTO.getParentId());
                    comment.setContent(commentCreateDTO.getContent());
                    comment.setType(commentCreateDTO.getType());
                    comment.setGmtCreate(System.currentTimeMillis());
                    comment.setGmtModified(comment.getGmtCreate());
                    comment.setCommentator(user.getId());
                    comment.setLikeCount(0L);
                    commentMapper.insert(comment);
                    HashMap<Object, Object> objectObjectHashMap = new HashMap<>();
                    objectObjectHashMap.put("message", "成功");
                    return objectObjectHashMap;
                }
            }
```
4. 以2开头的，都是系统级的异常；

## 建表语句
```mysql
  CREATE TABLE `community`.`comment` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `parent_id` BIGINT NOT NULL COMMENT '指向问题的id，父类id',
  `type` INT NULL COMMENT '父类类型，type可以定义为枚举类，1的时候做什么事情，2的时候做什么事情',
  `commentator` INT NULL COMMENT '评论人id',
  `content` VARCHAR(1024) NULL,
  `gmt_create` BIGINT NOT NULL COMMENT '创建时间',
  `gmt_modified` BIGINT NOT NULL,
  `like_count` BIGINT NULL DEFAULT 0,
  PRIMARY KEY (`id`));
```

# 什么是json？
   + 它是一种网络传输数据结构：不同语言之间传递对象，约定俗成这么个语言json来传递
    
# 问题三十一：实现回复功能-添加事务
1. 当从页面提交过来一个评论的请求，如对回复问题进行操作，它会进行两个操作，一个是插入评论，另一个是更新问题的评论数：当提交评论后，由于网络延迟
或者数据库抖动，导致插入评论的动作成功，更新问题评论数抛异常，则会产生更新了评论，却没有更新评论数的现象
2. 解决办法：在方法上加Spring提供的注解@Transactional，把整个的方法体加上一个事务；当两个动作都包裹在一个事务中的时候，若有一个动作失败了，整
体就会回滚掉
3. Spring有AOP的技能加持，方法打一个@Transactional注解，Spring就自动的管理这个事务，P39事务的源码分析
4. 前端页面的编写，根据以往的语法和bootstrap模板，使用community.js;在回复的button绑定一个onclick事件,绑定到post()这么一个方法,post()方法在
     community.js中实现,前端向后端异步传入JSON对象的方法实现：
```js
    function post() {
        var questionId = $("#question_id").val();
        var content = $("#comment_content").val();
        $.ajax({
            type: "POST",
            url: "/comment",
            data: JSON.stringify({
                "parentId":questionId,
                "content":content,
                "type":1
            }),
            success: function (response) {
                console.log(response)
            },
            dataType: "json",
            contentType:"application/json"
        });
        console.log("所评论问题id " + questionId);
        console.log("评论内容 " + content);
    }
```
*     注意：url要回到根目录；
*     通过前端页面标签的ID，获取对象，然后在data中一一对应起来，和commentDTO中的属性对应，这里注意data的格式必须是
     JSON格式，所以用JSON.Stringify()将对象处理下；
*    我还添加了contentType:"application/json"，这也是一个必要属性


5. local storage
6. 前端调试是在js代码中加入：debugger;
7. 炉石：缺法力飓风、血法师萨尔诺斯
8. <h1></h1>标题标签传值可以用th:text
9. <a>标签，要修饰margin，需要加上display：block
10. 在判断评论内容是否为"" 或 null，我们用或者判断了两次
```java
            if(commentCreateDTO == null || commentCreateDTO.getContent() == "" || commentCreateDTO.getContent() == null){
                return ResultDTO.errorOf(CustomErrorCodeEnumImp.CONTENT_IS_EMPTY);
            }
```
这里我们引入commons.lang3的包，使用StringUtils的isBlank()方法判断是否是以上的两种情况，比较方便
11. 提交问题后，window.location.reload()重新加载一下页面。这样刷新 了整个页面，不好
12. 在前端也校验一下content是否为空，即!content
13. 我么设置评论的顺序为按照时间顺序倒序排
```java
    commentExample.setOrderByClause("gmt_create desc");
```

# 问题三十二：实现二级评论功能
1. 首先对于一个问题，点击进入后，先展示一级评论（考虑的是当问题比较多的话，会这影响页面的加载速度）
2. 当鼠标悬浮在评论图标的时候，图标高亮，点击后会触发一个弹框，将一级评论的子评论都查询出来并展示在一级评论的页面下面
3. 前端用th:data-id获取当前评论的id，绑定在onclick的方法上，将整个span对象传给onclick的方法上，然后在js代码就能获取到span对象，
通过 对象.getAttribute("data-id");就能获取到id。 然后通过id选择器就能选到二级评论模块的div，在js代码中通过getAttribute获取是否addClass("in")的标志
data-collapse，有则remove Class和Attribute，没有则设置Class和Attribute
4. 点击之后触发active样式被添加|删除到页面的class中去： e.classList.remove("active");|e.classList.add("active");
5. 二级评论套用一级评论展示的逻辑
6.     border-radius: 20px;修饰边框为圆角；border: 2px solid #ebebeb;给div块加一个外边框，宽度2px
7. 鼠标悬浮，光标变小手cursor: pointer;添加hover可以悬浮添加样式
8. 在后端controller处理的时候，queryListByTargetId查询出来的数组用ResultDTO返回一个数组，但是在ResultDTO中没有这样的方法，于是需要在ResultDTO中重新修改，
增加方法。
* 由于传给ResultDTO的可能是一个List，也可能会是一个Object，于是这里就可以引入了泛型的概念在ResultDTO类上面
9. 在js代码的展开、折叠方法collapseComment中，触发展开的时候，获取新的请求，拿到标签属性后，将后端获取到的内容写到页面上去，二级评论的这段代码，是要在js代码中手写去完成的
```html
    <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12 collapse sub-comment"
                             th:id="${'comment-'+comment.id}">
                            <!--展示二级评论(套用一级评论HTML代码)-->
                            <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
                                <h4>
                                    <span th:text="${question.commentCount}"></span> 个回复
                                </h4>
                                <hr>
                                <div class="comment-foreach-sub" th:each="comment : ${comments}">
                                    <div class="media-left">
                                        <a href="/index">
                                            <img class="media-object img-rounded"
                                                 th:src="${comment.user.avatarUrl}">
                                        </a>
                                    </div>
                                    <div class="media-body menu-tag-sub ">
                                        <a th:href="'/index'">
                                            <span class="media-heading" style="font-size: 18px;">亮亮</span>
                                        </a> •
                                        <span th:text="${#dates.format(comment.gmtCreate,'yyyy-MM-dd')}"
                                              class="date-tag"> </span>
                                        <span class="glyphicon glyphicon-comment icon pull-right" aria-hidden="true"
                                              th:data-id="${comment.id}"
                                              onclick="collapseComment(this)"></span>
                                        <br>
                                        <span th:text="${comment.content}"></span><br>
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
                                <input type="text" class="form-control" placeholder="请输入你的评论..."
                                       th:id="${'input-'+comment.id}">
                                <button type="button" class="btn btn-success pull-right sub-comment-btn"
                                        onclick="comment(this)" th:data-id="${comment.id}">回复
                                </button>
                            </div>
                        </div>
```
10. jquery中有getJSON,将后台查到的数据查到页面上，用js拼接
当点击collapseComment时，往th:id="${'comment-body-'+comment.id}"这个id追加标签；
 * 去js代码中追加标签
 * 关于时间的格式化，可以使用moment js,有一个data format处理方法
[js中格式化时间的网站momentjs](http://momentjs.cn/)
11. 查询数据倒序排列@Select("select * from question order by gmt_create desc")
 
# 问题三十二：实现模糊查询，相似推荐
在数据库我们可以这样模糊查询字段相关的数据：
```mysql
  select id,title,tag from question where (tag like "%Spring%" or tag like "%Boot%"  or tag like "%Java%") and id != 110 ;
```
1. 加括号是排除了id = 110的数据，不加括号排除不了
2. 以上的方式我们也可以使用正则表达式实现
```mysql
  select id,title,tag from question where tag regexp 'Spring|Boot|Java' and id != 110;
```
3. HTML <input> autocomplete 属性启动自动完成功能
4. js中通过input框的id属性拿到前一次的值，如果有值，则在后面追加逗号并把当前value追加上去，若没有值，则直接赋值
```js
    function selectTags(value) {
    var previous = $("#tag").val();
    if (previous) {
        var currentVal = $("#tag").val(previous + ',' + value);
    } else {
        $("#tag").val(value);
    }
}
```

5. 增加判断插入的标签值是否已经存在
```js
    function selectTags(value) {
    var previous = $("#tag").val();
    if (previous.indexOf(value) == -1) {
        //    说明之前的值是存在的了
        //    不追加
        //    不存在的话就添加 == -1 说明说明不存在
        if (previous) {
            var currentVal = $("#tag").val(previous + ',' + value);
        } else {
            $("#tag").val(value);
        }
    }
}
```
6. 将展示标签的导航栏用一个大的div封装起来，当点击input框的时候，让display: block;导航栏就会展示出来，当变成
display: none;时，导航栏就会消失
 * 实现方式：给input框增加一个onclick事件，和大的div通过id进行绑定
```js
    /**
 * 当点击input框的时候，改变display：none属性为，display:block,将标签导航栏展示出来
 */
    function showSelectTags() {
        //jquery自带的方式，将style中的display:none属性改变，展示大div
        var val = $("#select-tag").show();
        console.log(val);
    
    }
```
7. 点击其他地方的时候，这个标签展示的界面消失，可以考虑得焦失焦，focus and blur and on、遮蔽层解决
8. 将所有的标签写在java代码中catch文件夹中
9. 提交标签的时候校验一下，必须符合缓存cache中录入的字段,校验在publishController中
10. 如何在th:each遍历的时候，发现当遍历的子对象是第一个的时候就展示呢？
    * th:each遍历的时候有诸如 th:each="selectCategory,selectCategoryStat: ${tags}" th:class="${selectCategoryStat.first?'tab-pane active':'tab-pane'}"
    selectCategoryStat判断是first，则使用三目运算符追加class
    * 也可以实现js代码中showSelectTags()对应的的li的first-child的class加active
    * 还可以用th:classappend
    
# 问题三十三：当别人回复时，记录一下通知加一
1. 新建表
```mysql
  CREATE TABLE `community`.`notification` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `notifier` BIGINT NULL COMMENT '通知人',
  `receiver` BIGINT NULL,
  `outerid` BIGINT NULL,
  `type` INT NULL,
  `gmt_create` BIGINT NULL,
  `status` INT NULL DEFAULT 0 COMMENT '已读未读',
  PRIMARY KEY (`id`))
COMMENT = '用户通知';
```
2. 创建枚举
    * 优点，当看到这个枚举的时候，下意识就知道代码中包含的数字是干什么的，也方便代码的重构
    
# 问题三十四：增加markdown富文本编辑功能
[markdown富文本编辑功能开源网址](https://pandao.github.io/editor.md/)
1. 下载源码，将需要的文件相应的放在项目的static目录下
2. 将文本域获取markdown样式
```html
    <link rel="stylesheet" href="/css/editormd.css">
    <script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>    
    <script src="/js/editormd.min.js" type="application/javascript"></script>

    <div class="form-group" id="editor-publish">
                    <textarea name="description" th:text="${description}" id="description" cols="30" rows="10"
                              class="form-control" style="display:none;"
                              placeholder="请输入问题描述..."></textarea>
                </div>
                <script type="text/javascript">
                    $(function () {
                        var editor = editormd("editor-publish", {
                            width: "100%",
                            height: 350,
                            path: "js/lib/",
                            delay: 0,
                            placeholder: "请编辑问题描述..."
                        });
                    });
                </script>
```
3. markdown to HTML
    * 将数据库中查询到的markdown格式内容展示在页面上
```html
    <link rel="stylesheet" href="/css/editormd.preview.css">
    <script src="/js/editormd.js" type="application/javascript"></script>
    <script src="/js/lib/marked.min.js" type="application/javascript"></script>
    <script src="/js/lib/prettify.min.js" type="application/javascript"></script>
    <div id="markdown-view-showQuestion">
                    <!-- Server-side output Markdown text -->
                    <textarea style="display:none;" th:text="${question.description}"></textarea>
                </div>
                <script type="text/javascript">
                    $(function() {
                        var testView = editormd.markdownToHTML("markdown-view-showQuestion", {
                            // markdown : "[TOC]\n### Hello world!\n## Heading 2", // Also, you can dynamic set Markdown text
                            // htmlDecode : true,  // Enable / disable HTML tag encode.
                            // htmlDecode : "style,script,iframe",  // Note: If enabled, you should filter some dangerous HTML tags for website security.
                        });
                    });
                </script>
```
4. 注意当点击浏览问题时，我们是根据问题的id找到问题的，例如这个路径：http://localhost:8080/question/122
 * 当需要对问题进行编辑的时候，跳转到编辑页面，我们是根据问题编号，找到问题，并展示在publish.html;http://localhost:8080/publish/122
 * 如果js代码中path: "js/lib/",意味着在122所在的目录中找js文件，显然是找不到的，path: "/js/lib/，则可以在相对的路径下找到js代码。markdown edit在寻找js
 * 根据官方文档的upload Image 操作，还需要注意添加插件
    
    
# 问题三十五：添加日志
1. 
```properties
    #打出日志的方式
    logging.file=logs/community.log
    logging.level.root=INFO
    logging.level.com.community.mapper=DEBUG
    #生产标准的最多看30天日志，每个日志200MB
    logging.file.max-history=30
    logging.file.max-size=200MB
```
2. 追加日志
类上添加@Slf4j
log.error("callBack get github error,{}",gitHubUser);


3. 打开虚拟机后，在windows界面下输入ssh root@虚拟机ip;eg:ssh root@192.168.255.128,回复yes加输入虚拟机密码，就能访问到虚拟机了
    * fdisk -l 查看挂在磁盘
        
## 部署
### 依赖
- 安装git
- 用maven执行，那么我们就需要JDK
- 编译java，就要安装maven工具
- 安装MySQL
   * springboot项目，不需要tomcat

## 步骤
- yum update  从数据源更新centos的数据源
- y 更新安装
- yum install git  安装git 点击y  ,之后就安装成功了
- git status 查看git状态，为安装成功
- pwd 在root目录下面，创建一个文件夹
- mkdir App ;cd App;ls;
- git clone +项目地址;拉去github项目
- ls 当前目录下有项目community
- cd community  进入项目
- mvn compile 执行maven文件，但是发现命令用不了，于是就需要安装maven
- yum install maven
### 一、两步安装maven

+ 下载包

wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

+ 安装maven
yum -y install apache-maven

### 二、查找maven安装路径

+ 查找包路径

rpm -qa|grep apache-maven

+ 根据包路径查找安装目录

rpm -ql apache-maven-3.5.2-1.el7.noarch

在搜索结果中就有maven的安装目录。
 - mvn -v  查看maven版本号
 - mvn clean compile package 清除之前的并打包
## 重新安装一下JDK
 - rpm -qa | grep java 检查自己linux中是否有已经安装的
 - rpm -e xxx --nodeps ;如果安装了，通过 rpm -e xxx --nodeps 命令进行装卸，xxx表示你通过 rpm -qa | grep java 命令 查到的安装包的名字
 - cd /  返回根目录
 - cd usr/local/  通过该命令进入根目录下的usr目录下的local目录，这个目录是放一些本地的共享资源的
 - mkdir java  创建java文件夹
 - cd java  进入文件夹
 - tar -xvf jdk-8u191-linux-x64.tar.gz 把linux的java压缩包放进来，解压
 - ll  查看解压后得到的文件夹
 -  rm -rf jdk-8u191-linux-x64.tar.gz 删除压缩包
 - cd jdk1.8.0_191/ 进入文件夹
 - pwd 获取 /usr/local/java/jdk1.8.0_191，后面用
 - 在文件末尾将下面倒数四段配置代码追加在前两行后面
 
```properties
    JAVA_HOME=/usr/local/java/jdk1.8.0_191
    CLASSPATH=.:$JAVA_HOME/lib.tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH
```
 - source /etc/profile 使更改的配置立即生效
 - java -version 命令和 javac -version 命令来查看 jdk 是否安装成功
 
## linux安装MySQL
 - rpm -qa | grep MySQL 检查自己linux中是否有已经安装的
 - rpm -qa | grep mysql
 - rpm -e xxx --nodeps ;
 - 复制粘贴文件　　cp  [选项]  源文件或目录  目标文件或目录
 - 剪切粘贴文件　　mv [选项]  源文件或目录  目标文件或目录
 - 删除文件　　　　rm 文件　　　　　　慎用 rm -rf  
 - wget http://dev.mysql.com/get/mysql57-community-release-el6-7.noarch.rpm  ;下载MySQL
 - rpm -ivh mysql57-community-release-el6-7.noarch.rpm ;安装下载的mysql仓库
 - yum repolist enabled ;查看源的启用情况,查看源的启用情况
 - yum list mysql-community* ;查看下现在mysql的版本
 - yum -y install mysql-server;安装mysql5.7.27
 - rpm -qa | grep mysql ;查看安装情况
 - service mysqld start ;启动
 - service mysqld start ；设置为开机启动
 - grep 'temporary password' /var/log/mysqld.log ; mysql安装成功后创建的超级用户'root'@'localhost'
 的密码会被存储在/var/log/mysqld.log，可以使用如下命令查看密码。初始密码：UkAYvqs&d5,/
 - 使用mysql生成的'root'@'localhost'用户和密码登录数据库，并修改 其密码，具体命令
   * mysql -uroot -p  登录并修改密码
   * ALTER USER 'root'@'localhost' IDENTIFIED BY 'Sxl1234!';
 - grant all on *.* to  test@"%" Identified by "test"; 创建用户，允许远程接入
 - FLUSH PRIVILEGES;
 - service mysqld restart 开通3306端口，就可以远程接入了,重启mysql服务:
 - service mysqld stop
 - netstat -tunlp|grep 3306  ;启动成功后执行netstat -tunlp|grep 3306就可以看到mysqld已经启动了3306端口的监听
 - mysqldump -u root -p community > E:\programmer\a.sql   ；导出数据库文件
 - CREATE DATRABSE [数据库名字]创建一个数据库，然后使用use [数据库名]选择要使用的数据库
 - source [文件路径]  ；导入sql语句
 - 此时就能执行mvn compile package
 - cp src/main/resources/application.properties src/main/resources/application-production.properties   复制一份配置文件
 - vi src/main/resources/application-production.properties   编辑生产配置文件
 - spring boot profile
 - java -jar -Dspring.profiles.active=production target/demo-0.0.1-SNAPSHOT.jar     //进入community文件夹后执行该命令
 - 之前在index页面的登录按钮链接中的重定向IP是写死的，考虑能不能更换，thymeleaf获取当前项目的路径
 - ps -aux | grep java 检查当前进程是否存在
 - 192.168.255.128:81   localhost:8080
 
## 网络映射
 - VMware 的 编辑-虚拟网络编辑器 需要管理员权限才能修改
 - ifconfig |grep 192.168     查看虚拟机的IP
 - 选择VMnet8-NAT设置-点击添加
   * 主机端口为未被占用的主机端口
   * 虚拟机IP地址为之前查到
   * 虚拟机端口-为项目启动的端口
   * 描述任意写，最后点击确认添加
 - 最后修改下生产配置文件和认证授登录的github重定向URI为主机IP：端口号
 - 可成功访问
 - 备注：手机设备访问的是路由器的ip地址
 
 
 







    
    
    
    
    
    
    
    
           
    

# 问题
   * 这个项目是什么类型的软件，目标用户是什么群体，我负责哪些模块，这些说完，要有突出的亮点
   * 商品添加的功能，添加的时候也可以加一些别的逻辑
   * Java如何将方法作为参数进行传递
    
    

    
   


