

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
    用户请求服务端，由于前后端分离，前端发起http请求，不会携带任何状态，当用户第一次请求后，我们手动设置一个token，作为会话，放入redis中，如此作为redis-session，并且这个token设置后放入前端的cookie中，如此后续的交互，前端只需要传递token给后端，后端就能识别这个用户来自谁了。
    ```

    [![cTY7Os.png](https://z3.ax1x.com/2021/04/19/cTY7Os.png)](https://imgtu.com/i/cTY7Os)

- 集群分布系统会话

    ```
    集群或者分布式系统本质是多个系统，假设这个里有两个服务器节点，分别是AB系统，一开始用户和A系统交互，那么这个时候的用户状态，我们可以保存到redis中，作为A系统的会话信息，随后用户的请求进入B系统，那么B系统中的会话我也同样和redis关联，如此AB系统的session就统一了。当然cookie是会随着用户的访问携带的。这个其实就是分布式会话，通过redis来保存用户的状态
    
    ```

    [![cTYTyj.png](https://z3.ax1x.com/2021/04/19/cTYTyj.png)](https://imgtu.com/i/cTYTyj)

 ### 用户会话
 - redis用户会话
 - SpringSession