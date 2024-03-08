## Vue

### Node.js

- 什么是Node.js：简单的说 Node.js 就是运行在服务端的 JavaScript。Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。

- Node.js有什么用：如果你是一个前端程序员，你不懂得像PHP、Python或Ruby等动态编程语言，然后你想创建自己的服务，那么Node.js是一个非常好的选择。Node.js 是运行在服务端的 JavaScript，如果你熟悉Javascript，那么你将会很容易的学会Node.js。当然，如果你是后端程序员，想部署一些高性能的服务，那么学习Node.js也是一个非常好的选择。

- Node.js安装：

  > node -v  安装和查看版本 

### NPM 

- 什么是NPM：NPM全称Node Package Manager，是Node.js包管理工具，是全球最大的模块生态系统，里面所有的模块都是开源免费的；也是Node.js的包管理工具，相当于后端的Maven 。
- NPM工具的安装位置：我们通过npm 可以很方便地下载js库，管理前端工程。Node.js默认安装的npm包和工具的位置：Node.js目录\node_modules
  - 在这个目录下你可以看见 npm目录，npm本身就是被NPM包管理器管理的一个工具，说明 Node.js已经集成了npm工具

### NPM 常用命令

```
# 查看版本
npm --version 
# 直接使用默认配置
npm init -f # npm init -y
# 安装某个模块 (安装的模块名不会记录在 package.json 中)
npm install webpack
# 安装某个模块 (模块记录在 package.json 的 dependencies 中)
npm install webpack --save # 或者 -S
# 安装某个模块 (模块记录在 package.json 的 devDependencies 中)
npm install webpack --save-dev # 或者 -D
# 全局安装模块
npm install webpack -g
# 安装指定版本
npm install webpack@3.10.0 --save 
# 指定安装版本范围
npm insatll webpack@">=2.0.0 <3.10.0"
# 安装模块最新版本
npm install webpack@latest
# 安装模块的确切版本 默认安装 "^3.10.0" 表示 3.10.0 及其以上、大版本号为3的版本 使用 --save-exact 后 "3.10.0" 表示只有 3.10.0 版本
npm install webpack --save --save-exact 
# 安装 package.json 文件中记录的模块
sudo rm -rf node_modules # 删除安装的模块
npm install # 需要有 package.json 文件

# 卸载模块
npm uninstall webpack
# 卸载全局模块
npm uninstall webpack -g
# 卸载 dependencies 中模块
npm uninstall webpack --save # -S
# 卸载 devDependencies 中模块
npm uninstall webpack --save-dev # -D

# 更新全部 package.json 中的模块
npm update
# 更新 dependencies 中模块
npm update --save # -S
# 更新 devDependencies 中模块
npm update --save-dev # -D
# 更新指定的模块
npm update webpack --save-dev
# 更新全局模块
npm update webpack -g
# 从npm v2.6.1 开始，npm update只更新顶层模块，而不更新依赖的依赖，以前版本是递归更新的。如果想取到老版本的效果，要使用下面的命令
npm --depth 9999 update
```

### vue-admin-template

- 简介：vue-admin-template是基于vue-element-admin的一套后台管理系统基础模板（最少精简版），可作为模板进行二次开发。

  > **建议：**你可以在 `vue-admin-template` 的基础上进行二次开发，把 `vue-element-admin`当做工具箱，想要什么功能或者组件就去 `vue-element-admin` 那里复制过来。

- 安装：

  ```
  #1、下载项目（zip或者git clone）
  **GitHub地址：**https://github.com/PanJiaChen/vue-admin-template
  #2、修改项目名称 vue-admin-template 改为 pokanoo-auth-ui
  #3、解压压缩包
  #4、进入目录
  cd pokaboo-auth-ui
  #5、安装依赖
  npm install
  #6、启动。执行后，浏览器自动弹出并访问http://localhost:9528/
  npm run dev
  ```

