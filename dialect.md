# Dialect

Dialect is for unify variant DB while building SQL, so far, Dialect could inject offset and limit, specify driver class name.

### Support Dialects

- CubridDialect
- SQLiteDialect
- DB2Dialect
- DerbyDialect (EmbeddedDerbyDialect/NetworkDerbyDialect)
- H2Dialect
- MariaDBDialect
- MySQLDialect
- OracleDialect
- PostgreSQLDialect
- SQLServer2012PlusDialect (version >= 2012)

### Custom your own Dialect

- Implements `cn.icuter.jsql.dialect.Dialect` or extends `cn.icuter.jsql.dialect.AbstractDialect`

    ```java
    public CustomDialect implements Dialect {
        public String getDriverClassName() {
            return "my.custom.jdbc.MyDriver";
        }
        public void injectOffsetLimit(BuilderContext builderCtx) {
            boolean offsetExists = builderContext.getOffset() > 0;
            StringBuilder offsetLimitBuilder = new StringBuilder(offsetExists ? " limit ?,?" : " limit ?");
            if (offsetExists) {
                builderContext.addCondition(Cond.value(builderContext.getOffset()));
            }
            builderContext.addCondition(Cond.value(builderContext.getLimit()));
    
            StringBuilder preparedSqlBuilder = builderContext.getPreparedSql();
            if (builderContext.getForUpdatePosition() > 0) {
                preparedSqlBuilder.insert(builderContext.getForUpdatePosition(), offsetLimitBuilder);
            } else {
                preparedSqlBuilder.append(offsetLimitBuilder);
            }
        }
        public boolean supportOffsetLimit() {
            return true;
        }
    }
    ```

- How to make `CustomDialect` work

    ```java
    JSQLDataSource datasource = new JSQLDataSource(url, username, password, new CustomDialect());
    
    Builder builder = new SelectBuilder(new CustomDialect());
    ```

- `Join On` Support Features


 dialect name            | inner join on | left \[outer] join on | right \[outer] join on | full \[outer] join on
---|---|---|---|---
CubridDialect            | √             | √                     | √                      | ✘
SQLiteDialect            | √             | √                     | ✘                      | ✘
DB2Dialect               | √             | √                     | √                      | ✘
DerbyDialect             | √             | √                     | √                      | ✘
H2Dialect                | √             | √                     | √                      | ✘
MariaDBDialect           | √             | √                     | √                      | ✘
MySQLDialect             | √             | √                     | √                      | ✘
OracleDialect            | √             | √                     | √                      | √
PostgreSQLDialect        | √             | √                     | √                      | √
SQLServer2012PlusDialect | √             | √                     | √                      | √

- `Join Using` Support Features


 dialect name            | inner join using | left \[outer] join using | right \[outer] join using | full \[outer] join using
---|---|---|---|---
CubridDialect            | ✘                | ✘                        | ✘                         | ✘
SQLiteDialect            | √                | √                        | ✘                         | ✘
DB2Dialect               | √                | √                        | √                         | √
DerbyDialect             | √                | √                        | √                         | ✘
H2Dialect                | ✘                | ✘                        | ✘                         | ✘
MariaDBDialect           | √                | √                        | √                         | ✘
MySQLDialect             | √                | √                        | √                         | ✘
OracleDialect            | √                | √                        | √                         | √
PostgreSQLDialect        | √                | √                        | √                         | √
SQLServer2012PlusDialect | ✘                | ✘                        | ✘                         | ✘