# Spring-data-JPA

## ORM 思想

​	对象关系映射，将对象与数据库表建立映射关系。ORM框架封装了jdbc操作。使用ORM思想可以使得开发者不再关注sql语句的编写，根据映射关系自动生成sql语句。

​	mybatis， hibernate 都实现了ORM思想。

## 映射关系

```java
Java类-----------------------表
类的属性----------------------表的字段
类的对象----------------------表的数据行
```

## JPA 规范

​	使用ORM思想所遵循的规范，sun公司对其定义的JPA规范，包含多个规范接口和抽象类。

​	优势：

​	1、标准化

​	2、容器级特性的支持

​	3、简单方便

​	4、查询能力

​	5、高级特性



## 配置文件（hibernate）

在resource目录下创建META-INF目录，新建配置文件persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
    http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">

    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa实现方式-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <!-- 数据库驱动 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
            <!-- 数据库地址 -->
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://192.168.153.130:3306/ssh?characterEncoding=utf-8" />
            <!-- 数据库用户名 -->
            <property name="javax.persistence.jdbc.user" value="root" />
            <!-- 数据库密码 -->
            <property name="javax.persistence.jdbc.password" value="mysql" />

            <!--jpa提供者的可选配置：我们的JPA规范的提供者为hibernate，所以jpa的核心配置中兼容hibernate的配 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />

        </properties>
    </persistence-unit>
</persistence>
```

## 配置文件（spring 容器）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!--spring 和 spring data jpa的配置-->

    <!-- 1.创建entityManagerFactory对象交给spring容器管理-->
    <bean id="entityManagerFactoty" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!--配置的扫描的包（实体类所在的包） -->
        <property name="packagesToScan" value="cn.itcast.domain" />
        <!-- jpa的实现厂家 -->
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>

        <!--jpa的供应商适配器 -->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!--配置是否自动创建数据库表 -->
                <property name="generateDdl" value="false" />
                <!--指定数据库类型 -->
                <property name="database" value="MYSQL" />
                <!--数据库方言：支持的特有语法 -->
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect" />
                <!--是否显示sql -->
                <property name="showSql" value="true" />
            </bean>
        </property>

        <!--jpa的方言 ：高级的特性 -->
        <property name="jpaDialect" >
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
        </property>

    </bean>

    <!--2.创建数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user" value="root"></property>
        <property name="password" value="mysql"></property>
        <property name="jdbcUrl" value="jdbc:mysql://192.168.153.130/ssh?characterEncoding=utf-8" ></property>
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    </bean>

    <!--3.整合spring dataJpa-->
    <jpa:repositories base-package="cn.itcast.dao" transaction-manager-ref="transactionManager"
                      entity-manager-factory-ref="entityManagerFactoty" ></jpa:repositories>

    <!--4.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactoty"></property>
    </bean>

    <!-- 4.txAdvice-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 5.aop-->
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* cn.itcast.service.*.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
    </aop:config>


    <!--5.声明式事务 -->

    <!-- 6. 配置包扫描-->
    <context:component-scan base-package="cn.itcast" ></context:component-scan>
</beans>
```



## 使用

```java
//创建entity工厂对象
EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
//创建entityManager对象
EntityManager entityManager = factory.createEntityManager();
//创建事务管理
EntityTransaction transaction = entityManager.getTransaction();
//开启事务
transaction.begin();
//创建数据对象
Customer customer = new Customer();
customer.setCustName("testName");
//提交事务
entityManager.persist(customer);
//释放资源
entityManager.close();
factory.close();
```



## 懒查询（延迟加载）

```java
//根据id查询表
Custom c = entityManager.reference(Custorm.class, 1L);
//提交查询事务，但不执行操作
transaction.commit();
//使用c对象，执行查询操作
System.out.print(c);
```

懒查询是当使用到 c 对象时， 再开始执行查询操作

## 主键策略

- SEQUENCE:序列(Oracle数据库用)
- IDENTITY:数据库自增(Mysql数据库用)
- TABLE:Hibernate负责生成主键值,并自动创建一个序列表,存储实体类对应表中的主键值.
- AUTO:Hibernate自动选择主键生成策略(优先选择使用SEQUENCE)

## 方法名称规则查询

​	使用名称规则查询，方法名按照规定格式声明，则可以不使用 @Query 注解查询语句。

