

##  分布式会话与单点登录SSO

### 分布式会话
- 什么是会话：

    ```
    会话session代表的是客户端与服务器的一次交互过程，这个过程可以是连续的也可以是时断时续的。
    对于servlet，一旦用户与服务器端交互，服务器tomcat就会为用户创建一个session，
    同时前端会有一个jsessionid，每次交互都会携带。如此一来，服务器只要在接到用户请求的时候，
    就可以拿到jsessionid，并根据这个id在内存中找到对于的session，当拿到session会话后，
    那么我们就可以操作会话了。会话存活期间，我们就能认为用户一直处于使用着网站的状态，
    一旦session超期过时，那就认为用户已经离开了网站，停止交互了。用户的身份信息，
    我们也是通过session来判断的，在session中可以保存不同用户的信息。
    
    ```

- 无状态会话：

    ```
    HTTP请求是无状态的，用户向服务器发起多个请求，服务端并不会知道这多次请求是来自同一个用户，
    这个是无状态的。cookie的出现就是为了有状态的记录用户。常见的，ios与服务器交互，安卓与服务器交互，
    前后端分离等待，都是通过发起http请求来调用接口数据的，每次交互服务端都不会拿到客户端的状态，
    但是我们可以通过手段去处理。比如每次用户发起请求的时候，携带一个userid或者user-token，
    如此一来，就能让服务器根据用户id或token来获得相应数据，
    每个用户的下一次请求都能被识别为来自同一个用户。
    
    ```

    

- 有状态会话：

    ```
    Tomcat的会话，就是有状态的。一旦用户和服务器交互，就有会话，会话保存了用户的信息，
    这样用户就有状态了。服务端会和每个客户端都保持着这样的一层关系，这个由容器来管理，
    这个session会话是保存到内存空间的，如此一来，当不同的用户访问服务端，
    那么就能通过会话知道是谁了。tomcat会话的出现也是为了让http请求变得有状态。
    如果用户不再和服务端交互，那么会话就会超时而消失，结束了他的生命周期，
    如此一来，每个用户其实都会有一个会话被维护，这就是有状态的会话。
    
    注：tomcat会话可以通过手段实现多个系统之间的状态同步，但是会损耗一定的时间，
    一旦发生同步那么用户请求就会等待，这种做法不可取。
    
    ```

