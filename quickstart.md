# Quick Start

As following example, you can learn how to new a `Connection` from `JSQLDataSource`, Build SQL with Builder, and finally execute with JdbcExecutor.

## Maven dependency

```text
<!-- for jdk1.8+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql</artifactId>
  <version>1.0.3</version>
</dependency>

<!-- for jdk1.6+ -->
<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql-jdk1.6</artifactId>
  <version>1.0.3</version>
</dependency>
```
## Example

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Connection connection = dataSource.newConnection();
try {
    JdbcExecutor executor = new DefaultJdbcExecutor(connection);
    Builder builder = new SelectBuilder().select().from("table").where().eq("name", "jsql").build()};
    List<Map<String, Object>> list = executor.execQuery(builder);
} finally {
    connection.close();
}
```

Maybe you just need `JdbcExecutor` rather than `Connection`, and `JSQLDataSource` can also create a Builder for less coding and convenience we could simplfy our example as follow

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutor executor = dataSource.createJdbcExecutor(connection);
try {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
} finally {
    executor.close();
}
```

> **Suggestion**: JSQLDataSource is singleton for each url/username