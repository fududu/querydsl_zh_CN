### 2.3. Querying SQL
本章节主要讲述使用 `Querydsl-SQL` 模块生成查询类型和进行查询的功能。

#### 2.3.1. Maven 整合
在你的 `Maven` 项目中添加下列依赖：

    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-sql</artifactId>
        <version>${querydsl.version}</version>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-sql-codegen</artifactId>
        <version>${querydsl.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.6.1</version>
    </dependency>

如果已经使用Maven插件生成了代码，则可以不需要 `querydsl-sql-codegen` 这个依赖包。

#### 2.3.2. 通过Maven生成代码
这个功能主要通过Maven插件的方式来使用，例如：

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
              <packageName>com.myproject.domain</packageName>
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
    
Use the goal test-export to treat the target folder as a test source folder for use with test code.

**表 2.1. 参数说明**

|名称|说明|
|---|---|
|`jdbcDriver`|JDBC 驱动支持的类名|
|`jdbcUrl`|JDBC url|
|`jdbcUser`|JDBC user|
|`jdbcPassword`|JDBC password|
|`namePrefix`|生成查询类类名的前缀，默认：`"Q"`|
|`nameSuffix`|生成查询类类名的后缀，默认：`""`|
|`beanPrefix`|生成的`Bean`类类名的前缀|
|`beanSuffix`|生成的`Bean`类类名的前缀|
|`packageName`|生成的源文件的包名称|
|`beanPackageName`|生成的`Bean`文件的包名称，默认：`packageName`|
|`beanInterfaces`|生成的`Bean`类实现的一组接口的类全名，默认：`""`|
|`beanAddToString`|设置`true`时，生成的类中将自动添加`toString()`方法的实现，默认：`false`|
|`beanAddFullConstructor`|设置`true`时，除了原始的默认构造方法外，还额外生成一个全参数完整的构造方法，默认：`false`|
|`beanPrintSupertype`|设置`true`时，将同时打印出超类，默认：`false`|
|`schemaPattern`|使用`LIKE`模式中的`schema`名称模式；它必须匹配数据库存储中的`schema`名称|
|`tableNamePattern`|使用`LIKE`模式中表名称模式；它必须匹配数据库中的表名称，多个模式使用逗号`,`分隔，默认：`null`|
|`targetFolder`|生成源文件的目标目录|
|`beansTargetFolder`|生成的`Bean`源文件的目标目录，默认与 `targetFolder` 设置的目录相同|
|`namingStrategyClass`|命名策略的实现类类名，默认：`DefaultNamingStrategy`|
|`beanSerializerClass`|`Bean`序列化的实现类类名，默认：`BeanSerializer`|
|`serializerClass`|序列化实现类类名，默认：`MetaDataSerializer`|
|`exportBeans`|设置为`true`时，同时导出`Bean`类，请参与第 `2.14.13` 部份，默认：`false`|
|`innerClassesForKeys`|设置为`true`时，会为主键生成一个内部类，默认：`false`|
|`validationAnnotations`|设置为`true`时，序列化时将加入`validation`验证标签，默认：`false`|
|`columnAnnotations`|导出列标签，默认：`false`|
|`createScalaSources`|是否导出`Scala`源文件，而不是`Java`源文件，默认：`false`|
|`schemaToPackage`|追加`schema`名称至包名称，默认：`false`|
|`lowerCase`|是否转换为小写名称，默认：`false`|
|`exportTables`|导出数据表，默认：`true`|
|`exportViews`|导出视图，默认：`true`|
|`exportPrimaryKeys`|导出主键，默认：`true`|
|`tableTypesToExport`|导出使用逗号分隔的表类型的列表（可能的值取决于JDBC驱动）。允许导出任意的类型，例如：`TABLE,MATERIALIZED VIEW`。如果此值被设置，那么 `exportTables`和`exportViews`的设置将会无效|
|`exportForeignKeys`|是否导出外键设置，默认：`true`|
|`customTypes`|用户自定义类型，默认：`none`|
|`typeMappings`|`table.column` 和 `JavaType`的映射设置，默认：`none`|
|`numericMappings`|`size/digits` 和 `JavaType` 的映射设置，默认：`none`|
|`imports`|生成的查询类中要引入的一组类：`com.bar`(引入包，不要加`.*`标识)，`com.bar.Foo`\(引入类\)|

自定义类型转换的实现可以被额外添加：

    <customTypes>
      <customType>com.querydsl.sql.types.InputStreamType</customType>
    </customTypes>

可以注册指定的 `table.column` 与 `JavaType` 的类型映射：

    <typeMappings>
      <typeMapping>
        <table>IMAGE</table>
        <column>CONTENTS</column>
        <type>java.io.InputStream</type>
      </typeMapping>
    </typeMappings>

默认的数值映射：

**表 2.2. 数值映射**

|总位数\(`Total digits`\)|小数位数\(`Decimal digits`\)|类型|
|---|---|---|
|> 18|0|`BigInteger`|
|> 9|0|`Long`|
|> 4|0|`Integer`|
|> 2|0|`Short`|
|> 0|0|`Byte`|
|> 0|> 0|`BigDecimal`|

它们还可以用以下列方式定制：

    <numericMappings>
      <numericMapping>
        <total>1</total>
        <decimal>0</decimal>
        <javaType>java.lang.Byte</javaType>
      </numericMapping>
    </numericMappings>

`Schema`、表名和列名能够使用插件来重新命名，下面给出几个例子：

`schema` 的重命名

    <renameMappings>
      <renameMapping>
        <fromSchema>PROD</fromSchema>
        <toSchema>TEST</toSchema>
      </renameMapping>
    </renameMappings>

表的重命名：

    <renameMappings>
      <renameMapping>
        <fromSchema>PROD</fromSchema>
        <fromTable>CUSTOMER</fromTable>
        <toTable>CSTMR</toTable>
      </renameMapping>
    </renameMappings>

列的重命名：

    <renameMappings>
      <renameMapping>
        <fromSchema>PROD</fromSchema>
        <fromTable>CUSTOMER</fromTable>
        <fromColumn>ID</fromColumn>
        <toColumn>IDX</toTable>
      </renameMapping>
    </renameMappings>

注：当表和列重命名时，`fromSchema` 可以被省略，
相比于APT（包管理工具）生成的代码，某此功能将不可用，例如，`QueryDelegate` 注解功能

#### 2.3.3. 使用Ant生成代码

使用 Querydsl-SQL 模块的 `com.querydsl.sql.codegen.ant.AntMetaDataExporter` 作为 ANT任务支持，提供了相同的功能，Ant 的 task 配置与 Maven 配置基本相同，作了复合型类型外。

复合类型要使用没有包装的元素（ANT: without wrapper element）

    <project name="testproject" default="codegen" basedir=".">

      <taskdef name="codegen" classname="com.querydsl.sql.codegen.ant.AntMetaDataExporter";/>

      <target name="codegen">
        <codegen
          jdbcDriver="org.h2.Driver"
          jdbcUser="sa"
          jdbcUrl="jdbc:h2:/dbs/db1"
          packageName="test"
          targetFolder="target/generated-sources/java">
          <renameMapping fromSchema="PUBLIC" toSchema="PUB"/>
        </codegen>
      </target>
    </project>

#### 2.3.4. 编码方式生成代码

下面是JAVA编码方式生成相关的查询类：

    java.sql.Connection conn = ...;
    MetaDataExporter exporter = new MetaDataExporter();
    exporter.setPackageName("com.myproject.mydomain");
    exporter.setTargetFolder(new File("target/generated-sources/java"));
    exporter.export(conn.getMetaData());

上面的代码说明，生成的查询类将会被生成至 `target/generated-sources/java` 作为源目录的 `com.myproject.mydomain` JAVA包目录中。
生成的查询类中，下划线连接的表名使用驼峰式命名作为查询类类名（`USER_PROFILE` => `QUserProfile`），下划线连接的列名使用驼峰式命名作为查询类型路径(`Path`)属性的名称（`user_id` => `userId`）。
另外，还生成了 `PrimaryKey` 和 `ForeignKey`(如果有) 作为查询类属性，可用于进行`join`查询。

#### 2.3.5. 配置

使用 `Querydsl-SQL` 的方言实现作为参数，构造一个 `com.querydsl.sql.Configuration` 作为配置信息。如果H2作为数据库，则可以这样做：

    SQLTemplates templates = new H2Templates();
    Configuration configuration = new Configuration(templates);
    
Querydsl 使用SQL方言来定制实现不同的关系数据库的SQL的序列化，目前可用的SQL方言实现有：

- CUBRIDTemplates (tested with CUBRID 8.4)
- DB2Templates (tested with DB2 10.1.2)
- DerbyTemplates (tested with Derby 10.8.2.2)
- FirebirdTemplates (tested with Firebird 2.5)
- HSQLDBTemplates (tested with HSQLDB 2.2.4)
- H2Templates (tested with H2 1.3.164)
- MySQLTemplates (tested with MySQL 5.5)
- OracleTemplates (test with Oracle 10 and 11)
- PostgreSQLTemplates (tested with PostgreSQL 9.1)
- SQLiteTemplates (tested with xerial JDBC 3.7.2)
- SQLServerTemplates (tested with SQL Server)
- SQLServer2005Templates (for SQL Server 2005)
- SQLServer2008Templates (for SQL Server 2008)
- SQLServer2012Templates (for SQL Server 2012 and later)
- TeradataTemplates (tested with Teradata 14)

为了构建 `SQLTemplates` 实例，可以使用如下面中的 `builder` 模式：

    H2Templates.builder()
      .printSchema() // to include the schema in the output
      .quote()       // to quote names
      .newLineToSingleSpace() // to replace new lines with single space in the output
      .escape(ch)    // to set the escape char
      .build();      // to get the customized SQLTemplates instance

通过 `Configuration.setUseLiterals(true)` 方法来设置直接使用常量来序列化SQL语句，而不是使用参数式绑定的预编译语句，并且覆盖 `schema`、`table`和自定义的类型，具体的说明请参见 `Configuration` 类的javadoc文档。

#### 2.3.6. 查询

下面的例子中，我们将使用工厂类 `SQLQueryFactory` 来创建 `SQLQuery` 对象，它比使用 `SQLQuery` 的构造方法来创建实例更简便：

    SQLQueryFactory queryFactory = new SQLQueryFactory(configuration, dataSource);

使用 Querydsl SQL 进行查询就是这么简单：

    QCustomer customer = new QCustomer("c");

    List<String> lastNames = queryFactory.select(customer.lastName).from(customer)
            .where(customer.firstName.eq("Bob"))
            .fetch();

假设关系数据库中表名为 `customer`，列为 `first_name` 和 `last_name`，则上述代码将转换为下列的SQL语句：

    SELECT c.last_name FROM customer c WHERE c.first_name = 'Bob'

#### 2.3.7. 一般用法

使用 `SQLQuery` 类的级联方法调用

`select`: 设置查询的映射（如果是通过 `SQLQueryFactory` 创建，则不是必须的）  
`from`: 添加查询的源（`Q`开头的查询映射对象）  
`innerJoin, join, leftJoin, rightJoin, fullJoin, on`: 添加要 `join` 的查询元素，`join` 方法的第一个参数是 `join` 的查询元素，第二个参数是 `join` 的查询元素的目标（别名）  
`where`: 添加查询过滤器，以 `and` 操作符连接的，或是以逗号分隔的可变参数  
`groupBy`: 以可变参数的方式添加 `GROUP BY` 参数  
`having`: 添加具有 `"GROUP BY"` 分组作为断言（`Predicate`）表达式的可变参数数组的 `HAVING` 过滤器  
`orderBy`: 添加结果的排序方式的排序表达式。使用 `asc()` 和 `desc()` 方法获取基于数字、字符串或其他可进行比较的表达式的 `OrderSpecifier` 实例  
`limit, offset, restrict`: 设置分页查询的结果。`limit` 为结果的最大行数，`offset` 是偏移的行数，`restrict` 则是两者的限定约束

#### 2.3.8. 联合查询

联合查询使用下面给出的语法来构建：

    QCustomer customer = QCustomer.customer;
    QCompany company = QCompany.company;
    List<Customer> cList = queryFactory.select(
            customer.firstName, customer.lastName, company.name)
            .from(customer)
            .innerJoin(customer.company, company)
            .fetch();

`LEFT JOIN`:

    queryFactory.select(customer.firstName, customer.lastName, company.name)
            .from(customer)
            .leftJoin(customer.company, company)
            .fetch();

此外，还可以使用 `on(..)` 加入 `join` 条件：

    queryFactory.select(customer.firstName, customer.lastName, company.name)
            .from(customer)
            .leftJoin(company).on(customer.company.eq(company.id))
            .fetch();

#### 2.3.9. 结果排序

加入排序的语法

    queryFactory.select(customer.firstName, customer.lastName)
            .from(customer)
            .orderBy(customer.lastName.asc(), customer.firstName.asc())
            .fetch();

相当于以下的SQL语句：

    SELECT c.first_name, c.last_name
    FROM customer c
    ORDER BY c.last_name ASC, c.first_name ASC

#### 2.3.10. 分组查询

通过下面的方式进行分组查询

    queryFactory.select(customer.lastName)
            .from(customer)
            .groupBy(customer.lastName)
            .fetch();

上面的代码等效于下面的SQL语句：

    SELECT c.last_name
    FROM customer c
    GROUP BY c.last_name

#### 2.3.11. 使用子查询

要创建一个子查询，可以使用 `SQLExpressions` 的工厂方法，可以通过 `from`、`where` 等添加查询参数

    QCustomer customer = QCustomer.customer;
    QCustomer customer2 = new QCustomer("customer2");
    queryFactory.select(customer.all())
            .from(customer)
            .where(customer.status.eq(
                SQLExpressions.select(customer2.status.max()).from(customer2)))
            .fetch();

#### 2.3.12. 查询常量

要查询常量，你需要为他们创建相应的常量实例，就像下面这样：

    queryFactory.select(Expressions.constant(1),
                    Expressions.constant("abc"));

`com.querydsl.core.types.dsl.Expressions` 类还提供了映射，运算和创建模板等静态方法。

#### 2.3.13. 扩展查询的支持

可以通过继承 `AbstractSQLQuery` 类并添加相应的标记（就像下列中的 `MySQLQuery` 的例子），用来支持指定数据库引擎的特有的语法。

    public class MySQLQuery<T> extends AbstractSQLQuery<T, MySQLQuery<T>> {
    
        public MySQLQuery(Connection conn) {
            this(conn, new MySQLTemplates(), new DefaultQueryMetadata());
        }

        public MySQLQuery(Connection conn, SQLTemplates templates) {
            this(conn, templates, new DefaultQueryMetadata());
        }

        protected MySQLQuery(Connection conn, SQLTemplates templates, QueryMetadata metadata) {
            super(conn, new Configuration(templates), metadata);
        }

        public MySQLQuery bigResult() {
            return addFlag(Position.AFTER_SELECT, "SQL_BIG_RESULT ");
        }

        public MySQLQuery bufferResult() {
            return addFlag(Position.AFTER_SELECT, "SQL_BUFFER_RESULT ");
        }
    
        // ...
    }

此标记是可以在序列化后的特定点插入的自定义SQL片段。Querydsl 所支持的插入点是由`com.querydsl.core.QueryFlag.Position` 枚举类定义的。

#### 2.3.14. 窗口函数(Window functions)

在 Querydsl 中，`SQLExpressions` 的类方法用于支持数据库窗口函数。

    queryFactory.select(SQLExpressions.rowNumber()
            .over()
            .partitionBy(employee.name)
            .orderBy(employee.id))
            .from(employee)

#### 2.3.15. 公用的表(table)表达式

在 Querydsl 中，通过两种不同的方式支持表(table)表达式

    QEmployee employee = QEmployee.employee;
    queryFactory.with(employee, 
            SQLExpressions.select(employee.all)
                    .from(employee)
                    .where(employee.name.startsWith("A")))
            .from(...)

使用列(columns)列表：

    QEmployee employee = QEmployee.employee;
    queryFactory.with(employee, employee.id, employee.name)
            .as(SQLExpressions.select(employee.id, employee.name)
                    .from(employee)
                    .where(employee.name.startsWith("A")))
            .from(...)

如果公用表(table)表达式的列（columns）是一个已存在的表的子集或视图，建议使用已生成的 `Path` 类型，例如本例中的 `QEmployee`，但是，如果列不存在于任何的已有的表中，则使用 `PathBuilder` 来构建。  
下列是对应这种情况下的一个例子：

    QEmployee employee = QEmployee.employee;
    QDepartment department = QDepartment.department;
    PathBuilder<Tuple> emp = new PathBuilder<Tuple>(Tuple.class, "emp");
    queryFactory.with(emp, SQLExpressions.select(
            employee.id, employee.name, employee.departmentId, 
            department.name.as("departmentName"))
                    .from(employee)
                    .innerJoin(department).on(employee.departmentId.eq(department.id))))
            .from(...)

