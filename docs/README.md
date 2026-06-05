# Config Server

> HiveMind Centralized Configuration Server

## Overview

The config-server provides centralized externalized configuration for all HiveMind microservices using Spring Cloud Config. In development, it uses a native (filesystem) profile to serve configs without requiring a Git repository. In production, it can switch to a Git-backed store.

## Service Info

| Property | Value |
|----------|-------|
| Port | 8888 |
| Service Name | `config-server` |
| Spring Boot | 3.3.5 |
| Spring Cloud | 2023.0.3 |
| Java | 17 |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│               Config Server (:8888)                   │
│                                                      │
│  Backend Profiles:                                   │
│  ┌─────────────────────────────────────────────┐    │
│  │ native (dev): classpath:/config-repo        │    │
│  │ git (prod):   https://github.com/.../config │    │
│  └─────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
         │
         ▼
  Microservices fetch config on startup:
  spring.config.import: "optional:configserver:"
```

## Configuration

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  profiles:
    active: native  # filesystem-based, no Git in dev
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config-repo
        git:
          uri: ${CONFIG_GIT_URI:https://github.com/your-org/hivemind-config}
          default-label: main
          clone-on-start: false

eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_SERVER:http://localhost:8761/eureka}
  instance:
    prefer-ip-address: true
```

## Configuration Profiles

### Native (Development)

- Configs stored in `src/main/resources/config-repo/`
- File naming: `{service-name}.yml` or `{service-name}-{profile}.yml`
- No Git repository required
- Changes require server restart

### Git (Production)

- Configs stored in a Git repository
- Supports branch-based profiles via `default-label`
- Auto-refreshes on Git push (with webhook)
- Encrypted secrets via `{cipher}` prefix

## How Services Connect

Each microservice includes:
```yaml
spring:
  config:
    import: "optional:configserver:"
```

The `optional:` prefix means services start normally even if config-server is unavailable — they fall back to their local `application.yml`.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| CONFIG_GIT_URI | https://github.com/... | Git repo URL (prod) |
| EUREKA_SERVER | http://localhost:8761/eureka | Eureka URL |

## Dependencies

- spring-cloud-config-server
- spring-cloud-starter-netflix-eureka-client
- spring-boot-starter-actuator

## Running Locally

```bash
cd microservices/config-server
mvn spring-boot:run
```

### Verify

```bash
curl http://localhost:8888/auth-service/default
```

Should return the configuration properties for auth-service.

## Usage in Development

Currently, the config-server is **not required** for local development. Each service contains its own `application.yml` with all necessary configuration and environment variable substitution. The config-server becomes useful when:

- You want to change config without redeploying services
- You need environment-specific overrides in staging/production
- You want encrypted secrets managed centrally

## Production Considerations

- Switch to Git backend for version-controlled config
- Enable encryption for secrets (`encrypt.key` property)
- Add Spring Security to protect the config endpoint
- Use webhooks + Spring Cloud Bus for config refresh without restart
