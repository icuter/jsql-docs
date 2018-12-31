# ORM

You can use simple orm helper utily which name is `cn.icuter.jsql.orm.ORMapper`. Using `@ColumnName` annotation associate db table column and object field name.

### Select Value
`cn.icuter.jsql.executor.DefaultJdbcExecutor.execQuery(Builder, Class<T>)` map to customer object

```java
Builder builder = new SelectBuilder().select().from("t_table t").build();
JdbcExecutor jdbcExecutor = new DefaultJdbcExecutor(connection);
List<Map<String, Object>> resultMap = jdbcExecutor.execQuery(builder);
List<Table> resultORM = jdbcExecutor.execQuery(builder, Table.class);
```

### Insert Value
If field value is null will be ignore
```java
Student student = new Student();
student.setGrade(2);
student.setName("Edward");
student.setAge(20);
student.setClass("04-1");

Builder insert = new InsertBuilder().insertInto("t_student").values(student).build();
```

### Update Value
If field value is null will be ignore
```java
Student student = new Student();
student.setGrade(2);
student.setName("Edward");
student.setAge(20);
student.setClass(null); // don't set class null

Builder update = new UpdateBuilder().update("t_student").set(student).build();
```

If you want to set null value, you can use `ORMapper.toMap` instead of `ORMapper.toMapIgnoreNullValue`
```java
Student student = new Student();
student.setGrade(2);
student.setName("Edward");
student.setAge(20);
student.setClass(null);

Builder update = new UpdateBuilder().update("t_student").set(ORMapper.of(student).toMap()).build();
```