## 2.1. Querying JPA

Querydsl 定义了一个通用静态类型的语法用于查询持久化的领域模型数据。JDO 和 JPA 是 Querydsl 主要的集成技术。这篇手册介绍了如何让Querydsl与JPA整合使用。

Querydsl JPA 是JPQL和标准条件查询（Criteria queries）的新的使用方式。它结合了条件查询的动态性和JPQL的表达能力，并且使用了完全的类型安全方式。

### 2.1.1. Maven 集成

在你的maven项目中添加下面的依赖：

    <dependency>
      <groupId>com.querydsl</groupId>
      <artifactId>querydsl-apt</artifactId>
      <version>${querydsl.version}</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>com.querydsl</groupId>
      <artifactId>querydsl-jpa</artifactId>
      <version>${querydsl.version}</version>
    </dependency>

    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.6.1</version>
    </dependency>

现在，来配置 maven APT 插件：

    <project>
      <build>
      <plugins>
        ...
        <plugin>
          <groupId>com.mysema.maven</groupId>
          <artifactId>apt-maven-plugin</artifactId>
          <version>1.1.3</version>
          <executions>
            <execution>
              <goals>
                <goal>process</goal>
              </goals>
              <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
              </configuration>
            </execution>
          </executions>
        </plugin>
        ...
      </plugins>
      </build>
    </project>

`JPAAnnotationProcessor` 查找带有 `javax.persistence.Entity` 注解的领域类型并为其生成查询类型。
如果你在领域类型中使用的是Hibernate的注解，那应该使用 APT 处理器 `com.querydsl.apt.hibernate.HibernateAnnotationProcessor` 来代替 `JPAAnnotationProcessor`。

运行 `clean install` 命令后，在 `target/generated-sources/java` 目录下将生成相应的查询类型。  
如果你使用Eclipse，运行 `mvn eclipse:eclipse` 更新你的Eclipse项目，将 `target/generated-sources/java` 作为源目录。  
现在，你就可以构建 JPA 查询实例和查询领域模型的实例了。

### 2.1.2. Ant 集成

将 Querydsl 的所有依赖包(jar)放到你的 classpath 目录下，使用并执行下面的Querydsl代码生成的任务：

    <!-- APT based code generation -->
    <javac srcdir="${src}" classpathref="cp">
      <compilerarg value="-proc:only"/>
      <compilerarg value="-processor"/>
      <compilerarg value="com.querydsl.apt.jpa.JPAAnnotationProcessor"/>
      <compilerarg value="-s"/>
      <compilerarg value="${generated}"/>
    </javac>

    <!-- compilation -->
    <javac classpathref="cp" destdir="${build}">
      <src path="${src}"/>
      <src path="${generated}"/>
    </javac>

将 `src` 替换为你的主要的源代码目录，`generated` 替换为你需要生成的源代码的目标目录。

### 2.1.3. 在 Roo 中使用 Querydsl JPA

如果你正在使用 Querydsl JPA 和 Spring Roo，你可以使用 `com.querydsl.apt.roo.RooAnnotationProcessor`  替换 `com.querydsl.apt.jpa.JPAAnnotationProcessor`，它将解析并处理 `@RooJpaEntity` 和 `@RooJpaActiveRecord` 注解，而不是 `@Entity`。

基于APT的代码生成器 **不支持** 与 AspectJ IDTs 整合。

### 2.1.4. 从 hbm.xml 文件中生成模型

如果你使用的Hibernate是基于XML配置的，那么你可以使用XML元数据来生成Querydsl模型。
`com.querydsl.jpa.codegen.HibernateDomainExporter` 提供了这样的功能：

    HibernateDomainExporter exporter = new HibernateDomainExporter(
            "Q",                    // 名称前缀
          new File("target/gen3"),  // 生成的目标文件夹
          configuration);           // org.hibernate.cfg.Configuration 实例

    exporter.export();

