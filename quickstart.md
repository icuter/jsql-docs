# Quick Start

As following example, you can learn how to new a `Connection` from `JSQLDataSource`, Build SQL with Builder, and finally execute with JdbcExecutor.

## Maven dependency

```text
<!-- for jdk1.8+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql</artifactId>
  <version>1.1.2</version>
</dependency>

<!-- for jdk1.6+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql-jdk1.6</artifactId>
  <version>1.1.2</version>
</dependency>
```
## Example

Assume we want to find something in database as follow
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
List<Map<String, Object>> list = dataSource.select()
                                           .from("table")
                                           .where().eq("name", "jsql")
                                           .execQuery();
```

Same as

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select()
                                               .from("table")
                                               .where().eq("name", "jsql")
                                               .execQuery(executor);
}
```

What has done above examples?
1. JSQLDataSource embedded a Connection Pool
2. `dataSource.getJdbcExecutor()` get Jdbc Executor from Connection Pool
3. use JdbcExecutor from pool to execute `dataSource.select()` 
4. close JdbcExecutor for returning back to the Connection Pool in JSQLDataSource
