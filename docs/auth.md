## 通用权限系统

### 介绍

- 权限管理是所有后台系统都会涉及的一个重要组成部分，而权限管理的核心流程是相似的，如果每个后台单独开发一套权限管理系统，就是重复造轮子，是人力的极大浪费，本项目就是针对这个问题，提供了一套通用的权限解决方案。

- 项目服务器端架构：SpringBoot + MyBatisPlus + SpringSecurity

- 前端架构：Node.js + Npm + Vue + ElementUI + Axios

  | 基础框架：SpringBoot                              |
  | ------------------------------------------------- |
  | 数据缓存：Redis                                   |
  | 数据库：Mysql                                     |
  | 权限控制：SpringSecurity                          |
  | 全局日志记录：AOP                                 |
  | 前端模板：vue-admin-template                      |
  | 前端技术：Node.js + Npm + Vue + ElementUI + Axios |

### 项目搭建

```
pokaboo-auth-parent：根目录，管理子模块：

​	common：公共类父模块

​		common-log：系统操作日志模块

​		common-util：核心工具类

​		service-util：service模块工具类

​		spring-security：spring-security业务模块

​	model：实体类模块

​	service-system：系统权限模块
```

### 数据库

### 角色管理

- 定义统一返回结果对象：

  > 项目中我们会将响应封装成json返回，一般我们会将所有接口的数据格式统一， 使前端(iOS Android, Web)对数据的操作更一致、轻松。一般情况下，统一返回数据格式没有固定的格式，只要能描述清楚返回的数据状态以及要返回的具体数据就可以。但是一般会包含状态码、返回消息、数据这几部分内容例如，我们的系统要求返回的基本数据格式如下：

  - 列表：

    ```json
    {
      "code": 200,
      "message": "成功",
      "data": [
        {
          "id": 2,
          "roleName": "系统管理员"
        }
      ],
      "ok": true
    }
    
    ```

  - 分页:

    ```json
    {
      "code": 200,
      "message": "成功",
      "data": {
        "records": [
          {
            "id": 2,
            "roleName": "系统管理员"
          },
          {
            "id": 3,
            "name": "普通管理员"
          }
        ],
        "total": 10,
        "size": 3,
        "current": 1,
        "orders": [],
        "hitCount": false,
        "searchCount": true,
        "pages": 2
      },
      "ok": true
    }
    ```

  - 没有返回数据

    ```json
    {
      "code": 200,
      "message": "成功",
      "data": null,
      "ok": true
    }
    ```