`HibernateDomainExporter` 需要在领域类型可见的 `classpath` 中执行，因为属性类型是通过反射机制获得的。

所有的 JPA 注解都会被忽略，但是 Querydsl 的注解不会被忽略，比如 `@QueryInit` 和 `@QueryType`。

### 2.1.5. 使用查询类型

To create queries with Querydsl you need to instantiate variables and Query implementations. We will start with the variables.

Let's assume that your project has the following domain type:

    @Entity
    public class Customer {
        private String firstName;
        private String lastName;

        public String getFirstName() {
            return firstName;
        }

        public String getLastName() {
            return lastName;
        }

        public void setFirstName(String fn) {
            firstName = fn;
        }

        public void setLastName(String ln)[
            lastName = ln;
        }
    }
    
Querydsl 会在 `Customer` 类的同一个包内生成一个名称为 `QCustomer` 的查询类。`QCustomer` 的静态实例变量可在Querydsl查询中作为 `Customer` 类的代表。  
`QCustomer` 有一个默认的静态实例变量：

    QCustomer customer = QCustomer.customer;

或者你可以像下面列子一样定义查询类实例变量：

    QCustomer customer = new QCustomer("myCustomer");

### 2.1.6. 查询

Querydsl JPA 模块同时支持 JPA 和 Hibernate API。

当使用 JPA 时，需要用到 `JPAQuery` 实例，就像下面的例子：

    // where entityManager is a JPA EntityManager
    JPAQuery<?> query = new JPAQuery<Void>(entityManager);

如是你使用的是 Hibernate API，你可以实例化一个 `HibernateQuery`：

    // where session is a Hibernate session
    HibernateQuery<?> query = new HibernateQuery<Void>(session);

`JPAQuery` 和 `HibernateQuery` 都实现了 `JPQLQuery` 这个接口。

本章中的例子中的查询是通过 `JPAQueryFactory` 实例创建的。最佳实践是通过 `JPAQueryFactory` 来获取 `JPAQuery` 实例。

如需 Hibernate API，可以使用 `HibernateQueryFactory`。

构造一个获取名字为 "Bob" 的一个 customer 的信息的查询：

    // queryFactory => JPAQueryFactory
    QCustomer customer = QCustomer.customer;
    Customer bob = queryFactory.selectFrom(customer)
      .where(customer.firstName.eq("Bob"))
      .fetchOne();

调用 `selectFrom` 方法是定义查询的来源与映射，`where` 则定义查询的过滤器，`fetchOne` 则要告诉 Querydsl 返回单个元素（`SQLQuery<?>` 中的泛型指定元素类型）。很简单，不是吗！

创建多资源（表）查询：

    QCustomer customer = QCustomer.customer;
    QCompany company = QCompany.company;
    query.from(customer, company);

使用多个查询条件过滤器：

    queryFactory.selectFrom(customer)
        .where(customer.firstName.eq("Bob"), customer.lastName.eq("Wilson"));

或者：

    queryFactory.selectFrom(customer)
        .where(customer.firstName.eq("Bob").and(customer.lastName.eq("Wilson")));

使用原生的 JPQL 查询语句：

    select customer from Customer as customer
    where customer.firstName = "Bob" and customer.lastName = "Wilson"

如果你想使用 `"or"` 来组合条件过滤器，可以使用下面的方式：

    queryFactory.selectFrom(customer)
        .where(customer.firstName.eq("Bob").or(customer.lastName.eq("Wilson")));

### 2.1.7. 联合查询

Querydsl 在JPQL中支持的联合查询：`inner join, join, left join, right join`。联合查询是类型安全的，并且遵循下面的模式：

    QCat cat = QCat.cat;
    QCat mate = new QCat("mate");
    QCat kitten = new QCat("kitten");
    queryFactory.selectFrom(cat)
        .innerJoin(cat.mate, mate)
        .leftJoin(cat.kittens, kitten)
        .fetch();

