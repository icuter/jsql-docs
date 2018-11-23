# Executor

## JdbcExecutor
When Builder has been built, we could use JdbcExecutor for execution. `dataSource.createJdbcExecutor()` will create a `CloseableJdbcExecutor` object, while calling `close` method, `Conection` in `JdbcExecutor` will be closed as well.

### Select
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    List<Map<String, Object>> list = dataSource.select().from("table").where().eq("name", "jsql").execQuery(executor);
}
```

### Insert/Update/Delete
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    int count = delete().from("table").where().eq("name", "jsql").execUpdate(executor);
}
```

### Batch Update
Builder List in batch group by the same sql, if sql is different, will seperate different batch update.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    List<Builder> builderList = new LinkedList<>() {{
        add(new DeleteBuilder() {{
            delete().from("table").where().eq("name", "Jhon").build();
        }});
        add(new DeleteBuilder() {{
            delete().from("table").where().eq("name", "Edward").build();
        }});
        add(new DeleteBuilder() {{
            delete().from("table").where().eq("name", "Jack").build();
        }});
    }};
    executor.execBatch(builderList);
}
```

As following example, I try to delete and update in `execBatch` with different executable SQL.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
try (JdbcExecutor executor = dataSource.createJdbcExecutor()) {
    List<Builder> builderList = new LinkedList<>() {{
        add(new DeleteBuilder() {{
            delete().from("table").where().eq("name", "Jhon").build();
        }});
        add(new DeleteBuilder() {{
            delete().from("table").where().eq("name", "Edward").build();
        }});
        add(new UpdateBuilder() {{
            update("table").set(Cond.eq("name", "Edward")).where().eq("id", 12345678).build();
        }});
        add(new UpdateBuilder() {{
            update("table").set(Cond.eq("name", "John")).where().eq("id", 123456789).build();
        }});
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
Builder builder = new SelectBuilder() {{
    select().from("table").where().eq("name", "jsql").build();
}};
List<Map<String, Object>> resultList = executor.execQuery(builder);
```

## Transaction
At this section, we could learn TransactionExecutor's usage and we would pay attention it's detail for commit/rollback/end and even Connection close.

### TransactionExecutor
Now, let'u create a `TransactionExecutor` with `Connection` parameter. At following example, while we commit/rollback, `Connection` will be closed as well.
```java
Connection connection = createConnection(false);
TransactionExecutor executor = new TransactionExecutor(connection);
executor.setStateListener((tx, state) -> {
    if (tx.wasCommitted() || tx.wasRolledBack()) {
        try {
            if (connection != null && !connection.isClosed()) {
                connection.close();
            }
        } catch (SQLException e) {
            throw new ExecutionException("closing Connection error", e);
        }
    }
});
Builder delete = new DeleteBuilder() {{
    delete().from("table_name").where().eq("id", "<UUID>").build();
}};
executor.execUpdate(delete);

executor.end(); // auto commit and close Connection by using StateListener
```

### JSQLDataSource
We could create a `TransactionExecutor` by `JSQLDataSource`, and let's simplfy the example above.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
TransactionExecutor executor = dataSource.createTransaction();
try {
    dataSource.delete().from("table_name").where().eq("id", "<UUID>").execUpdate(executor);
    executor.commit();
} catch (Exception e) {
    executor.rollback();
}
```

**Note**
> Please note that, Connection will be closed while calling commit or rollback, if don't commit or rollback, Conneciton in TransactionExecutor will be always alive and keep connecting to DB server.

### JdbcExecutorPool
From JdbcExecutor Pool we could get `TransactionExecutor` and process jdbc SQL with it and do transaction manually.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutorPool pool = dataSource.createExecutorPool();
TransactionExecutor executor = pool.getTransactionExecutor()
try {
    ...
    executor.commit();
} catch (Exception e) {
    executor.rollback();
} finally {
    pool.returnExecutor(executor);
}
```

For convenience, we could simplfy our sample as follow, if you forget to commit transaction, all DML operation to DB will be auto commit before return to the jdbc executor pool. When call `executor.close`/`executor.end` will auto return to executor pool.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
JdbcExecutorPool pool = dataSource.createExecutorPool();
try (TransactionExecutor executor = pool.getTransactionExecutor()) {
    ... auto commit when executor.close or end
}
```
