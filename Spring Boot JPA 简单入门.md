# Spring Boot JPA 简单入门

[toc]

## 环境搭建 ##

在一个Spring Boot的项目基础上，在pom.xml添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.15</version>
</dependency>
```

在application.yml中配置数据源、数据连接池和JPA配置项：

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/retard?useUnicode=yes&characterEncoding=UTF-8&serverTimezone=GMT%2B8 # 数据库URL
    username: root # 数据库用户名
    password: 123456 # 数据库密码
    driver-class-name: com.mysql.cj.jdbc.Driver # JDBC类
    type: com.zaxxer.hikari.HikariDataSource # 数据连接池类,hikari是Spring Boot的默认连接池
    hikari:
      minimum-idle: 5 # 最小空闲连接，默认值10，小于0或大于maximum-pool-size，都会重置为maximum-pool-size
      maximum-pool-size: 15 # 最大连接数，小于等于0会被重置为默认值10；大于零小于1会被重置为minimum-idle的值
      auto-commit: true # 自动提交从池中返回的连接
      idle-timeout: 30000 # 只有空闲连接数大于最大连接数且空闲时间超过该值，才会被释放
      pool-name: DatebookHikariCP # 连接池名称，默认HikariPool-1
      max-lifetime: 1800000 # #连接最大存活时间.不等于0且小于30秒，会被重置为默认值30分钟.设置应该比mysql设置的超时时间短
      connection-timeout: 30000 # 连接超时时间:毫秒，小于250毫秒，否则被重置为默认值30秒
      connection-test-query: SELECT 1 # #连接测试查询
  jpa:
    hibernate:
      ddl-auto: update # jpa数据表更新模式
    show-sql: true # 显示sql
    open-in-view: true # 解决一个测试时no Session错误，此配置只针对web开发
    database-platform: org.hibernate.dialect.MySQL8Dialect # 指定sql方言
```

## Entity 映射（实体映射） ##

JPA需要将数据库中的数据映射成Entity Class，我们只需要掌握基于注解的映射配置即可。

### 基础映射 ###

实体类最好实现Serializable接口，当然不实现也不会影响程序。

```java
package com.retard.oauth.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonIgnore;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.ToString;
import lombok.experimental.Accessors;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Collection;
import java.util.Date;

/**
 * 用户实体
 */
@Data
@Accessors(chain = true)
@ToString(exclude = "password")
@Entity
@Table(name = "tb_user")
@EntityListeners(AuditingEntityListener.class)
@ApiModel(value = "用户", description = "实现了Spring Security的UserDetails接口")
public class User implements UserDetails, Serializable {

	@ApiModelProperty(value = "主键", dataType = "integer")
	@Id
	@Column(name = "id")
	@GeneratedValue(strategy = GenerationType.AUTO)
	private Long id;

	@Column(name = "username", unique = true, nullable = false, length = 32)
	private String username;

	@JsonIgnore
	@Column(name = "password", nullable = false, length = 64)
	private String password;

	@Column(name = "enable", nullable = false)
	private boolean enable = true;

	@Column(name = "expired", nullable = false)
	private boolean expired = false;

	@Column(name = "locked", nullable = false)
	private boolean locked = false;

	@Column(name = "credential_expired", nullable = false)
	private boolean credentialExpired = false;

	@CreatedDate
	@Temporal(TemporalType.TIMESTAMP)
	@Column(name = "created_time", nullable = false)
	@JsonFormat(pattern = "yyyy-MM-dd hh:mm:ss")
	private Date createdTime;

	@LastModifiedDate
	@Temporal(TemporalType.TIMESTAMP)
	@Column(name = "modified_time")
	@JsonFormat(pattern = "yyyy-MM-dd hh:mm:ss")
	private Date modifiedTime;

	@Transient
	private Collection<GrantedAuthority> authorities;

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		return null;
	}

	@Override
	public String getPassword() {
		return password;
	}

	@Override
	public String getUsername() {
		return username;
	}

	@Transient
	@Override
	public boolean isAccountNonExpired() {
		return !expired;
	}

	@Transient
	@Override
	public boolean isAccountNonLocked() {
		return !locked;
	}

	@Transient
	@Override
	public boolean isCredentialsNonExpired() {
		return !credentialExpired;
	}

	@Override
	public boolean isEnabled() {
		return enable;
	}
}
```

`@Entity`: 标注在类之上，表示一个实体。

`@Table`: 标注在类之上，设置数据库表名称，参数name为表名称。eg: `@Table(name="user")`。

`@Id`: 标注在属性上，表示主键。

`@GeneratedValue(strategy=GenerationType.IDENINY)`: 标注在属性之上，表示自增。strategy有四种自增策略:
- AUTO 主键由程序控制, 是默认选项 ,不设置就是这个
- IDENTITY 主键由数据库生成, 采用数据库自增长, Oracle不支持这种方式
- SEQUENCE 通过数据库的序列产生主键, MYSQL  不支持
- TABLE 提供特定的数据库产生主键, 该方式更有利于数据库的移植

