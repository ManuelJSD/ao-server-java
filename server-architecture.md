# 📖 Documentación Técnica — AO-Server

> **Análisis completo del servidor de Argentum Online en Java**
> Fecha: 2026-05-28 · Actualizada manualmente desde análisis de código fuente

---

## 📌 Resumen General

**AO-Server** es una reimplementación del servidor del MMORPG *Argentum Online* (originalmente en Visual Basic 6), escrita en **Java 17** con un enfoque moderno basado en:

- **Netty** para networking asíncrono de alto rendimiento
- **Google Guice** para inyección de dependencias (IoC)
- **Maven** como build system multi-módulo
- **JUnit 5 + AssertJ + Mockito** para testing
- **tinylog 2.7.0** para logging

El proyecto está basado en [AOXP-Server](https://github.com/aoxp/AOXP-Server) (commit `daa8d10`) ya que la implementación original dejó de mantenerse.

---

## 🏗️ Arquitectura de Módulos

```mermaid
graph TD
    A["aoserver (POM padre)"] --> B["server (Server Core)"]
    A --> C["server-security (Server Security)"]
    B -->|depende de| C
```

| Módulo | Artifact | Descripción |
|--------|----------|-------------|
| `aoserver` | `com.ao:aoserver:1.0-SNAPSHOT` | POM padre con gestión de dependencias y plugins |
| `server` | `com.ao:server` | Módulo principal: lógica de juego, red, datos, servicios |
| `server-security` | `com.ao:server-security` | Módulo de seguridad: cifrado/descifrado de tráfico de red |

---

## 📦 Stack Tecnológico

| Categoría | Tecnología | Versión |
|-----------|-----------|---------| 
| **Lenguaje** | Java | 17 |
| **Build** | Maven | 3.8+ |
| **Networking** | Netty (NIO) | 4.1.119.Final |
| **IoC/DI** | Google Guice | 7.0.0 |
| **Utilidades** | Google Guava | 32.0.1-jre |
| **Logging** | tinylog | 2.7.0 |
| **Configuración** | Apache Commons Configuration2 | 2.12.0 |
| **Validación** | Hibernate Validator + Expressly | 9.0.0 / 6.0.0 |
| **Serialización** | Jackson YAML + Databind | 2.18.2 |
| **Testing** | JUnit 5 + AssertJ + Mockito | 5.13.4 / 3.27.3 / 5.18.0 |
| **Cobertura** | JaCoCo + Coveralls | 0.8.13 |
| **CI/CD** | GitHub Actions | — |

---

## 📂 Estructura de Paquetes (Módulo `server`)

```
com.ao
├── Bootstrap.java              ← Punto de entrada (main)
├── AOServer.java               ← Servidor Netty (Runnable)
│
├── action/                     ← Sistema de acciones asíncronas
│   ├── Action.java
│   └── ActionExecutor.java     ← Executor de single-thread con cola
│
├── config/                     ← Configuración del servidor
│   ├── ArchetypeConfiguration.java
│   ├── IntervalsConfig.java    ← Intervalos de timers del juego
│   ├── ServerConfig.java       ← Interfaz de configuración
│   └── ini/
│       ├── ArchetypeConfigurationIni.java
│       └── ServerConfigIni.java ← Implementación basada en INI
│
├── context/                    ← Contexto de aplicación (Service Locator)
│   ├── ApplicationContext.java
│   └── ApplicationProperties.java
│
├── data/dao/                   ← Capa de Acceso a Datos
│   ├── AccountDAO.java
│   ├── CityDAO.java
│   ├── MapDAO.java
│   ├── NpcCharacterDAO.java
│   ├── ObjectDAO.java
│   ├── UserCharacterDAO.java
│   ├── exception/
│   │   ├── DAOException.java
│   │   └── NameAlreadyTakenException.java
│   ├── ini/                    ← Implementaciones basadas en INI
│   │   ├── CityDAOIni.java
│   │   ├── LegacyObjectType.java
│   │   ├── NpcDAOIni.java
│   │   ├── ObjectDAOIni.java
│   │   └── UserDAOIni.java
│   └── map/
│       └── MapDAOImpl.java     ← Lectura de mapas binarios
│
├── ioc/                        ← Módulos Guice (DI)
│   ├── ArchetypeLocator.java
│   ├── InjectorFactory.java    ← Fábrica central del Injector (6 módulos)
│   └── module/
│       ├── ArchetypeModule.java    ← @Provides para 17 arquetipos
│       ├── BootstrapModule.java
│       ├── ConfigurationModule.java
│       ├── DaoModule.java
│       ├── SecurityModule.java
│       └── ServiceModule.java
│
├── model/                      ← Modelo de dominio
│   ├── character/              ← Personajes (jugadores y NPCs)
│   │   ├── Character.java      ← Interfaz base
│   │   ├── UserCharacter.java  ← Personaje de jugador
│   │   ├── NpcCharacter.java   ← Personaje NPC
│   │   ├── Race.java, Gender.java, Skill.java, Attribute.java...
│   │   ├── archetype/          ← 17 arquetipos concretos (+ DefaultArchetype, UserArchetype)
│   │   ├── attack/             ← Estrategias de ataque
│   │   ├── behavior/           ← Comportamientos NPC (Hostile, Pet, Null)
│   │   ├── movement/           ← Estrategias de movimiento (Greedy, Quiet)
│   │   └── npc/
│   │       ├── Drop.java
│   │       ├── drop/           ← Estrategias de drop (Random, DropAll)
│   │       └── properties/     ← Tipos de NPC (Creature, Guard, Merchant...)
│   │
│   ├── inventory/              ← Sistema de inventario
│   │   ├── Inventory.java
│   │   └── InventoryImpl.java
│   │
│   ├── map/                    ← Sistema de mapas
│   │   ├── Map.java            ← 100x100 tiles, búsqueda de caminos
│   │   ├── Tile.java           ← Celda del mapa
│   │   ├── Position.java       ← Coordenadas (mapa, x, y)
│   │   ├── City.java           ← Ciudades
│   │   ├── Heading.java        ← Direcciones (N/S/E/W)
│   │   ├── Trigger.java        ← Triggers de tiles
│   │   └── area/AreaInfo.java  ← Info de área visible
│   │
│   ├── object/                 ← 43 tipos de objetos del juego
│   │   ├── Object.java         ← Interfaz base
│   │   ├── Item.java, Weapon.java, Armor.java, Shield.java...
│   │   ├── Food.java, Drink.java, HPPotion.java, ManaPotion.java...
│   │   ├── Boat.java, Door.java, Key.java, Sign.java...
│   │   └── properties/         ← Propiedades de objetos + crafting
│   │
│   ├── spell/                  ← Sistema de hechizos
│   │   ├── Spell.java
│   │   └── effect/             ← 8 efectos (HitPoints, Poison, Paralysis...)
│   │
│   └── user/                   ← Usuarios y cuentas
│       ├── User.java           ← Interfaz base
│       ├── ConnectedUser.java  ← Usuario conectado (pre-login)
│       ├── LoggedUser.java     ← Usuario autenticado (post-login)
│       ├── Account.java / AccountImpl.java
│       └── Guild.java
│
├── network/                    ← Capa de red
│   ├── Connection.java         ← DTO usuario+canal
│   ├── DataBuffer.java         ← Wrapper sobre ByteBuf de Netty
│   ├── ClientPacketsManager.java ← Dispatcher de paquetes entrantes
│   ├── ServerPacketsManager.java ← Dispatcher de paquetes salientes
│   └── packet/
│       ├── IncomingPacket.java  ← Interfaz para paquetes cliente→servidor
│       ├── OutgoingPacket.java  ← Interfaz para paquetes servidor→cliente
│       ├── incoming/           ← 11 paquetes entrantes implementados
│       │   ├── LoginExistingCharacterPacket
│       │   ├── LoginNewCharacterPacket
│       │   ├── TalkPacket, YellPacket, WhisperPacket
│       │   ├── WalkPacket, ChangeHeadingPacket
│       │   ├── LeftClickPacket
│       │   ├── QuitPacket
│       │   ├── PingPacket
│       │   └── ThrowDicesPacket
│       └── outgoing/           ← 25 paquetes salientes implementados
│           ├── AreaChangedPacket, BlockPositionPacket
│           ├── ChangeInventorySlotPacket, ChangeMapPacket, ChangeSpellSlotPacket
│           ├── CharacterCreatePacket, ConsoleMessagePacket
│           ├── DiceRollPacket, DisconnectPacket, ErrorMessagePacket
│           ├── GuildChatPacket, MultiMessagePacket
│           ├── ObjectCreatePacket, ParalyzedPacket
│           ├── PlayMidiPacket, PlayWavePacket, PongPacket
│           ├── SetInvisiblePacket, UpdateDexterityPacket
│           ├── UpdateHungerAndThirstPacket, UpdateStrengthAndDexterityPacket
│           ├── UpdateStrengthPacket, UpdateUserStatsPacket
│           ├── UserCharacterIndexInServerPacket, UserIndexInServer
│
├── service/                    ← Capa de servicios
│   ├── AreaService.java → AreaServiceImpl
│   ├── CharacterBodyService.java → CharacterBodyServiceImpl
│   ├── LoginService.java → LoginServiceImpl
│   ├── MapService.java → MapServiceImpl
│   ├── NpcService.java → NpcServiceImpl
│   ├── ObjectService.java → ObjectServiceImpl
│   ├── TimedEventsService.java → TimedEventsServiceImpl
│   ├── UserService.java → UserServiceImpl
│   ├── PrivilegesService.java
│   ├── ValidatorService.java
│   └── CharacterIndexManager.java
│
└── utils/                      ← Utilidades
    ├── IniUtils.java           ← Lectura de archivos INI legacy
    └── RangeParser.java        ← Parser de rangos numéricos (ej: "1-40")
```

### Módulo `server-security` (3 archivos)

```
com.ao.security
├── Hashing.java                ← Utilidad de hashing MD5
├── SecurityManager.java        ← Interfaz de seguridad (encrypt/decrypt)
└── impl/
    └── DefaultSecurityManager.java ← Implementación sin cifrado (desarrollo)
```

---

## 🔄 Flujo de Ejecución

```mermaid
sequenceDiagram
    participant Main as Bootstrap.main()
    participant Ctx as ApplicationContext
    participant Guice as Guice Injector
    participant Services as Services Layer
    participant Timers as ScheduledExecutorService
    participant Server as AOServer

    Main->>Ctx: loadApplicationContext()
    Ctx->>Guice: InjectorFactory.get(properties)
    Guice-->>Ctx: Injector con 6 módulos

    Main->>Services: MapService.loadMaps()
    Main->>Services: MapService.loadCities()
    Main->>Services: ObjectService.loadObjects()
    Main->>Services: NpcService.loadNpcs()

    Main->>Server: configureNetworking()
    Note right of Server: Bind 0.0.0.0:7666

    Main->>Timers: startTimers()
    Note right of Timers: 7 hilos: HP/mana, stamina,\nhambre, sed, IA NPC,\nworld save, efectos temp.

    Main->>Server: server.run()
    Note right of Server: Netty EventLoop activo
```

---

## 🌐 Pipeline de Red (Netty)

El servidor usa **Netty** con el patrón **Reactor** (boss/worker event loop groups):

```mermaid
graph LR
    subgraph "Inbound (Cliente → Servidor)"
        A[Bytes crudos] --> B[Decrypter]
        B --> C["Decoder (ByteToMessage)"]
        C --> D[ClientPacketsManager]
    end

    subgraph "Outbound (Servidor → Cliente)"
        E[OutgoingPacket] --> F["Encoder (MessageToMessage)"]
        F --> G[Encrypter]
        G --> H[Bytes cifrados]
    end
```

### Paquetes Entrantes Implementados (11)

| ID | Paquete | Descripción |
|----|---------|-------------|
| 0 | `LoginExistingCharacterPacket` | Login con personaje existente |
| 1 | `ThrowDicesPacket` | Tirar dados para atributos |
| 2 | `LoginNewCharacterPacket` | Crear nuevo personaje |
| 3 | `TalkPacket` | Mensaje de chat |
| 4 | `YellPacket` | Gritar (chat amplio) |
| 5 | `WhisperPacket` | Susurro (chat privado) |
| 6 | `WalkPacket` | Movimiento del personaje |
| — | `ChangeHeadingPacket` | Cambiar orientación (N/S/E/W) |
| — | `LeftClickPacket` | Click izquierdo sobre entidad |
| — | `QuitPacket` | Desconexión limpia del cliente |
| 119 | `PingPacket` | Heartbeat para latencia |

### Paquetes Salientes Implementados (25)

| Categoría | Paquetes |
|-----------|----------|
| Auth/Sesión | `UserIndexInServer`, `UserCharacterIndexInServerPacket`, `CharacterCreatePacket`, `DisconnectPacket` |
| Mapa/Movimiento | `ChangeMapPacket`, `AreaChangedPacket`, `BlockPositionPacket` |
| Inventario | `ChangeInventorySlotPacket`, `ChangeSpellSlotPacket`, `ObjectCreatePacket` |
| Estadísticas | `UpdateUserStatsPacket`, `UpdateStrengthPacket`, `UpdateDexterityPacket`, `UpdateStrengthAndDexterityPacket`, `UpdateHungerAndThirstPacket` |
| Estado | `ParalyzedPacket`, `SetInvisiblePacket` |
| Chat | `ConsoleMessagePacket`, `ErrorMessagePacket`, `GuildChatPacket`, `MultiMessagePacket` |
| Juego | `DiceRollPacket`, `PlayMidiPacket`, `PlayWavePacket`, `PongPacket` |

---

## ⏱️ Game Timers (Bootstrap)

El método `startTimers()` inicializa un `ScheduledExecutorService` con **7 hilos** (`game-timer-N`) que ejecutan las siguientes tareas en paralelo:

| Timer | Intervalo | Estado |
|-------|-----------|--------|
| Regeneración HP/Mana | `intervals.regen.hp` | ✅ Implementado |
| Regeneración Stamina | `intervals.regen.stamina` | ✅ Implementado |
| Hambre | `intervals.survival.hunger` | ✅ Implementado |
| Sed | `intervals.survival.thirst` | ✅ Implementado |
| IA de NPCs | `intervals.npc.aiTick` | 🔧 Timer activo, lógica pendiente |
| World Save | `intervals.world.saveInterval` (min) | 🔧 Timer activo, escritura pendiente |
| Efectos temporales | `intervals.states.poison` | 🔧 Timer activo, lógica pendiente |

---

## 🎮 Modelo de Dominio

### Character (Interfaz central)

El modelo de personajes usa una arquitectura basada en interfaces y patrones Strategy:

```mermaid
classDiagram
    class Character {
        <<interface>>
        +getHitPoints()
        +getMana()
        +getPosition()
        +attack(Character)
        +cast(Spell, Character)
        +moveTo(Heading)
        +getInventory()
        ...
    }

    Character <|-- UserCharacter
    Character <|-- NpcCharacter

    Character --> Inventory
    Character --> Position
    Character --> Reputation
    Character --> Privileges
    Character --> Archetype

    NpcCharacter --> Behavior
    NpcCharacter --> MovementStrategy
    NpcCharacter --> AttackStrategy
    NpcCharacter --> Dropable

    Behavior <|-- HostileBehavior
    Behavior <|-- PetBehavior
    Behavior <|-- NullBehavior

    MovementStrategy <|-- GreedyMovementStrategy
    MovementStrategy <|-- QuietMovementStrategy
```

### Arquetipos (17 clases concretas)

Warrior, Mage, Paladin, Cleric, Assasin, Bard, Druid, Bandit, Thief, Pirate, Hunter, Fisher, Lumberjack, Miner, Blacksmith, Carpenter, Worker.

> Además existen `DefaultArchetype` (base de comportamiento compartido) y `UserArchetype` (archetype de usuario genérico).

### Razas y Géneros

Definidos como enums: `Race` (Human, Elf, DarkElf, Dwarf, Gnome) y `Gender` (Male, Female).

### Tipos de NPC

Creature, Guard, Merchant, Trainer, Governor, Noble (cada uno en `npc.properties`).

---

## 🗺️ Sistema de Mapas

- **Grilla**: 100×100 tiles por mapa
- **Área visible**: 8×6 tiles (VISIBLE_AREA_WIDTH × VISIBLE_AREA_HEIGHT)
- **Distancia máxima**: 12 tiles
- **Total de mapas configurados**: 290 (archivos `.map`, `.inf`, `.dat`)
- **Archivos de mapa en resources**: 870 archivos
- **Formato**: Binario (legacy del AO original en VB6)
- **Tipos de tile**: Ground, Water, Lava, con triggers y salidas

---

## 💉 Sistema de Inyección de Dependencias

El IoC usa **Google Guice** con **6 módulos**:

| Módulo | Bindings principales |
|--------|---------------------|
| `BootstrapModule` | `ServerConfig` → `ServerConfigIni`, `IntervalsConfig` |
| `ConfigurationModule` | `ArchetypeConfiguration` → INI |
| `DaoModule` | Los 6 DAO interfaces → implementaciones (Singleton) |
| `ServiceModule` | Todos los service interfaces → implementaciones (Singleton); parámetros de configuración nombrados |
| `SecurityModule` | `SecurityManager` → cargado dinámicamente por reflection |
| `ArchetypeModule` | 17 métodos `@Provides` (uno por clase de arquetipo) |

> [!WARNING]
> El `ApplicationContext` usa un **patrón Service Locator estático** que los propios desarrolladores marcan como TODO para eliminar. Esto dificulta la testabilidad y acopla el código.

---

## 📁 Persistencia de Datos

La capa de datos lee formatos **legacy INI** del AO original:

| DAO | Archivo fuente | Datos |
|-----|---------------|-------|
| `ObjectDAOIni` | `data/objects.dat` | 43 tipos de objeto del juego |
| `NpcDAOIni` | `data/npcs.dat` | NPCs del mundo |
| `CityDAOIni` | `data/cities.dat` | Ciudades y spawn points |
| `UserDAOIni` | `charfiles/*.chr` | Archivos de personaje |
| `MapDAOImpl` | `data/maps/MapaN.{map,inf,dat}` | Mapas binarios |

> [!NOTE]
> No hay base de datos relacional. Todo se persiste en archivos INI y binarios, manteniendo compatibilidad con el formato original de Argentum Online. El guardado (write) aún no está implementado.

---

## ⚙️ Configuración

### `project.properties` — Configuración del contexto de aplicación

Rutas a archivos de datos, configuración de razas (heads y bodies), inventario, seguridad.

### `server.ini` — Configuración del servidor en formato INI

- **Puerto**: 7666 (por defecto)
- **Versión del cliente**: 0.13.0
- **Max usuarios**: 550
- **Creación de personajes**: Activada
- **Staff**: Listas de Dioses, Semidioses, Consejeros, RolesMasters
- **Intervalos de juego**: Regeneración, hambre, sed, veneno, movimiento, ataque, etc.
- **MD5 Hashes**: Verificación de integridad del cliente

### `tinylog.properties` — Configuración de logging

Configuración del sistema de logging tinylog (escrito por classpath scan en runtime).

---

## 🧪 Testing

| Métrica | Valor |
|---------|-------|
| **Archivos de test** | 72 |
| **Archivos de producción** | 266 (+3 security) |
| **Ratio test/src** | ~27% |

### Cobertura por capa:

| Capa | Tests | Observación |
|------|-------|-------------|
| **Model (objects)** | ✅ 35 tests | Excelente cobertura de 43 tipos de objeto |
| **Model (spell effects)** | ✅ 7 tests | Buena cobertura |
| **Model (character)** | ⚠️ 4 tests | Solo Reputation, Archetype, Movement |
| **Model (map)** | ✅ 2 tests | Map y Position |
| **Data (DAO)** | ✅ 5 tests | CityDAO, NpcDAO, ObjectDAO, UserDAO, MapDAO |
| **Network** | ⚠️ 2 tests | Solo LoginPackets |
| **Service** | ⚠️ 5 tests | CharacterBody, MapService, TimedEvents, otros |
| **Config** | ✅ 2 tests | Configuración de servidor e intervalos |
| **Model (user)** | ✅ 1 test | AccountImpl |

### CI/CD

GitHub Actions ejecuta en cada push/PR a `main`:
1. Checkout + Setup Java 17 (Temurin)
2. `mvn -B package`
3. `mvn clean test jacoco:report`
4. Envío de cobertura a Coveralls
5. Dependency submission para Dependabot

---

## 📊 Estado de Madurez del Servidor

```mermaid
pie title Estado de Implementación
    "Modelo de dominio" : 38
    "Capa de datos (DAOs)" : 15
    "Red (Netty + paquetes)" : 17
    "Servicios + Timers" : 15
    "Testing" : 10
    "Configuración + IoC" : 5
```

### ✅ Lo que FUNCIONA

- **Modelo de dominio completo**: Personajes, NPCs, objetos (43 tipos), hechizos, mapas, inventario
- **Capa de datos**: Lectura de archivos INI y mapas binarios legacy. Carga tolerante con huecos en `objects.dat` e ignorando falsos errores de NPCs
- **Infraestructura de red**: Pipeline Netty con cifrado/descifrado, decodificación/codificación
- **Inyección de dependencias**: 6 módulos Guice configurados (incluido ArchetypeModule)
- **CI/CD**: Pipeline completo con tests y cobertura
- **Paquetes de red básicos**: Login (existente + nuevo), chat (talk/yell/whisper), movimiento, orientación, click izquierdo, desconexión
- **Game Timers**: HP/mana regen, stamina regen, hambre, sed (todos activos con lógica completa)

### ⚠️ Lo que está INCOMPLETO

1. **IA de NPCs**: Timer activo cada `aiTick` ms, pero la lógica de comportamiento es un TODO
2. **World Save**: Timer activo periódicamente, pero la escritura a disco es un TODO
3. **Efectos temporales**: Timer activo (poison interval), lógica de limpieza de estados es un TODO
4. **Service Locator** (`ApplicationContext`): Eliminar Injector estático pendiente
5. **Paquetes entrantes**: Solo 11 de ~129 del protocolo original
6. **Paquetes salientes**: 25 de ~104 definidos
7. **Sistema de persistencia**: Solo lectura de datos

### 🔴 Lo que FALTA implementar

- Lógica completa de IA para NPCs
- Escritura/guardado de personajes (world save)
- Efectos temporales (veneno, parálisis, invisibilidad)
- Sistema de combate PvP y PvE
- Sistema de comercio (NPCs y entre jugadores)
- Sistema de crafting
- Sistema de guilds/clanes completo
- Sistema de misiones/quests
- Administración en runtime (comandos GM)
- Cifrado real del tráfico
- **Implementar paquetes prioritarios para la jugabilidad (ver Roadmap Prioritario)**

---

## 🚀 Roadmap Prioritario: Paquetes de Red (MVP Jugable)

Dado que el cliente ya tiene implementados 129 paquetes de entrada y 104 de salida, pero el servidor apenas tiene una fracción, la prioridad para lograr una versión **jugable (MVP)** es implementar la siguiente lista de paquetes.

### 📥 Paquetes Entrantes (Cliente → Servidor) Prioritarios

1. **Acciones y Combate**
   - `Attack` (Golpe físico con arma o puño)
   - `CastSpell` (Lanzar hechizo a un target seleccionado)
2. **Interacción con Objetos e Inventario**
   - `EquipItem` (Equipar/Desequipar arma, armadura, escudo, casco)
   - `UseItem` (Beber pociones de HP/Mana, comer)
   - `Drop` (Tirar objetos del inventario al mapa)
   - `Take` (Agarrar objetos del mapa con tecla Q)
3. **Mundo y NPCs**
   - `CommerceStart` / `CommerceBuy` / `CommerceSell` (Comprar y vender a NPCs)
   - `Resurrect` (Pedir resurrección a un Priest NPC)

### 📤 Paquetes Salientes (Servidor → Cliente) Prioritarios

1. **Sincronización de Entidades**
   - `CharacterCreate` / `CharacterRemove` / `CharacterMove` (Sincronizar spawn, movimiento y desaparición de usuarios y NPCs en área visible)
   - `ObjectCreate` / `ObjectDelete` (Ver items en el suelo)
2. **Feedback Visual y Combate**
   - `UpdateHP` / `UpdateMana` / `UpdateStamina` (Cambio de estado al usar pociones o recibir daño)
   - `UpdateGold` / `UpdateExp` (Al matar NPCs o comerciar)
   - `CreateFX` / `Blood` (Sangre y efectos visuales de hechizos)
   - `PlayWave` (Sonidos de golpes, hechizos, fallos)
3. **Feedback de Interfaz**
   - `ConsoleMessage` (Mensajes de sistema: "Fallas el golpe", "No tienes suficiente maná", etc.)
   - `ChatOverHead` (Textos flotantes encima de la cabeza de los personajes)
   - `UpdateInventorySlot` / `ChangeSpellSlot` (Refrescar inventario)

---

## 🔗 Referencias

- **Repositorio original**: [AOXP-Server](https://github.com/aoxp/AOXP-Server)
- **Commit base**: `daa8d10b83b762a0072dd022e99fdfab1c57bb6b`
- **Argentum Online**: MMORPG argentino desarrollado en 1999 por Pablo Márquez
