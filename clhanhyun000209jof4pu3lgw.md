---
title: "Adding Correlation IDs to Easily Track Down Errors"
datePublished: Wed Jul 13 2022 20:20:35 GMT+0000 (Coordinated Universal Time)
cuid: clhanhyun000209jof4pu3lgw
slug: correlation-ids
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683231937187/c6c3977b-ce4c-42f6-ab20-12684edd23f5.png
tags: java, error-handling, springboot, error-tracking, spring-cloud

---

In this post, we'll look at how we can use Spring Cloud Sleuth to add trace IDs to our application to track down calls and exceptions.

# Why Would We Want To Use Correlation IDs?

If you've ever worked on an application that has numerous concurrent calls happening you'll know that it's hard to figure out from the logs which message belongs to which call.  
Likewise, if you get an exception in your application and you want to find the root cause it can be hard to know where to look in the logs.

We can use correlation IDs to correlate our log messages and exception stack traces to specific calls in our web application. As well, if we include correlation IDs in our error responses sent back to clients we can use those IDs to trace back to the cause of the error.

One way we can easily add correlation IDs to our application is Spring Cloud Sleuth.

# Spring Cloud Sleuth

[Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) is part of the wider Spring Cloud library and provides the configuration required for distributed tracing.  
Distributed tracing is a great tool for tracing your calls through various microservices, however, the part that we're interested in here is this:

> Adds trace and span ids to the Slf4J MDC, so you can extract all the logs from a given trace or span in a log aggregator.

The trace ID is what we will use as our correlation ID.

By default, Spring Cloud Sleuth will add the trace ID as a header when calling other services, which is how the distributed trace is tracked. However, it does not automatically add the trace id to responses from our application. This can easily be added with a filter, which we'll show below.

# Setting Up A Spring Boot App with Sleuth

## Adding a Simple Controller and Service

Let's start by creating a Spring Boot application with a simple controller and service.

**Pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>Blogging-correlation-ids</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.1</version>
        <relativePath/>
    </parent>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <java.version>17</java.version>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

</project>
```

**StackToBasicsApplication**

```java
package com.stacktobasics;

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
package com.stacktobasics;

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
}
```

**HelloWorldService**

```java
package com.stacktobasics;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class HelloWorldService {

    public String sayHello() {
        log.info("Returning hello from service");
        return "hello";
    }
}
```

When we launch this application and call `GET /hello` we see the following:

```plaintext
2022-07-13 18:30:39.545  INFO [,10f5c8744d01e2d4,10f5c8744d01e2d4] 19460 --- [nio-8080-exec-6] com.stacktobasics.HelloWorldController   : Someone called the /hello endpoint
2022-07-13 18:30:39.545  INFO [,10f5c8744d01e2d4,10f5c8744d01e2d4] 19460 --- [nio-8080-exec-6] com.stacktobasics.HelloWorldService      : Returning hello from service
```

Notice the `10f5c8744d01e2d4` ID in our logs. This is repeated twice, once for the span ID and once for the trace ID. In this case they are the same value. We can see that the ID is injected into both our controller and our service logs. This means we can trace the call through whatever classes are used!

If we call `GET /hello` again we can see we get a different ID:

```plaintext
2022-07-13 18:32:51.910  INFO [,4a6379092ac32a93,4a6379092ac32a93] 19460 --- [nio-8080-exec-7] com.stacktobasics.HelloWorldController   : Someone called the /hello endpoint
2022-07-13 18:32:51.911  INFO [,4a6379092ac32a93,4a6379092ac32a93] 19460 --- [nio-8080-exec-7] com.stacktobasics.HelloWorldService      : Returning hello from service
```

## Adding the Correlation ID to Exception Responses

It would be useful to be able to return the trace ID when sending back error responses. That way if a user of the application tells us about an error they are experiencing, we can ask them for the response they received and look for the trace ID in our logs.

**HttpExceptionHandler**

```java
package com.stacktobasics;

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
package com.stacktobasics;

import org.springframework.cloud.sleuth.Span;
import org.springframework.cloud.sleuth.TraceContext;
import org.springframework.cloud.sleuth.Tracer;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class CorrelationIDHandler {
    private final Tracer tracer;

    public CorrelationIDHandler(Tracer tracer) {
        this.tracer = tracer;
    }

    public String getCorrelationId() {
        return Optional.of(tracer).map(Tracer::currentSpan).map(Span::context).map(TraceContext::traceId).orElse("");
    }
}
```

**ExceptionResponse**

```java
package com.stacktobasics;

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

