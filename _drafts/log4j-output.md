# Log4J输出日志的几种方式


<!-- @import "[TOC]" {cmd="toc" depthFrom=5 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [1. 发送日志邮件](#1-发送日志邮件)
- [2. 输出到数据库](#2-输出到数据库)
- [3. 输出到文件](#3-输出到文件)
- [4. 通过Socket输出](#4-通过socket输出)
- [5. 输出到 Kafka](#5-输出到-kafka)
- [6. 自定义输出](#6-自定义输出)

<!-- /code_chunk_output -->


##### 1. 发送日志邮件

pass the vm arguement first i.e `-Dmail.smtp.starttls.enable=true`

```
log4j.rootLogger=DEBUG, sendMail
log4j.appender.sendMail=org.apache.log4j.net.SMTPAppender
log4j.appender.sendMail.SMTPProtocol=smtps
log4j.appender.sendMail.Threshold=ERROR
log4j.appender.sendMail.SMTPPort=465
log4j.appender.sendMail.SMTPUsername=xxxx@gmail.com
log4j.appender.sendMail.From=xxxx@gmail.com
log4j.appender.sendMail.SMTPPassword=.........
log4j.appender.sendMail.To=yyyyy@gmail.com
log4j.appender.sendMail.SMTPHost=smtp.gmail.com
log4j.appender.sendMail.Subject=Error Alert
log4j.appender.sendMail.layout=org.apache.log4j.PatternLayout
log4j.appender.sendMail.layout.ConversionPattern=%d{yyyy-MM-ddHH:mm:ss.SSS} [%p] %t %c - %m%n
log4j.appender.sendMail.smtp.starttls.enable=true
log4j.appender.sendMail.smtp.auth=true
log4j.appender.sendMail.BufferSize=1
```


##### 2. 输出到数据库
##### 3. 输出到文件
##### 4. 通过Socket输出

```properties
log4j.logger.com.demo.elk=DEBUG, socket

# appender socket
log4j.appender.socket=org.apache.log4j.net.SocketAppender
log4j.appender.socket.Port=4567
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=%d [%-5p] [%l] %m%n
log4j.appender.socket.ReconnectionDelay=10000
```

##### 5. 输出到 Kafka

[KafkaAppender](https://logging.apache.org/log4j/2.x/manual/appenders.html#KafkaAppender)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="info" name="spring-boot-kafka-log" packages="com.example">
    <Appenders>
        <Kafka name="Kafka" topic="kafka-test">
            <PatternLayout pattern="%date %message"/>
            <Property name="bootstrap.servers">localhost:9092</Property>
        </Kafka>
        <Async name="Async">
            <AppenderRef ref="Kafka"/>
        </Async>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5p [%-7t] %F:%L - %m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="Kafka"/>
            <AppenderRef ref="stdout"/>
        </Root>
        <Logger name="org.apache.kafka" level="WARN" />
    </Loggers>
</Configuration>
```

##### 6. 自定义输出
