# Logging

While using JSQL, we must concern SQL and it's value log. For this, we design a interface name `cn.icuter.jsql.log.JSQLLogger`, if you want to use your own Logger which your system is in use. We have implemented SLF4j/Log4j2/Log4j2/JUL, so that logging into your log automatically.

Note
> JUL is the short name of `java.util.logging.Logger`, logback not in implemented list is due to it's using SLF4j api as well.

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