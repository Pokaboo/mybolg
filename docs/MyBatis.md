## MyBatis

### MyBatis历史

> MyBatis最初是Apache的一个开源项目<span style="color:blue;font-weight:bold;">iBatis</span>, 2010年6月这个项目由Apache Software Foundation迁移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于2013年11月迁移到Github。
>
> iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。 iBatis提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

### Mybatis特性

- MyBatis支持定制化SQL、存储过程以及高级映射
- MyBatis避免了几乎所有的JDBC代码和手动设置参数以及结果集解析操作
- MyBatis可以使用简单的XML或注解实现配置和原始映射；将接口和Java的POJO（Plain Ordinary Java Object，普通的Java对象）映射成数据库中的记录
- Mybatis是一个半自动的ORM（Object   Relation  Mapping）框架

### 和其它持久化层技术对比

- JDBC
  - SQL 夹杂在Java代码中耦合度高，导致硬编码内伤
  - 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见
  - 代码冗长，开发效率低
- Hibernate 和 JPA
  - 操作简便，开发效率高
  - 程序中的长难复杂 SQL 需要绕过框架
  - 内部自动生产的 SQL，不容易做特殊优化
  - 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。
  - 反射操作太多，导致数据库性能下降
- MyBatis
  - 轻量级，性能出色
  - SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
  - 开发效率稍逊于HIbernate，但是完全能够接收

### #{}和${}区别

- #{}：Mybatis会在运行过程中，把配置文件中的SQL语句里面的**#{}**转换为“**?**”占位符，发送给数据库执行。
- ${}：将来会**根据${}拼字符串**

### 应用场景举例

- 在SQL语句中，数据库表的表名不确定，需要外部动态传入，此时不能使用#{}，因为数据库不允许表名位置使用问号占位符，此时只能使用${}。其他情况，**只要能用#{}肯定不用${}**，避免SQL注入。

### 数据输出

- 返回单个简单数据类型

  ```xml
  <select id="selectEmpCount" resultType="int">
      select count(*) from t_emp
  </select>
  ```

- 返回实体类对象

  ```xml
  <!-- 编写具体的SQL语句，使用id属性唯一的标记一条SQL语句 -->
  <!-- resultType属性：指定封装查询结果的Java实体类的全类名 -->
  <select id="selectEmployee" resultType="com.pokaboo.mybatis.entity.Employee">
      <!-- Mybatis负责把SQL语句中的#{}部分替换成“?”占位符 -->
      <!-- 给每一个字段设置一个别名，让别名和Java实体类中属性名一致 -->
      select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{maomi}
  </select>
  <!-- 增加全局配置自动识别对应关系 -->
  <!-- 做了下面的配置，select语句中可以不给字段设置别名 -->
  <!-- 在全局范围内对Mybatis进行配置 -->
  <settings>
      <!-- 具体配置 -->
      <!-- 从org.apache.ibatis.session.Configuration类中可以查看能使用的配置项 -->
      <!-- 将mapUnderscoreToCamelCase属性配置为true，表示开启自动映射驼峰式命名规则 -->
      <!-- 规则要求数据库表字段命名方式：单词_单词 -->
      <!-- 规则要求Java实体类属性名命名方式：首字母小写的驼峰式命名 -->
      <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```

- 返回Map类型：适用于SQL查询返回的各个字段综合起来并不和任何一个现有的实体类对应，没法封装到实体类对象中。**能够封装成实体类类型的，就不使用Map类型**。

  ```xml
  <!-- Map<String,Object> selectEmpNameAndMaxSalary(); -->
  <!-- 返回工资最高的员工的姓名和他的工资 -->
  <select id="selectEmpNameAndMaxSalary" resultType="map">
          SELECT
              emp_name 员工姓名,
              emp_salary 员工工资,
              (SELECT AVG(emp_salary) FROM t_emp) 部门平均工资
          FROM t_emp WHERE emp_salary=(
              SELECT MAX(emp_salary) FROM t_emp
          )
  </select>
  ```

- 返回List类型：查询结果返回多个实体类对象，希望把多个实体类对象放在List集合中返回。此时不需要任何特殊处理，在resultType属性中还是设置实体类类型即可。

  ```xml
  <!-- List<Employee> selectAll(); -->
  <select id="selectAll" resultType="com.pokaboo.mybatis.entity.Employee">
      select emp_id empId,emp_name empName,emp_salary empSalary from t_emp
  </select>
  ```

