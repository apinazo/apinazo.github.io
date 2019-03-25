# Distributed session with Spring Session and Redis

## Overview

When there are several instances of the same application under a load balancer, all of them must share the session. But how to do it the microservices way?

## Redis

Redis is a NoSQL simple in-memory database storing key-value pairs. It is ideal for what it's needed here: having a persistent store shared by all microservice instances.

The simplest way to get a Redis instance up and running is using its official image in [Docher Hub](https://hub.docker.com/_/redis).

Just execute this:

```bash
docker run --name local-redis -d redis
```

This will pull the Redis image and create a container named `local-redis` with the default configuration:

* Port: 6379
* Host: 127.0.0.1
* No password.
* Non-persistent in-memory storage.

## Spring Session

### Adding Spring Session to the application

Although sometimes the official documentation is not very clear, adding Spring Session to the project is very straight-forward. 

First of all, add the dependencies:

```gradle
compile 'org.springframework.boot:spring-boot-starter-data-redis'
compile 'org.springframework.session:spring-session-data-redis'
```
You need both of them. With the starter you can take advantage of autoconfiguration. A bean is created with the driver needed to connect to Redis.

Sometime ago there was the need of using `@EnableRedisHttpSession` in some configuration bean but there is no more. Spring Boot will enable it if the Redis dependencies are in the classpath.

Then is the turn to define where is the Redis server located and set it to store the session.

Just add this to your application.yml:

```yaml
spring:
  session:
    store:
      store-type: redis
  redis:
    host: 127.0.0.1
    password: # There is no password by default.
    port: 6379
```

### Using Spring Session

Spring Session adds a filter that:
 
* Intercepts all requests.
* Extracts the session data.
* Persists it in Redis.

Using this is totally transparent. The apps should continue using the classic `HttpSession` with no problems at all. Magically all data read and written in the HttpSession will be available to all apps using the same Redis instance. 

Let's say you have two microservices: A and B. 

In A you can create a @RestController with this endpoint:

```java
@RestController
public class SessionController {

    @GetMapping("/session")
    private void doSomething(HttpSession session) {

        session.setAttribute("x", "y");
    }
}
```

Then in B you can have this one:

```java
@RestController
public class SessionController {

    @GetMapping("/session")
    private String doSomething(HttpSession session) {

        return session.getAttribute("x");
    }
}
```

When you `GET http://A/session`, the key-value pair `("x", "y")` is stored in Redis - distributed. 

Then when you `GET http://B/session`, the right value for `"x"` is returned, retrieved from Redis.

## References

* [Spring Session reference documentation](https://docs.spring.io/spring-session/docs/current/reference/html5/).
* [Introduction to Spring Session Redis in Cloud-Native Environments](https://medium.com/@odedia/spring-session-redis-part-i-overview-a5f6c7446c8b).
* [Guide to Spring Session](https://www.baeldung.com/spring-session).