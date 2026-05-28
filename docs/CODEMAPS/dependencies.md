<!-- Generated: 2026-05-28 | Files scanned: 266 classes / 47 packages | Token estimate: ~400 -->

# Dependencies

## Runtime Libraries

| Library                                       | Version       | Role                                                     |
|-----------------------------------------------|---------------|----------------------------------------------------------|
| `io.netty:netty-all`                          | 4.1.119.Final | TCP server, channel pipeline, ByteBuf                    |
| `com.google.inject:guice`                     | 7.0.0         | Dependency injection container (6 modules)               |
| `com.google.guava:guava`                      | 32.0.1-jre    | Utilities; pins version to fix CVE-2023-2976             |
| `org.tinylog:tinylog-api`                     | 2.7.0         | Logging API (static `Logger` calls)                      |
| `org.tinylog:tinylog-impl`                    | 2.7.0         | Logging implementation (console/file writers)            |
| `org.apache.commons:commons-configuration2`   | 2.12.0        | INI + properties file parsing                            |
| `org.apache.commons:commons-lang3`            | 3.18.0        | General utilities; pins version to fix CVE-2025-48924    |
| `org.hibernate.validator:hibernate-validator` | 9.0.0.Final   | Bean validation (JSR-380)                                |
| `org.glassfish.expressly:expressly`           | 6.0.0         | EL expression language (required by Hibernate Validator) |
| `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml` | 2.18.2 | YAML parsing support                            |
| `com.fasterxml.jackson.core:jackson-databind` | 2.18.2        | JSON/YAML data binding                                   |
| `com.ao:server-security`                      | 1.0-SNAPSHOT  | Internal — Netty Encrypter/Decrypter handlers            |

## Test Libraries (not shipped)

| Library                           | Version | Role                |
|-----------------------------------|---------|---------------------|
| `org.junit.jupiter:junit-jupiter` | 5.13.4  | Unit test framework |
| `org.assertj:assertj-core`        | 3.27.3  | Fluent assertions   |
| `org.mockito:mockito-core`        | 5.18.0  | Mocking framework   |
| `org.mockito:mockito-junit-jupiter` | 5.18.0 | Mockito JUnit 5 extension |
| `org.jacoco:jacoco-maven-plugin`  | 0.8.13  | Coverage reporting  |

## Build Tools

| Tool                    | Version | Role                                            |
|-------------------------|---------|-------------------------------------------------|
| Maven                   | 3.8+    | Build, dependency management, multi-module      |
| `maven-compiler-plugin` | 3.14.0  | Java 17 compilation                             |
| `maven-assembly-plugin` | —       | Creates fat JAR (`*-jar-with-dependencies.jar`) |
| `spotless-maven-plugin` | 2.44.0  | Code formatting enforcement                     |
| Java                    | 17      | Language target and runtime                     |

## External Services / Integrations

None — the server is fully self-contained. All game data is in local files under `data/`. No external APIs, databases,
message brokers, or cloud services.

## Internal Module Dependency

```
server  ──depends on──►  server-security
```

`server-security` provides `SecurityManager` implementations injected via `SecurityModule` at runtime using the class
name from `project.properties`.

> **Nota**: `DefaultSecurityManager` no cifra el tráfico. Es la implementación de desarrollo. El cifrado real
> queda pendiente de implementar en una clase concreta diferente.
