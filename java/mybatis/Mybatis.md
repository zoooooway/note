# Mybatis

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202306011537127.png)
> https://www.cnblogs.com/cxuanBlog/p/11172792.html

## Configuration
`Configuration` 类中存放了 `Mybatis` 的所有配置信息, 如果使用 `XML` 文件来配置 `Mybatis`, `Mybatis` 的配置文件的 <configuration> 标签的内容解析后就全在此类中.

## SqlSession
`SqlSession` 是 `Mybatis` 提供的 API 接口, 通过 `SqlSession` 可以执行 `SQL`, 获取映射程序(`Mapper`), 管理事务等操作.

## SqlSessionFactory
通过 `SqlSessionFactory` 可以获取 `SqlSession` 实例.
一般来说程序中会实例化一个单例 `SqlSessionFactory` 并一直持有到应用程序销毁为止, 应用程序需要执行 `SQL` 时通过此单例对象来获取一个 `SqlSession`. 
`SqlSessionFactory` 本身是由 `SqlSessionFactoryBuilder` 创建的, 支持使用 `XML`, 注解, 或者 `Java` 代码配置来创建. 如果使用 `Spring` 这类依赖注入框架, 那只需要一些配置即可自动创建， 并由框架来管理 `SqlSessionFactory` 实例.

## Executor
`SqlSession` 执行 `SQL` 的操作其实是交由其内部的 `Executor` 来处理. 下面是 `Configuration` 类中创建不同 `Executor` 的代码:
```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

可以看到创建 `Executor` 需要一个 `Transaction` 实例，`Transaction` 定义如下:
```java
public interface Transaction {
  Connection getConnection() throws SQLException;

  void commit() throws SQLException;

  void rollback() throws SQLException;

  void close() throws SQLException;

  Integer getTimeout() throws SQLException;
}
```

也就是说 `Executor` 内部持有 `Transaction` 实例， 并且通过该实例可以进行事务的管理. 

## Transaction
其实 `Transaction` 相当于对 `Connection` 的包装, 对事务的管理仍然是通过调用 `Connection` 的方法. 当需要获取 `Connection` 时, 比如 `sqlSeesion.getConnection()`, 实际会通过 `transaction.getConnection()` 方法去获取 `Connection`. `Transaction` 获取 `Connection` 是通过 `DataSource` 对象, 其内部持有一个 `DataSource` 实例, 并通过 `dataSource.getConnection()` 来获取连接. 通常, 应用程序中都会使用连接池, 这个 `DataSource` 对象就是各个连接池的数据源对象. 

## Handler
`Mybatis` 中有四种 `handler`:
* `StatementHandler`: 封装了 `JDBC` 的 `Statement` 操作: `query`, `update` ，`Statement` 是负责执行静态 `SQL` 并返回结果的对象. `StatementHandler` 主要负责创建 `Statement`， 设置参数等操作. `StatementHandler` 有四种实现类:
  * `SimpleStatementHandler`:  管理**不需要预编译** `SQL` 语句的 `Statement`.
  * `PreparedStatementHandler`: 管理**需要预编译** `SQL` 语句的 `Statement`.
  * `CallableStatementHandler`: 管理**需要调用存储过程** 的 `SQL` 语句的 `Statement`.
  * `RoutingStatementHandler`: `RoutingStatementHandler` 并不直接管理 `Statement`, 它内部持有上述三种实现类中的任意一种实例, 并通过该实例来操作 `Statement`. 
* `ParameterHandler`: 负责参数处理，为需要预编译的 `SQL` 动态赋值.
* `ResultSetHandler`: 处理 `Statement` 执行后产生的结果集，生成结果列表.
* `TypeHandler`: 类型转换处理器，在 `ResultSetHandler` 生成结果列表前进行类型处理.