- 代码实例：

  ```java
  
  import lombok.Data;
  
  /**
   * 全局统一返回结果类
   *
   */
  @Data
  public class Result<T> {
  
      //返回码
      private Integer code;
  
      //返回消息
      private String message;
  
      //返回数据
      private T data;
  
      public Result(){}
  
      // 返回数据
      protected static <T> Result<T> build(T data) {
          Result<T> result = new Result<T>();
          if (data != null)
              result.setData(data);
          return result;
      }
  
      public static <T> Result<T> build(T body, Integer code, String message) {
          Result<T> result = build(body);
          result.setCode(code);
          result.setMessage(message);
          return result;
      }
  
      public static <T> Result<T> build(T body, ResultCodeEnum resultCodeEnum) {
          Result<T> result = build(body);
          result.setCode(resultCodeEnum.getCode());
          result.setMessage(resultCodeEnum.getMessage());
          return result;
      }
  
      public static<T> Result<T> ok(){
          return Result.ok(null);
      }
  
      /**
       * 操作成功
       * @param data  baseCategory1List
       * @param <T>
       * @return
       */
      public static<T> Result<T> ok(T data){
          Result<T> result = build(data);
          return build(data, ResultCodeEnum.SUCCESS);
      }
  
      public static<T> Result<T> fail(){
          return Result.fail(null);
      }
  
      /**
       * 操作失败
       * @param data
       * @param <T>
       * @return
       */
      public static<T> Result<T> fail(T data){
          Result<T> result = build(data);
          return build(data, ResultCodeEnum.FAIL);
      }
  
      public Result<T> message(String msg){
          this.setMessage(msg);
          return this;
      }
  
      public Result<T> code(Integer code){
          this.setCode(code);
          return this;
      }
  }
  ```

  ```java
  import lombok.Getter;
  
  /**
   * 统一返回结果状态信息类
   *
   */
  @Getter
  public enum ResultCodeEnum {
  
      SUCCESS(200,"成功"),
      FAIL(201, "失败"),
      SERVICE_ERROR(2012, "服务异常"),
      DATA_ERROR(204, "数据异常"),
      ILLEGAL_REQUEST(205, "非法请求"),
      REPEAT_SUBMIT(206, "重复提交"),
      ARGUMENT_VALID_ERROR(210, "参数校验异常"),
  
      LOGIN_AUTH(208, "未登陆"),
      PERMISSION(209, "没有权限"),
      ACCOUNT_ERROR(214, "账号不正确"),
      PASSWORD_ERROR(215, "密码不正确"),
      LOGIN_MOBLE_ERROR( 216, "账号不正确"),
      ACCOUNT_STOP( 217, "账号已停用"),
      NODE_ERROR( 218, "该节点下有子节点，不可以删除")
      ;
  
      private Integer code;
  
      private String message;
  
      private ResultCodeEnum(Integer code, String message) {
          this.code = code;
          this.message = message;
      }
  }
  ```


### 集成knife4j

文档地址：https://doc.xiaominfo.com/

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案。

- 添加依赖

  ```xml
  <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
  </dependency>
  ```

- 添加knife4j配置类

  ```java
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import springfox.documentation.builders.ApiInfoBuilder;
  import springfox.documentation.builders.ParameterBuilder;
  import springfox.documentation.builders.PathSelectors;
  import springfox.documentation.builders.RequestHandlerSelectors;
  import springfox.documentation.schema.ModelRef;
  import springfox.documentation.service.ApiInfo;
  import springfox.documentation.service.Contact;
  import springfox.documentation.service.Parameter;
  import springfox.documentation.spi.DocumentationType;
  import springfox.documentation.spring.web.plugins.Docket;
  import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;
  
  import java.util.ArrayList;
  import java.util.List;
  
  /**
   * knife4j配置信息
   */
  @Configuration
  @EnableSwagger2WebMvc
  public class Knife4jConfig {
  
      @Bean
      public Docket adminApiConfig(){
          List<Parameter> pars = new ArrayList<>();
          ParameterBuilder tokenPar = new ParameterBuilder();
          tokenPar.name("token")
                  .description("用户token")
                  .defaultValue("")
                  .modelRef(new ModelRef("string"))
                  .parameterType("header")
                  .required(false)
                  .build();
          pars.add(tokenPar.build());
          //添加head参数end
  
          Docket adminApi = new Docket(DocumentationType.SWAGGER_2)
                  .groupName("adminApi")
                  .apiInfo(adminApiInfo())
                  .select()
                  //只显示admin路径下的页面
                  .apis(RequestHandlerSelectors.basePackage("com.atguigu"))
                  .paths(PathSelectors.regex("/admin/.*"))
                  .build()
                  .globalOperationParameters(pars);
          return adminApi;
      }
  
      private ApiInfo adminApiInfo(){
  
          return new ApiInfoBuilder()
                  .title("后台管理系统-API文档")
                  .description("本文档描述了后台管理系统微服务接口定义")
                  .version("1.0")
                  .contact(new Contact("qy", "http://pokaboo.cn", "pokaboo@163.com"))
                  .build();
      }
  
  }
  ```

