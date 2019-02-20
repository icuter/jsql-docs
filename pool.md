# Jdbc Object Pool

## JSQLDataSource
At first, we'd better new a `JSQLDataSource` instance that is the JSQL programing entrance,
Connection pool even JdbcExecutor pool can be created by `JSQLDataSource`, please refer to
the following section.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Connection connection = dataSource.newConnection();
// TODO do something with connection
```

### Connection Pool
Create `ConnectionPool` by `JSQLDataSource`.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
ConnectionPool pool = dataSource.createConnectionPool();
Connection connection = null;
try {
    connection = pool.getConnection(); // This is a PooledConnection
    // TODO do something with connection
} finally {
    connection.close(); // same to pool.returnConnection(connection);
}
```

### JdbcExecutor Pool
Like Connection Pool creation, but provide a executor object which combine JdbcExecutor and Connection to work together.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutorPool pool = dataSource.createExecutorPool();
JdbcExecutor executor = pool.getExecutor();
try {
    // TODO do something with executor
} finally {
    executor.close();
}
```

Let's refactor our quick start example by using `JdbcExecutorPool`
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutorPool pool = dataSource.createExecutorPool();
JdbcExecutor executor = pool.getExecutor();
try {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
} finally {
    executor.close();
}
```

### Pool Configuration
If we want to configure pool, do it by `PoolConfiguration`.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");

PoolConfiguration poolConfiguration = PoolConfiguration.defaultPoolCfg();
poolConfiguration.setMaxPoolSize(32);
ConnectionPool pool = dataSource.createConnectionPool(poolConfiguration);
```

 name | comment
---|---|---
maxPoolSize | The max object pool size <br> *default: 20*
idleTimeout | The max idle time of object after returned, and will check it' idle timeout when borrow object from pool.<br>&nbsp;0: timeout immediately when returning or checked by pool maintainer<br>-1: never timeout<br> *default: 30 minutes*
idleCheckInterval | The interval time of milliseconds to trigger pool maintainer checking for removing timeout idle object. Usually, when borrowing object also check it's idle timeout and try to remove it from pool, so idle object checking don't depend on idle object checker.<br> *default: 0* <br> **NOTE:** If you need to check idle object with interval time, especially, fire wall auto close the long time connection and without sending reset signal.
pollTimeout | Time of milliseconds to wait for borrowing a object.<br>&nbsp;0: no wait<br>-1: hold waiting until success<br> *default: 5 seconds*

`PoolConfiguration.defaultPoolCfg()` could get the default Pool Configuration.

Object Pool should be static final, borrow and return object globally