#### 2.3.16. 其他表达式

其他的表达式，同样可以调用 `SQLExpression` 类中的相关的静态方法中获取或构建。

#### 2.3.17. 数据操作命令 (DML)

##### 2.3.17.1. 插入(Insert)

带列(Column)方式：

    QSurvey survey = QSurvey.survey;

    queryFactory.insert(survey)
            .columns(survey.id, survey.name)
            .values(3, "Hello").execute();

不带列(Column)方式：

    queryFactory.insert(survey).values(4, "Hello").execute();

有子查询：

    queryFactory.insert(survey)
            .columns(survey.id, survey.name)
            .select(SQLExpressions.select(survey2.id.add(1), survey2.name).from(survey2))
            .execute();

有子查询，不带列：

    queryFactory.insert(survey)
            .select(SQLExpressions.select(survey2.id.add(10), survey2.name).from(survey2))
            .execute();

除了 columns/values 方式外，Querydsl 还提供了 `set` 方法用于代替这一方式：

    QSurvey survey = QSurvey.survey;

    queryFactory.insert(survey)
            .set(survey.id, 3)
            .set(survey.name, "Hello").execute();

这和第一个例子（带列方式插入）是相同的。内部实现中，Querydsl 自动映射列和指定的值。  
要注意的是

    columns(...).select(...)

