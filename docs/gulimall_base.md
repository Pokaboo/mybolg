## 谷粒商城-分布式基础
### 1、微服务

- 微服务架构风格，就像是把一个单独的应用程序开发为一套小服务，每个小服务运行在自己的进程中，并且使用轻量级机制通信，通常是HTTP API。这些服务围绕业务能力来构建，并且通过完全自动化部署机制来独立部署。这些服务使用不同的编程语言书写，以及不同数据存储技术，并保持最低限度的集中式管理。<font color='red'>简而言之，拒绝大型单体应用，基于业务边界进行服务微化拆分，各个服务独立部署运行。</font>

### 2、集群&分布式&节点
- 分布式：是指工作方式
- 集群：是指物理形态
- 《分布式系统原理与范型》中定义：分布式系统是建立在网络之上的软件系统，分布式是指将不同的业务分布在不同的地方；集群指的是将几台服务器集中在一起，实现同一业务。<font color='red'>分布式中的每一个节点，都可以做集群。而集群并不一定就是分布式的。</font>
- 节点：集群中的每一个服务器

### 3、远程调用
- 在分布式系统中，各个服务器可能处于不同的主机，但是服务之间不可避免的需要相互调用，称之为远程调用。SpringCloud中使用Http + Json 方式完成远程调用。

### 4、负载均衡
- 分布式系统中，A服务需要调用B服务，B服务在多台机器中都存在，A调用任意一个服务器均可完成。
- 常见负载均衡算法
  - 轮询：为第一个请求选择健康池中的第一个服务器，然后按顺序往后依次选择，直到最后一个
  - 最小连接：优选选择连接数最小的，也就是压力最小的后端服务器，在会话较长的情况下可以考虑采用这种方式
  - 散列：很久请求源的IP的散列（hash）来选择要转发的服务器。这种方式可以一定程度上保证特定用户能连接到相同的服务器。

### 5、服务注册/发现&注册中心
- A服务调用B服务，A服务并不知道B服务当前状态，引入注册中心

### 6、配置中心
- 配置中心用来集中管理微服务的配置信息

### 7、服务熔断&服务降级
- 在微服务架构中，微服务之间通过网络进行通信，存在相互依赖，当其中一个服务不可用是，可能造成雪崩发生。
- 服务熔断
  - 设置服务的超时，当被调用的服务经常失败到达某个阈值。我们可以开启断路保护机制，后来的请求不再去调用这个服务，本地直接返回默认的数据
- 服务降级
  - 在运维期间，当系统处于高峰期，系统资源紧张，可以让非核心服务降级运行

### 8、API网关
- 负载均衡、服务熔断、灰度发布、统一认证、限流管控、日志统计