下面是原生的JPQL查询的语句：

    select cat from Cat as cat
    inner join cat.mate as mate
    left outer join cat.kittens as kitten

再来一个例子：

    queryFactory.selectFrom(cat)
        .leftJoin(cat.kittens, kitten)
        .on(kitten.bodyWeight.gt(10.0))
        .fetch();

上述最终生成的原生的JPQL的语句：

    select cat from Cat as cat
    left join cat.kittens as kitten
    on kitten.bodyWeight > 10.0

### 2.1.8. 一般用法

`JPQLQuery` 接口可以进行级联调用的方法说明：

`select`: 设置查询的映射（如果由查询工厂创建查询，则不需要调用）  
`from`: 添加查询的源（表）  
`innerJoin, join, leftJoin, rightJoin, on`: 使用这些方法添加要 `join` 的元素。连接方法的第一个参数是要连接的源，第二个参数是源目标（别名）。  
`where`: 添加查询过滤器，以逗号分隔的可变参数传入，或是使用 `and` 或 `or` 操作连接。  
`groupBy`: 以可变参数形式添加查询的分组。  
`having`: 添加具有 `"GROUP BY"` 分组作为断言（`Predicate`）表达式的可变参数数组的 `HAVING` 过滤器  
`orderBy`: 添加结果的排序方式的排序表达式。使用 `asc()` 和 `desc()` 方法获取基于数字、字符串或其他可进行比较的表达式的 `OrderSpecifier` 实例  
`limit, offset, restrict`: 设置分页查询的结果。`limit` 为结果的最大行数，`offset` 是偏移的行数，`restrict` 则是两者的限定约束

### 2.1.9. 结果排序(Ordering)

声明结果排序：

    QCustomer customer = QCustomer.customer;
    queryFactory.selectFrom(customer)
            .orderBy(customer.lastName.asc(), customer.firstName.desc())
            .fetch();

等效于下面的原生JPQL语句：

    select customer from Customer as customer
    order by customer.lastName asc, customer.firstName desc

### 2.1.10. 分组(Grouping)

下面的是对结果分组：

    queryFactory.select(customer.lastName).from(customer)
            .groupBy(customer.lastName)
            .fetch();

等效于下面的原生JPQL语句：

    select customer.lastName
    from Customer as customer
    group by customer.lastName

### 2.1.11. 删除语句

在Querydsl JPA中，删除语句是简单的 `delete-where-execute` 形式。下面是一个例子：

    QCustomer customer = QCustomer.customer;
    // delete all customers
    queryFactory.delete(customer).execute();
    // delete all customers with a level less than 3
    queryFactory.delete(customer).where(customer.level.lt(3)).execute();

调用 `where` 方法是可选的，调用 `execute` 方法是执行删除操作并返回被删除实体的数量。

JPA中的DML语句并不考虑JPA级联规则，也不提供细粒度二级缓存的交互。

### 2.1.12. 更新语句

在Querydsl JPA中，更新语句是简单的 `update-set/where-execute` 形式。下面是一个例子：

    QCustomer customer = QCustomer.customer;
    // rename customers named Bob to Bobby
    queryFactory.update(customer).where(customer.name.eq("Bob"))
            .set(customer.name, "Bobby")
            .execute();

调用 `set` 方法以 `SQL-Update-style` 方式定义要更新的属性，`execute` 调用指行更新操作并返回被更新的实体的数量。

JPA中的DML语句并不考虑JPA级联规则，也不提供细粒度二级缓存的交互。

### 2.1.13. 子查询

使用 `JPAExpressions` 的静态工厂方法创建一个子查询，并且通过调用 `from`，`where` 等定义查询参数。

    QDepartment department = QDepartment.department;
    QDepartment d = new QDepartment("d");
    queryFactory.selectFrom(department)
        .where(department.size.eq(
            JPAExpressions.select(d.size.max()).from(d)))
         .fetch();