![方法命名规则](D:\Java\笔记\方法命名规则.png)

```
 //findBy 规则查询
 Customer findByCustName(String custName);
 
 // select * from Custormer where custName like ?
 List<Customer> findByCustNameLike(String custName);

 //select * from Custormer where custNmae like ? and custIndustry=?
 Customer findByCustNameLikeAndCustIndustry(String custName, String custIndustry);
```



# specifications 动态查询

​	借助 Specification<T> 条件拼接实现动态查询， 体现出查询的面向对象特点。

## 查询方式

| 方法名称                    | Sql对应关系          |
| --------------------------- | -------------------- |
| equle                       | filed = value        |
| gt（greaterThan ）          | filed > value        |
| lt（lessThan ）             | filed < value        |
| ge（greaterThanOrEqualTo ） | filed >= value       |
| le（ lessThanOrEqualTo）    | filed <= value       |
| notEqule                    | filed != value       |
| like                        | filed like value     |
| notLike                     | filed not like value |

## 查询案例

```java
//创建条件组合对象 
Specification<Customer> spec = new Specification<Customer>() {
            @Override
    		/**
    		*匿名内部类来设置查询条件
            *root:	通过root对象，获取实体类对象中的属性（对应数据库字段名）地址
            *criteriaQuery: 必须在实体类型或嵌入式类型上的Criteria 查询上起作用。用来自定义查询
            *criteriaBuilder: 该对象用来对属性赋值，并创建jpa查询条件对象，如：like，and，or等
            **/
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                //获取属性地址
                Path<Object> custName = root.get("custName");
				//创建like条件
                Predicate pre = criteriaBuilder.like(custName.as(String.class), "%好%");
                //创建完全匹配条件
                Predicate pre2 = criteriaBuilder.equal(custName, "好");
				//追加and条件查询
                Predicate and = criteriaBuilder.and(pre);
                //返回jpa查询对象
                return and;
            }
        };
		//提交查询
        List<Customer> all = customerDao.findAll(spec);
        for (Customer customer : all) {
            System.out.println(customer);
        }
```



## 多表查询

### 注解说明

```txt
@OneToMany:
   	作用：建立一对多的关系映射
    属性：
    	targetEntityClass：指定多的多方的类的字节码
    	mappedBy：指定从表实体类中引用主表对象的名称。
    	cascade：指定要使用的级联操作
    	fetch：指定是否采用延迟加载
    	orphanRemoval：是否使用孤儿删除

@ManyToOne
    作用：建立多对一的关系
    属性：
    	targetEntityClass：指定一的一方实体类字节码
    	cascade：指定要使用的级联操作
    	fetch：指定是否采用延迟加载
    	optional：关联是否可选。如果设置为false，则必须始终存在非空关系。
    	
    	
3.4	映射的注解说明
@OneToMany:
   	作用：建立一对多的关系映射
    属性：
    	targetEntityClass：指定多的多方的类的字节码
    	mappedBy：指定从表实体类中引用主表对象的名称。
    	cascade：指定要使用的级联操作
    	fetch：指定是否采用延迟加载
    	orphanRemoval：是否使用孤儿删除

@ManyToOne
    作用：建立多对一的关系
    属性：
    	targetEntityClass：指定一的一方实体类字节码
    	cascade：指定要使用的级联操作
    	fetch：指定是否采用延迟加载
    	optional：关联是否可选。如果设置为false，则必须始终存在非空关系。

@JoinColumn
     作用：用于定义主键字段和外键字段的对应关系。
     属性：
    	name：指定外键字段的名称
    	referencedColumnName：指定引用主表的主键字段名称
    	unique：是否唯一。默认值不唯一
    	nullable：是否允许为空。默认值允许。
    	insertable：是否允许插入。默认值允许。
    	updatable：是否允许更新。默认值允许。
    	columnDefinition：列的定义信息。
```

1、明确关系注解：@OneToMany, @ManyToOne, @ManyToMany, @OneToOne

2、确定由哪方维护外键关系， 另一方放弃维护外键

3、维护关系的一方添加外键维护注解：@JoinColumn(name="customer")

### 一对多

