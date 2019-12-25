# Quick Start

As following example, you can learn how to new a `Connection` from `JSQLDataSource`, Build SQL with Builder, and finally execute with JdbcExecutor.

## Maven dependency

```text
<!-- for jdk1.8+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql</artifactId>
  <version>1.1.1</version>
</dependency>

<!-- for jdk1.6+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql-jdk1.6</artifactId>
  <version>1.1.1</version>
</dependency>
```
## Example

Assume we want to find something in database as follow
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
List<Map<String, Object>> list = dataSource.select().from("table")
                                                    .where().eq("name", "jsql")
                                                    .execQuery();
```

Same as

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select().from("table")
                                                        .where().eq("name", "jsql")
                                                        .execQuery(executor);
}
```

what has done above examples ?
1. JSQLDataSource embedded a Connection Pool
2. `dataSource.getJdbcExecutor()` get Jdbc Executor from Connection Pool
3. use JdbcExecutor from pool to execute `dataSource.select()` 
4. close JdbcExecutor for returning back to the Connection Pool in JSQLDataSource


Maybe you just need `JdbcExecutor` rather than `Connection` and `JSQLDataSource` can also create a Builder for less coding and convenience we could simplfy our example as follow

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutor executor = dataSource.createJdbcExecutor();
try {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
} finally {
    executor.close();
}
```

If you are under the jdk1.7 +, you can code with `try(resource){}` instead of `try{..} finally {..}`

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
}
```
> **Suggestion**: JSQLDataSource is singleton for each url and username

*NOT* recommended way that you can also using Connection and JdbcExecutor to execute Builder

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Connection connection = dataSource.newConnection();
try {
    JdbcExecutor executor = new DefaultJdbcExecutor(connection);
    Builder builder = new SelectBuilder().select().from("table").where().eq("name", "jsql").build();
    List<Map<String, Object>> list = executor.execQuery(builder);
} finally {
    connection.close();
}
```

If you are under the jdk1.7 +, you can code with `try(resource){}` instead of `try{..} finally {..}`

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (Connection connection = dataSource.newConnection()) {
    JdbcExecutor executor = new DefaultJdbcExecutor(connection);
    Builder builder = new SelectBuilder().select().from("table").where().eq("name", "jsql").build();
    List<Map<String, Object>> list = executor.execQuery(builder);
}
```