- 单Tomcat会话：

    ```
    这个就有状态的，用户首次访问服务端，这个时候会话产生，并且会设置jsessionid放入cookie，
    后续每次请求都会携带jsessionid以保持会话状态
    ```

    [![cTYolQ.png](https://z3.ax1x.com/2021/04/19/cTYolQ.png)](https://imgtu.com/i/cTYolQ)

- 动静分离会话：

    ```
    用户请求服务端，由于前后端分离，前端发起http请求，不会携带任何状态，当用户第一次请求后，
    我们手动设置一个token，作为会话，放入redis中，如此作为redis-session，
    并且这个token设置后放入前端的cookie中，如此后续的交互，前端只需要传递token给后端，
    后端就能识别这个用户来自谁了。
    ```

    [![cTY7Os.png](https://z3.ax1x.com/2021/04/19/cTY7Os.png)](https://imgtu.com/i/cTY7Os)

- 集群分布系统会话

    ```
    集群或者分布式系统本质是多个系统，假设这个里有两个服务器节点，分别是AB系统，
    一开始用户和A系统交互，那么这个时候的用户状态，我们可以保存到redis中，
    作为A系统的会话信息，随后用户的请求进入B系统，那么B系统中的会话我也同样和redis关联，
如此AB系统的session就统一了。当然cookie是会随着用户的访问携带的。
    这个其实就是分布式会话，通过redis来保存用户的状态
    
    ```
    
    [![cTYTyj.png](https://z3.ax1x.com/2021/04/19/cTYTyj.png)](https://imgtu.com/i/cTYTyj)

### 用户会话
 - redis用户会话
 - SpringSession

### 分布式会话拦截器
- 前端：header中携带：headerUserId、headerUserToken

  ```javascript
  axios.post(
      serverUrl + '/userInfo/update?userId=' + userInfo.id,
      userInfoMore, {
        headers: {
          'headerUserId': userInfo.id,
          'headerUserToken': userInfo.userUniqueToken
        }
      })
    .then(res => {
      if (res.data.status == 200) {
        alert("用户信息修改成功!");
        window.location.reload();
      } else {
        alert(res.data.msg);
        console.log(res.data.msg);
      }
    });
  ```

- 后端：自定义拦截器，拦截请求

  ```java
  package com.pookaboo.interceptor;
  
  import com.pookaboo.utils.CustomJSONResult;
  import com.pookaboo.utils.JsonUtils;
  import com.pookaboo.utils.RedisOperator;
  import org.apache.commons.lang3.StringUtils;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.servlet.HandlerInterceptor;
  import org.springframework.web.servlet.ModelAndView;
  
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.io.OutputStream;
  
  /**
   * UserToken拦截器
   */
  public class UserTokenInterceptor implements HandlerInterceptor {
  
      public static final String REDIS_USER_TOKEN = "redis_user_token";
  
      @Autowired
      private RedisOperator redisOperator;
  
      /**
       * 拦截请求，在调用controller之前调用
       *
       * @param request
       * @param response
       * @param handler
       * @return
       * @throws Exception
       */
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
  
          String userId = request.getHeader("headerUserId");
          String userToken = request.getHeader("headerUserToken");
  
          if (StringUtils.isNotBlank(userId) && StringUtils.isNotBlank(userToken)) {
              String uniqueToken = redisOperator.get(REDIS_USER_TOKEN + ":" + userId);
              if (StringUtils.isBlank(uniqueToken)) {
                  returnErrorResponse(response, CustomJSONResult.errorMsg("请登录..."));
                  return false;
              } else {
                  if (!uniqueToken.equals(userToken)) {
                      returnErrorResponse(response, CustomJSONResult.errorMsg("账号在异地登录..."));
                      return false;
                  }
              }
          } else {
              returnErrorResponse(response, CustomJSONResult.errorMsg("请登录..."));
              return false;
          }
          return true;
      }
      
      
      /**
       * 拦截信息返回
       * @param response
       * @param customJSONResult
       */
      private void returnErrorResponse(HttpServletResponse response, CustomJSONResult customJSONResult) {
          OutputStream out = null;
          try {
              response.setCharacterEncoding("utf-8");
              response.setContentType("text/json");
              out = response.getOutputStream();
              out.write(JsonUtils.objectToJson(customJSONResult).getBytes("utf-8"));
              out.flush();
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (out != null) {
                      out.close();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
      /**
       * 在请求访问controller之后，渲染视图之前
       *
       * @param request
       * @param response
       * @param handler
       * @param modelAndView
       * @throws Exception
       */
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
  
      }
  
      /**
       * 在请求访问controller之后，渲染视图之后
       *
       * @param request
       * @param response
       * @param handler
       * @param ex
       * @throws Exception
       */
      @Override
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
  
      }
  }
  
  ```
  
- 自定义拦截器需要配置

  ```java
  package com.pookaboo.config;
  
  import com.pookaboo.interceptor.UserTokenInterceptor;
  import org.springframework.boot.web.client.RestTemplateBuilder;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.client.RestTemplate;
  import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
  import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
  
  @Configuration
  public class WebMvcConfig implements WebMvcConfigurer {
  
      @Bean
      public UserTokenInterceptor buildUserTokenInterceptor() {
          return new UserTokenInterceptor();
      }
  
      /**
       * 注册拦截器
       *
       * @param registry
       */
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(buildUserTokenInterceptor())
                  .addPathPatterns("/hello")
                  .addPathPatterns("/shopcart/add")
                  .addPathPatterns("/shopcart/del")
                  .addPathPatterns("/address/list")
                  .addPathPatterns("/address/add")
                  .addPathPatterns("/address/update")
                  .addPathPatterns("/address/setDefalut")
                  .addPathPatterns("/address/delete")
                  .addPathPatterns("/orders/*")
                  .addPathPatterns("/center/*")
                  .addPathPatterns("/userInfo/*")
                  .addPathPatterns("/myorders/*")
                  .addPathPatterns("/mycomments/*")
                  .excludePathPatterns("/myorders/deliver")
                  .excludePathPatterns("/orders/notifyMerchantOrderPaid");
  
          WebMvcConfigurer.super.addInterceptors(registry);
      }
  }
  
  ```


### 单点登录SSO
- 相同顶级域名的单点登录SSO
    - 比如说现在有个一级域名为 www.imooc.com ，是教育类网站，但是慕课网有其他的产品线，可以通过构建二级域名提供服务给用户访问，比如： music.imoo
  op.imooc.com ， blog.imooc.com 等等，分别为慕课音乐，慕课电商以及慕课博客等，用户只需要在其中一个站点登录，那么其他站点也会随之而登录。
  也就是说，用户自始至终只在某一个网站下登录后，那么他所产生的会话，就共享给了其他的网站，实现了单点网站登录后，同时间接登录了其他的网站，那么
  单点登录，他们的会话是共享的，都是同一个用户会话。
    
    - Cookie + Redis 实现 SSO：
    
      ```
      那么之前我们所实现的分布式会话后端是基于redis的，如此会话可以流窜在后端的任意系统，都能获取到缓存中的用户数据信息，前端通过使用cookie，可以保
      一级二级下获取，那么这样一来，cookie中的信息userid和token是可以在发送请求的时候携带上的，这样从前端请求后端后是可以获取拿到的，这样一来，其实
      登录注册以后，其实cookie和redis中都会带有用户信息，只要用户不退出，那么就能在任意一个站点实现登录了。
      那么这个原理主要也是cookie和网站的依赖关系，顶级域名 www.imooc.com 和*.imooc.com 的cookie值是可以共享的，可以被携带
      比如设置为 .imooc.com ， .t.mukewang.com ，如此是OK的。
      二级域名自己的独立cookie是不能共享的，不能被其他二级域名获取，比如： music.imooc.com 的cookie是不能被mtv.imooc.com 共
      互不影响，要共享必须设置为.imooc.com 。
      ```
    
  
- 不同顶级域名的单点登录CAS	
    么如果顶级域名都不一样，咋办？比如 www.imooc.com 要和www.mukewang.com 的会话实现共享，这个时候又
下图，这个时候的cookie由于顶级域名不同，就不能实现cookie跨域了，每个站点各自请求到服务端，cookie无法同步。比如，www.imooc.com下的用户发起请
ie，但是他又访问了www.abc.com，由于cookie无法携带，所以会要你二次登录。
[![gN6aWR.jpg](https://z3.ax1x.com/2021/05/10/gN6aWR.jpg)](https://imgtu.com/i/gN6aWR)

- SSO时序图
[![gNymgx.png](https://z3.ax1x.com/2021/05/10/gNymgx.png)](https://imgtu.com/i/gNymgx)

```
用户访问系统1的受保护资源，系统1发现用户未登录，跳转至sso认证中心，并将自己的地址作为参数
sso认证中心发现用户未登录，将用户引导至登录页面
用户输入用户名密码提交登录申请
sso认证中心校验用户信息，创建用户与sso认证中心之间的会话，称为全局会话，同时创建授权令牌
sso认证中心带着令牌跳转会最初的请求地址（系统1）
系统1拿到令牌，去sso认证中心校验令牌是否有效
sso认证中心校验令牌，返回有效，注册系统1
系统1使用该令牌创建与用户的会话，称为局部会话，返回受保护资源
用户访问系统2的受保护资源
系统2发现用户未登录，跳转至sso认证中心，并将自己的地址作为参数
sso认证中心发现用户已登录，跳转回系统2的地址，并附上令牌
系统2拿到令牌，去sso认证中心校验令牌是否有效
sso认证中心校验令牌，返回有效，注册系统2
系统2使用该令牌创建与用户的局部会话，返回受保护资源
　　用户登录成功之后，会与sso认证中心及各个子系统建立会话，用户与sso认证中心建立的会话称为全局会话，用户与各个子系统建立的会话称为局部会话，局部会话建立之后，用户访问子系统受保护资源将不再通过sso认证中心，全局会话与局部会话有如下约束关系

局部会话存在，全局会话一定存在
全局会话存在，局部会话不一定存在
全局会话销毁，局部会话必须销毁

```

- 单点注销时序图

[![gNc81I.png](https://z3.ax1x.com/2021/05/10/gNc81I.png)](https://imgtu.com/i/gNc81I)

```
用户向系统1发起注销请求
系统1根据用户与系统1建立的会话id拿到令牌，向sso认证中心发起注销请求
sso认证中心校验令牌有效，销毁全局会话，同时取出所有用此令牌注册的系统地址
sso认证中心向所有注册系统发起注销请求
各注册系统接收sso认证中心的注销请求，销毁局部会话
sso认证中心引导用户至登录页面

```

### 简单实例
[![g6EfdP.png](https://z3.ax1x.com/2021/05/15/g6EfdP.png)](https://imgtu.com/i/g6EfdP)

- 后端代码

```java
package com.pookaboo.controller;

import com.pookaboo.pojo.Users;
import com.pookaboo.pojo.vo.UsersVO;
import com.pookaboo.service.UserService;
import com.pookaboo.utils.*;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.UUID;

@Controller
public class SSOController {

    @Autowired
    private UserService userService;

    @Autowired
    private RedisOperator redisOperator;

    public static final String REDIS_USER_TOKEN = "redis_user_token";
    public static final String REDIS_USER_TICKET = "redis_user_ticket";
    public static final String REDIS_TMP_TICKET = "redis_tmp_ticket";
    public static final String COOKIE_USER_TICKET = "cookie_user_ticket";

    /**
     * 跳转到cas登录页面
     *
     * @param returnUrl
     * @param model
     * @param request
     * @param response
     * @return
     */
    @GetMapping("/login")
    public String login(String returnUrl, Model model, HttpServletRequest request, HttpServletResponse response) {
        model.addAttribute("returnUrl", returnUrl);
        // 1. 获取userTicket门票，如果cookie中能够获取到，证明用户登录过，此时签发一个一次性的临时票据并且回跳
        String userTicket = getCookie(request, COOKIE_USER_TICKET);
        boolean isVerified = verifyUserTicket(userTicket);
        if (isVerified) {
            String tmpTicket = createTmpTicket();
            return "redirect:" + returnUrl + "?tmpTicket=" + tmpTicket;
        }
        return "login";
    }

    /**
     * 校验CAS全局用户门票
     *
     * @param userTicket
     * @return
     */
    private boolean verifyUserTicket(String userTicket) {
        // 0. 验证CAS门票不能为空
        if (StringUtils.isBlank(userTicket)) {
            return false;
        }
        // 1. 验证CAS门票是否有效
        String userId = redisOperator.get(REDIS_USER_TICKET + ":" + userTicket);
        if (StringUtils.isBlank(userId)) {
            return false;
        }
        // 2. 验证门票对应的user会话是否存在
        String userRedis = redisOperator.get(REDIS_USER_TOKEN + ":" + userId);
        if (StringUtils.isBlank(userRedis)) {
            return false;
        }
        return true;
    }

    /**
     * cas登录
     *
     * @param username
     * @param password
     * @param returnUrl
     * @param model
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @PostMapping("/doLogin")
    public String doLogin(String username,
                          String password,
                          String returnUrl,
                          Model model,
                          HttpServletRequest request,
                          HttpServletResponse response) throws Exception {

        // 0. 判断用户名和密码必须不为空
        if (StringUtils.isBlank(username) ||
                StringUtils.isBlank(password)) {
            model.addAttribute("errmsg", "用户名或密码不能为空");
            return "login";
        }
        // 1. 实现登录
        Users users = userService.queryUserForLogin(username,
                MD5Utils.getMD5Str(password));
        if (users == null) {
            model.addAttribute("errmsg", "用户名或密码不正确");
            return "login";
        }
        // 2、创建用户的redis会话
        String uniqueToken = UUID.randomUUID().toString().trim();
        UsersVO usersVO = new UsersVO();
        BeanUtils.copyProperties(users, usersVO);
        usersVO.setUserUniqueToken(uniqueToken);
        redisOperator.set(REDIS_USER_TOKEN + ":" + users.getId(), JsonUtils.objectToJson(usersVO));

        // 3、创建用户全局门票，代表用户在cas系统登录过
        String userTikect = UUID.randomUUID().toString().trim();
        // 3.1、用户全局门票需要存放到cas端的cookie中
        setCookie(COOKIE_USER_TICKET, userTikect, response);

        // 4、userTikect关联用户id,并且放入到redis中，代表这个用户有了门票
        redisOperator.set(REDIS_USER_TICKET + ":" + userTikect, users.getId());
        // 5、创建临时票据,回调到调用端网站，是由cas端签发的一个一次性的临时的票据
        String tmpTicket = createTmpTicket();
        /**
         * userTicket: 用于表示用户在CAS端的一个登录状态：已经登录
         * tmpTicket: 用于颁发给用户进行一次性的验证的票据，有时效性
         */
        return "redirect:" + returnUrl + "?tempTicket=" + tmpTicket;
    }

    @PostMapping("/verifyTmpTicket")
    @ResponseBody
    public CustomJSONResult verifyTmpTicket(String tempTicket,
                                            HttpServletRequest request,
                                            HttpServletResponse response) throws Exception {

        // 使用一次性临时票据来验证用户是否登录，如果登录过，把用户会话信息返回给站点
        // 使用完毕后，需要销毁临时票据
        String tempTikectValue = redisOperator.get(REDIS_TMP_TICKET + ":" + tempTicket);
        if (StringUtils.isBlank(tempTikectValue)) {
            return CustomJSONResult.errorUserTicket("用户票据异常");
        }
        // 0. 如果临时票据OK，则需要销毁，并且拿到CAS端cookie中的全局userTicket，以此再获取用户会话
        if (!tempTikectValue.equals(MD5Utils.getMD5Str(tempTicket))) {
            return CustomJSONResult.errorUserTicket("用户票据异常");
        } else {
            // 销毁临时票据
            redisOperator.del(REDIS_TMP_TICKET + ":" + tempTicket);
        }

        // 1. 验证并且获取用户的userTicket
        String userTicket = getCookie(request, COOKIE_USER_TICKET);
        String userId = redisOperator.get(REDIS_USER_TICKET + ":" + userTicket);
        if (StringUtils.isBlank(userId)) {
            return CustomJSONResult.errorUserTicket("用户票据异常");
        }
        String userRedis = redisOperator.get(REDIS_USER_TOKEN + ":" + userId);
        if (StringUtils.isBlank(userRedis)) {
            return CustomJSONResult.errorUserTicket("用户票据异常");
        }
        return CustomJSONResult.ok(JsonUtils.jsonToPojo(userRedis, UsersVO.class));
    }

    /**
     * 退出登录
     *
     * @param userId
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @PostMapping("/logout")
    @ResponseBody
    public CustomJSONResult logout(String userId,
                                   HttpServletRequest request,
                                   HttpServletResponse response) throws Exception {
        // 0. 获取CAS中的用户门票
        String userTicket = getCookie(request, COOKIE_USER_TICKET);
        // 1. 清除userTicket票据，redis/cookie
        deleteCookie(COOKIE_USER_TICKET, response);
        redisOperator.del(REDIS_USER_TICKET + ":" + userTicket);
        // 2. 清除用户全局会话（分布式会话）
        redisOperator.del(REDIS_USER_TOKEN + ":" + userId);

        return CustomJSONResult.ok();
    }


    /**
     * 创建临时票据
     *
     * @return
     */
    private String createTmpTicket() {
        String tmpTicket = UUID.randomUUID().toString().trim();
        try {
            redisOperator.set(REDIS_TMP_TICKET + ":" + tmpTicket,
                    MD5Utils.getMD5Str(tmpTicket), 600);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return tmpTicket;
    }

    /**
     * 设置cookie
     *
     * @param key
     * @param val
     * @param response
     */
    private void setCookie(String key,
                           String val,
                           HttpServletResponse response) {
        Cookie cookie = new Cookie(key, val);
        cookie.setDomain("sso.com");
        cookie.setPath("/");
        response.addCookie(cookie);
    }

    /**
     * 删除cookie
     *
     * @param key
     * @param response
     */
    private void deleteCookie(String key,
                              HttpServletResponse response) {
        Cookie cookie = new Cookie(key, null);
        cookie.setDomain("sso.com");
        cookie.setPath("/");
        cookie.setMaxAge(-1);
        response.addCookie(cookie);
    }

    /**
     * 获取cookie
     *
     * @param request
     * @param key
     * @return
     */
    private String getCookie(HttpServletRequest request, String key) {
        Cookie[] cookieList = request.getCookies();
        if (cookieList == null || StringUtils.isBlank(key)) {
            return null;
        }
        String cookieValue = null;
        for (int i = 0; i < cookieList.length; i++) {
            if (cookieList[i].getName().equals(key)) {
                cookieValue = cookieList[i].getValue();
                break;
            }
        }
        return cookieValue;
    }
}

```

- 前端示例

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0,maximum-scale=1.0, user-scalable=0">

  <title>MTV系统</title>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.9/dist/vue.js"></script>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
</head>

<body>
  <div id="mtv">
    <h1>MTV系统</h1>
    <div v-if="userIsLogin != true">
      欢迎陌生人，请<a>登录</a>!
    </div>
    <div v-if="userIsLogin == true">
      欢迎<span style="color: green;">{{userInfo.username}}</span>登录系统!
      <br />
      <button @click="logout">点我退出登录</button>
    </div>
  </div>
  <script type="text/javascript " src="js/app.js"></script>
  <script type="text/javascript">
    var index = new Vue({
      el: "#mtv",
      data: {
        userIsLogin: false,
        userInfo: {},
      },
      created() {
        var me = this;
        // 通过cookie判断用户是否登录
        this.judgeUserLoginStatus();

        // http://www.mtv.com:8080/sso-mtv/index.html

        // 判断用户是否登录
        var userIsLogin = this.userIsLogin;
        if (!userIsLogin) {
          // 如果没有登录，判断一下是否存在tmpTicket临时票据
          var tmpTicket = app.getUrlParam("tmpTicket");
          console.log("tmpTicket: " + tmpTicket);
          if (tmpTicket != null && tmpTicket != "" && tmpTicket != undefined) {
            // 如果有tmpTicket临时票据，就携带临时票据发起请求到cas验证获取用户会话
            var serverUrl = app.serverUrl;
            axios.defaults.withCredentials = true;
            axios.post('http://www.sso.com:8090/verifyTmpTicket?tmpTicket=' + tmpTicket)
              .then(res => {
                if (res.data.status == 200) {
                  var userInfo = res.data.data;
                  console.log(res.data.data);
                  this.userInfo = userInfo;
                  this.userIsLogin = true;
                  app.setCookie("user", JSON.stringify(userInfo));
                  window.location.href = "http://www.mtv.com:8080/sso-mtv/index.html";
                } else {
                  alert(res.data.msg);
                  console.log(res.data.msg);
                }
              });
          } else {
            // 如果没有tmpTicket临时票据，说明用户从没登录过，那么就可以跳转至cas做统一登录认证了
            window.location.href = app.SSOServerUrl + "/login?returnUrl=http://www.mtv.com:8080/sso-mtv/index.html";
          }

          console.log(app.SSOServerUrl + "/login?returnUrl=" + window.location);
        }
      },
      methods: {
        logout() {
          var userId = this.userInfo.id;
          axios.defaults.withCredentials = true;
          axios.post('http://www.sso.com:8090/logout?userId=' + userId)
            .then(res => {
              if (res.data.status == 200) {
                var userInfo = res.data.data;
                console.log(res.data.data);
                this.userInfo = {};
                this.userIsLogin = false;

                app.deleteCookie("user");

                alert("退出成功!");

              } else {
                alert(res.data.msg);
                console.log(res.data.msg);
              }
            });
        },
        // 通过cookie判断用户是否登录
        judgeUserLoginStatus() {
          var userCookie = app.getCookie("user");
          if (
            userCookie != null &&
            userCookie != undefined &&
            userCookie != ""
          ) {
            var userInfoStr = decodeURIComponent(userCookie);
            // console.log(userInfo);
            if (
              userInfoStr != null &&
              userInfoStr != undefined &&
              userInfoStr != ""
            ) {
              var userInfo = JSON.parse(userInfoStr);
              // 判断是否是一个对象
              if (typeof(userInfo) == "object") {
                this.userIsLogin = true;
                // console.log(userInfo);
                this.userInfo = userInfo;
              } else {
                this.userIsLogin = false;
                this.userInfo = {};
              }
            }
          } else {
            this.userIsLogin = false;
            this.userInfo = {};
          }
        }
      }
    });

  </script>
</body>

</html>

```


### 单点登录存在的问题
- 跨域cookie共享：由于Chrome更改了默认的SameSite属性（`http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html`）导致在获取cas的cookie时，一直获取不到

- 解决方案
  
    - 1、禁用`SameSite`验证：打开`Chrome`设置，将`chrome://flags/#same-site-by-default-cookies`先禁用，然后重启浏览器。
      
    - 2、cookie的SameSite属性改成最低级：**不过，这只是一种权宜之计，因为设置`sameSite`为`None`之后，`CSRF`的风险又回来了。所以，换成`token`的检验方式而不依赖`Cookie`，或许才是更合理的解决方案**
    
        ```java
        public class SameSiteInterceptor implements HandlerInterceptor {
            @Override
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
                return true;
            }
        
            @Override
            public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            }
        
            @Override
            public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
                Collection<String> headers = response.getHeaders(HttpHeaders.SET_COOKIE);
                boolean firstHeader = true;
                // there can be multiple Set-Cookie attributes
                for (String header : headers) { 
                    // "SameSite=None; Secure;"
                    response.addHeader(HttpHeaders.SET_COOKIE, String.format("%s; %s", header, "SameSite=None; Secure;"));
                }
            }
        }
        ```
    
        