将查询的结果映射至指定的列  
如果要获取数据库自动生成的主键值，则不是获取受影响的行数，则使用 `executeWithKey` 方法。

    set(...)

可用于映射单列，子查询返回空时，全部映射为 `null` 值。

使用一个bean实例，填充一个语句实例（SQLXxxxClause）

    queryFactory.insert(survey)
            .populate(surveyBean).execute();

上述代码将忽略 `surveyBean` 中的值为 `null` 的属性，如果需要绑定 `null` 值属性，则需要：

    queryFactory.insert(survey)
            .populate(surveyBean, DefaultMapper.WITH_NULL_BINDINGS).execute();

##### 2.3.17.2. 更新(Update)

有 `where`:

    QSurvey survey = QSurvey.survey;

    queryFactory.update(survey)
            .where(survey.name.eq("XXX"))
            .set(survey.name, "S")
            .execute();

没有 `where`:

    queryFactory.update(survey)
            .set(survey.name, "S")
            .execute();

使用bean填充：

    queryFactory.update(survey)
            .populate(surveyBean)
            .execute();

##### 2.3.17.3. 删除(Delete)

有 `where`:

    QSurvey survey = QSurvey.survey;

    queryFactory.delete(survey)
            .where(survey.name.eq("XXX"))
            .execute();