- 返回自增主键

  ```xml
  <!-- int insertEmployee(Employee employee); -->
  <!-- useGeneratedKeys属性字面意思就是“使用生成的主键” -->
  <!-- keyProperty属性可以指定主键在实体类对象中对应的属性名，Mybatis会将拿到的主键值存入这个属性 -->
  <insert id="insertEmployee" useGeneratedKeys="true" keyProperty="empId">
      insert into t_emp(emp_name,emp_salary) values (#{empName},#{empSalary})
  </insert>
  ```

  - 使用场景：例如：保存订单信息。需要保存Order对象和List<OrderItem>。其中，OrderItem对应的数据库表，包含一个外键，指向Order对应表的主键。
  - Mybatis是将自增主键的值设置到实体类对象中，而**不是以Mapper接口方法返回值**的形式返回。

- 不支持自增主键的数据库：而对于不支持自增型主键的数据库（例如 Oracle），则可以使用 selectKey 子元素：selectKey  元素将会首先运行，id  会被设置，然后插入语句会被调用

  ```xml
  <insert id="insertEmployee" parameterType="com.pokaboo.mybatis.beans.Employee"  databaseId="oracle">
          <selectKey order="BEFORE" keyProperty="id" resultType="integer">
              select employee_seq.nextval from dual 
          </selectKey>    
          insert into orcl_employee(id,last_name,email,gender) values(#{id},{lastName},#{email},#{gender})
  </insert>
  <!-- 或者 -->
  <insert id="insertEmployee" parameterType="com.pokaboo.mybatis.beans.Employee"  databaseId="oracle">
          <selectKey order="AFTER" keyProperty="id"  resultType="integer">
              select employee_seq.currval from dual 
          </selectKey>    
      insert into orcl_employee(id,last_name,email,gender) values(employee_seq.nextval,#{lastName},#{email},#{gender})
  </insert>
  ```

- 数据库字段和实体类属性对应关系

  - 别名：将字段的别名设置成和实体类属性一致、

    ```xml
    <!-- 编写具体的SQL语句，使用id属性唯一的标记一条SQL语句 -->
    <!-- resultType属性：指定封装查询结果的Java实体类的全类名 -->
    <select id="selectEmployee" resultType="com.pokaboo.mybatis.entity.Employee">
        <!-- Mybatis负责把SQL语句中的#{}部分替换成“?”占位符 -->
        <!-- 给每一个字段设置一个别名，让别名和Java实体类中属性名一致 -->
        select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{maomi}
    </select>
    ```

    >关于实体类属性的约定：

    > getXxx()方法、setXxx()方法把方法名中的get或set去掉，首字母小写。

  - 全局配置自动识别驼峰式命名规则：在Mybatis全局配置文件加入如下配置

    ```xml
    <!-- 使用settings对Mybatis全局进行设置 -->
    <settings>
        <!-- 将xxx_xxx这样的列名自动映射到xxXxx这样驼峰式命名的属性名 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```

- <font color='red;font-weight:bold;'>使用resultMap</font>：使用resultMap标签定义对应关系，再在后面的SQL语句中引用这个对应关系

  ```xml
  <!-- 专门声明一个resultMap设定column到property之间的对应关系 -->
  <resultMap id="selectEmployeeByRMResultMap" type="com.pokaboo.mybatis.entity.Employee">  
      <!-- 使用id标签设置主键列和主键属性之间的对应关系 -->
      <!-- column属性用于指定字段名；property属性用于指定Java实体类属性名 -->
      <id column="emp_id" property="empId"/>
      
      <!-- 使用result标签设置普通字段和Java实体类属性之间的关系 -->
      <result column="emp_name" property="empName"/>
      <result column="emp_salary" property="empSalary"/>
  </resultMap>    
  <!-- Employee selectEmployeeByRM(Integer empId); -->
  <select id="selectEmployeeByRM" resultMap="selectEmployeeByRMResultMap">
      select emp_id,emp_name,emp_salary from t_emp where emp_id=#{empId}
  </select>
  ```

### 对象关联

- 一对一 ：<font color='red'>association</font>、<font color='red'>javaType</font>

  ```xml
  <!-- 创建resultMap实现“对一”关联关系映射 -->
  <!-- id属性：通常设置为这个resultMap所服务的那条SQL语句的id加上“ResultMap” -->
  <!-- type属性：要设置为这个resultMap所服务的那条SQL语句最终要返回的类型 -->
  <resultMap id="selectOrderWithCustomerResultMap" type="com.pokaboo.mybatis.entity.Order">
      <!-- 先设置Order自身属性和字段的对应关系 -->
      <id column="order_id" property="orderId"/>
      <result column="order_name" property="orderName"/>
      <!-- 使用association标签配置“对一”关联关系 -->
      <!-- property属性：在Order类中对一的一端进行引用时使用的属性名 -->
      <!-- javaType属性：一的一端类的全类名 -->
      <association property="customer" javaType="com.pokaboo.mybatis.entity.Customer">
          <!-- 配置Customer类的属性和字段名之间的对应关系 -->
          <id column="customer_id" property="customerId"/>
          <result column="customer_name" property="customerName"/>
      </association>
  </resultMap>
  <!-- Order selectOrderWithCustomer(Integer orderId); -->
  <select id="selectOrderWithCustomer" resultMap="selectOrderWithCustomerResultMap">
      SELECT order_id,order_name,c.customer_id,customer_name
      FROM t_order o
      LEFT JOIN t_customer c
      ON o.customer_id=c.customer_id
      WHERE o.order_id=#{orderId}
  </select>
  ```

