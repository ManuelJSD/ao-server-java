<!-- Generated: 2026-05-28 | Files scanned: 266 classes / 47 packages | Token estimate: ~950 -->

# Backend

## Packet Dispatch

### Incoming (Client → Server) — 11 packets

```
ClientPacketsManager.dispatch(packetId, DataBuffer, Connection)
  LOGIN_EXISTING  → LoginExistingCharacterPacket → LoginService.connectExistingCharacter()
  LOGIN_NEW       → LoginNewCharacterPacket       → LoginService.connectNewCharacter()
  WALK            → WalkPacket                    → MapService.moveCharacterTo()
  CHANGE_HEADING  → ChangeHeadingPacket           → [update character heading]
  LEFT_CLICK      → LeftClickPacket               → [interact with entity at position]
  TALK            → TalkPacket                    → [broadcast to area]
  WHISPER         → WhisperPacket                 → [send to target Connection]
  YELL            → YellPacket                    → [broadcast to all]
  THROW_DICES     → ThrowDicesPacket              → DiceRollPacket (outgoing)
  QUIT            → QuitPacket                    → [clean disconnect, free resources]
  PING            → PingPacket                    → PongPacket (outgoing)
```

### Outgoing (Server → Client) — 25 packets

```
ServerPacketsManager → OutgoingPacket.write(DataBuffer) → Connection.send()

Auth/Session:  UserIndexInServer, UserCharacterIndexInServerPacket, CharacterCreatePacket,
               DisconnectPacket
Map/Movement:  ChangeMapPacket, AreaChangedPacket, BlockPositionPacket
Inventory:     ChangeInventorySlotPacket, ChangeSpellSlotPacket, ObjectCreatePacket
Stats:         UpdateUserStatsPacket, UpdateStrengthPacket, UpdateDexterityPacket,
               UpdateStrengthAndDexterityPacket, UpdateHungerAndThirstPacket
Status:        ParalyzedPacket, SetInvisiblePacket
Chat:          ConsoleMessagePacket, ErrorMessagePacket, GuildChatPacket, MultiMessagePacket
Game:          DiceRollPacket, PlayMidiPacket, PlayWavePacket, PongPacket
```

## Service → DAO Mapping

```
LoginService (LoginServiceImpl)
  ├── AccountDAO      → UserDAOIni       → charfiles/*.chr
  ├── UserCharacterDAO→ UserDAOIni       → charfiles/*.chr
  ├── UserService     → UserServiceImpl  (session tracking)
  ├── PrivilegesService                  (role checks)
  └── CharacterIndexManager              (slot allocation)

MapService (MapServiceImpl)
  ├── MapDAO          → MapDAOImpl       → data/maps/*.map (binary, 290 files)
  ├── CityDAO         → CityDAOIni       → data/cities.dat (INI)
  └── AreaService     → AreaServiceImpl

ObjectService (ObjectServiceImpl)
  └── ObjectDAO       → ObjectDAOIni     → data/objects.dat (INI, 43 types)

NpcService (NpcServiceImpl)
  ├── NpcCharacterDAO → NpcDAOIni        → data/npcs.dat (INI)
  └── NpcManager                         (spawn/behavior loop — lógica pendiente)
```

## Guice Module → Bindings

| Module              | Key Bindings                                                                         |
|---------------------|--------------------------------------------------------------------------------------|
| BootstrapModule     | ServerConfig → ServerConfigIni, IntervalsConfig → IntervalsConfigIni                |
| ConfigurationModule | ArchetypeConfiguration → ArchetypeConfigurationIni                                   |
| DaoModule           | All 6 DAO interfaces → implementations (Singleton)                                   |
| ServiceModule       | All service interfaces → implementations (Singleton); named int/string config params |
| SecurityModule      | SecurityManager → dynamically loaded class from config                               |
| ArchetypeModule     | 17 archetype @Provides methods (one per archetype class)                             |

## Game Timers (`Bootstrap.startTimers`)

Pool de 7 hilos (`game-timer-N`) vía `ScheduledExecutorService`:

| Timer | Método disparado | Estado |
|-------|-----------------|--------|
| HP/Mana regen | `character.regenHpAndMana()` + `UpdateUserStatsPacket` | ✅ Completo |
| Stamina regen | `character.regenStamina()` + `UpdateUserStatsPacket` | ✅ Completo |
| Hambre | `character.tickHunger()` + `UpdateHungerAndThirstPacket` | ✅ Completo |
| Sed | `character.tickThirst()` + `UpdateHungerAndThirstPacket` | ✅ Completo |
| IA NPCs | TODO: lógica de NPCs | 🔧 Timer activo |
| World Save | TODO: guardado masivo | 🔧 Timer activo |
| Efectos temporales | TODO: limpieza estados temporales | 🔧 Timer activo |

Cada timer captura excepciones individualmente y loguea con tinylog en lugar de fallar el hilo completo.

## Key Files

```
com/ao/Bootstrap.java              entry point, startup orchestration, game timers
com/ao/AOServer.java               Netty server bootstrap, pipeline config
com/ao/ioc/InjectorFactory.java    assembles all 6 Guice modules
com/ao/context/ApplicationContext.java  Service Locator wrapper
com/ao/network/ClientPacketsManager.java  incoming packet dispatch
com/ao/network/ServerPacketsManager.java  outgoing packet factory
com/ao/action/ActionExecutor.java  single-threaded async action queue
com/ao/service/login/LoginServiceImpl.java  auth + character load
com/ao/service/map/MapServiceImpl.java      map state + movement
```

## Async Model

`ActionExecutor<T extends ActionExecutor<?>>` wraps a `BlockingQueue<Action>` processed by a single background thread.
Used by `MapServiceImpl` to serialize world-map mutations without explicit locks.

Game timers run in a separate `ScheduledExecutorService` (7 threads). Each timer iterates over
`UserService.getConnectedUsers()` and applies actions to alive `UserCharacter` instances.