没有 `where`:

    queryFactory.delete(survey).execute()

#### 2.3.18. DML 语句批量操作

Querydsl SQL 通用调用 DML API来使用JDBC批处理更新操作。如果有一系列相似结构的操作，可以调用 `addBatch()` 方法将多个操作绑定到同一个 `DMLClause` 中来进行统一调用。下面有几个例子，说明了 `UPDATE`, `INSERT` 和 `DELETE` 的批量操作。

**Update:**

    QSurvey survey = QSurvey.survey;

    queryFactory.insert(survey).values(2, "A").execute();
    queryFactory.insert(survey).values(3, "B").execute();

    SQLUpdateClause update = queryFactory.update(survey);
    update.set(survey.name, "AA").where(survey.name.eq("A")).addBatch();
    update.set(survey.name, "BB").where(survey.name.eq("B")).addBatch();

**Delete:**

    queryFactory.insert(survey).values(2, "A").execute();
    queryFactory.insert(survey).values(3, "B").execute();

    SQLDeleteClause delete = queryFactory.delete(survey);
    delete.where(survey.name.eq("A")).addBatch();
    delete.where(survey.name.eq("B")).addBatch();
    assertEquals(2, delete.execute());

**Insert:**

    SQLInsertClause insert = queryFactory.insert(survey);
    insert.set(survey.id, 5).set(survey.name, "5").addBatch();
    insert.set(survey.id, 6).set(survey.name, "6").addBatch();
    assertEquals(2, insert.execute());

