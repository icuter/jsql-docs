# Jdbc Object Pool

## Connection Pool

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
PooledConnection pool = dataSource.getPooledConnection();
try (Connection connection = pool.getConnection()) {
    // This is a PooledConnection
    // TODO do something with connection
}
```

Get Connection directly from pool

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
try (Connection connection = dataSource.getConnection()) {
    // This is a PooledConnection
    // TODO do something with connection
}
```

## JdbcExecutor Pool

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
JdbcExecutorPool pool = dataSource.getExecutorPool();
JdbcExecutor executor = pool.getExecutor();
try {
    // TODO do something with executor
} finally {
    executor.close();
}
```

Get jdbc executor directly from pool
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();

try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
}
```

## Pool Configuration
Now, let's customize out connection object pool by `PoolConfiguration`.

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();

PoolConfiguration poolConfiguration = PoolConfiguration.defaultPoolCfg();
poolConfiguration.setMaxPoolSize(32);
ConnectionPool pool = dataSource.createConnectionPool(poolConfiguration);
```

For better and shorter is to get Connection Pool from inner `JSQLDataSource`

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass")
                            .poolMaxSize(32)
                            .build();
PooledConnection pool = dataSource.getPooledConnection();
```

Pool Configuration settings overall

 name | comment
---|---
maxPoolSize | The max object pool size <br> *default: 20*
idleTimeout | The max idle milliseconds time of object after returned, and will check it' idle timeout when borrow object from pool.<br>&nbsp;0: timeout immediately when returning or checked by pool maintainer<br>-1: never timeout<br> *default: 30 minutes*
~~idleCheckInterval~~ | The interval time of milliseconds to trigger pool maintainer checking for removing timeout idle object. Usually, when borrowing object also check it's idle timeout and try to remove it from pool, so idle object checking don't depend on idle object checker. But we suggest you don't turn it off in case over the max database connections <br> **less or equals 0**: never interval check <br> *default: 10 minutes* <br> **NOTE:** If you need to check idle object with interval time, especially, fire wall auto close the long time connection and without sending reset signal.<br> **Removed after v1.0.4** 
scheduledThreadLifeTime | Set the schedule thread's life time in milliseconds, but set it to negative or 0 means never timeout<br> *default: 5 minutes* <br>since: v1.0.4
pollTimeout | Time of milliseconds to wait for borrowing a object.<br>&nbsp;0: no wait<br>< 0: hold waiting until success, e.g. -1<br> *default: 5 seconds*
validateOnBorrow | Validate on borrowing an object from pool <br> *default: true* 
validateOnReturn | Validate on returning an object to pool <br> *default: false* 
createRetryCount | Retry to create pool object if exception occur <br> *default: 0*


`PoolConfiguration.defaultPoolCfg()` could get the default Pool Configuration.

Object Pool should be static final, borrow and return object globally
