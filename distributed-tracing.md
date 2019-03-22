# Overview
Distributed tracing is the technique for following a request from a client through all the microservices architecture back to the client.

# Unifiying request logs with Sleuth

Sleuth will give a unique ID to the request, shared by every service implied in it, just as a breadcrumb. Each service would have its own ID so creting a hierarchy.

First of all, add the Sleuth library to all services using distributed tracing. In cloudsample, this is set in the parent `pom.xml` of the project.

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```
Then, configure Sleuth and the Zipking URL. This is done in the common services configuration in the Config Server.

```yaml
spring:
  sleuth:
    traceId128: true
    sampler:
      probability: 1.0
  zipkin:
    baseUrl: http://localhost:9411/
```

A better practice is using discovering by using the Zipkin service name in the URL:
```yaml
baseUrl: http://zipkinserver:9411/
```

**NOTE:** All clients must declare the `baseUrl`. Failing to do so will prevent the service to show in Zipkin.

Properties:
- probability: Percentage of requests to track, from 0 to 1. Set 1.0 to track them all.


# Timing requests with Zipkin

Zipkin is a tool for timing distributed requests, using Sleuth correlation IDs to identify them. 

**NOTE:** Zipkin is not meant to be a logging system for the services - it is all just about timing requests and tracing intercommunication between services. 

Set up Zipkin following the [quickstart](https://zipkin.io/pages/quickstart.html).

Then the app must be started:

```java
java -jar zipkin-server-2.11.12-exec
```

When the other services are up and running they will start sending logs to Zipkin. You can view this logs in the Zipkin default URL, at [http://localhost:9411/zipkin/](http://localhost:9411/zipkin/).

**NOTE:** Services will not start to show in the select until there are requests to them.


# Configuring Zipkin as a microservice

Sometimes it can be a better approach serving it as a microservice - specially if there is interest in customizing it.

**WARNING:** '@EnableZipkinServer' is to be [deprecated](https://github.com/openzipkin/zipkin/issues/2043). It's encouraged to use the binary distribution or the Docker version.

```xml
<!-- Zipkin server -->
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-server</artifactId>
    <version>2.12.0</version>
</dependency>

<!-- Zipkin UI -->
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-ui</artifactId>
    <scope>runtime</scope>
    <version>2.12.0</version>
</dependency>
```

If there is any conflict with the loggers just add this dependency:

```xml
<!-- Needed here to avoid dependency conflict
     with loggers. -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

To put Zipkin to run is as easy as adding the `@EnableZipkinServer` configuration:

```java
@EnableZipkinServer
@SpringBootApplication
public class ZipkinApp {

    public static void main(String[] args) {
        SpringApplication.run(ZipkinApp.class, args);
    }

}
```


# Business logic distributed logs

Before we learnt how to measere service response times taking advantage of distributed tracing. But what if we want to see the actual logs of a request implying several services?

The solution is to store the logs in a time series database and then search for them by its correlation identifier which is genereated - as we already know - by Sleuth.

# Storing distributed logs

TODO. Kafka, ELK...


# Searching distributed logs

TODO: Kibana, Grafana...
