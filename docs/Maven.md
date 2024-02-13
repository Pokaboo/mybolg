## Maven

### 中央仓库、镜像仓库、本地仓库

### Maven核心概念：坐标

- 使用x、y、z三个“向量”作为空间的坐标系，可以在空间中唯一的定位到一个点。使用三个向量在Maven的仓库中唯一的定位到一个jar包

  - groupId：公司或组织的id

  - artifactId：一个项目或者是项目中的一个模块的id

  - version：版本号


### Maven核心概念：POM

- POM含义：**P**roject **O**bject **M**odel，项目对象模型。和POM类似的是：DOM：Document Object Model，文档对象模型。
- 思想：POM表示将工程抽象为一个模型，再用程序中的对象来描述这个模型。这样我们就可以用程序来管理项目了。我们在开发过程中，最基本的做法就是将现实生活中的事物抽象为模型，然后封装模型相关的数据作为一个对象，这样就可以在程序中计算与现实事物相关的数据。
- 对应的配置文件：POM理念集中体现在Maven工程根目录下**pom.xml**这个配置文件中。所以这个pom.xml配置文件就是Maven工程的核心配置文件。其实学习Maven就是学这个文件怎么配置，各个配置有什么用。

### Maven构建命令

- mvn clean ：清理操作。效果：删除target目录
- mvn compile：主程序编译。编译结果存放的目录：target/classes
- mvn test-compile：测试程序编译。编译结果存放的目录：target/test-classes
- mvn test：测试操作。测试的报告会存放在target/surefire-reports目录下
- mvn package：打包操作。打包的结果会存放在target目录下
- mvn install：安装操作。安装的效果是将本地构建过程中生成的jar包存入Maven本地仓库。这个jar包在Maven仓库中的路径是根据它的坐标生成的。

### maven packaging的几种形式如下

- Maven的三种packaging方式(pom、jar、war)。pom是maven依赖文件，jar是java普通项目打包，war是java web项目打包。
  - pom：打出来可以作为其他项目的maven依赖，在工程A中添加工程B的pom，A就可以使用B中的类。用在父级工程或聚合工程中。用来做jar包的版本控制。
  - jar包：通常是开发时要引用通用类，打成jar包便于存放管理。当你使用某些功能时就需要这些jar包的支持，需要导入jar包。
  - war包：是做好一个web网站后，打成war包部署到服务器。目的是节省资源，提供效率。

### scope的三个取值

- compile：通常使用的第三方框架的jar包这样在项目实际运行时真正要用到的jar包都是以compile范围进行依赖的。比如SSM框架所需jar包。
- test：测试过程中使用的jar包，以test范围依赖进来。比如junit。
- provided：在开发过程中需要用到的“服务器上的jar包”通常以provided范围依赖进来。比如servlet-api、jsp-api。而这个范围的jar包之所以不参与部署、不放进war包，就是避免和服务器上已有的同类jar包产生冲突，同时减轻服务器的负担。说白了就是：“服务器上已经有了，你就别带啦！”

- compile、provided、test 对比

  |          | main目录（空间） | test目录（空间） | 开发过程（时间） | 部署到服务器（时间） |
  | :------: | :--------------: | :--------------: | :--------------: | :------------------: |
  | compile  |       有效       |       有效       |       有效       |         有效         |
  | provided |       有效       |       有效       |       有效       |         无效         |
  |   test   |       无效       |       有效       |       有效       |         无效         |

### 依赖的传递性

- 概念：A依赖B，B依赖C，那么在A没有配置对C的依赖的情况下，A里面能不能直接使用C？
- 传递的原则：在A依赖B，B依赖C的前提下，C是否能够传递到A，取决于B依赖C时使用的依赖范围
  - B依赖C时使用compile范围：可以传递。
  - B依赖C时使用test或provided范围：不能传递，所以需要这样的jar包时，就必须在需要的地方明确配置依赖才可以。

### 依赖的排除

- 概念：当A依赖B，B依赖C而且C可以传递到A的时候，但是A不想要C，需要在A里面把C排除掉。而往往这种情况都是为了避免jar包之间的冲突。所以配置依赖的排除其实就是阻止某些jar包的传递。因为这样的jar包传递过来会和其他jar包冲突。

- 配置方式：使用excludes标签配置依赖的排除。

  ```xml
  <dependency>
      <groupId>com.pokaboo</groupId>
      <artifactId>pro01-maven-java</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>compile</scope>
      <!-- 使用excludes标签配置依赖的排除    -->
      <exclusions>
          <!-- 在exclude标签中配置一个具体的排除 -->
          <exclusion>
              <!-- 指定要排除的依赖的坐标（不需要写version） -->
              <groupId>commons-logging</groupId>
              <artifactId>commons-logging</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

  
### 继承

- 本质上是A工程的pom.xml中的配置继承了B工程中pom.xml的配置。
