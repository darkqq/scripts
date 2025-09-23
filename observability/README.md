### Turbo-detailed installation guide

#### Infra preparation

```bash
git clone https://github.com/darkqq/scripts.git
```
```bash
cd scripts/observability
```
```bash
docker compose up -d 
```

#### Delivery side
1. Get javaagent (https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases)
2. Update Dockerfile
    ```Dockerfile
    COPY opentelemetry-javaagent.jar /app/opentelemetry-javaagent.jar

    CMD ["java", "-javaagent:/app/opentelemetry-javaagent.jar", "-jar", "/app/app.jar"]
    ```
3. Add envs
   ```env
      OTEL_EXPORTER_OTLP_ENDPOINT=http://<collector-addr>:4318
      OTEL_LOGS_EXPORTER=otlp;OTEL_METRICS_EXPORTER=otlp
      OTEL_RESOURCE_ATTRIBUTES=deployment.environment\=<profile>
      OTEL_SERVICE_NAME=<unique-service-name>
      OTEL_TRACES_EXPORTER=otlp
   ```
4. Bingo bango bongo, bish bash bosh.

#### Application side (Maven/Spring)
Basic opentelemetry
```xml
<dependencies>
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-api</artifactId>
        <version>1.49.0</version> 
    </dependency>

    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk</artifactId>
        <version>1.49.0</version>
    </dependency>

    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-context</artifactId>
        <version>1.49.0</version>
    </dependency>
</dependencies>
```

Logback appender for otel collector
```xml
        <dependency>
            <groupId>io.opentelemetry.instrumentation</groupId>
            <artifactId>opentelemetry-logback-appender-1.0</artifactId>
            <version>2.20.0-alpha</version>
            <scope>runtime</scope>
        </dependency>
```

logback-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %5p ${PID:-} --- [%15.15t] %-40.40logger{39} : %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="OTEL" class="io.opentelemetry.instrumentation.logback.appender.v1_0.OpenTelemetryAppender">
        <captureExperimentalAttributes>true</captureExperimentalAttributes>
        <captureCodeAttributes>true</captureCodeAttributes>
        <captureMarkerAttribute>true</captureMarkerAttribute>
        <captureKeyValuePairAttributes>true</captureKeyValuePairAttributes>
        <captureLoggerContext>true</captureLoggerContext>
        <captureMdcAttributes>
            <pattern>.*</pattern>
        </captureMdcAttributes>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="OTEL" />
    </root>
</configuration>
```