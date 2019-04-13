# Condition

There are variant condition and you will find out the method name is same to SQL syntax. Condition is used in Builder, sometimes also used as value. Here docs is for special condition, such as `gt(>)`/`ge(>=)`/`lt(<)`/`le(<=)`/`like`/`ne(<>)`/`eq(=)` I thought you can handle it yourself.

### var
Sometimes, we need to make condition without place holder which display in SQL with `?`, but directly display in SQL, just like `Cond.var("key", "value")` output `key = value`, check example as follow.

```java
Builder builder = new SelectBuilder()
    .select()
    .from("t_table", "t_table1")
    .where()
    .var("t_table.id", "t_table1.id")
    .build();
```
*SQL*: select * from t_table, t_table1 where `t_table.id=t_table1.id`

### groupBy/having
```java
Builder builder = new SelectBuilder()
    .select("name", "age")
    .from("t_table")
    .groupBy("name", "age").having(Cond.gt("age", 18))
    .build();
```
*SQL*: select name, age from t_table group by name,age having ( age > ?)
*VALUE*: 18

### orderBy
```java
Builder builder = new SelectBuilder()
    .select("name", "age")
    .from("t_table")
    .orderBy("name desc", "age")
    .build();
```
*SQL*: select name, age from t_table order by name desc,age

### isNull/isNotNull
```java
Builder builder = new SelectBuilder()
    .select("name", "age")
    .from("t_table")
    .where()
    .isNull("name")
    .and()
    .isNotNull("age")
    .build();
```
*SQL*: select name, age from t_table where name is null and age is not null

### forUpdate
```java
Builder builder = new SelectBuilder()
    .select("name", "age")
    .from("t_table")
    .where()
    .eq("id", 123456789)
    .forUpdate()
    .build();
```
*SQL*: select name, age from t_table where id = ? for update

*VALUE*: 123456789

### and/or
Sometimes we need to resolve multi conditions combination, then `and(Condition... conditions)` and `or(Condition... conditions)` could be used, as following example you can find out their usage.
```java
Builder builder = new SelectBuilder()
    .select("name", "age")
    .from("t_table")
    .where()
    .and(Cond.like("name", "%Lee"), Cond.gt("age", 18))
    .and()
    .or(Cond.eq("age", 12), Cond.eq("age", 16))
    .build();
```
*SQL*: select name, age from t_table where ( name like ? and age > ?) and ( age = ? or age = ?)

*VALUE*: "%Lee", 18, 12, 16


### exists
```java
Builder existsSelect = new SelectBuilder()
    .select("1")
    .from("t_table1")
    .where()
    .var("t_table.id", "t_table1.id")
    .build();
Builder builder = new SelectBuilder()
    .select()
    .from("t_table")
    .where()
    .exists(existsSelect)
    .build();
```
*SQL*: select * from t_table where exists (select 1 from t_table1 where t_table.id=t_table1.id)

### notExists
```java
Builder existsSelect = new SelectBuilder()
    .select("1")
    .from("t_table1")
    .where()
    .var("t_table.id", "t_table1.id")
    .build();
Builder builder = new SelectBuilder()
    .select()
    .from("t_table")
    .where()
    .notExists(existsSelect)
    .build();
```
*SQL*: select * from t_table where not exists (select 1 from t_table1 where t_table.id=t_table1.id)

### in
#### with Array
```java
Builder select = new SelectBuilder().select().from("table").where().in("lang", "TW", "CN", "HK").build();
```
*SQL*: select * from table where lang in (?,?,?)

*VALUE*: "TW", "CN", "HK"

#### with `java.util.Collection`
```java
Builder select = new SelectBuilder()
    .select()
    .from("table")
    .where()
    .in("lang", new LinkedList<String>() {{
        add("TW");
        add("CN");
        add("HK");
    }}})
    .build();
```
*SQL*: select * from table where lang in (?,?,?)
 
*VALUE*: "TW", "CN", "HK"

#### with `SelectBuilder`
```java
Builder selectIn = new SelectBuilder().select("name").from("table_1").where().like("name", "%jsql%").build();
Builder select = new SelectBuilder().select().from("table").where().in("name", selectIn).build();
```
*SQL*: select * from table where name in (select name from table where name like ?)

*VALUE*: "%jsql%"

### Join Table
#### inner Join
```java
Builder builder = new SelectBuilder()
    .select()
    .from("table_1").joinOn("table_2", Cond.var("table1.id", "table2.id"), Cond.eq("t.framework", "jsql"))
    .build();
```
*SQL*: select * from table_1 join table_2 on (table1.id=table2.id and t.framework = ?)

*VALUE*: "jsql"

#### left Join
```java
Builder builder = new SelectBuilder()
    .select()
    .from("table_1").leftJoinOn("table_2", Cond.var("table1.id", "table2.id"))
    .build();
```
*SQL*: select * from table_1 left join table_2 on (table1.id=table2.id and t.framework = ?)

*VALUE*: "jsql"

#### right Join
```java
Builder builder = new SelectBuilder()
    .select()
    .from("table_1").rightJoinOn("table_2", Cond.var("table1.id", "table2.id"))
    .build();
```
*SQL*: select * from table_1 right join table_2 on (table1.id=table2.id and t.framework = ?)

*VALUE*: "jsql"

#### outer Join
```java
Builder builder = new SelectBuilder()
    .select()
    .from("table_1").outerJoinOn("table_2", Cond.var("table1.id", "table2.id"))
    .build();
```
*SQL*: select * from table_1 outer join table_2 on (table1.id=table2.id and t.framework = ?)

*VALUE*: "jsql"

#### full join
```java
Builder builder = new SelectBuilder()
    .select()
    .from("table_1").fullJoinOn("table_2", Cond.var("table1.id", "table2.id"))
    .build();
```
*SQL*: select * from table_1 full join table_2 on (table1.id=table2.id and t.framework = ?)

*VALUE*: "jsql"

### offset/limit
Offset and limit will make different paging SQL by different Dialect, as following example will show you MySQL and Oracle Dialect offset and limit setting. Only set offset without limit is forbidden, but only set limit is allow, e.g. `Builder.offset(6).build()` will occur error.

#### MySQL
```java
Builder select = new SelectBuilder(Dialects.MYSQL).select().from("table").where().eq("id", "0123456789").offset(5).limit(10).build();
```
*SQL*: select * from table where id = ? limit ?,?

*VALUE*: "0123456789", 5, 10

```java
Builder select = new SelectBuilder(Dialects.MYSQL).select().from("table").where().eq("id", "0123456789").limit(10).build();
```
*SQL*: select * from table where id = ? limit ?

*VALUE*: "0123456789", 10

#### Oracle
```java
Builder select = new SelectBuilder(Dialects.ORACLE).select().from("table").where().eq("id", "0123456789").offset(5).limit(10).build();
```

*SQL*: 
```sql
select * from ( select source_.*, rownum rownum_0_ from (select * from table where id = ? ) source_ where rownum <= ?) where rownum_0_ > ?
```

*VALUE*: "0123456789", 10, 5

```java
Builder select = new SelectBuilder(Dialects.ORACLE).select().from("table").where().eq("id", "0123456789").limit(10).build();
```
*SQL*: select * from ( select * from table where id = ? ) where rownum <= ?

*VALUE*: "0123456789", 10