- 注解

  - @Api ：添加在控制器类上，通过此注解的tags属性，可以指定模块名称，并且，在指定名称时，建议在名称前添加数字作为序号，Knife4j会根据这些数字将各模块升序排列，例如：

    > ```java
    > @Api(value = "提供商品添加、修改、删除及查询的相关接⼝",tags = "01.商品管理")
    > ```

  - @ApiOpearation：添加在Api中处理请求的方法上，通过此注解的value属性，可以指定业务/请求资源的名称，例如：

    > ```java
    > @ApiOperation("添加商品")
    > ```

  - @ApiOperationSupport：添加在Api中处理请求的方法上，通过此注解的order属性（int），可以指定排序序号，Knife4j会根据这些数字将各业务/请求资源升序排列，例如：

    > ```java
    > @ApiOperationSupport(order = 100) 
    > ```

  - @ApiImplicitParams 和 @ApiImplicitParam：对于处理请求的方法的参数列表中那些未封装的参数（例如String、Long），需要在处理请求的方法上使用此注解来配置参数的说明，并且，必须配置name属性，此属性的值就是方法的参数名称，使得此注解的配置与参数对应上，然后，再通过value属性对参数进行说明，还要注意，此属性的required属性表示是否必须提交此参数，默认为false。另外，还可以通过dataType配置参数的数据类型，如果未配置此属性，在API文档中默认显示为string，可以按需修改为int、long等。例如：

    > ```java
    > @ApiImplicitParams({
    >  @ApiImplicitParam(dataType = "string",name = "username", value = "⽤户登录账号",required =
    > true),
    >  @ApiImplicitParam(dataType = "string",name = "password", value = "⽤户登录密码",required =
    > false,defaultValue = "111111")
    > })
    > ```

  - @ApiModel：用来对实体类进行说明，例如：

    > ```java
    > @ApiModel(value = "User对象",description = "⽤户信息")
    > ```

  - @ApiModelProperty:作用在实体类的参数上，如果处理请求时，参数是封装的POJO类型，需要对各请求参数进行说明时，应该在此POJO类型的各属性上使用此注解，通过此注解的value属性配置请求参数的名称，通过requeired属性配置是否必须提交此请求参数（并不具备检查功能），例如：

    > ```java
    > @ApiModelProperty(dataType = "String",required = true, value = "⽤户注册账号")
    > ```

- 访问：http://ip:port/doc.html

​	![Knife4j](icon/Knife4j.png)



### JWT

- JWT是JSON Web Token的缩写，即JSON Web令牌，是一种自包含令牌。 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源。比如用在用户登录上。JWT最重要的作用就是对 token信息的防伪作用。

- **JWT令牌的组成**：一个JWT由三个部分组成：**JWT头、有效载荷、签名哈希**最后由这三者组合进行base64url编码得到JWT。典型的，一个JWT看起来如下图：该对象为一个很长的字符串，字符之间通过"."分隔符分为三个子串。https://jwt.io/

  ![JWT](icon/3402e929-2225-4c64-8f2e-4471b63366d0.png)

- **JWT头**：WT头部分是一个描述JWT元数据的JSON对象，通常如下所示。

  ```json
   {  
     "alg": "HS256",  
     "typ": "JWT"
   }
  在上面的代码中，alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；
  typ属性表示令牌的类型，JWT令牌统一写为JWT。
  最后，使用Base64 URL算法将上述JSON对象转换为字符串保存。
  ```

- **有效载荷**：有效载荷部分，是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。 JWT指定七个默认字段供选择。

  ```json
  iss: jwt签发者
  sub: 主题
  aud: 接收jwt的一方
  exp: jwt的过期时间，这个过期时间必须要大于签发时间
  nbf: 定义在什么时间之前，该jwt都是不可用的.
  iat: jwt的签发时间
  jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
  ```

  - 除以上默认字段外，我们还可以自定义私有字段，如下例：

    ```
    {
      "name": "Helen",
      "role": "editor",
      "avatar": "helen.jpg"
    }
    ```

  - 请注意，默认情况下JWT是未加密的，任何人都可以解读其内容，因此不要构建隐私信息字段，存放保密信息，以防止信息泄露。

  - JSON对象也使用Base64 URL算法转换为字符串保存。