```java
@Entity//表示当前类是一个实体类
@Table(name="cst_customer")//建立当前实体类和表之间的对应关系
public class Customer implements Serializable {
  	
    ...
   
    //配置客户和联系人的一对多关系
  	@OneToMany(targetEntity=LinkMan.class, mappedBy="customer")
    //name:外键
    //referenceColumnName：主键
    //一对多情况下，一方放弃维护外键关系，在OneToMany中添加一方在多方中的属性名。
	//@JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
	private Set<LinkMan> linkmans = new HashSet<LinkMan>(0);

    ...
}

@Entity
@Table(name="cst_linkman")
public class LinkMan implements Serializable {
	
    ...
	
    //多对一关系映射：多个联系人对应客户
	@ManyToOne(targetEntity=Customer.class)
	@JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
	private Customer customer;//用它的主键，对应联系人表中的外键
    
    ...
}

public void test(){
    Customer customer = new Customer();
    customer.setCustName = "abc";
    
    LinkMan linkMan = new LinkMan();
    linkMan.setCustomer(customer);
    
    //若一方没有放弃维护主键，保存时则会执行多余的update操作，影响执行效率。
    customerDao.save();
    linkManDao.save();
}
```

### 级联操作

​	一般避免使用级联操作。

​	级联删除过程：先删除被级联的表，后删除维护主键的表

​	1、在默认情况下，它会把外键字段置为 null，然后删除主表数据。如果在数据库的表结构上，外键字段有非空约束，默认情况就会报错了。

​    2、如果配置了放弃维护关联关系的权利，则不能删除（与外键字段是否允许为 null，没有关系）因为在删除时，它根本不会去更新从表的外键字段了。

​    3、如果还想删除，使用级联删除引用



​	使用方式：在关联注解中开启 cascade 属性

```java
	/**
	 * cascade:配置级联操作, 有效执行
	 * 		CascadeType.MERGE	级联更新
	 * 		CascadeType.PERSIST	级联保存：
	 * 		CascadeType.REFRESH 级联刷新：
	 * 		CascadeType.REMOVE	级联删除：
	 * 		CascadeType.ALL		包含所有
	 */
@OneToMany(mappedBy="customer", cascade=CascadeType.ALL)
```

### 多对多

```txt
@ManyToMany
	作用：用于映射多对多关系
	属性：
		cascade：配置级联操作。
		fetch：配置是否采用延迟加载。
    	targetEntity：配置目标的实体类。映射多对多的时候不用写。

@JoinTable
    作用：针对中间表的配置
    属性：
    	nam：配置中间表的名称
    	joinColumns：中间表的外键字段关联当前实体类所对应表的主键字段  本类外键属性
    	inverseJoinColumn：中间表的外键字段关联对方表的主键字段  关联类外键属性
    	
@JoinColumn
    作用：用于定义主键字段和外键字段的对应关系。
    属性：
    	name：指定外键字段的名称
    	referencedColumnName：指定引用主表的主键字段名称
    	unique：是否唯一。默认值不唯一
    	nullable：是否允许为空。默认值允许。
    	insertable：是否允许插入。默认值允许。
    	updatable：是否允许更新。默认值允许。
    	columnDefinition：列的定义信息。
```



```java
@ManyToMany
@JoinTable(name = "u_r", joinColumns = {@JoinColumn(name="u_id")}, inverseJoinColumns = {@JoinColumn(name="r_id")})
```



## 对象导航查询

```java
/*注：一方查询多方时，默认为懒加载，会抛出no-session异常。
	 	在注解中开启事务@Transactional  
	 多方查询一方时，默认为立即加载*/
@Test
@Transactional
	//1、查询单个对象。
	LinkMan linkMan = linkManDao.findById(2L);
	//2、调用get方法，查询出关联对象集合。
	Customer customer = linkMna.getCustomer();
```

### 懒加载处理

​	通过查询一方，再查询关联多方时，默认为懒加载。可以在查询主体的实体类表关系注解中对fetch属性进行修改。

​	如：@ManyToMany (fetch = fetchType.EAGER)    LAZY



### 多表 specification 查询

​	使用 root.join 方式，进行连接查询。

```java
/*
*JoinType.LEFT
*JoinType.RIGHT
*JoinType.INNER
*/
Join<LinkMan, Customer> join = root.join("customer", JoinType.LEFT);
return cb.like(join.get("custName").as(String.class), "%好%")；
```



# 应用场景

​	简单的单表 CRUD 查询操作。

​	对多表关联查询使用相对 mybatis 开发效率略低。