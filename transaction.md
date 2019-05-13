# Transaction
At this section, we could learn `TransactionExecutor`'s usage and we would note that it's detail for commit/rollback/end and even Connection close.

### TransactionExecutor
Now, let'u create a `TransactionExecutor` with `Connection` parameter. At following example, while we commit/rollback, `Connection` will be closed as well.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Connection connection = dataSource.createConnection(false);
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

### TransactionDataSource

Do transaction under TransactionDataSource that created by JSQLDataSource.
Please play attention, if exception occurs will cause transaction rollback.

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
dataSource.transaction(tx -> {
    tx.insert("table")
      .values(Cond.eq("col1", "val1"), Cond.eq("col2", 102),Cond.eq("col3", "val3")).execUpdate();
    // tx.commit(); // try to commit if transaction ended
});
```

```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
TransactionDataSource tx = dataSource.transaction();
tx.insert("table")
  .values(Cond.eq("col1", "val1"), Cond.eq("col2", 102),Cond.eq("col3", "val3"))
  .execUpdate();
tx.close(); // try to commit if close
```

> Perform committing when calling `TransactionDataSource.close()` if transaction does NOT commit or rollback