#### 2.3.19. 生成Bean类

使用 `MetaDataExporter` 类来生成数据库表对应的 JavaBean 类作为 `DTO`

    java.sql.Connection conn = ...;
    MetaDataExporter exporter = new MetaDataExporter();
    exporter.setPackageName("com.myproject.mydomain");
    exporter.setTargetFolder(new File("src/main/java"));
    exporter.setBeanSerializer(new BeanSerializer());
    exporter.export(conn.getMetaData());

现在，可以使用 `DMLClause` 中使用Bean和查询对象Bean作为参数，直接进行查询了：

    QEmployee e = new QEmployee("e");

    // Insert
    Employee employee = new Employee();
    employee.setFirstname("John");
    Integer id = queryFactory.insert(e).populate(employee).executeWithKey(e.id);
    employee.setId(id);

    // Update
    employee.setLastname("Smith");
    assertEquals(1l,
            queryFactory.update(e).populate(employee)
            .where(e.id.eq(employee.getId())).execute());

    // Query
    Employee smith = queryFactory.selectFrom(e).where(e.lastname.eq("Smith")).fetchOne();
    assertEquals("John", smith.getFirstname());

    // Delete
    assertEquals(1l, queryFactory.delete(e).where(e.id.eq(employee.getId())).execute());

#### 2.3.20. 提取SQL语句和查询绑定

    SQLBindings bindings = query.getSQL();
    System.out.println(bindings.getSQL());

