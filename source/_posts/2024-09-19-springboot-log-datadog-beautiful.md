---
title: 美化Spring Boot在datadog上的日志
date: 2024-09-19 15:57:11
tags:
  - SpringBoot
---

我们目前现有的SpringBoot项目所有日志都是用logback框架打印，发送到datadog上，目前发现一个问题是默认的logback打印的日志在datadog上非常不美观，类似下面这种

<!--more-->
![未美化的日志](img1.webp)
所有日志都是INFO，没有不同日志等级显示不同颜色的区分。异常栈没有折叠，占用一大片位置，非常难看。而且也没法用datadog的查询attribute功能查traceId。原因是datadog读取日志是按照json格式读取的，而默认的logback输出就是个字符串，于是datadog就直接把整个日志字符串展示出来了。

经过一番搜索，发现有个库叫[logstash-logback-encoder](https://github.com/logfellow/logstash-logback-encoder)，作用就是把logback输出的日志格式化成logstash的格式，再发送给datadog，具体配置如下

```
<configuration>
    <property name="LOG_HOME" value="logs"/>
    <property name="CONSOLE_LOG_PATTERN" value="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] [%X{traceId}] %-5level %logger{36} - %msg%n"/>

    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <includeCallerData>true</includeCallerData>
            <providers>
                <pattern>
                    <pattern>
                        {
                            "timestamp": "%date{\"yyyy-MM-dd HH:mm:ss.SSS\", UTC}",
                            "level": "%level",
                            "logger": "%logger",
                            "className": "%class",
                            "lineNumber": "%line",
                            "message": "[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] [%X{traceId}] %-5level %logger{36} - %msg%n",
                            "traceId": "%mdc{traceId}"
                        }
                    </pattern>
                </pattern>
                <stackTrace>
                    <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">
                        <maxDepthPerThrowable>30</maxDepthPerThrowable>
                        <maxLength>2048</maxLength>
                        <shortenedClassNameLength>20</shortenedClassNameLength>
                        <rootCauseFirst>true</rootCauseFirst>
                    </throwableConverter>
                </stackTrace>
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>

```
配完后的datadog日志长这样，可以看到有颜色筛选，异常栈都折叠在一个日志里了，看起来清爽多了。

![美化后的日志](img2.webp)

