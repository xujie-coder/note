logback是一个日志框架，通过他可以帮我们实现把日志输出到文件中，默认情况下，如果不使用logback，日志是不会输出到文件中，只会输出到控制台。



接入logback非常简单，首先需要依赖spring-boot-starter-logging，但是这个包我们一般不需要主动依赖，spring-boot-starter已经依赖他了。



接下来我们只需要在我们的resources目录下，增加配置文件 logback-spring.xml即可

## logback-spring.xml
```plain
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="app.name" source="spring.application.name"/>

    <property name="APP_NAME" value="${app.name}"/>
    <property name="LOG_PATH" value="${user.home}/${APP_NAME}/logs"/>
    <property name="LOG_FILE" value="${LOG_PATH}/application.log"/>
    <property name="FILE_LOG_PATTERN" value="%d %-5level [%thread %logger - %sensitive%n"/>

    <appender name="APPLICATION"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="APPLICATION"/>
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

配置说明：

1.  `<include resource="org/springframework/boot/logging/logback/defaults.xml"/>`

这一行包含了一个默认的Logback配置文件`org/springframework/boot/logging/logback/defaults.xml`，提供了Spring Boot项目中通用的日志配置设置。

2. `<springProperty scope="context" name="app.name" source="spring.application.name"/>`通过springProperty标签，可以从Spring Boot的上下文中引用属性（比如`spring.application.name`），并给这个属性命名为`app.name`。
3. property

```plain
<property name="APP_NAME" value="${app.name}"/>
<property name="LOG_PATH" value="${user.home}/${APP_NAME}/logs"/>
<property name="LOG_FILE" value="${LOG_PATH}/application.log"/>
<property name="FILE_LOG_PATTERN" value="%d %-5level [%thread %logger - %msg%n"/>
```

这些行定义了多个变量，包括

+ APP_NAME：使用了上面从Spring Boot上下文中获取的app.name属性。
+ LOG_PATH：定义了日志文件存储的路径。
+ LOG_FILE：定义了日志文件的具体名称，位于LOG_PATH指定的目录下。
+ FILE_LOG_PATTERN：定义了文件日志的格式，包括时间戳、日志级别、线程名、日志产生的类或接口名以及日志信息。
4. `application appender`这个部分配置了一个基于文件的日志处理器，包含如下设置：

```plain
 <appender name="APPLICATION"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
    </appender>
```

+ 日志文件的具体位置。
+ <font style="color:rgb(38, 38, 38);">日志的格式编码器，指定使用FILE_LOG_PATTERN变量。</font>
+ <font style="color:rgb(38, 38, 38);">一个基于大小和时间的滚动策略，配置包括日志文件滚动前的最大大小、保留旧日志文件的天数、文件大小上限以及总大小上限。</font>
5. `root`

```plain
<root level="INFO">
    <appender-ref ref="APPLICATION"/>
    <appender-ref ref="CONSOLE"/>
</root>
```

定义日志的全局级别（在这里是INFO），并关联了`application appender`，表明所有INFO级别以上的日志都将通过`APPLICATION appender`处理。

