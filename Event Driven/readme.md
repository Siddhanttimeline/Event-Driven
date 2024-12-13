# **Real-Time Configuration Updates in Microservices with Spring Cloud and RabbitMQ**

*Author: Siddhant Tripathi*  
*Date: Dec 13, 2024*  
*Reading Time: 7 min*

---

## **Table of Contents**

1. [Config Server](#config-server)
2. [Connect Config Server with Services](#connect-config-server-with-services)
3. [Spring Cloud Bus](#spring-cloud-bus)
4. [Create a Webhook in the GitHub Repository using Hookdeck](#create-a-webhook-in-the-github-repository-using-hookdeck)
5. [Summary](#summary)
6. [Resources](#resources)

---

Spring Cloud provides tools for developers to quickly implement common patterns in distributed systems, such as:

- Configuration management
- Service discovery
- Circuit breakers
- Intelligent routing
- Micro-proxy
- Control bus
- Short-lived microservices
- Contract testing

This guide demonstrates how to integrate **Spring Cloud Config Server** with **Spring Cloud Bus** to manage configurations for microservices efficiently. We centralize configuration files in a **GitHub repository** and use **RabbitMQ** to broadcast configuration updates to all microservices.

---

## **Config Server**

### **Dependencies**

Add these dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### **Enable Config Server**

In your `ConfigServer` main class, add the `@EnableConfigServer` annotation:

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer.class, args);
    }
}
```

### **GitHub Repository Configuration**

Add the following to `application.yml`:

```yaml
spring:
  application:
    name: "configserver"
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: "provide the uri to your github repo"
          default-label: master
          timeout: 5 
          clone-on-start: true
          force-pull: true

server:
  port: 8071
```

### **Access Configurations**

Request URL format:

```
http://<config-server-host>:<port>/<application-name>/<profile>
```

Example:

```
http://localhost:8071/dataCatalog/prod
```

---

## **Connect Config Server with Services**

### **Dependencies**

Add the following to your microservices' `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### **Spring Cloud Version**

Add the Spring Cloud version property:

```xml
<properties>
    <spring-cloud.version>2023.0.0</spring-cloud.version>
</properties>
```

### **Dependency Management**

Add this section above your `<build>` section:

```xml
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
```

### **Configuration in `application.yml`**

```yaml
config:
  import: "optional:configserver:http://localhost:8071/"

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

---

## **Spring Cloud Bus**

### **RabbitMQ Setup**

Run RabbitMQ via Docker:

```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.13-management
```

### **Dependencies**

Add these dependencies to the `pom.xml` of the Config Server and microservices:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### **RabbitMQ Configuration**

Add RabbitMQ connection details to `application.yml`:

```yaml
rabbitmq:
  host: "localhost"
  port: 5672 
  username: "guest" 
  password: "guest"

management:
  endpoints:
    web:
      exposure:
        include: "busrefresh"
```

### **Invoke Bus Refresh**

Use this endpoint to refresh configurations across all microservices:

```
http://<microservice-host>:<port>/actuator/busrefresh
```

---

## **Create a Webhook in the GitHub Repository using Hookdeck**

### **Config Server Dependency**

Add this dependency to the Config Server's `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-monitor</artifactId>
</dependency>
```

### **Webhook Setup**

1. Install **Hookdeck**:

    ```bash
    hookdeck login
    hookdeck listen [config server port]
    ```

2. Configure the webhook in your GitHub repository to point to the Hookdeck public URL.

---

## **Summary**

This guide covers:

- Centralized configuration management with Spring Cloud Config Server.
- Broadcasting configuration updates using Spring Cloud Bus and RabbitMQ.
- Automating updates with GitHub webhooks and Hookdeck.

---

## **Resources**

- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
- [RabbitMQ Documentation](https://www.rabbitmq.com/)
- [Hookdeck](https://hookdeck.com/)

