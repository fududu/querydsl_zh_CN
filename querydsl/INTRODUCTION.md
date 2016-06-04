# 1. 介绍

### 1.1. 背景

Querydsl 能够诞生，是因为需要在类型安全的方式下进行HQL查询。构造的HQL查询需要拼接字符串，并且会导致代码难以阅读。通过纯字符串对领域类型和属性的不安全引用是基于字符串构建HQL的另一个问题。

随着类型安全的领域模型的不断的发展，给软件开发带了巨大的好处。领域最大的改变直接反应在查询和自动查询的构建，它使查询构建更快且安全。

Querydsl最先支持的是Hibernate的HQL语言，现如今，Querydsl已经支持JPA，JDO，JDBC，Lucene，Hibernate Search，MangoDB，Collections 和RDF(Relational Data Format) Bean作为后端。

### 1.2. 原则

类型安全是Querydsl的核心原则。查询是基于与领域类型的属性映射生成的查询类型构建的。同时，函数/方法的调用也是使用完全的类型安全的方式构建的。

保持一致是另一个重要的原则。查询路径(Path)和操作在所有实现中都是相同的，它们都具有一个公用的基本接口。

要想获得更多Querydsl查询和表达式的说明，请查看javadoc中的 `com.querydsl.core.Query`，`com.querydsl.core.Fetchable` 和 `com.querydsl.core.types.Expression` 类的文档。