- **签名哈希**：签名哈希部分是对上面两部分数据签名，通过指定的算法生成哈希，以确保数据不会被篡改。首先，需要指定一个密码（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用标头中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名。

  ```
  HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(claims), secret)    ==>   签名hash
  ```

  - 在计算出签名哈希后，JWT头，有效载荷和签名哈希的三个部分组合成一个字符串，每个部分用"."分隔，就构成整个JWT对象。

- **Base64URL算法**：如前所述，JWT头和有效载荷序列化的算法都用到了Base64URL。该算法和常见Base64算法类似，稍有差别。作为令牌的JWT可以放在URL中（例如api.example/?token=xxx）。 Base64中用的三个字符是"+"，"/"和"="，由于在URL中有特殊含义，因此Base64URL中对他们做了替换："="去掉，"+"用"-"替换，"/"用"_"替换，这就是Base64URL算法。

### 项目集成JWT

- 引入依赖

  ```xml
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt</artifactId>
  </dependency>
  ```

- 添加JWT帮助类

  ```java
  import io.jsonwebtoken.*;
  import org.springframework.util.StringUtils;
  
  import java.util.Date;
  
  /**
   * 生成JSON Web令牌的工具类
   */
  public class JwtHelper {
  
      private static long tokenExpiration = 365 * 24 * 60 * 60 * 1000;
      private static String tokenSignKey = "123456";
  
      public static String createToken(Long userId, String username) {
          String token = Jwts.builder()
                  .setSubject("AUTH-USER")
                  .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))
                  .claim("userId", userId)
                  .claim("username", username)
                  .signWith(SignatureAlgorithm.HS512, tokenSignKey)
                  .compressWith(CompressionCodecs.GZIP)
                  .compact();
          return token;
      }
  
      public static Long getUserId(String token) {
          try {
              if (StringUtils.isEmpty(token)) return null;
  
              Jws<Claims> claimsJws = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token);
              Claims claims = claimsJws.getBody();
              Integer userId = (Integer) claims.get("userId");
              return userId.longValue();
          } catch (Exception e) {
              e.printStackTrace();
              return null;
          }
      }
  
      public static String getUsername(String token) {
          try {
              if (StringUtils.isEmpty(token)) return "";
  
              Jws<Claims> claimsJws = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token);
              Claims claims = claimsJws.getBody();
              return (String) claims.get("username");
          } catch (Exception e) {
              e.printStackTrace();
              return null;
          }
      }
  
      public static void removeToken(String token) {
          //jwttoken无需删除，客户端扔掉即可。
      }
  
      public static void main(String[] args) {
          String token = JwtHelper.createToken(1L, "admin");//"eyJhbGciOiJIUzUxMiIsInppcCI6IkdaSVAifQ.H4sIAAAAAAAAAKtWKi5NUrJSCjAK0A0Ndg1S0lFKrShQsjI0MzY2sDQ3MTbQUSotTi3yTFGyMjKEsP0Sc1OBWp6unfB0f7NSLQDxzD8_QwAAAA.2eCJdsJXOYaWFmPTJc8gl1YHTRl9DAeEJprKZn4IgJP9Fzo5fLddOQn1Iv2C25qMpwHQkPIGukTQtskWsNrnhQ";//JwtHelper.createToken(7L, "admin");
          System.out.println(token);
          System.out.println(JwtHelper.getUserId(token));
          System.out.println(JwtHelper.getUsername(token));
      }
  }
  ```

### JWT认证流程

![JWT认证流程](icon/JWT认证流程.png)

- 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。

- 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT(Token)。形成的JWT就是一个形同lll.zzz.xxx的字符串。 token head.payload.singurater

- 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。

- 前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题) HEADER

- 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。

- 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。
  
