**<center>Mybatis</center>**

- [1. MyBatis获取参数值的两种方式](#1-mybatis获取参数值的两种方式)
  - [1.1. 参数值的不同情况](#11-参数值的不同情况)
    - [1.1.1. 实体类类型的参数](#111-实体类类型的参数)
    - [1.1.2. 使用@Param标识参数](#112-使用param标识参数)
- [2. MyBatis的各种查询功能](#2-mybatis的各种查询功能)
- [3. 特殊SQL的执行](#3-特殊sql的执行)
  - [3.1. 模糊查询](#31-模糊查询)
  - [3.2. 批量删除](#32-批量删除)
  - [3.3. 动态设置表名](#33-动态设置表名)
  - [3.4. 添加功能获取自增的主键](#34-添加功能获取自增的主键)
- [4. 自定义映射resultMap](#4-自定义映射resultmap)
  - [4.1. 字段名与属性名不一致的情况](#41-字段名与属性名不一致的情况)
  - [4.2. 处理多对一映射关系](#42-处理多对一映射关系)
  - [4.3. 处理一对多映射关系](#43-处理一对多映射关系)
- [5. 动态SQL](#5-动态sql)
- [6. MyBatis的缓存](#6-mybatis的缓存)
  - [6.1. 一级缓存](#61-一级缓存)
  - [6.2. 二级缓存](#62-二级缓存)
    - [6.2.1. 二级缓存条件](#621-二级缓存条件)
    - [6.2.2. 二级缓存的相关配置](#622-二级缓存的相关配置)
  - [6.3. MyBatis缓存查询的顺序](#63-mybatis缓存查询的顺序)
- [7. Mybatis的逆向工程](#7-mybatis的逆向工程)

# 1. MyBatis获取参数值的两种方式
MyBatis获取参数值的两种方式：${}和#{}
- ${}的本质就是字符串拼接，#{}的本质就是占位符赋值
- ${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号；但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自
动添加单引号。

## 1.1. 参数值的不同情况
### 1.1.1. 实体类类型的参数
若mapper接口中的方法参数为实体类对象时。此时可以使用\${}和#{}，通过访问实体类对象中的属性名获取属性值，注意${}需要手动加单引号

### 1.1.2. 使用@Param标识参数
以@Param注解命名参数：此时MyBatis会将这些参数放在一个map集合中，以两种方式存储：
- 以@Param注解的值为键，以参数为值
- 以param1,param2...为键，以参数为值
因此只需要通过#{}或'${}'以键的方式访问值即可。

# 2. MyBatis的各种查询功能

1. 若查询出的数据只有一条
    - 通过实体类对象接收
    - 通过list集合接收
    - 通过map结合接收
2. 若查询出的数据有多条
     - 通过实体类类型的list集合接收
     - 通过map类型的list集合接收
     - 可以在mapper接口的方法上添加@Mapkey注解，此时就可以将每条数据转换的map集合作为值，以某个字段的值（通常为唯一约束的字段）作为键，放在同一个map集合。可以理解为map里面套map。

# 3. 特殊SQL的执行
## 3.1. 模糊查询
三种方式：
```sql
# 方式一
select * from t_user where username like concat('%',#{username},'%')
# 方式二
select * from  t_user where username like '%${username}%'
# 方式三（推荐）
select * from  t_user where username "%"#{username}"%"
```

## 3.2. 批量删除
接口方法：
```java
int deleteMore(@Param("ids") String ids);
```
SQL语句：
```sql
delete from t_user where id in (${ids})
```

## 3.3. 动态设置表名
接口方法：
```java
List<User> getUserByTableName(@Param("tableName") String tableName);
```
SQL语句：
```sql
select * from ${tableName}
```

## 3.4. 添加功能获取自增的主键
接口方法：
```java
void insertUser(User user);
```
SQL语句：
```xml
<!--
        void insertUser(User user);
        useGeneratedKeys:设置当前标签中的sql使用了自增的主键
        keyProperty:将自增的主键的值赋值给传输到的映射文件中参数的某个属性
-->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    insert into t_user values (null,#{username},#{password},#{age},#{sex},#{email})
</insert>
```

# 4. 自定义映射resultMap
## 4.1. 字段名与属性名不一致的情况
- 在sql语句中为字段取别名，保持与属性名一致
- 设置全局配置，将下划线自动映射为驼峰。如：emp_name -> empName
    ```xml
    <settings>
        <!-- 将字段名中的下划线自动映射为驼峰:emp_name->empName -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```
- 通过resultMap设置自定义的映射关系(通常用处理多对一或一对多关系)
    ```xml
    <!--  
        resultMap:设置自定义映射关系
        id:唯一标识，不能重复
        type:设置映射关系中的实体类类型
        子标签：
            id:设置主键的映射关系
            result:设置普通字段的映射关系
        属性：
            property:设置映射关系中的属性名，必须是type属性所设置的实体类类型中的属性名
            column:设置映射关系中的字段名，必须是sql语句查询出的字段名
    -->
    <resultMap id="empResultMap" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
    </resultMap>
    ```

## 4.2. 处理多对一映射关系
- 级联属性赋值
    ```xml
    <resultMap id="empAndDeptResultMapOne" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <result property="dept.did" column="did"/>
        <result property="dept.deptName" column="dept_name"/>
    </resultMap>
    ```
- 通过association
    ```xml
    <!--
        association:处理多对一的映射关系
        property:需要处理多对的映射关系的属性名
        javaType:该属性的类型
    -->
    <resultMap id="empAndDeptResultMapTwo" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <association property="dept" javaType="Dept">
            <id property="did" column="did"/>
            <result property="deptName" column="dept_name"/>
        </association>
    </resultMap>
    ```
- 分布查询
    ```xml
    <!-- Emp中的reslutMap配置 -->
    <resultMap id="empAndDeptByStepResultMap" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <!--
            select:设置分布查询的sql的唯一标识(namespace.SQLId或mapper接口的全类名.方法名)
            column:设置分布查询的条件
            fetchType="eager">
        -->
        <association property="dept"
                     select="com.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                     column="did"
                     fetchType="eager">
        </association>
    </resultMap>

    <!-- DeptMapper中的SQL:根据员工所对应的部门id查询部门信息-->
    <!--  Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);  -->
    <select id="getEmpAndDeptByStepTwo" resultType="Dept">
        select * from t_dept where did = #{did}
    </select>


    <!--  Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);  -->
    <select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
        select * from t_emp where eid = #{eid}
    </select>
    ```

## 4.3. 处理一对多映射关系

- collection方法
    ```xml
     <resultMap id="deptAndEmpResultEmp" type="Dept">
        <id property="did" column="did"/>
        <result property="deptName" column="dept_name"/>
        <!--
            collection:处理一对多的映射关系
            ofType:表示该属性所对应的集合中存储数据的类型
        -->
        <collection property="emps" ofType="Emp">
            <id property="eid" column="eid"/>
            <result property="empName" column="emp_name"/>
            <result property="age" column="age"/>
            <result property="sex" column="sex"/>
            <result property="email" column="email"/>
        </collection>
    </resultMap>
    
    <!--  Dept getDeptAndEmp(@Param("did") Integer did);  -->
    <select id="getDeptAndEmp" resultMap="deptAndEmpResultEmp">
        select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did=#{did}
    </select>
    ```
- 分布查询方法
    ```xml
    <resultMap id="deptAndEmpByStepResultMap" type="Dept">
        <id property="did" column="did"/>
        <result property="deptName" column="dept_name"/>
        <collection property="emps"
                    select="com.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                    column="did"
                    fetchType="eager"/>
    </resultMap>
    <!--Dept getDeptAndEmpByStepOne(@Param("did") Integer did);-->
    <select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpByStepResultMap">
        select * from t_dept where did = #{did}
    </select>

    <!--  List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);  -->
    <select id="getDeptAndEmpByStepTwo" resultType="emp">
        select * from t_emp where did = #{did}
    </select>
    ```

# 5. 动态SQL
1. `if`:根据标签中test属性所对应的表达式决定标签中的内容是否需要拼接到SQL中
2. `where`:
    - 当where标签中有内容时，会自动生成where关键字，并且将内容前多余的and或or去掉
    - 当where标签中没有内容时，此时where标签没有任何效果
    - where标签不能将其中内容后面多余的and或or去掉
3. `trim`:
    - 若标签中有内容时：
      - `prefix|suffix`:将trim标签中内容前面或后面添加指定内容
      - `suffixOverrides|prefixOverrides`:将trim标签中内容前面或后面去掉指定内容
    - 若标签中没有内容时，trim标签也没有任何效果
4. `choose,when,otherwise`（相当于`if,else if,else`）
   when至少要有一个，otherwise最多有一个
5. `foreach`
    - collection:设置需要循环的数据或集合
    - item:表示数组或集合中的每一个数据
    - separator:循环体之间的分隔符
    - open:foreach标签所循环的所有内容的开始符
    - close:foreach标签所循环的所有内容的结束符
6. sql标签
    - 设置SQL片段：
        ```xml
        <sql id="empColumns">
        eid,emp_name,age,sex,email
        </sql>
        ```
    - 引用SQL片段：
        ```xml
        <include refid="empColumns"></include>
        ```

# 6. MyBatis的缓存
## 6.1. 一级缓存
一级缓存是`SqlSession`级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问。使**一级缓存失效**的四种情况：
1. 不同的SqlSession对应不同的一级缓存
2. 同一个SqlSession但是查询条件不同
3. 同一个SqlSession两次查询期间执行了任何一次增删改操作
4. 同一个SqlSession两次查询期间手动清空了缓存

**任意的一次增删改操作都会清空一级缓存。**

## 6.2. 二级缓存
### 6.2.1. 二级缓存条件
二级缓存是`SqlSessionFactory`级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取二级缓存开启的条件：
1. 在核心配置文件中，设置全局配置属性`cacheEnabled="true"`，默认为true，不需要设置
2. 在映射文件中设置标签<cache />
3. 二级缓存必须在SqlSession关闭或提交之后有效
4. 查询的数据所转换的实体类类型必须实现序列化的接口

**使二级缓存失效的情况：**
两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

### 6.2.2. 二级缓存的相关配置
在mapper配置文件中添加的cache标签可以设置一些属性：
- eviction属性：缓存回收策略
    - **LRU**（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。
    - FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。
    - SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
    - WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
    - **默认的是 LRU。**

- flushInterval属性：刷新间隔，单位毫秒
默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新
- size属性：引用数目，正整数
代表缓存最多可以存储多少个对象，太大容易导致内存溢出
- readOnly属性：只读，true/false
    - true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
    - **false**：读写缓存；会返回缓存对象的拷贝（通过序列化）。
    - 为了保证安全，默认是**false**。

## 6.3. MyBatis缓存查询的顺序
- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。
- 如果二级缓存没有命中，再查询一级缓存
- 如果一级缓存也没有命中，则查询数据库
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

# 7. Mybatis的逆向工程
1. 添加依赖和插件
   ```xml
    <!-- 依赖MyBatis核心包 -->
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
    </dependencies>
    <build>
        <!-- 构建过程中用到的插件 -->
        <plugins>
            <!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.0</version>
                <!-- 插件的依赖 -->
                <dependencies>
                    <!-- 逆向工程的核心依赖 -->
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.2</version>
                    </dependency>
                    <!-- 数据库连接池 -->
                    <dependency>
                        <groupId>com.mchange</groupId>
                        <artifactId>c3p0</artifactId>
                        <version>0.9.2</version>
                    </dependency>
                    <!-- MySQL驱动 -->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.28</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
   ```
2. 创建MyBatis的核心配置文件:mybatis-config
3. 创建逆向工程的配置文件
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
            "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    <generatorConfiguration>
        <!--
            targetRuntime: 执行生成的逆向工程的版本
            MyBatis3Simple: 生成基本的CRUD（清新简洁版）
            MyBatis3: 生成带条件的CRUD（奢华尊享版）
        -->
        <context id="DB2Tables" targetRuntime="MyBatis3">
            <!-- 数据库的连接信息 -->
            <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                            connectionURL="jdbc:mysql://localhost:3306/mybatis"
                            userId="user"
                            password="password">
            </jdbcConnection>
            <!-- javaBean的生成策略-->
            <javaModelGenerator targetPackage="com.mybatis.pojo"
                            targetProject=".\src\main\java">
                <property name="enableSubPackages" value="true" />
                <property name="trimStrings" value="true" />
            </javaModelGenerator>
            <!-- SQL映射文件的生成策略 -->
            <sqlMapGenerator targetPackage="com.mybatis.mapper"
                        targetProject=".\src\main\resources">
                <property name="enableSubPackages" value="true" />
            </sqlMapGenerator>
            <!-- Mapper接口的生成策略 -->
            <javaClientGenerator type="XMLMAPPER"
                    targetPackage="com.mybatis.mapper" targetProject=".\src\main\java">
                <property name="enableSubPackages" value="true" />
            </javaClientGenerator>
            <!-- 逆向分析的表 -->
            <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
            <!-- domainObjectName属性指定生成出来的实体类的类名 -->
            <table tableName="t_emp" domainObjectName="Emp"/>
            <table tableName="t_dept" domainObjectName="Dept"/>
        </context>
    </generatorConfiguration>
    ```
4. 执行MBG插件的generate目标