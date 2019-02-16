# Dialect

Dialect is for unify variant DB while building SQL, so far, Dialect could inject offset and limit, specify driver class name.

### Support Dialects

- CubridDialect
- SQLiteDialect
- DB2Dialect
- DerbyDialect
- H2Dialect
- MariaDBDialect
- MySQLDialect
- OracleDialect
- PostgreSQLDialect
- SQLServer2012PlusDialect
- SqlServerDialect
- UnknownDialect (Other kinds of DB JSQL don't support)

### Custom your own Dialect

1. Implements Dialect interface
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

1. How to make `CustomDialect` work
    ```java
    JSQLDataSource datasource = new JSQLDataSource(url, username, password, new CustomDialect());
    
    Builder builder = new SelectBuilder(new CustomDialect());
    ```