- 一对多：<font color='red'>collection</font>、<font color='red'>ofType</font>

  ```xml
  <!-- 配置resultMap实现从Customer到OrderList的“对多”关联关系 -->
  <resultMap id="selectCustomerWithOrderListResultMap" type="com.pokaboo.mybatis.entity.Customer">
      <!-- 映射Customer本身的属性 -->
      <id column="customer_id" property="customerId"/>
      <result column="customer_name" property="customerName"/>
      
      <!-- collection标签：映射“对多”的关联关系 -->
      <!-- property属性：在Customer类中，关联“多”的一端的属性名 -->
      <!-- ofType属性：集合属性中元素的类型 -->
      <collection property="orderList" ofType="com.pokaboo.mybatis.entity.Order">
          <!-- 映射Order的属性 -->
          <id column="order_id" property="orderId"/>
          <result column="order_name" property="orderName"/>
      </collection>
  </resultMap>
      
  <!-- Customer selectCustomerWithOrderList(Integer customerId); -->
  <select id="selectCustomerWithOrderList" resultMap="selectCustomerWithOrderListResultMap">
      SELECT c.customer_id,c.customer_name,o.order_id,o.order_name
      FROM t_customer c
      LEFT JOIN t_order o
      ON c.customer_id=o.customer_id
      WHERE c.customer_id=#{customerId}
  </select>
  ```

### 分步查询

```xml
<!-- orderList集合属性的映射关系，使用分步查询 -->
<!-- 在collection标签中使用select属性指定要引用的SQL语句 -->
<!-- select属性值的格式是：Mapper配置文件的名称空间.SQL语句id -->
<!-- column属性：指定Customer和Order之间建立关联关系时所依赖的字段 -->
<collection property="orderList" select="com.pokaboo.mybatis.mapper.CustomerMapper.selectOrderList" column="customer_id"/>
```

### 延迟加载

