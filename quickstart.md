# Quick Start

As following example, you can learn how to new a `Connection` from `JSQLDataSource`, Build SQL with Builder, and finally execute with JdbcExecutor.

## Configure in maven pom.xml file

```text
<repositories>
  ...
  <repository>
    <id>icuter</id>
    <url>https://github.com/icuter/mvn-repository/tree/master</url>
  </repository>
  ...
</repositories>

<dependency>
  <groupId>cn.icuter</groupId>
  <artifactId>jsql</artifactId>
  <version>1.0.2</version>
</dependency>
```
## Coding

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (Connection connection = dataSource.newConnection()) {
    JdbcExecutor executor = new DefaultJdbcExecutor(connection);
    Builder builder = new SelectBuilder() {{
        select().from("table").where().eq("name", "jsql").build();
    }};
    List<Map<String, Object>> list = executor.execQuery(builder);
}
```

Maybe you just need `JdbcExecutor` rather than `Connection`, and `JSQLDataSource` can also create a Builder for less coding and convenience we could simplfy our example as follow
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor(connection)) {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
}
```

JSQLDataSource should be singleton per url/username