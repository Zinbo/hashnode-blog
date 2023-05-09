---
title: "Adding Correlation IDs to Easily Track Down Errors - Spring Boot 3 Edition"
datePublished: Tue May 09 2023 20:10:28 GMT+0000 (Coordinated Universal Time)
cuid: clhgphuwj000609lfgdup1dip
slug: adding-correlation-ids-to-easily-track-down-errors-spring-boot-3-edition
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683662936268/69a30362-7d90-4051-8eca-3b6c5566248c.png
tags: spring, error-handling, springboot, tracing, springboot3

---

You may have read my previous post, [Adding Correlation IDs to Easily Track Down Errors](https://stacktobasics.com/correlation-ids), which used Spring Cloud Sleuth and Spring Boot 2.X to add correlation IDs to our logs and error responses.

In this post, I'll go over how we can do the same in Spring Boot 3. If you'd like more background on why correlation IDs are useful and why you should use them, please read my previous post [here](https://stacktobasics.com/correlation-ids).

You can find all of the code referenced in this post [here](https://github.com/Zinbo/blog-examples/tree/master/correlation-ids-spring-boot-3).

# What's Different In Spring Boot 3?

One of the big changes in Spring Boot 3 was the switch over to using [Micrometer](https://micrometer.io/docs/tracing) for tracing. With that change, Spring Cloud Sleuth has been discontinued and is incompatible with Spring Boot 3.

Thankfully the switch over to Micrometer is easy!

In the rest of this post we will go over the examples given in the previous blog [here](https://stacktobasics.com/correlation-ids), but using Micrometer.

# **Setting Up A Spring Boot App with Micrometer**

## **Adding a Simple Controller and Service**

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.stacktobasics</groupId>
    <artifactId>correlation-ids-spring-boot-3</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>correlation-ids-spring-boot-3</name>
    <description>Showcase correlation ids in Spring Boot 3</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bridge-brave</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**StackToBasicsApplication**

```java
package com.stacktobasics.correlationidsspringboot3;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StackToBasicsApplication {

	public static void main(String[] args) {
		SpringApplication.run(StackToBasicsApplication.class, args);
	}

}
```

**HelloWorldController**

```java
package com.stacktobasics.correlationidsspringboot3.api;

import com.stacktobasics.correlationidsspringboot3.service.HelloWorldService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
@RequestMapping
@Slf4j
public class HelloWorldController {

    private final HelloWorldService helloWorldService;

    public HelloWorldController(HelloWorldService helloWorldService) {
        this.helloWorldService = helloWorldService;
    }

    @GetMapping("/hello")
    public ResponseEntity<String> sayHello() {
        log.info("Someone called the /hello endpoint");
        return ResponseEntity.ok(helloWorldService.sayHello());
    }

    @GetMapping("/bad-call")
    public void badCall() {
        log.info("Someone called the /bad-call endpoint");
        helloWorldService.fakeBadCall();
    }
}
```

**HelloWorldService**

```java
package com.stacktobasics.correlationidsspringboot3.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class HelloWorldService {

    public String sayHello() {
        log.info("Returning hello from service");
        return "hello";
    }

    public void fakeBadCall() {
        log.info("About to throw IllegalArgumentException...");
        throw new IllegalArgumentException("Exception from Hello World Service");
    }
}
```

**application.yml**

```yaml
spring:
  application:
    name: correlation-ids-spring-boot-3

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

Note that, unlike with Sleuth, we need to manually set out logging pattern to include our traceId and spanId.

When we launch this application and call `GET /hello` we see the following:

```bash
2023-05-09T19:30:49.423+01:00  INFO [correlation-ids-spring-boot-3,645a91593eebd66457eb88b96d17bcc9,57eb88b96d17bcc9] 41420 --- [nio-8080-exec-2] c.s.c.api.HelloWorldController           : Someone called the /hello endpoint
2023-05-09T19:30:49.423+01:00  INFO [correlation-ids-spring-boot-3,645a91593eebd66457eb88b96d17bcc9,57eb88b96d17bcc9] 41420 --- [nio-8080-exec-2] c.s.c.service.HelloWorldService          : Returning hello from service
```

We can see our traceId and spanId being printed, which is the same as what we saw for our Spring Boot 2.X application with Spring Sleuth.

## **Adding the Correlation ID to Exception Responses**

**HttpExceptionHandler**

```java
package com.stacktobasics.correlationidsspringboot3.infra.exceptionhandler;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
@Slf4j
public class HttpExceptionHandler {

    private final CorrelationIDHandler correlationIDHandler;

    public HttpExceptionHandler(CorrelationIDHandler correlationIDHandler) {
        this.correlationIDHandler = correlationIDHandler;
    }

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<ExceptionResponse> handleIllegalArgumentException(IllegalArgumentException exception) {
        log.warn(exception.getMessage(), exception);
        var exceptionResponse = new ExceptionResponse(HttpStatus.BAD_REQUEST, exception, correlationIDHandler.getCorrelationId());
        return new ResponseEntity<>(exceptionResponse, exceptionResponse.getStatus());
    }
}
```

**CorrelationIDHandler**

```java
package com.stacktobasics.correlationidsspringboot3.infra.exceptionhandler;

import io.micrometer.tracing.CurrentTraceContext;
import io.micrometer.tracing.TraceContext;
import io.micrometer.tracing.Tracer;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class CorrelationIDHandler {

    private final Tracer tracer;

    public CorrelationIDHandler(Tracer tracer) {
        this.tracer = tracer;
    }

    public String getCorrelationId() {
        return Optional.of(tracer).map(Tracer::currentTraceContext).map(CurrentTraceContext::context).map(TraceContext::traceId).orElse("");
    }
}
```

Note here that we now use io.micrometer.tracing, rather than org.springframework.cloud.sleuth.

**ExceptionResponse**

```java
package com.stacktobasics.wownamechecker.infra.exceptionhandler;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Getter;
import lombok.NonNull;
import org.springframework.http.HttpStatus;

import java.time.LocalDateTime;

@Getter
public class ExceptionResponse {
    private final String correlationId;
    private final HttpStatus status;
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
    private final LocalDateTime datetime;
    private final String error;

    ExceptionResponse(@NonNull HttpStatus status, @NonNull Exception ex, String correlationId) {
        datetime = LocalDateTime.now();
        this.status = status;
        this.error = ex.getMessage();
        this.correlationId = correlationId;
    }
}
```

When we call `GET /bad-call` we see the following in our logs:

```bash
2023-05-09T19:35:00.844+01:00  INFO [correlation-ids-spring-boot-3,645a925467299fb619c6ac98ab7f704b,19c6ac98ab7f704b] 41420 --- [nio-8080-exec-7] c.s.c.api.HelloWorldController           : Someone called the /bad-call endpoint
2023-05-09T19:35:00.844+01:00  INFO [correlation-ids-spring-boot-3,645a925467299fb619c6ac98ab7f704b,19c6ac98ab7f704b] 41420 --- [nio-8080-exec-7] c.s.c.service.HelloWorldService          : About to throw IllegalArgumentException...
2023-05-09T19:35:00.845+01:00  WARN [correlation-ids-spring-boot-3,645a925467299fb619c6ac98ab7f704b,19c6ac98ab7f704b] 41420 --- [nio-8080-exec-7] c.s.c.i.e.HttpExceptionHandler           : Exception from Hello World Service

java.lang.IllegalArgumentException: Exception from Hello World Service
	at com.stacktobasics.correlationidsspringboot3.service.HelloWorldService.fakeBadCall(HelloWorldService.java:17)
```

We get the following response:

```json
{
    "correlationId": "645a925467299fb619c6ac98ab7f704b",
    "status": "BAD_REQUEST",
    "datetime": "2023-05-09 19:35:00",
    "error": "Exception from Hello World Service"
}
```

As expected, the `correlationId` field in the response matches the `traceId` in the logs.

# Testing Trace Propagation

Before we go any further, let's test that our trace IDs are being propagated to downstream requests. In case you're new to tracing, this is a key aspect as to how tracing works. The trace ID gets propagated to any requests that you make, meaning that you can trace the whole request across multiple services.

Trace IDs are propagated by using an HTTP header. By default, in Spring Sleuth this was the `X-B3-TraceId` header. Spring Boot 3 with Micrometer uses the `traceparent` header by default.

Let's add a new endpoint to our application and call this endpoint from `HelloWorldService` to check that the trace ID is being propagated properly.

**OtherController**

```java
package com.stacktobasics.correlationidsspringboot3.api;

import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
@Slf4j
public class OtherController {
    @GetMapping("/other")
    public ResponseEntity<String> sayHello(HttpServletRequest request) {
        log.info("traceparent: {}", request.getHeader("traceparent"));
        log.info("Someone called the /other endpoint");
        return ResponseEntity.ok("other");
    }
}
```

HelloWorldService

```java
package com.stacktobasics.correlationidsspringboot3.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
@Slf4j
public class HelloWorldService {

    private final RestTemplate restTemplate;

    public HelloWorldService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }


    public String sayHello() {
        log.info("Returning hello from service");
        restTemplate.getForEntity("http://localhost:8080/other", String.class);
        return "hello";
    }

    public void fakeBadCall() {
        log.info("About to throw IllegalArgumentException...");
        throw new IllegalArgumentException("Exception from Hello World Service");
    }
}
```

We also need to add a `RestTemplate` bean to ensure that the trace ID gets propagated automatically:

```java
package com.stacktobasics.correlationidsspringboot3.infra.web;

import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfiguration {
    
    @Bean
    RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder.build();
    }
}
```

When we call `GET /hello` we see the following in our logs:

```bash
2023-05-09T19:44:17.569+01:00  INFO [correlation-ids-spring-boot-3,645a94813f591c8b1e2d81517af62961,1e2d81517af62961] 12636 --- [nio-8080-exec-2] c.s.c.api.HelloWorldController           : Someone called the /hello endpoint
2023-05-09T19:44:17.569+01:00  INFO [correlation-ids-spring-boot-3,645a94813f591c8b1e2d81517af62961,1e2d81517af62961] 12636 --- [nio-8080-exec-2] c.s.c.service.HelloWorldService          : Returning hello from service
2023-05-09T19:44:17.591+01:00  INFO [correlation-ids-spring-boot-3,645a94813f591c8b1e2d81517af62961,c85d5409fb9a2748] 12636 --- [nio-8080-exec-5] c.s.c.api.OtherController                : traceparent: 00-645a94813f591c8b1e2d81517af62961-5225ae36a413cb58-00
2023-05-09T19:44:17.593+01:00  INFO [correlation-ids-spring-boot-3,645a94813f591c8b1e2d81517af62961,c85d5409fb9a2748] 12636 --- [nio-8080-exec-5] c.s.c.api.OtherController                : Someone called the /other endpoint
```

We can see that `traceparent` contains 4 values, delimited by `-`. These values are:

* `version`
    
* `trace-id`
    
* `parent-id`
    
* `trace-flags`
    

We can see that the second value, which is the trace ID, matches the traceID that we see in our logs.

You can read more about the `traceparent` header [here](https://www.w3.org/TR/trace-context/#traceparent-header).

Note that this is different from our previous implementation using Spring Boot 2.X and Sleuth, which used the `X-B3-TraceId` header, which only contained the trace ID.

### Using Old Headers

If you would prefer to use the older `X-B3-*` headers, you can do this by simply adding the following bean:

```java
    @Bean
    public Tracing braveTracing() {
        return Tracing.newBuilder()
                .propagationFactory(B3Propagation.newFactoryBuilder().injectFormat(B3Propagation.Format.MULTI).build())
                .build();
    }
```

Your requests will now contain the headers `x-b3-traceid, x-b3-spanid, x-b3-parentspanid, x-b3-sampled`, rather than `traceparent`.

## **Adding the Trace ID to the Response**

**TraceFilter**

```java
package com.stacktobasics.correlationidsspringboot3.infra.web;

import io.micrometer.tracing.CurrentTraceContext;
import io.micrometer.tracing.Span;
import io.micrometer.tracing.Tracer;
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Optional;

@Component
public class TraceFilter implements Filter {

    private static final String TRACE_ID_HEADER_NAME = "traceparent";
    public static final String DEFAULT = "00";
    private final Tracer tracer;

    public TraceFilter(Tracer tracer) {
        this.tracer = tracer;
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletResponse response = (HttpServletResponse) res;
        if (!response.getHeaderNames().contains(TRACE_ID_HEADER_NAME)) {
            if(Optional.of(tracer).map(Tracer::currentTraceContext).map(CurrentTraceContext::context).isEmpty()) {
                chain.doFilter(req, res);
                return;
            }
            var context = tracer.currentTraceContext().context();
            var traceId = context.traceId();
            var parentId = context.spanId();
            var traceparent = DEFAULT + "-" + traceId + "-" + parentId + "-" + DEFAULT;
            response.setHeader(TRACE_ID_HEADER_NAME, traceparent);
        }
        chain.doFilter(req, res);
    }
}
```

Note that this is a bit different from our `TraceFilter` implementation using Spring Sleuth. Here we're building the whole `traceparent` header rather than just the trace ID.

When we now call `GET /hello` we'll see that the `traceparent` header is returned in the response:

```bash
traceparent: 00-645aa0cb147f7334b00a46bd38797e6c-b00a46bd38797e6c-00
```

# **Conclusion**

In this post, we've learned how to use Spring Boot 3 with Micrometer, which has superseded Spring Cloud Sleuth, to trace calls through our application. We've also learned how to include those trace IDs in error responses sent back to the client, as well as including the trace ID as an HTTP header to propagate it through other clients.

You can find all of the code referenced in this post [here](https://github.com/Zinbo/blog-examples/tree/master/correlation-ids-spring-boot-3).

Till next time!