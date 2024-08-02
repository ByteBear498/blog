---
date: 2017-04-06 21:38:00
status: public
tags: 
 - logback
 - log
categoies:
 - java
title: java日志框架-logback
---

### 简单介绍
logback当前分成三个模块：logback-core,logback- classic和logback-access。
1.  logback-core 是其它两个模块的基础模块。
2.  logback-classic是log4j的一个 改良版本。此外logback-classic完整实现SLF4J API。
3.  logback-access提供HTTP访问日志功能。
### MAVEN配置
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>
```
### 测试代码
```java
Logger logger = LoggerFactory.getLogger("test");
logger.info("this is test string");
LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
StatusPrinter.print(lc);
```

### logback.xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [ %t:%r ] [ %p ] %m%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/info.log</file>
        <append>true</append>
        <immediateFlush>true</immediateFlush>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [ %t:%r ] [ %p ] %m%n</pattern>
        </encoder>
    </appender>

    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

logback官网：https://logback.qos.ch/manual/configuration.html