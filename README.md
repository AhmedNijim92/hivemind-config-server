# HiveMind Config Server

> Centralized configuration management for all HiveMind microservices via Spring Cloud Config.

## Overview

The Config Server provides externalized configuration for the entire HiveMind platform. In development, it uses the `native` profile to serve configuration from the local filesystem. In production, it connects to a Git repository for versioned configuration. All microservices fetch their configuration from this server on startup. The Config Server registers itself with Eureka for discoverability.

## Features

- Centralized configuration for all services
- Native profile (local filesystem) for development
- Git-backed configuration for production
- Registered with Eureka for service discovery
- Environment-specific config profiles (dev, prod)

## Configuration

| Property | Description | Default |
|----------|-------------|---------|
| `server.port` | Config server port | `8888` |
| `spring.profiles.active` | Active profile (`native` or `git`) | `native` |
| `spring.cloud.config.server.native.search-locations` | Local config path | `classpath:/configs` |
| `spring.cloud.config.server.git.uri` | Git repo URI (prod) | — |
| `eureka.client.serviceUrl.defaultZone` | Eureka registry URL | `http://localhost:8761/eureka` |

## Tech Stack

- Java 17
- Spring Boot 3.x
- Spring Cloud Config Server
- Eureka Client
- Maven

## Docker

```
Port: 8888
Base image: eclipse-temurin:17-jre-alpine
JVM flags: -XX:MaxRAMPercentage=75.0 -XX:+UseG1GC
User: non-root (spring)
```

## CI/CD

- **Build**: Maven `clean package` with JDK 17 (Temurin)
- **Test**: Unit tests run during build phase
- **Docker**: Build and push to Docker Hub on `main` branch merge
- **Security**: Trivy vulnerability scan (CRITICAL, HIGH) on built image
