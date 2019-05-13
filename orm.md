# ORM

Class of `cn.icuter.jsql.orm.ORMapper` and annotation of `@ColumnName` to associate
the DB table columns with class's fields. Of course, if you feel tedious to define a relational class mapping field and column,
you would like to operate `Map<String, Object>` instead of class. As the following invocation
- `InsertBuilder.values(Map<String, Object>)`
- `UpdateBuilder.set(Map<String, Object>)`
- `DefaultJdbcExecutor.execQuery(Builder)`

## Support Class Field Type
- short/Short
- int/Integer
- long/Long
- float/Float
- double/Double
- byte/Byte
- boolean/Boolean
- BigInteger
- BigDecimal
- String
- Clob
- NClob
- Blob
- byte[]

## Select
JSQL wrap result from DB by using DQL(`select * from table ...`) as a map of list or relational class of list.
Let's check out the example below to understand it's usage.

First, we create a class which named `ORMClass`.
```java
public class ORMClass {
    @ColumnName("orm_id")
    private String ormId;
    @ColumnName("f_blob")
    private byte[] fBlob;
    @ColumnName("f_blob")
    private Blob fBlobObj;
    @ColumnName("f_clob")
    private String fClob;
    @ColumnName("f_clob")
    private Clob fClobObj;
    @ColumnName("f_string")
    private String fString;
    @ColumnName("f_int")
    private int fInt;
    @ColumnName("f_integer")
    private Integer fInteger;
    @ColumnName("f_double")
    private double fDouble;
    @ColumnName("f_decimal")
    private BigDecimal fDecimal;

    // Collapse get/set method
}
```

Second, query DB result to `Map<String, Object>` or `ORMClass` for each record.
```java
JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
Builder builder = new SelectBuilder().select().from("t_table t").build();
try (JdbcExecutor jdbcExecutor = dataSource.createJdbcExecutor()) {
    List<Map<String, Object>> resultMap = jdbcExecutor.execQuery(builder);
    List<ORMClass> resultORM = jdbcExecutor.execQuery(builder, ORMClass.class);
}
```

## Insert
In this section, we could learn how to insert an `ORMClass` or `Map<String, Object>` to DB. We will reuse the
`ORMClass` in the previous section.

```java
ORMClass ormClass = new ORMClass();
ormClass.setOrmId(UUID.randomUUID().toString());
ormClass.setfString(null); // ignore inserting
ormClass.setfInteger(100);
ormClass.setfInt(100);
ormClass.setfDecimal(new BigDecimal("100.002"));
ormClass.setfDouble(100.00d);
ormClass.setfClobObj(new JSQLClob("test jsql clob"));
ormClass.setfBlobObj(new JSQLBlob("test jsql clob".getBytes()));

JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
dataSource.insert("t_test_table").values(ormClass).execUpdate();
```

### NOTE

    As default, the null value of field will be ignored while processing,
    and Clob/NClob/Blob could be created by JSQLClob/JSQLNClob/JSQLBlob.

## Update
Update DB record with ORM, as default, you can not update null value, if you want to do so, 
try to call `UpdateBuilder.set(Object, FieldInterceptor)` instead of `UpdateBuilder.set(Object)`.

```java
ORMClass ormClass = new ORMClass();
ormClass.setOrmId(UUID.randomUUID().toString());
ormClass.setfString(null); // ignore update
ormClass.setfInteger(100);
ormClass.setfInt(100);
ormClass.setfDecimal(new BigDecimal("100.002"));
ormClass.setfDouble(100.00d);
ormClass.setfClobObj(new JSQLClob("test jsql clob"));
ormClass.setfBlobObj(new JSQLBlob("test jsql clob".getBytes()));

JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
dataSource.update("t_test_table").set(ormClass)
          .where().eq("orm_id", ormClass.getOrmId()).execUpdate();
```

As following example, let's to use `UpdateBuilder.set(Object, FieldInterceptor)` to ignore orm_id update.
```java
ORMClass ormClass = new ORMClass();
ormClass.setOrmId(UUID.randomUUID().toString());
ormClass.setfString(null); // ignore update
ormClass.setfInteger(100);
ormClass.setfInt(100);
ormClass.setfDecimal(new BigDecimal("100.002"));
ormClass.setfDouble(100.00d);
ormClass.setfClobObj(new JSQLClob("test jsql clob"));
ormClass.setfBlobObj(new JSQLBlob("test jsql clob".getBytes()));

JSQLDataSource dataSource = new JSQLDataSource("url", "username", "password");
    dataSource.update("t_test_table")
              .set(ormClass, (object, field, colName, value, resultMap) -> !"orm_id".equals(colName))
              .where().eq("orm_id", ormClass.getOrmId()).execUpdate();
}
```