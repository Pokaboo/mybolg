## Maven

#### 中央仓库、镜像仓库、本地仓库

#### Maven核心概念：坐标

- 使用x、y、z三个“向量”作为空间的坐标系，可以在空间中唯一的定位到一个点。使用三个向量在Maven的仓库中唯一的定位到一个jar包

  - groupId：公司或组织的id

  - artifactId：一个项目或者是项目中的一个模块的id

  - version：版本号


#### Maven核心概念：POM

- POM含义：**P**roject **O**bject **M**odel，项目对象模型。和POM类似的是：DOM：Document Object Model，文档对象模型。
- 思想：POM表示将工程抽象为一个模型，再用程序中的对象来描述这个模型。这样我们就可以用程序来管理项目了。我们在开发过程中，最基本的做法就是将现实生活中的事物抽象为模型，然后封装模型相关的数据作为一个对象，这样就可以在程序中计算与现实事物相关的数据。
- 对应的配置文件：POM理念集中体现在Maven工程根目录下**pom.xml**这个配置文件中。所以这个pom.xml配置文件就是Maven工程的核心配置文件。其实学习Maven就是学这个文件怎么配置，各个配置有什么用。

#### Maven构建命令

- mvn clean ：清理操作。效果：删除target目录

- mvn compile：主程序编译。编译结果存放的目录：target/classes

- mvn test-compile：测试程序编译。编译结果存放的目录：target/test-classes

- mvn test：测试操作。测试的报告会存放在target/surefire-reports目录下

- mvn package：打包操作。打包的结果会存放在target目录下

- mvn install：安装操作。安装的效果是将本地构建过程中生成的jar包存入Maven本地仓库。这个jar包在Maven仓库中的路径是根据它的坐标生成的。

  