再来一个例子：

    QEmployee employee = QEmployee.employee;
    QEmployee e = new QEmployee("e");
    queryFactory.selectFrom(employee)
            .where(employee.weeklyhours.gt(
                JPAExpressions.select(e.weeklyhours.avg())
                    .from(employee.department.employees, e)
                    .where(e.manager.eq(employee.manager))))
            .fetch();

### 2.1.14. 使用原始查询

如果你在查询执行前需要调整原有的查询，则可以像下面这样暴露她：

    Query jpaQuery = queryFactory.selectFrom(employee).createQuery();
    // ...
    List results = jpaQuery.getResultList();

### 2.1.15. JPA 查询中使用本地SQL查询

Querydsl 支持通过 `JPASQLQuery` 类在JPA中执行本地SQL查询。

要使用它，你必须为SQL schema 生成 Querydsl 查询类。通过下列所述的 maven 配置来完成：

    <project>
      <build>
        <plugins>
          ...
          <plugin>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-maven-plugin</artifactId>
            <version>${querydsl.version}</version>
            <executions>
              <execution>
                <goals>
                  <goal>export</goal>
                </goals>
              </execution>
            </executions>
            <configuration>
              <jdbcDriver>org.apache.derby.jdbc.EmbeddedDriver</jdbcDriver>
              <jdbcUrl>jdbc:derby:target/demoDB;create=true</jdbcUrl>
              <packageName>com.mycompany.mydomain</packageName>
              <targetFolder>${project.basedir}/target/generated-sources/java</targetFolder>
            </configuration>
            <dependencies>
              <dependency>
                <groupId>org.apache.derby</groupId>
                <artifactId>derby</artifactId>
                <version>${derby.version}</version>
              </dependency>
            </dependencies>
          </plugin>
          ...
        </plugins>
      </build>
    </project>

当查询类已经成功生成在你所选的位置，就可以在查询中使用他们了。

单列查询：

    // serialization templates
    SQLTemplates templates = new DerbyTemplates();
    // 查询类 (S* -> SQL, Q* -> 领域类)
    SAnimal cat = new SAnimal("cat");
    SAnimal mate = new SAnimal("mate");
    QCat catEntity = QCat.cat;

    JPASQLQuery<?> query = new JPASQLQuery<Void>(entityManager, templates);
    List<String> names = query.select(cat.name).from(cat).fetch();

如果在你的查询中混合引用了实体（如：QCat）和表（如：SAnimal），你要确保他们使用变量名称一致。`SAnimal.animal` 的变量名称是 `"animal"`，因此使用了一个新的实例（`new SAnimal("cat")`）

另一种模式可以是：

    QCat catEntity = QCat.cat;
    SAnimal cat = new SAnimal(catEntity.getMetadata().getName());

查询多列：

    query = new JPASQLQuery<Void>(entityManager, templates);
    List<Tuple> rows = query.select(cat.id, cat.name).from(cat).fetch();

查询所有列:

    List<Tuple> rows = query.select(cat.all()).from(cat).fetch();
 
SQL中查询，但是映射为实体：

    query = new JPASQLQuery<Void>(entityManager, templates);
    List<Cat> cats = query.select(catEntity).from(cat).orderBy(cat.name.asc()).fetch();

联合查询:

    query = new JPASQLQuery<Void>(entityManager, templates);
    cats = query.select(catEntity).from(cat)
        .innerJoin(mate).on(cat.mateId.eq(mate.id))
        .where(cat.dtype.eq("Cat"), mate.dtype.eq("Cat"))
        .fetch();

查询并映射为DTO（Data Transform Object）:

    query = new JPASQLQuery<Void>(entityManager, templates);
    List<CatDTO> catDTOs = query.select(Projections.constructor(CatDTO.class, cat.id, cat.name))
        .from(cat)
        .orderBy(cat.name.asc())
        .fetch();

如果你正在使用Hibernate API，而不是JPA的API，替换为 `HibernateSQLQuery` 即可。
