# 引言

Querydsl 是一个能使用静态类型构建类似于SQL的查询的框架。Querydsl 通过流畅的API调用来构建原本需要用字符串连接或外部XML文件配置的查询。

相比于简单的字符串，使用级联API方法调用的好处在于

- IDE的代码提示
- 几乎没有语法来允许无效的查询
- 领域类和属性可以线程安全的引用
- 变更领域类时可以更好的重构