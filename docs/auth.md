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

  
