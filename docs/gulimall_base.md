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

  