```xml
<!-- Mybatis全局配置 -->
<settings>
    <!-- 开启延迟加载功能 -->
    <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

### 动态SQL

- if和where标签

  ```xml
  <!-- List<Employee> selectEmployeeByCondition(Employee employee); -->
  <select id="selectEmployeeByCondition" resultType="com.pokaboo.mybatis.entity.Employee">
      select emp_id,emp_name,emp_salary from t_emp
      
      <!-- where标签会自动去掉“标签体内前面、后面多余的and/or” -->
      <where>
          <!-- 使用if标签，让我们可以有选择的加入SQL语句的片段。这个SQL语句片段是否要加入整个SQL语句，就看if标签判断的结果是否为true -->
          <!-- 在if标签的test属性中，可以访问实体类的属性，不可以访问数据库表的字段 -->
          <if test="empName != null">
              <!-- 在if标签内部，需要访问接口的参数时还是正常写#{} -->
              or emp_name=#{empName}
          </if>
          <if test="empSalary &gt; 2000">
              or emp_salary>#{empSalary}
          </if>
          <!--
           第一种情况：所有条件都满足 WHERE emp_name=? or emp_salary>?
           第二种情况：部分条件满足 WHERE emp_salary>?
           第三种情况：所有条件都不满足 没有where子句
           -->
      </where>
  </select>
  ```

- set标签：实际开发时，对一个实体类对象进行更新。往往不是更新所有字段，而是更新一部分字段。此时页面上的表单往往不会给不修改的字段提供表单项

  ```xml
  <!-- void updateEmployeeDynamic(Employee employee) -->
  <update id="updateEmployeeDynamic">
      update t_emp
      <!-- set emp_name=#{empName},emp_salary=#{empSalary} -->
      <!-- 使用set标签动态管理set子句，并且动态去掉两端多余的逗号 -->
      <set>
          <if test="empName != null">
              emp_name=#{empName},
          </if>
          <if test="empSalary &lt; 3000">
              emp_salary=#{empSalary},
          </if>
      </set>
      where emp_id=#{empId}
      <!--
           第一种情况：所有条件都满足 SET emp_name=?, emp_salary=?
           第二种情况：部分条件满足 SET emp_salary=?
           第三种情况：所有条件都不满足 update t_emp where emp_id=?
              没有set子句的update语句会导致SQL语法错误
       -->
  </update>
  ```

- trim：使用trim标签控制条件部分两端是否包含某些字符

  ```xml
  <!-- List<Employee> selectEmployeeByConditionByTrim(Employee employee) -->
  <select id="selectEmployeeByConditionByTrim" resultType="com.pokaboo.mybatis.entity.Employee">
      select emp_id,emp_name,emp_age,emp_salary,emp_gender
      from t_emp
      
      <!-- prefix属性指定要动态添加的前缀 -->
      <!-- suffix属性指定要动态添加的后缀 -->
      <!-- prefixOverrides属性指定要动态去掉的前缀，使用“|”分隔有可能的多个值 -->
      <!-- suffixOverrides属性指定要动态去掉的后缀，使用“|”分隔有可能的多个值 -->
      <!-- 当前例子用where标签实现更简洁，但是trim标签更灵活，可以用在任何有需要的地方 -->
      <trim prefix="where" suffixOverrides="and|or">
          <if test="empName != null">
              emp_name=#{empName} and
          </if>
          <if test="empSalary &gt; 3000">
              emp_salary>#{empSalary} and
          </if>
          <if test="empAge &lt;= 20">
              emp_age=#{empAge} or
          </if>
          <if test="empGender=='male'">
              emp_gender=#{empGender}
          </if>
      </trim>
  </select>
  ```

  - prefix属性：指定要动态添加的前缀
  - suffix属性：指定要动态添加的后缀
  - prefixOverrides属性：指定要动态去掉的前缀，使用“|”分隔有可能的多个值
  - suffixOverrides属性：指定要动态去掉的后缀，使用“|”分隔有可能的多个值

- choose/when/otherwise标签：多个分支条件中，仅执行一个

  ```xml
  <!-- List<Employee> selectEmployeeByConditionByChoose(Employee employee) -->
  <select id="selectEmployeeByConditionByChoose" resultType="com.pokaboo.mybatis.entity.Employee">
      select emp_id,emp_name,emp_salary from t_emp
      where
      <choose>
          <when test="empName != null">emp_name=#{empName}</when>
          <when test="empSalary &lt; 3000">emp_salary &lt; 3000</when>
          <otherwise>1=1</otherwise>
      </choose>
      
      <!--
       第一种情况：第一个when满足条件 where emp_name=?
       第二种情况：第二个when满足条件 where emp_salary < 3000
       第三种情况：两个when都不满足 where 1=1 执行了otherwise
       -->
  </select>
  ```

- foreach标签：

  ```xml
  <!--
      collection属性：要遍历的集合
      item属性：遍历集合的过程中能得到每一个具体对象，在item属性中设置一个名字，将来通过这个名字引用遍历出来的对象
      separator属性：指定当foreach标签的标签体重复拼接字符串时，各个标签体字符串之间的分隔符
      open属性：指定整个循环把字符串拼好后，字符串整体的前面要添加的字符串
      close属性：指定整个循环把字符串拼好后，字符串整体的后面要添加的字符串
      index属性：这里起一个名字，便于后面引用
          遍历List集合，这里能够得到List集合的索引值
          遍历Map集合，这里能够得到Map集合的key
   -->
  <foreach collection="empList" item="emp" separator="," open="values" index="myIndex">
      <!-- 在foreach标签内部如果需要引用遍历得到的具体的一个对象，需要使用item属性声明的名称 -->
      (#{emp.empName},#{myIndex},#{emp.empSalary},#{emp.empGender})
  </foreach>
  ```

  > 上面批量插入的例子本质上是一条SQL语句，而实现批量更新则需要多条SQL语句拼起来，用分号分开。也就是一次性发送多条SQL语句让数据库执行。此时需要在**数据库连接信息的URL地址**中设置：
  >
  > ```
  > dev.url=jdbc:mysql://192.168.198.100:3306/test?allowMultiQueries=true
  > ```
  >
  > ```
  > 对应的foreach标签如下:
  > <!-- int updateEmployeeBatch(@Param("empList") List<Employee> empList) -->
  > <update id="updateEmployeeBatch">
  >     <foreach collection="empList" item="emp" separator=";">
  >         update t_emp set emp_name=#{emp.empName} where emp_id=#{emp.empId}
  >     </foreach>
  > </update>
  > ```

- SQL标签：

  ```xml
  <!-- 使用sql标签抽取重复出现的SQL片段 -->
  <sql id="mySelectSql">
  	select emp_id,emp_name,emp_age,emp_salary,emp_gender from t_emp
  </sql>
  
  <!-- 使用include标签引用声明的SQL片段 -->
  <include refid="mySelectSql"/>
  ```

### 缓存

### 逆向工程







