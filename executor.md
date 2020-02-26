# Executor

## JdbcExecutor
JdbcExecutor for executing Builder.

### Select

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery();
```

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
}
```

### Insert/Update/Delete
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
int count = dataSource.insert("table").values(Cond.eq("name", "jsql")).execUpdate();
count = dataSource.update("table").set(Cond.eq("name", "icuter")).execUpdate();
count = dataSource.delete().from("table").where().eq("name", "jsql").execUpdate();
```

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    int count = dataSource.insert("table").values(Cond.eq("name", "jsql")).execUpdate(executor);
    count = dataSource.update("table").set(Cond.eq("name", "icuter")).execUpdate(executor);
    count = dataSource.delete().from("table").where().eq("name", "jsql").execUpdate(executor);
}
```

### Batch Update
Builder List in batch group by the same sql, if sql is different, will separate different batch update.

```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();

try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Builder> builderList = new LinkedList<>() {{
        add(new DeleteBuilder().delete().from("table").where().eq("name", "Jhon").build());
        add(new DeleteBuilder().delete().from("table").where().eq("name", "Edward").build());
        add(new DeleteBuilder().delete().from("table").where().eq("name", "Jack").build());
    }};
    executor.execBatch(builderList);
}
```

As following example, I try to delete and update in `execBatch` with different executable SQL.
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();

try (JdbcExecutor executor = dataSource.getJdbcExecutor()) {
    List<Builder> builderList = new LinkedList<>() {{
        add(new DeleteBuilder().delete().from("table").where().eq("name", "Jhon").build());
        add(new DeleteBuilder().delete().from("table").where().eq("name", "Edward").build());
        add(new UpdateBuilder().update("table").set(Cond.eq("name", "Edward")).where().eq("id", 12345678).build());
        add(new UpdateBuilder().update("table").set(Cond.eq("name", "John")).where().eq("id", 123456789).build());
    }};
    executor.execBatch(builderList);
}
```
At this time, `delete from table where name = ?` and `update table set name = ? where id = ?` will create two batch execution.

### Note
If you want to new a `JdbcExcecutor` instance without `JSQLDataSource`, you could pass your `Connection` as parameter to do that.
```java
Connection connection = yourConnection;
JdbcExecutor executor = new DefaultJdbcExecutor(connection);
Builder builder = new SelectBuilder().select().from("table").where().eq("name", "jsql").build();
List<Map<String, Object>> resultList = executor.execQuery(builder);
```

## TransactionExecutor
Now, let's create a `TransactionExecutor` with `Connection` parameter. At following example, while we commit/rollback, `Connection` will be closed as well.
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
Connection connection = dataSource.getConnection();
TransactionExecutor executor = new TransactionExecutor(connection);
try {
    Builder delete = new DeleteBuilder().delete().from("table_name").where().eq("id", "<UUID>").build()};
    executor.execUpdate(delete);
    executor.commit();
} catch (Exception e) {
    executor.rollback();
} finally {
    executor.close();
}
```

### JSQLDataSource
`TransactionExecutor` could created by `JSQLDataSource` directly.
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
TransactionExecutor executor = dataSource.getTransactionExecutor();
try {
    dataSource.delete().from("table_name").where().eq("id", "<UUID>").execUpdate(executor);
    executor.commit();
} catch (Exception e) {
    executor.rollback();
} finally {
    executor.close();
}
```

**Note**
> Please note that, Connection will be closed while calling commit or rollback, if don't commit or rollback, Connection in TransactionExecutor will be always alive and keep connecting to DB server.

### JdbcExecutorPool
From JdbcExecutor Pool we could get `TransactionExecutor` and process jdbc SQL with it and do transaction manually.
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
JdbcExecutorPool pool = dataSource.getExecutorPool();
TransactionExecutor executor = pool.getTransactionExecutor();
try {
    ...
    executor.commit();
} catch (Exception e) {
    executor.rollback();
} finally {
    pool.returnExecutor(executor);
}
```

Another example show you calling `executor.close`/`executor.end` same as returning to executor pool.
```java
JSQLDataSource dataSource = JSQLDataSource.newDataSourceBuilder()
                            .url("jdbcUrl").user("jsql").password("pass").build();
JdbcExecutorPool pool = dataSource.getExecutorPool();
TransactionExecutor executor = pool.getTransactionExecutor();
try {
    // TODO TransactionExecutor which from JdbcExecutorPool will be return after closed 
    executor.commit();
} finally {
    executor.close();
}
```