`@Column`: 最重要的注解之一，标注于属性之上，用于设置数据表中的字段属性，它的参数较多，以下逐一介绍：
- name: 字段名称，如果不设置，默认使用属性名称
- unique: 唯一值或不重复，默认为false
- nullable: 可以为空，默认为true
- insertable: 出现在insert语句，默认为true。如果设置为false，新建实体时，不会在此字段插入值。
- updatable: 出现在update语句中，默认为true。如果设置为false，本字段在新建之后，不会被修改。
- columnDefinition: 属性表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用。（也就是说，如果DB中表已经建好，该属性没有必要使用。）eg:`@Column(columnDefinition="varchar(255) COMMENT '测试字段'")`。一般不使用。
- table: 指定表名称，一般不适用。
- length: 长度（主要针对varchar等类型）
- precision和scale: 用于双精度数字，precision表示数字总长度，scale表示小数点后的精度。

`@Transient`: 表示忽略此字段，不会在表中创建此字段。

`@Temporal(TemporalType.TIMESTAMP)`: 指定日期类型,有DATA、TIME、TIMESTAMP三种

`@Lob`: 表示大文本或二进制字段，使用它的时候最好同时加上`@Basic(fetch=FetchType.LAZY)`。

`@Enumerated(EnumType.ORDINAL)`: 用于枚举类型的属性，ORDINAL会将枚举映射为枚举顺序值，STRING会将枚举映射为字符串。

`@Convert(converter = GenderConverter.class)`: 自定类型属性转化，需要指定`AttributeConverter<F,A>`的实现类:

```java
public interface AttributeConverter<X,Y> {
    public Y convertToDatabaseColumn (X attribute);
    public X convertToEntityAttribute (Y dbData);
}
```

`@CreatedDate`和`@LastModifiedDate`：添加在日期属性上，当实体被创建或修改，都会自动修改数据库字段。但是必须在实体上添加`@EntityListeners(AuditingEntityListener.class)`字段，在Spring Boot的配置类上添加`@EnableJpaAuditing`字段，这两个注解才会生效。

`@EntityListeners(TestEntityListener.class)`: 标注在实体类之上，设置实体监听器。监听器实现方式如下：

```java
public class TestEntityListener{
    @PrePersist
    public void prePersist(Entitiy entity){
        // 保存之前调用
    }
    @PreUpdate
    public void preUpdate(Entity entity){
        // 更新之前调用
    }
    @PreRemove
    public void preRemove(Entity entity){
        // 删除之前调用
    }
    @PostPersist
    public void postPersist(Entity entity){
        // 保存之后调用
    }
    @PostUpdate
    public void postUpdate(Entity entity){
        // 更新之后调用
    }
    @PostRemove
    public void postRemove(Entity entity){
        // 删除之后调用
    }
}
```

`@MappedSuperClass`:  标注在实体类的父类之上，可以将属相封装在一个基类中，实体类调用可以复用相同的属性映射。

`@Basic(fetch = LAZY)`:设置属性懒加载，一般于`@Lob`配合使用。

`@Embeddable`和`@Embedded`:  `@Embeddable`标注在java类上，表示该类时一个可以被实体类嵌入的类，必须实现Serializable接口。`@Embedded`注解在属性上，表示属性类是一个嵌入类。嵌入类中的属性可展开到实体类中，映射成实体类对应表的字段。

`@AttributeOverrides`和`@AttributeOverride`: 可以和`@Embedded`配合使用，可标注在属性上，也可以标注在实体类上，用于覆盖实体类的父类或者时嵌入类的属性，eg:

```java
@AttributeOverrides({
    @AttributeOverride(name = "test1", column = @Column(name="test", length=10)),
    @AttributeOverride(name = "test2", column = @Column(name="test2", length=20, nullable=false))
})
```

`@IdClass(TestId.class)`: 复合主键特殊类，标注于实体之上。如果实体类使用了复合主键，必须给实体类指定一个特殊类。这个注解就是用于指定这个特殊类的。eg：

```java
@Entity @IdClass(TestId.class) public class Test{
    @Id int id1;
    @Id int id2;
}

class TestId{
    int id1;
    int id2;
}
```

`@EmbeddableId`: 嵌入式主键，eg：

```java
@Entity public class Test{
    @EmbeddableId TestId id;
}

@Embeddable
class TestId{
    int id1;
    int id2;
}
```

### 关系型映射 ###

