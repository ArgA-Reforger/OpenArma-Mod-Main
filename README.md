<div align="center">

# OpenArma Mod Main

**Arma Reforger core AI commander mod**

[OpenArma main repo](https://github.com/ArgA-Reforger/OpenArma) | [MapExporter](https://github.com/ArgA-Reforger/OpenArma-Mod-MapExporter) | [MapScanner](https://github.com/ArgA-Reforger/OpenArma-Mod-MapScanner)

🇪🇸 [Español](#español) | 🇬🇧 [English](#english)

</div>

---

## Español

### Introducción

OpenArma Mod Main es el mod de runtime principal de [OpenArma](https://github.com/ArgA-Reforger/OpenArma) dentro de Arma Reforger. Implementa el ciclo operativo completo del comandante IA: observar el campo de batalla → reportar la situación → recibir órdenes → ejecutar comandos.

El mod en sí no toma decisiones — es un **cliente Agent MCP (Model Context Protocol)** que se comunica con el backend de OpenArma vía REST API, mientras que el motor Multi-Agent del backend maneja todas las decisiones tácticas.

### Arquitectura

```
OA_Main (controlador singleton)
  ├── OA_WorldObserver     Observación: escanea el campo de batalla, arma el reporte de situación en JSON
  ├── OA_DecisionBridge    Comunicación: envío/recepción vía REST API (polling por heartbeat)
  ├── OA_CommandExecutor   Ejecución: convierte las órdenes de la IA en Waypoints del juego
  ├── OA_EventTracker      Eventos: rastrea eventos de combate (contacto/baja/avistamiento)
  ├── OA_HudIndicator      UI: estado de conexión/estado de ejecución/facción
  └── OA_Config            Configuración: URL de la API/clave/intervalo de decisión
```

### Flujo de ejecución

```
1. Login     El jugador ingresa la API Key → se establece la conexión con el backend
2. Heartbeat Cada N segundos: WorldObserver escanea → arma el JSON de situación → lo envía al backend
3. Response  El backend responde con: órdenes pendientes + configuración actualizada (inicio/parada, facción, intervalo)
4. Execute   CommandExecutor convierte las órdenes JSON en Waypoints del juego (mover/defender/patrullar/atacar)
5. Loop      Se repiten los pasos 2-4 hasta el Logout
```

### Funciones principales

- **Soporte multi-facción**: configuración dinámica de N bandos (IA vs humano / IA vs IA), cada bando con su propio modo de control
- **Reportes de situación**: escaneo automático de posiciones amigas/enemigas, composición, armas, salud, munición y tarea actual
- **Ejecución de comandos**: soporta comandos tácticos como `move` / `defend` / `patrol` / `attack` / `regroup`
- **Rastreo de eventos de combate**: inicio/fin de contacto, bajas de unidades, avistamientos enemigos y otros eventos reportados
- **Reconexión**: reintento con backoff exponencial, hasta 10 intentos automáticos de reconexión
- **Visualización HUD**: indicador de estado de conexión (verde/amarillo/rojo/gris) + configuración de facción actual

### Instalación

1. Importar este mod en el Arma Reforger Workbench
2. Agregar `OA_GameModeInjector` o `OA_GameModeComponent` al GameMode de la escena
3. Asegurarse de que el backend de OpenArma esté corriendo
4. Ingresar la API Key en el juego para conectarse

### Descripción de archivos

```
OA/
├── Scripts/Game/OA/
│   ├── OA_Main.c                 Controlador singleton: Login → Heartbeat → Logout
│   ├── OA_WorldObserver.c        Escaneo de situación del campo de batalla y construcción del JSON
│   ├── OA_DecisionBridge.c       Capa de comunicación REST API
│   ├── OA_CommandExecutor.c      Conversión de órdenes de la IA en Waypoints del juego
│   ├── OA_EventTracker.c         Rastreo de eventos de combate
│   ├── OA_HudIndicator.c         Visualización del estado en el HUD
│   ├── OA_InputHandler.c         Manejo de entrada de teclado
│   ├── OA_DataStructs.c          Definiciones de estructuras de datos
│   ├── OA_Config.c               Gestión de configuración
│   ├── OA_GameModeComponent.c    Inyección de componente en el GameMode
│   ├── OA_GameModeInjector.c     Inyección automática en el GameMode
│   └── OA_PlayerControllerInjector.c  Inyección en el controlador del jugador
├── UI/
│   ├── layouts/                  Archivos de layout del HUD
│   └── Textures/                 Texturas del indicador de estado
└── addon.gproj                   Archivo de proyecto Enfusion
```

### Licencia

[MIT](LICENSE)

### Proyectos relacionados

- **[OpenArma](https://github.com/ArgA-Reforger/OpenArma)** — Backend + frontend (plataforma Multi-Agent)
- **[OpenArma-Mod-MapExporter](https://github.com/ArgA-Reforger/OpenArma-Mod-MapExporter)** — Herramienta de exportación de datos de mapas desde el Workbench
- **[OpenArma-Mod-MapScanner](https://github.com/ArgA-Reforger/OpenArma-Mod-MapScanner)** — Herramienta de escaneo de mapas dentro del juego

---

## English

### Introduction

OpenArma Mod Main is the core runtime mod for [OpenArma](https://github.com/ArgA-Reforger/OpenArma) inside Arma Reforger. It implements the AI commander's full operating loop: observe the battlefield → report the situation → receive orders → execute commands.

The mod itself makes no decisions — it's an **Agent MCP (Model Context Protocol) client** that talks to the OpenArma backend over REST API, while the backend's Multi-Agent engine drives all tactical decisions.

### Architecture

```
OA_Main (singleton controller)
  ├── OA_WorldObserver     Observation: scans the battlefield, builds a JSON situation report
  ├── OA_DecisionBridge    Communication: REST API send/receive (heartbeat polling)
  ├── OA_CommandExecutor   Execution: turns AI orders into in-game Waypoints
  ├── OA_EventTracker      Events: tracks combat events (engagement/casualty/contact)
  ├── OA_HudIndicator      UI: connection status / running state / faction display
  └── OA_Config            Config: API URL / key / decision interval
```

### Runtime flow

```
1. Login     Player enters the API Key → connection established with the backend
2. Heartbeat Every N seconds: WorldObserver scans → builds situation JSON → sends to backend
3. Response  Backend replies with: pending orders + latest config (start/stop, faction, interval)
4. Execute   CommandExecutor turns JSON orders into game Waypoints (move/defend/patrol/attack)
5. Loop      Repeat 2-4 until Logout
```

### Core features

- **Multi-faction support**: dynamic N-side configuration (AI vs human / AI vs AI), each side with its own control mode
- **Situation reports**: automatic scanning of friendly/enemy positions, composition, weapons, health, ammo, current task
- **Command execution**: supports `move` / `defend` / `patrol` / `attack` / `regroup` and other tactical commands
- **Combat event tracking**: engagement start/end, unit deaths, enemy contacts, and other reported events
- **Reconnection**: exponential backoff retry, up to 10 automatic reconnection attempts
- **HUD display**: connection status indicator (green/yellow/red/gray) + current faction configuration

### Installation

1. Import this mod in the Arma Reforger Workbench
2. Add `OA_GameModeInjector` or `OA_GameModeComponent` to the scene's GameMode
3. Make sure the OpenArma backend is running
4. Enter the API Key in-game to connect

### File overview

```
OA/
├── Scripts/Game/OA/
│   ├── OA_Main.c                 Singleton controller: Login → Heartbeat → Logout
│   ├── OA_WorldObserver.c        Battlefield situation scanning & JSON build
│   ├── OA_DecisionBridge.c       REST API communication layer
│   ├── OA_CommandExecutor.c      AI orders → game Waypoint conversion
│   ├── OA_EventTracker.c         Combat event tracking
│   ├── OA_HudIndicator.c         HUD status display
│   ├── OA_InputHandler.c         Keyboard input handling
│   ├── OA_DataStructs.c          Data structure definitions
│   ├── OA_Config.c               Config management
│   ├── OA_GameModeComponent.c    GameMode component injection
│   ├── OA_GameModeInjector.c     GameMode auto-injection
│   └── OA_PlayerControllerInjector.c  Player controller injection
├── UI/
│   ├── layouts/                  HUD layout files
│   └── Textures/                 Status indicator textures
└── addon.gproj                   Enfusion project file
```

### License

[MIT](LICENSE)

### Related projects

- **[OpenArma](https://github.com/ArgA-Reforger/OpenArma)** — Backend + frontend (Multi-Agent platform)
- **[OpenArma-Mod-MapExporter](https://github.com/ArgA-Reforger/OpenArma-Mod-MapExporter)** — Workbench map data export tool
- **[OpenArma-Mod-MapScanner](https://github.com/ArgA-Reforger/OpenArma-Mod-MapScanner)** — In-game map scanning tool
