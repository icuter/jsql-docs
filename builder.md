# Builder

Builder is responsible for SQL generation, like DQL and DML syntax, such as `select`/`insert`/`update`/`delete`. The best way to create a Builder instance is using `JSQLDataSource` which will set `Dialect` into Builder automatically. Let's refer as following examples to understand it's usage.
```java
JSQLDataSource dataSource = new JSQLDataSource("jdbc:mysql://host:3306/database", "username", "password");
SelectBuilder select = dataSource.select(String... cols); // same to new SelectBuilder(Dialects.MYSQL).select(cols);
UpdateBuilder update = dataSource.update(table);          // same to new UpdateBuilder(Dialects.MYSQL).update(table);
InsertBuilder insert = dataSource.insert(table);          // same to new InsertBuilder(Dialects.MYSQL).insert(table);
DeleteBuilder delete = dataSource.delete();               // same to new DeleteBuilder(Dialects.MYSQL).delete();
```

### SelectBuilder
Created by keyword of `new`, normally, we could new a Builder for SQL construction.
```java
Builder builder = new SelectBuilder() {{
    select().from("table").where().eq("name", "jsql").build();
}};
```
In less coding way, we could create a Builder by using `JSQLDataSource` as follow.
```java
JSQLDataSource dataSource = new JSQLDataSource(...);
dataSource.select().from("table").where().eq("name", "jsql").build();
```

**SQL**: select * from table where name = ?

**VALUE**: "jsql"

### InsertBuilder

```java
Builder insert = new InsertBuilder() {{
    insert("table")
        .values(Cond.eq("col1", "val1"), Cond.eq("col2", 102),Cond.eq("col3", "val3"))
        .build();
}};
```

Created by `JSQLDataSource` as follow.
```java
JSQLDataSource dataSource = new JSQLDataSource(...);
dataSource.insert("table")
        .values(Cond.eq("col1", "val1"), Cond.eq("col2", 102),Cond.eq("col3", "val3"))
        .build();
```
**SQL**: insert into t_table(col1,col2,col3) values(?,?,?)

**VALUE**: "val1", 102, "val3"

### UpdateBuilder

```java
Builder update = new UpdateBuilder() {{
    update("t_table")
        .set(Cond.eq("col1", "val1"), Cond.eq("col2", 102), Cond.eq("col3", "val3"))
        .where()
        .like("id", "123%")
        .build();
}};
```

Created by `JSQLDataSource` as follow.
```java
JSQLDataSource dataSource = new JSQLDataSource(...);
dataSOurce.update("t_table")
        .set(Cond.eq("col1", "val1"), Cond.eq("col2", 102), Cond.eq("col3", "val3"))
        .where()
        .like("id", "123%")
        .build();
```
**SQL**: update t_table set col1 = ?, col2 = ?, col3 = ? where id like ?

**VALUE**: "val1", 102, "val3"

### DeleteBuilder
```java
Builder delete = new DeleteBuilder() {{
    delete().from("t_table").where().eq("id", 123456789).build();
}};
```

Created by `JSQLDataSource` as follow.
```java
JSQLDataSource dataSource = new JSQLDataSource(...);
dataSOurce.delete().from("t_table").where().eq("id", 123456789).build();
```
**SQL**: delete from t_table where id = ?

**VALUE**: 123456789
