# Config Server

Centralized configuration server for the HiveMind microservices platform. Provides externalized configuration for all services via Spring Cloud Config.

## Details

| Property | Value |
|----------|-------|
| **Port** | `8888` |
| **Database** | None |
| **Role** | Centralized Configuration |

## Build & Run

```bash
# Build
mvn clean package

# Run
java -jar target/*.jar

# Docker
docker build -t hivemind/config-server .
```

## Links

- [Main Repository](https://github.com/AhmedNijim92/hivemind-backend)