- 源码目录结构：

  ```
  |-dist 生产环境打包生成的打包项目
  |-mock 产生模拟数据
  |-public 包含会被自动打包到项目根路径的文件夹
  	|-index.html 唯一的页面
  |-src
  	|-api 包含接口请求函数模块
  		|-table.js  表格列表mock数据接口的请求函数
  		|-user.js  用户登陆相关mock数据接口的请求函数
  	|-assets 组件中需要使用的公用资源
  		|-404_images 404页面的图片
  	|-components 非路由组件
  		|-SvgIcon svg图标组件
  		|-Breadcrumb 面包屑组件(头部水平方向的层级组件)
  		|-Hamburger 用来点击切换左侧菜单导航的图标组件
  	|-icons
  		|-svg 包含一些svg图片文件
  		|-index.js 全局注册SvgIcon组件,加载所有svg图片并暴露所有svg文件名的数组
  	|-layout
  		|-components 组成整体布局的一些子组件
  		|-mixin 组件中可复用的代码
  		|-index.vue 后台管理的整体界面布局组件
  	|-router
  		|-index.js 路由器
  	|-store
  		|-modules
  			|-app.js 管理应用相关数据
  			|-settings.js 管理设置相关数据
  			|-user.js 管理后台登陆用户相关数据
  		|-getters.js 提供子模块相关数据的getters计算属性
  		|-index.js vuex的store
  	|-styles
  		|-xxx.scss 项目组件需要使用的一些样式(使用scss)
  	|-utils 一些工具函数
  		|-auth.js 操作登陆用户的token cookie
  		|-get-page-title.js 得到要显示的网页title
  		|-request.js axios二次封装的模块
  		|-validate.js 检验相关工具函数
  		|-index.js 日期和请求参数处理相关工具函数
  	|-views 路由组件文件夹
  		|-dashboard 首页
  		|-login 登陆
  	|-App.vue 应用根组件
  	|-main.js 入口js
  	|-permission.js 使用全局守卫实现路由权限控制的模块
  	|-settings.js 包含应用设置信息的模块
  |-.env.development 指定了开发环境的代理服务器前缀路径
  |-.env.production 指定了生产环境的代理服务器前缀路径
  |-.eslintignore eslint的忽略配置
  |-.eslintrc.js eslint的检查配置
  |-.gitignore git的忽略配置
  |-.npmrc 指定npm的淘宝镜像和sass的下载地址
  |-babel.config.js babel的配置
  |-jsconfig.json 用于vscode引入路径提示的配置
  |-package.json 当前项目包信息
  |-package-lock.json 当前项目依赖的第三方包的精确信息
  |-vue.config.js webpack相关配置(如: 代理服务器)
  ```

- 实现登陆&自动登陆&退出登陆

  - vue.config.js：

    - 注释掉mock接口配置
    - 配置代理转发请求到目标接口

    ```js
    // before: require('./mock/mock-server.js')
    proxy: {
      '/dev-api': { // 匹配所有以 '/dev-api'开头的请求路径
        target: 'http://localhost:8800',
        changeOrigin: true, // 支持跨域
        pathRewrite: { // 重写路径: 去掉路径中开头的'/dev-api'
          '^/dev-api': ''
        }
      }
    }
    ```

  - src/utils/request.js：请求自动携带token, 且请求头名称为 token

    ```js
    const token = store.getters.token
    if (token) {
      // let each request carry token
      // ['X-Token'] is a custom headers key
      // please modify it according to the actual situation
      // config.headers['X-Token'] = getToken()
      config.headers['token'] = token
    }
    ```

    ```js
    if (res.code !== 200) {
      Message({
        message: res.message || 'Error',
        type: 'error',
        duration: 5 * 1000
      })
      return Promise.reject(new Error(res.message || 'Error'))
    } else {
      return res
    }
    ```

  - src/api/user.js：修改路由

    ```js
    import request from '@/utils/request'
    
    export function login(data) {
      return request({
        url: '/admin/system/index/login',
        method: 'post',
        data
      })
    }
    
    export function getInfo(token) {
      return request({
        url: '/admin/system/index/info',
        method: 'get',
        params: { token }
      })
    }
    
    export function logout() {
      return request({
        url: '/admin/system/index/logout',
        method: 'post'
      })
    }
    ```

    
