# JSQL

## Introduction

Welcome to JSQL. It's a lightweight JDBC DSL framework, JSQL means `Just SQL` and without ORM configuration.
It is a framework which is convenience and easy to use, just like sql syntax that you feel free to write SQL as usual and 
makes Java SQL development more easier.

If you are a Java developer searching for the jdbc framework satisfy functions as connection pool, super lightweight orm
and want to write sql like java code programing, then I suggest you could try to use JSQL framework for jdbc operation.

JSQL for Reasons:
- No SQL string and keep your code graceful
- No ORM bean code generation mass your git control
- Provide ExecutorPool/ConnectionPool for jdbc connection pooling without DBCP dependencies

## Requirements

- JDK6 or higher

## Features
1. Connection/JdbcExecutor pool
2. SQL syntax like builder
3. Transaction
4. Support customizing dialects
5. Pagination
6. Jdbc executor for query, update or batch update
7. Super lightweight ORM
8. Against SQL inject
9. Logging ability

## Support Databases
1. Cubrid
2. SQLite
3. DB2
4. Derby (EmbeddedDerby/NetworkDerby)
5. H2
6. MariaDB
7. MySQL
8. Oracle(Driver Version >= ojdbc6)
9. PostgreSQL
10. SQLServer2012(version >= 2012)
