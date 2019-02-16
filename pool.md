# Jdbc Object Pool

## JSQLDataSource
Now, let’s try to create a JSQLDataSource and new a Connection from JSQLDataSource.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Connection connection = dataSource.newConnection();
// TODO do something with connection
```

### Connection Pool
First, we'd create a JSQLDataSource, and then invoke `JSQLDataSource.createConnectionPool` to create a Connection Pool.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
ConnectionPool pool = dataSource.createConnectionPool();
Connection connection = null;
try {
    connection = pool.getConnection();
    // TODO do something with connection
} finally {
    pool.returnConnection(connection);
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
ObjectPool<Connection> pool = dataSource.createConnectionPool(poolConfiguration);
```

 name | comment | default value
---|---|---
maxPoolSize | Max object pool size | 20
idleTimeout | The max valid milliseconds between object returned till now, 0 will be timeout immediately when returning or checked by pool maintainer, and -1 means never timeout | 1 hour
idleCheckInterval | The interval time of milliseconds to trigger pool maintainer checking idle object | 30 minutes
pollTimeout | Setting time waiting for borrowing a object with milliseconds, -1 will never timeout | 5 seconds

`PoolConfiguration.defaultPoolCfg()` could get the default Pool Configuration.

Object Pool should be static final, borrow and return object globally
