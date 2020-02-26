# JSQLDataSource

JSQLDataSource embedded an Connection pool, but don't worry any daemon thread
running at backend and Connection keep alive, everything in Connection Pool will 
be faded.

## Create DataSource

New an instance by using keyword `new`

```java
JSQLDataSource datasource = new JSQLDataSource("jdbc:h2:tcp://192.168.200.96:1522/testdb;DB_CLOSE_ON_EXIT=FALSE", "your_name", "your_password");
```

Use `Properties` instead
```java
Properties props = new Properties();
props.setProperty("url", "jdbc:mariadb://192.168.200.96:3307/testdb?serverTimezone=GMT%2B8");
props.setProperty("loginTimeout", "10");
props.setProperty("username", "jsql"); // will be overridden by driver.user
props.setProperty("driver.user", "your_name");
props.setProperty("driver.password", "your_password");

JSQLDataSource datasource = new JSQLDataSource(props);
```
> Please use `Properties.setProperty` rather than `Properties.put`

Use `DataSourceBuilder` for the best

```java
JSQLDataSource datasource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbc:mariadb://192.168.200.96:3307/testdb?serverTimezone=GMT%2B8")
                            .user("your_name").password("your_password").loginTimeout(10)
                            .poolMaxSize(8)
                            .poolIdleTimeout(500000)
                            .poolObjectCreateRetryCount(2)
                            .poolPollTimeout(5000)
                            .poolScheduleThreadLifeTime(9000)
                            .poolValidationOnBorrow(false)
                            .poolValidationOnReturn(true)
                            .addMapProperties(() -> {
                                Map<String, String> props = new HashMap<>();
                                props.put("driver.socketFactory", "javax.net.DefaultSocketFactory");
                                return props;
                            }).build();
```

## Configuration

 property name | comment | mandatory | default 
---|---|---|---
url | connection url | Y | 
username | connection username, this is not necessary for memory DB, e.g. derby or sqlite | N | 
password | connection password, this is not necessary for memory DB, e.g. derby or sqlite | N |
driverClass | jdbc Driver class name | N | detected by url
dialect | your customized Dialect class name | N | detected by url
loginTimeout | seconds of connection timeout creation | N | 5 seconds
pool.maxPoolSize | max connection pool size | N | 20
pool.idleTimeout | milliseconds of object idle timeout | N | 30 minutes
pool.scheduledThreadLifeTime | idle schedule thread's life time in milliseconds | N | 5 minutes
pool.pollTimeout | milliseconds of borrowing a object timeout | N | 5 seconds
pool.validateOnBorrow | validate on borrowing an object from pool | N | true
pool.validateOnReturn | validate on returning an object to pool | N | false
pool.createRetryCount | retry to create pool object if exception occur | N | 0
driver.user | prior to username | N | 
driver.password | prior to password | N | 

**NOTE**
- `pool.*` configuration please checkout [Connection Pool](pool.md)
- `driver.*` configuration depend on Database Driver implementation, 
e.g. using proxy socket factory for creating MariaDB Connection by 
following configuration `driver.socketFactory=custom.proxy.Socks5SocketFactory`