如果想在打印的SQL语句中看到绑定的常量参数，可以调用配置类 `Configuration` 的 `setUseLiterals(true)`方法。

#### 2.3.21. 自定义类型

Querydsl SQL 提供了自定义类型与 `ResultSet/Statement` 映射的可能性。自定义类型可以在配置中进行注册，在构建查询（SQLQuery）时，`Configuration` 将作为参数被自动添加至查询中。

    Configuration configuration = new Configuration(new H2Templates());
    // overrides the mapping for Types.DATE
    configuration.register(new UtilDateType());

具体映射至数据库表中的列：

    Configuration configuration = new Configuration(new H2Templates());
    // declares a mapping for the gender column in the person table
    configuration.register("person", "gender",  new EnumByNameType<Gender>(Gender.class));

如果要自定义数值映射，可以调用 `registerNumeric` 方法：

    configuration.registerNumeric(5,2,Float.class);

上述调用，将 `NUMERIC(5,2)` 映射为 `java.lang.Float` 类型。

#### 2.3.22. 查询和更新监听器

`SQLListener` 是一个用于查询和执行 DML语句的监听器接口。可以通过 `addListener` 方法将 `SQLListener` 实例注册到 `Configuration` 配置中，或者注册到 query/clause 级别的查询或操作语句中。

监听器实现可用于数据同步，缓存，日志记录和数据校验等。

#### 2.3.23. Spring 框架整合

Querydsl SQL 可以通过 `querydsl-sql-spring` 模块来整合 spring 框架。

    <dependency>
      <groupId>com.querydsl</groupId>
      <artifactId>querydsl-sql-spring</artifactId>
      <version>${querydsl.version}</version>
    </dependency>

它提供了一个 Spring 异常转换器，和一个 Spring 数据库连接提供者实现，用于支持 Spring 框架中的事务管理。

    package com.querydsl.example.config;

    import com.querydsl.sql.H2Templates;
    import com.querydsl.sql.SQLQueryFactory;
    import com.querydsl.sql.SQLTemplates;
    import com.querydsl.sql.spring.SpringConnectionProvider;
    import com.querydsl.sql.spring.SpringExceptionTranslator;
    import com.querydsl.sql.types.DateTimeType;
    import com.querydsl.sql.types.LocalDateType;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.PropertySource;
    import org.springframework.core.env.Environment;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    import org.springframework.transaction.PlatformTransactionManager;

    import javax.inject.Inject;
    import javax.inject.Provider;
    import javax.sql.DataSource;
    import java.sql.Connection;
    
    @Configuration
    public class JdbcConfiguration {

        @Bean
        public DataSource dataSource() {
            // implementation omitted
        }
     
        @Bean
        public PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(dataSource());
        }

        @Bean
        public com.querydsl.sql.Configuration querydslConfiguration() {
            //change to your Templates
            SQLTemplates templates = H2Templates.builder().build();
            com.querydsl.sql.Configuration configuration = 
                    new com.querydsl.sql.Configuration(templates);
            configuration.setExceptionTranslator(new SpringExceptionTranslator());
            return configuration;
        }

        @Bean
        public SQLQueryFactory queryFactory() {
            Provider<Connection> provider = new SpringConnectionProvider(dataSource());
            return new SQLQueryFactory(querydslConfiguration(), provider);
        }
    }

译者注：翻译文档时的版本 v4.0.7 中，`querydsl-sql-spring` 中的 `SpringConnectionProvider` 中返回的 `Connection` 必须在事务开启且进行事务时才能获取，否则将抛出异常。