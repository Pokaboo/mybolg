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