Let's add a method to our controller and service to trigger an exception.

**HelloWorldController**

```java
...

@RestController
@RequestMapping
@Slf4j
public class HelloWorldController {
    
    ...
    
    @GetMapping("/bad-call")
    public void badCall() {
        log.info("Someone called the /bad-call endpoint");
        helloWorldService.fakeBadCall();
    }
}
```

\*\*HelloWorldService

```java
...

@Component
@Slf4j
public class HelloWorldService {

    ...

    public void fakeBadCall() {
        log.info("About to throw IllegalArgumentException...");
        throw new IllegalArgumentException("Exception from Hello World Service");
    }
}
```

Things to notice are:

* `CorrelationIDHandler` uses the injected `Tracer` object to get a copy of the trace ID.
    
* `ExceptionResponse` is the response object we return to users when we encounter an exception. It contains the trace ID as a correlation ID field.
    
* We have an exception handler `HttpExceptionHandler` which handles uncaught exceptions and provides and appropriate response to the user. Here we are only handling `IllegalArgumentException`.
    

When we call `GET /bad-call` we see the following in our logs:

```plaintext
2022-07-13 18:47:20.751  INFO [,b490ea4b7230260a,b490ea4b7230260a] 14716 --- [nio-8080-exec-5] com.stacktobasics.HelloWorldController   : Someone called the /bad-call endpoint
2022-07-13 18:47:20.751  INFO [,b490ea4b7230260a,b490ea4b7230260a] 14716 --- [nio-8080-exec-5] com.stacktobasics.HelloWorldService      : About to throw IllegalArgumentException...
2022-07-13 18:47:20.755  WARN [,b490ea4b7230260a,b490ea4b7230260a] 14716 --- [nio-8080-exec-5] com.stacktobasics.HttpExceptionHandler   : Exception from Hello World Service

java.lang.IllegalArgumentException: Exception from Hello World Service
	at com.stacktobasics.HelloWorldService.fakeBadCall(HelloWorldService.java:17) ~[classes/:na]
	at com.stacktobasics.HelloWorldController.badCall(HelloWorldController.java:29) ~[classes/:na]
	...
```

We also see the following response:

```json
{
"correlationId": "b490ea4b7230260a",
"status": "BAD_REQUEST",
"datetime": "2022-07-13 18:47:20",
"error": "Exception from Hello World Service"
}
```

We can see that the correlationID in the response matches the trace ID in the logs.

## Adding the Trace ID to the Header

Having the trace ID is great for tracing calls through our logs. However, if we want to track the trace ID in another application that doesn't use Spring Sleuth (such as a Node.JS application) then we need to find a way of exposing the trace ID to callers.  
This can easily be done by adding a filter to our application which adds the trace ID as a header.

**TraceFilter**

```java
@Component
public class TraceFilter implements Filter {

    private static final String TRACE_ID_HEADER_NAME = "X-B3-TraceId";
    private final CorrelationIDHandler correlationIDHandler;

    public TraceFilter(CorrelationIDHandler correlationIDHandler) {
        this.correlationIDHandler = correlationIDHandler;
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletResponse response = (HttpServletResponse) res;
        if (!response.getHeaderNames().contains(TRACE_ID_HEADER_NAME)) {
            var id = correlationIDHandler.getCorrelationId();
            if (!id.isEmpty()) response.setHeader(TRACE_ID_HEADER_NAME, id);
        }
        chain.doFilter(req, res);
    }
}
```

This filter adds the `X-B3-TraceId` header to any response that does not already contain a trace ID. The `X-B3-TraceId` header is what Sleuth adds when sending requests to other services to keep track of the trace through microservices, so we've used the same name to be consistent.

If we run our application again and call `GET /hello` we will find the trace ID has been added to the response headers:

```plaintext
HTTP/1.1 200
X-B3-TraceId: c9ac548611166be7
```

# Conclusion

In this post, we've learned how to use Spring Cloud Sleuth to trace calls through our application. We've also learned how to include those trace IDs in error responses sent back to the client, as well as including the trace ID as an HTTP header to propagate it through other clients.

Till next time!