### 9、微服务架构图
[![pSgpHf0.md.png](https://s1.ax1x.com/2023/02/06/pSgpHf0.md.png)](https://imgse.com/i/pSgpHf0)

  [![pSg9j8P.md.png](https://s1.ax1x.com/2023/02/06/pSg9j8P.md.png)](https://imgse.com/i/pSg9j8P)

### 10、环境搭建
- 安装Virtual Box

- 安装vagrant
  ```
  https://app.vagrantup.com/boxes/search Vagrant 官方镜像仓库
  a、打开 window cmd 窗口，输入vagrant，验证是否下载成功
  b、初始化centos7： Vagrant init centos/7 （注："centos/7 " 的名字和官方镜像仓库的名字是对应的）
  c、运行 vagrant up 即可启动虚拟机（系统 root 用户的密码是 vagrant）
  注：如果vagrant up过慢，可以使用中科大镜像
  vagrant init centos7 https://mirrors.ustc.edu.cn/centos-cloud/centos/7/vagrant/x86_64/images/CentOS-7.box
  d、vagrant常用命令  
  cmd窗口：
  vagrant ssh
  启动：vagrant up
  退出：exit
  ```
  
- 虚拟机网络配置

  ```
  a、当前用户目录下C:\Users\zhaowu，找到Vagrantfile文件。修改config.vm.network "private_network", ip: "192.168.56.10"（ip与cmd命令ipconfig/all下的Virtual Box保持一致：比如为，192.168.56.1则配置成192.168.56.1开头）
  c、配置完后vagrant reload，然后ping测试
  ```
  
- 安装Docker  

- 安装Mysql

  ```
  1、拉取mysql:docker pull mysql:5.7
  2、检查镜像：下载mysql5.7
  3、创建实例并且启动：
      docker run -p 3306:3306 --name mysql \
      -v /mydata/mysql/log:/var/log/mysql \
      -v /mydata/mysql/data:/var/lib/mysql \
      -v /mydata/mysql/conf:/etc/mysql \
      -e MYSQL_ROOT_PASSWORD=root \
      -d mysql:5.7、
  5、创建mysql配置文件
      vi /mydata/mysql/conf/my.cnf
      [client]
      default-character-set=utf8
  
      [mysql]
      default-character-set=utf8
  
      [mysqld]
      init_connect='SET collation_connection = utf8_unicode_ci'
      init_connect='SET collation_connection = utf8_unicode_ci'
      init_connect='SET NAMES utf8'
      character-set-server=utf8
      collation-server=utf8_unicode_ci
      skip-character-set-client-handshake
      skip-name-resolve
  6、重启mysql容器：docker restart mysql
  ```

- 安装redis

  ```
  1、拉取redis:docker pull redis
  2、创建配置文件
  	mkdir -p /mydata/redis/conf
  	touch /mydata/redis/conf/redis.conf
  3、启动容器
  	docker run -p 6379:6379 --name redis \
      -v /mydata/redis/data:/data \
      -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
      -d redis redis-server /etc/redis/redis.conf
  4、运行 redis
  	docker exec -it redis redis-cli
  5、开启 aof 持久化
  	vi /mydata/redis/conf/redis.conf
      # 添加如下内容
      appendonly yes
  6、重启 redis
  	docker restart redis
  ```

- 环境-开发工具&环境安装配置

- 数据库设计

- 前端部署

  ```
  1、git clone https://gitee.com/renrenio/renren-fast-vue.git
  2、将导入的前端模板导入VScode
  3、npm install
  4、npm run dev
  ```
  
- 后台管理系统

  ```
  1、git clone https://gitee.com/renrenio/renren-fast.git
  2、修改数据库相关配置文件
  ```

### 11、SpringCloud Alibaba
- Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布
  式应用服务。

- pom文件引入SpringCloud Alibaba

  ```xml
  <dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-alibaba-dependencies</artifactId>
              <version>2.1.0.RELEASE</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

### 12、Nacos
- nacos:Nacos 是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。他是使用 java 编写。需要依赖 java 环境Nacos 文档地址： https://nacos.io/zh-cn/docs/quick-start.htm

- 下载nacos

- 启动 **nacos-server**

  ```tex
  双击 bin 中的 startup.cmd 文件
  访问 http://localhost:8848/nacos/
  使用默认的 nacos/nacos 进行登录
  ```

- pom文件引入nacos

  ```xml
  <dependency>
  	<groupId>com.alibaba.cloud</groupId>
  	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

- 使用<font color='orange'>@EnableDiscoveryClient </font>**开启服务注册发现功能**

  ```java
  @EnableDiscoveryClient
  @SpringBootApplication
  public class MallCouponApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(MallCouponApplication.class, args);
      }
  
  }
  ```

- 修改配置文件

  ```yml
  server:
    port: 7000
  spring:
    application:
      name: mall-coupon
    datasource:
      username: root
      password: root
      url: jdbc:mysql://xxx.xxx.xx.xx:3306/mall_sms?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
      driver-class-name: com.mysql.cj.jdbc.Driver
    #配置Nacos Server 地址
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
  ```

### 13、openFeign

- feign是一个声明式的HTTP客户端，他的目的就是让远程调用更加简单。 给远程服务发的是HTTP请求

- 使用步骤

  - 1、使用注解@EnableFeignClients启用feign客户端

    ```java
    @SpringBootApplication
    @EnableFeignClients(basePackages = "com.pokaboo.mall.member.feign")
    @MapperScan("com.pokaboo.mall.member.dao")
    public class MallMemberApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(MallMemberApplication.class, args);
        }
    
    }
    ```

    

  - 2、使用注解@FeignClient 定义feign客户端

    ```java
    @FeignClient("mall-coupon")
    public interface CouponFenginService {
    
        @RequestMapping("coupon/coupon/coupons")
        public R coupons();
    }
    ```

  - 3、使用注解@Autowired使用上面所定义feign的客户端 

    ```java
    @Autowired
    private CouponFenginService couponFenginService;
    
    @RequestMapping("/coupons")
    public R test(){
        R coupons = couponFenginService.coupons();
        MemberEntity memberEntity = new MemberEntity();
        memberEntity.setNickname("Pokaboo");
        return R.ok().put("member",memberEntity).put("coupons",coupons.get("coupons"));
    }
    ```

- pom引入

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

- @EnableFeignClients

  ```
  注解告诉框架扫描所有使用注解@FeignClient定义的feign客户端，并把feign客户端注册到IOC容器中
  ```

- @FeignClient

  ```
  使用注解@FeignClient 定义feign客户端 ;
  ```
### 14、配置中心

- 引入pom

  ```xml
  <!--Nacos 配置中心-->
      <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  ```

- application.properties文件

  ```xml
  spring.application.name=mall-coupon
  spring.cloud.nacos.config.server-addr=192.168.11.1:8848
  ```

- @RefreshScope

  ```java
  @RefreshScope
  @RestController
  @RequestMapping("coupon/coupon")
  public class CouponController {
      @Autowired
      private CouponService couponService;
  
      @Value("${coupon.user.name}")
      private String userName;
      @Value("${coupon.user.age}")
      private Integer userAge;
  
      @RequestMapping("/test/configCenter")
      public R test(){
          return R.ok().put("userName",userName).put("userAge",userAge);
      }
  ```

  ```
  scope是如何做到热加载的呢？
  因为可以单独管理Bean的创建和销毁 创建Bean的时候如果scope为refresh，这个Bean就缓存在一个专门管理这类scope的map中，
  当外部配置更改过后，会触发一个刷新动作，这个动作将上面的map中的Bean清空，这样，当再次用到这个Bean的时候，
  这些Bean就会重新被IOC容器创建一次，使用最新的外部化配置的值注入类中，达到热加载新值的效果。
  ```
  
- 命名空间：
用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等

- 配置集：

  一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。

- **配置集** ID:

  Nacos 中的某个配置集的 ID。配置集 ID 是组织划分配置的维度之一。**Data ID** 通常用于组织划分系统的配置集。一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。Data ID 通常采用类 Java 包（如 com.taobao.tc.refund.log.level）的命名规则保证全局唯一性。此命名规则非强制。

- 配置分组:

  Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和MQ_topic 配置

- <font color='orange'>自动注入</font>：
NacosConfigStarter 实现了 org.springframework.cloud.bootstrap.config.PropertySourceLocator接口，并将优先级设置成了最高。在 Spring Cloud 应用启动阶段，会主动从 Nacos Server 端获取对应的数据，并将获取到的数据转换成 PropertySource 且注入到 Environment 的 PropertySources 属性中，所以使用@Value 注解也能直接获取 Nacos Server 端配置的内容。 
- <font color='cornflowerblue'>动态刷新</font>：
Nacos Config Starter 默认为所有获取数据成功的 Nacos 的配置项添加了监听功能，在监听到服务端配置发生变化时会实时触发
org.springframework.cloud.context.refresh.ContextRefresher 的 refresh 方法 。如果需要对 Bean 进行动态刷新，请参照 Spring 和 Spring Cloud 规范。推荐给类添加@RefreshScope 或 @ConfigurationProperties 注解，

### 15、网关gateway

- 网关提供 API 全托管服务，丰富的 API 管理功能，辅助企业管理大规模的 API，以降低管理成本和安全风险，包括协议适配、协议转发、安全策略、防刷、流量、监控日志等功能。Spring Cloud Gateway 旨在提供一种简单而有效的方式来对 API 进行路由，并为他们提供切面，例如：安全性，监控/指标 和弹性等。官方文档地址：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.3.RELEASE/single/spring-cloud-gateway.html

  ```
  思考：
  为什么使用 API 网关？
  API 网关出现的原因是微服务架构的出现，不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务的接口才能完成一个业务需求，
  如果让客户端直接与各个微服务通信，会有以下的问题：
  1、客户端会多次请求不同的微服务，增加了客户端的复杂性。
  2、存在跨域请求，在一定场景下处理相对复杂。
  3、认证复杂，每个服务都需要独立认证。
  4、难以重构，随着项目的迭代，可能需要重新划分微服务。
  例如，可能将多个服务合并成一个或者将一个服务拆分成多个。如果客户端直接与微服务通信，那么重构将会很难实施。
  某些微服务可能使用了防火墙 / 浏览器不友好的协议，直接访问会有一定的困难。
  
  以上这些问题可以借助 API 网关解决。API 网关是介于客户端和服务器端之间的中间层，所有的外部请求都会先经过 API 网关这一层。
  也就是说，API 的实现方面更多的考虑业务逻辑，而安全、性能、监控可以交由 API 网关来做，这样既提高业务灵活性又不缺安全性：
  使用 API 网关后的优点如下：
  1、易于监控。可以在网关收集监控数据并将其推送到外部系统进行分析。
  2、易于认证。可以在网关上进行认证，然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证。
  3、减少了客户端与各个微服务之间的交互次数。
  ```

- <font color='orange'>Route（路由）</font>：路由是网关最基础的部分，路由信息有一个 ID、一个目的 URL、一组断言和一组Filter 组成。如果断言路由为真，则说明请求的 URL 和配置匹配

- <font color='orange'>Predicate（断言）</font>：Java8 中的断言函数。Spring Cloud Gateway 中的断言函数输入类型是 Spring5.0 框架中的 ServerWebExchange。Spring Cloud Gateway 中的断言函数允许开发者去定义匹配来自于 http request 中的任何信息，比如请求头和参数等。

- <font color='orange'>Filter（过滤器）</font>：一个标准的 Spring webFilter。Spring cloud gateway 中的 filter 分为两种类型的Filter，分别是 Gateway Filter 和 Global Filter。过滤器 Filter 将会对请求和响应进行修改处理

- 工作原理：

  ```
  客户端发送请求给网关，弯管 HandlerMapping 判断是否请求满足某个路由，满足就发给网关的 WebHandler。
  这个 WebHandler 将请求交给一个过滤器链，请求到达目标服务之前，会执行所有过滤器的 pre 方法。
  请求到达目标服务处理之后再依次执行所有过滤器的 post 方法。
  一句话：满足某些断言（predicates）就路由到指定的地址（uri），使用指定的过滤器（filter）
  ```
  

-  PokabooMall 分布式基础篇-31章





