# Logging

While using JSQL, we must concern SQL and it's values' log.
So, JSQL design an interface of `cn.icuter.jsql.log.JSQLLogger` for your own Logger.
JSQL has implemented SLF4j/Log4j2/Log4j2/JUL, and auto detecting priority is `SLF4j > Log4j2 > Log4j2 > JUL`,
as long as your library including `slf4j.jar` or other else supported by JSQL.

**Note**
> JUL is the short name of `java.util.logging.Logger`, `logback` depends on SLF4j as well.

### Configuration
#### Log4j2
log4j2.xml
```
<Loggers>
    <Logger name="cn.icuter.jsql" level="info" additivity="false">
        <AppenderRef ref="Console"/>
    </Logger>
    ...
</Loggers>
```
#### Log4j
log4j.properties
```
log4j.logger.cn.icuter.jsql=INFO
```
#### logging.properties
```
cn.icuter.jsql.level=INFO
```

### Custom Logger
Define a CustomLogger class which extends JDKLogger, maybe Log4jLogger, Log4j2Logger or SLF4jLogger.
```
public class CustomLogger extends JDKLogger {

    @Override
    public void init(Class<?> aClass) {
        logger = (Logger) Main.LOGGER;
    }
}
```
Configure your custom Logger
```
static {
    Logs.setLogger(CustomLogger.class);
}
```