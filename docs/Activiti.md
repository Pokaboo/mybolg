##  Activiti

### 什么是工作流
```
工作流(Workflow)，就是通过计算机对业务流程自动化执行管理。它主要解决的是“使在多个参与者之间按照某种预定义的规则自动进行传递文档、信息或任务的过程，从而实现某个预期的业务目标，或者促使此目标的实现”。
```

### 什么是Activiti7

```
Activiti 是一个工作流引擎， activiti 可以将业务系统中复杂的业务流程抽取出来，使用专门的
建模语言（BPMN2.0）进行定义，业务系统按照预先定义的流程进行执行，实现了业务系统的业务
流程由 activiti 进行管理，减少业务系统由于流程变更进行系统升级改造的工作量，从而提高系统的
健壮性，同时也减少了系统开发维护成本。
官方网站：https://www.activiti.org/
```

### BPM & BPMN

```
BPM（Business Process Management），即业务流程管理，是一种以规范化的构造端到端的卓越业务流程为中心，以持续的提高组织业务绩效为目的系统化方法，常见商业管理教育如 EMBA、MBA等均将 BPM 包含在内
BPMN（Business Process Model And Notation）- 业务流程模型和符号 是由 BPMI（BusinessProcess Management Initiative）开发的一套标准的业务流程建模符号，使用 BPMN 提供的符号可以创建业务流程。 
```

### 简单示例

* activiti.cfg.xml配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation=
         "http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  		http://www.springframework.org/schema/contex http://www.springframework.org/schema/context/spring-context.xsd
  		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
  
      <!--配置数据源-->
      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
          <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
          <property name="url" value="jdbc:mysql://localhost:3306/activiti?serverTimezone=UTC&amp;nullCatalogMeansCurrent=true"/>
          <property name="username" value="root"/>
          <property name="password" value="root"/>
      </bean>
  
      <!--activiti单独运行的ProcessEngine配置，使用单独启动方式-->
      <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
          <!-- 数据源 -->
          <property name="dataSource" ref="dataSource"></property>
          <!-- activiti数据库表处理策略 -->
          <property name="databaseSchemaUpdate" value="true"/>
      </bean>
  
  </beans>
  ```

* 测试类

  ```java
  package com.pokaboo.test;
  
  import org.activiti.engine.ProcessEngine;
  import org.activiti.engine.ProcessEngineConfiguration;
  import org.junit.Test;
  
  public class Activititest {
  
      @Test
      public void creatActivitiTable() {
          //创建ProcessEngineConfiguration
          //通过ProcessEngineConfiguration创建ProcessEngine，此时会创建数据库
          ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml");
          ProcessEngine processEngine = configuration.buildProcessEngine();
      }
  }
  ```

* 查看数据库，创建了 25 张表

  ```
  Activiti 的表都以 ACT_开头。 第二部分是表示表的用途的两个字母标识。 用途也和服务的 API 对应。
  	ACT_RE_*: 'RE'表示 repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
  	ACT_RU_*: 'RU'表示 runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti 只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。
  	ACT_HI_*: 'HI'表示 history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
  	ACT_GE_*: GE 表示 general。通用数据， 用于不同场景下
  ```

  

   
