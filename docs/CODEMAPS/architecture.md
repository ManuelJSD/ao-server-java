<!-- Generated: 2026-05-28 | Files scanned: 266 classes / 47 packages | Token estimate: ~600 -->

# Architecture

## System Overview

Java 17 game server (Argentum Online reimplementation). Single deployable JAR, file-based persistence, no database.

```
Client (AO game client)
    │  TCP socket
    ▼
[Netty Pipeline per connection]
  Decrypter → Decoder → ClientPacketsManager
                             │ IncomingPacket.handle()
                             ▼
                       Service Layer (singletons, Guice)
                             │
                       DAO Layer (file readers)
                             │
                       data/ (INI + binary files)

  ServerPacketsManager → Encoder → Encrypter
         ▲
         │ OutgoingPacket.write()
    Service Layer
```

## Module Boundaries

```
aoserver (parent POM)
├── server/          ← core: networking, services, model, DAOs
└── server-security/ ← Encrypter/Decrypter Netty handlers
```

## Startup Sequence (`com.ao.Bootstrap`)

1. `InjectorFactory` creates Guice Injector (6 modules)
2. `MapService.loadMaps()` — 290 binary map files
3. `MapService.loadCities()` — INI cities.dat
4. `ObjectService.loadObjects()` — INI objects.dat
5. `NpcService.loadNpcs()` — INI npcs.dat
6. `configureNetworking()` — sets bind address from `server.ini`
7. `startTimers()` — 7-thread ScheduledExecutorService for game loops
8. `AOServer.run()` — Netty boss/worker event loops bind on `0.0.0.0:7666`

## Layer Responsibilities

| Layer           | Role                          | Location                                               |
|-----------------|-------------------------------|--------------------------------------------------------|
| Network         | Packet framing, en/decryption | `network/`, `server-security/`                         |
| Packet handlers | Protocol dispatch             | `network/packet/incoming/`, `network/packet/outgoing/` |
| Services        | Business logic, game state    | `service/`                                             |
| DAOs            | File I/O                      | `data/dao/`                                            |
| Domain model    | Entities, value objects       | `model/`                                               |
| IoC             | Guice wiring (6 modules)      | `ioc/module/`                                          |
| Actions         | Async command queue           | `action/`                                              |
| Timers          | Game loops (7 threads)        | `Bootstrap.startTimers()`                              |

## Key Design Patterns

- **Reactor** (Netty): non-blocking I/O, boss+worker thread groups
- **Guice DI**: constructor injection, all services/DAOs are singletons
- **Service Locator**: `ApplicationContext.getInstance(T)` used at startup boundaries only
- **Strategy**: `MovementStrategy`, `AttackStrategy`, `AIType` on NPC
- **Action Queue**: single-threaded `ActionExecutor` serializes world mutations
- **Template Method**: `DefaultArchetype` provides base behavior; concrete archetypes override specifics