JPA可以映射复杂的一对一(`@OneToOne`)、一对多(`@OneToMany` & `@ManyToOne`)、多对多关系(`@ManyToOne`)。
关系映射注解都有一个cascade属性，用于控制级联操作，cascade有如下选项(`CascadeType`)：
- `CascadeType.ALL`: 级联所有操作
- `CascadeType.PERSIST`: 级联持久化，当父实体保存后，父实体中映射的子实体也会级联保存
- `CascadeType.MERGE`: 级联合并，当查询父实体后，会通过join语句级联查询子实体，只能级联两个存在的实体
- `CascadeType.REMOVE`: 级联删除，当父实体删除，子实体也会跟着删除
- `CascadeType.REFRESH`: 级联刷新，当父实体修改属性并保存后，父实体中映射的子实体也会级联保存
- `CascadeType.DETACH`: 完全分离，不会进行级联操作

每个关系映射注解都有一个fetch属性，用于控制父实体中映射的子实体的加载方式:
- `FetchType.EAGER`: 立即加载，查询父实体后立即查询子实体
- `FetchType.LAZY`: 懒加载，查询子实体后，不查询子实体，访问时才查询子实体

每个关系映射注解都有一个mappedBy属性，这个属性的解释比较复杂，需要详细解释。如果映射时双向的，拥有关系的主体要设置`@JoinColumn`或`@JoinTable`，关系的客体需要设置mappedBy（mappedBy和`@JoinColumn`&`@JoinTable`不能同时设置），mappedBy指向拥有关系的主体中表示客体的字段，可能是一个字段，也可能是一个集合。最后mappedBy可以这么理解，mappedBy定义所在类的关系拥有者那一方的名称是xxx。mappedBy也可以避免生成中间表。

每个关系映射注解都有一个orphanRemoval属性，如果orphanRemoveal设置为true的话，要删掉子集合数据，需要调用clear方法

`@JoinColumn`: 表示一个外键字段，一般和关系映射注解配合使用。用法和@Column类似

`@JoinTable`: 多对多关系会需要一个中间表，@JoinTable就是控制这个中间表的。@JoinTable是非必须的。

## 持久层：Repository ##

持久层通过动态代理的方式实现。我们只需要指定接口和配置即可。持久层接口需要继承`JpaRepository<E,I>`。`JpaRepository`已经实现了部分基本CRUD方法，如findById，save和deleteById。

### 通过命名方法的方式查询 ###

Spring Data可以根据方法名称，自动生成SQL语句，方法命名需要遵循一定的规则：

keyword         | sample                   | JPQL snippet                  |
----------------------------------------|----------------------|---------------------------|
find                | findByA                  |...where a=?1                 |
And                |findByAandB           |...where a=?1 and b = ?2|
Or                   |findByAorB             |...where a=?1 or b= ?2   |
StartingWith   |findByAStartingWith|...where a like ?1+'%'      |
EndingWith     |findByAEndingWith|...where a like '%' + ?1    |
not                 |findByANot             |...where a <> ?1             |
findTop3,findFirst3        |findTop3ByA            |...where a = ?1 limit 3    |
findDistinct                                        | findDistinctByA|select distinct * ...|
OrderBy|findByOrderByAAscAndBDesc| ... order by a,b desc|

我们只需要在持久层接口中声明这些方法即可。

### 通过jpql\hql查询 ###

如果需要实现复杂的查询，需要在持久层接口方法上标注`@Query`，并配置jpql，eg：

```java
@Query(value="select e from Entity e where e.a =?1 and date between ?2 and ?3")
Set<Entity> findSomething(String a,Date beginDate, Date endDate);

@Query(value="from Entity where e.a like %:a%")
List<Entity> findSomething2(@Param("a") String a);

// 通过设置nativeQuery，可以使用原生的sql语言
@Query(value = "from tb_entity as e where e.a = ?1", nativeQuery=true)
List<Entity> findSomething3(String a);
```

### 结构化查询 ###

JPA不提倡直接拼接查询语句，我们可用通过Criteria的方式，生成查询语句。持久层接口需要继承`JpaSpecificationExecutor<E>`。`JpaSpecificationExecutor<E>`中的方法全部带有`Specification`参数，使用方法如下，eg:

```java
Page<User> users= userRepository.findAll((Root<User> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) -> {
    List<Predicate> predicates = new ArrayList<>();
    if(!Strings.isBlank(username)){
        predicates.add(criteriaBuilder.like(root.get("username"), username + "%"));
    }
    return query.where(predicates.toArray(new Predicate[0])).getRestriction();
}, PageRequest.of(pageNum - 1, pageSize, Sort.by(orderBy.toArray(new String[0]))));
```

当调用`JpaSpecificationExecutor<E>`中的方法时，需要实现`Specification`接口中的`toPredicate`方法（本接口是一个函数接口，可以使用匿名函数的形式简写）。参数`Root<?>`相当于查询语句中的from部分，参数`CriteriaQuery`相当于一个查询语句，用于设置where部分和select部分，参数`CriteriaBuilder`用于构建查询语句。

### 分页和排序 ###



### Entity Manager ###



## 事务管理 ##