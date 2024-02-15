## SpringMVC

### SpringMVC概述

>SpringMVC是Spring为表述层开发提供的一整套完备的解决方案。
>
>在表述层框架历经Strust、WebWork、Strust2等诸多产品的历代更迭之后，目前业界普遍选择了SpringMVC作为Java EE项目表述层开发的**首选方案**。之所以能做到这一点，是因为SpringMVC具备如下显著优势：

### 表述层框架要解决的基本问题

- 请求映射
- 数据输入
- 视图界面
- 请求分发
- 表单回显
- 会话控制
- 过滤拦截
- 异步交互
- 文件上传
- 文件下载
- 数据校验
- 类型转换

### 整体流程解析

[![pFGE8rF.png](https://s11.ax1x.com/2024/02/15/pFGE8rF.png)](https://imgse.com/i/pFGE8rF)

### @RequestMapping注解

- 从注解名称上我们可以看到，@RequestMapping注解的作用就是将请求的 URL 地址和处理请求的方式（handler方法）关联起来，建立映射关系。SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的方法来处理这个请求。


### @RequestParam注解

### @CookieValue注解

- 获取当前请求中的 Cookie 数据。

