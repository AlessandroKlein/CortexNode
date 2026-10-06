# API-SPECIFICATION.md

> **Sistema:** Plataforma de Automatización Distribuida
> **Tipo:** Especificación técnica / Contrato de API
> **Estado:** Planificación
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-06
> **Compatibilidad:** ESP32 / ESP32-S3 / ESP32-C6 / ESP32-C5 / ESP32-H2 / ESP32-P4 y nodos compatibles

---

# 1. Propósito

Este documento define la especificación oficial de la **API de la Plataforma de Automatización Distribuida**.

La API permite que:

* la interfaz web;
* aplicaciones móviles futuras;
* Central;
* controladores de zona;
* otros nodos;
* herramientas de administración;
* sistemas externos;
* Home Assistant;
* aplicaciones de terceros;
* servicios de integración;

puedan consultar y controlar el sistema mediante una interfaz común y estable.

La API debe abstraer completamente la implementación física del sistema.

Un cliente no debería necesitar conocer:

* GPIO;
* dirección I²C;
* dirección SPI;
* registro Modbus;
* ID CAN;
* tipo de expansor;
* modelo exacto del sensor;
* PHY Ethernet;
* W5500;
* LAN8720;
* IP101;
* tareas FreeRTOS;
* arquitectura interna del firmware.

El cliente trabaja con conceptos lógicos:

```text
Site
Zone
Group
Device
Resource
Capability
Entity
Function
Scene
Automation
Mode
State
Command
Event
Integration
```

---

# 2. Objetivos

La API debe proporcionar:

1. Una interfaz REST uniforme.
2. Identificadores estables.
3. Comunicación local y distribuida.
4. Operación Local-First.
5. Compatibilidad con Central y nodos autónomos.
6. Consultas de estado en tiempo real.
7. Envío de comandos.
8. Automatizaciones.
9. Escenas.
10. Eventos.
11. Historial.
12. Configuración.
13. Diagnóstico.
14. Descubrimiento.
15. Integraciones externas.
16. Control de acceso.
17. Versionado.
18. Compatibilidad hacia atrás.
19. Operaciones asíncronas.
20. Comunicación WebSocket.
21. Posibilidad de utilizar MQTT u otros transportes.
22. Posibilidad futura de utilizar CBOR u otros formatos compactos.

---

# 3. Documentos relacionados

La API no define nuevamente el modelo completo del sistema.

Se apoya en:

```text
ARCHITECTURE.md
        │
        ▼
DATA-MODEL.md
        │
        ▼
DATA-SCHEMAS.md
        │
        ├──────────────┐
        ▼              ▼
SYSTEM-BUS.md    API-AUTHENTICATION-AUTHORIZATION.md
        │              │
        └──────┬───────┘
               ▼
       API-SPECIFICATION.md
               │
        ┌──────┼─────────┐
        ▼      ▼         ▼
      Web    Mobile   Integrations
```

### Responsabilidad de cada documento

| Documento                             | Responsabilidad                   |
| ------------------------------------- | --------------------------------- |
| `ARCHITECTURE.md`                     | Arquitectura general              |
| `DATA-MODEL.md`                       | Conceptos y relaciones            |
| `DATA-SCHEMAS.md`                     | Estructuras serializadas          |
| `SYSTEM-BUS.md`                       | Comunicación interna              |
| `API-AUTHENTICATION-AUTHORIZATION.md` | Autenticación y permisos          |
| `API-SPECIFICATION.md`                | Interfaz externa de la plataforma |

---

# 4. Principios fundamentales

## 4.1 Local-First

La API debe poder funcionar sin Internet.

```text
Internet
   │
   X
   │
Central ───── Nodes
   │
   └──── Local API
```

La pérdida de Internet no debe impedir:

* consultar dispositivos localmente;
* controlar dispositivos localmente;
* ejecutar automatizaciones;
* utilizar escenas;
* utilizar funciones críticas;
* acceder a la interfaz web local.

---

# 5.2 La API no depende de Internet

La API puede estar disponible mediante:

* Ethernet;
* Wi-Fi;
* red local;
* Access Point del dispositivo;
* Central;
* controlador de zona;
* nodo individual.

Internet solamente es necesario para funciones externas.

---

# 5.3 La API no es el System Bus

La API y el System Bus son capas diferentes.

```text
                 CLIENTES
                    │
             REST / WebSocket
                    │
                    ▼
                  API
                    │
                    ▼
              DATA MODEL
                    │
                    ▼
               SYSTEM BUS
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Node      Zone      Central
```

La API está orientada a clientes.

El System Bus está orientado a comunicación interna entre componentes.

---

# 6. Modelo de abstracción

La API utiliza la siguiente jerarquía:

```text
Site
 │
 ├── Zone
 │    │
 │    ├── Device
 │    │     │
 │    │     ├── Resource
 │    │     │
 │    │     └── Entity
 │    │
 │    └── Group
 │
 ├── Function
 ├── Scene
 ├── Automation
 └── Mode
```

Las entidades constituyen la principal interfaz de control.

Ejemplos:

```text
light.living
light.kitchen
switch.pump
fan.bedroom
sensor.living.temperature
sensor.living.humidity
binary_sensor.front_door
cover.garage
climate.house
energy.house
```

---

# 7. Arquitectura de la API

## 7.1 Arquitectura lógica

```text
Client
  │
  ▼
HTTP / WebSocket
  │
  ▼
API Server
  │
  ├── Authentication
  │
  ├── Authorization
  │
  ├── Validation
  │
  ├── Rate Limiting
  │
  └── API Service
          │
          ▼
      Data Model
          │
          ▼
      System Bus
          │
    ┌─────┼─────┐
    ▼     ▼     ▼
 Central Zone Node
```

---

# 8. Base URL

La API utilizará:

```text
/api/v1
```

Ejemplo:

```text
http://192.168.1.100/api/v1
```

o:

```text
https://central.local/api/v1
```

La API no debe depender de un puerto específico en el diseño lógico.

La implementación podrá utilizar:

* `80` HTTP;
* `443` HTTPS;
* otro puerto configurable;
* mDNS;
* hostname local.

---

# 9. Versionado

La versión principal de la API estará incluida en la URL.

```text
/api/v1
/api/v2
```

Los cambios compatibles no requieren incrementar la versión principal.

### Ejemplo

Agregar un campo opcional:

```json
{
  "name": "Living",
  "description": "Sala principal"
}
```

es compatible.

Cambiar:

```text
entity_id
```

por:

```text
id
```

no es compatible y requiere una nueva versión.

---

# 10. Versiones independientes

Existen varios niveles de versión:

```text
API Version
    │
    ├── Data Schema Version
    │
    ├── Firmware Version
    │
    └── Integration Version
```

No deben confundirse.

Ejemplo:

```text
API:            v1
Entity Schema:  1.2.0
Firmware:       4.7.3
Integration:    2.1.0
```

---

# 11. Formato principal

El formato principal será:

```text
application/json
```

Codificación:

```text
UTF-8
```

El JSON será el formato principal para:

* REST;
* WebSocket;
* configuración;
* administración;
* integraciones.

---

# 12. Formatos futuros

La arquitectura permitirá:

```text
JSON
CBOR
MessagePack
Protobuf
```

Sin modificar el modelo lógico.

Por ejemplo:

```text
Data Model
    │
    ├── JSON
    ├── CBOR
    └── Binary
```

Esto permitirá utilizar formatos más eficientes en:

* System Bus;
* enlaces de baja velocidad;
* nodos con recursos limitados;
* telemetría masiva.

---

# 13. Headers

## 13.1 Headers generales

Los clientes deberían utilizar:

```http
Content-Type: application/json
Accept: application/json
```

---

# 13.2 Request ID

Cada solicitud puede incluir:

```http
X-Request-ID: req_01J...
```

El servidor debe devolverlo en la respuesta.

Sirve para:

* debugging;
* logs;
* correlación;
* diagnóstico;
* trazabilidad.

---

# 13.3 Idempotency-Key

Las operaciones que puedan generar efectos físicos deben soportar:

```http
Idempotency-Key: 7f8d...
```

Ejemplo:

```http
POST /api/v1/entities/switch.pump/commands
Idempotency-Key: cmd-client-12345
```

Esto evita que una misma solicitud sea ejecutada dos veces debido a:

* reintentos;
* pérdida de conexión;
* timeout;
* duplicación de paquetes.

---

# 13.4 Concurrencia

Para operaciones de configuración podrá utilizarse:

```http
If-Match
If-None-Match
ETag
```

Esto permite detectar que la configuración cambió entre:

```text
GET
   ↓
modificación local
   ↓
PUT
```

---

# 14. Estructura de respuesta

Las respuestas exitosas deben ser simples y consistentes.

Ejemplo:

```json
{
  "data": {
    "entity_id": "light.living",
    "state": {
      "on": true
    }
  },
  "request_id": "req_01JABC"
}
```

---

# 15. Respuesta de colección

```json
{
  "data": [
    {
      "entity_id": "light.living",
      "state": {
        "on": true
      }
    },
    {
      "entity_id": "light.kitchen",
      "state": {
        "on": false
      }
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 2
  },
  "request_id": "req_01JABC"
}
```

---

# 16. Errores

Todas las APIs deben utilizar una estructura de error común.

```json
{
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "The requested entity does not exist.",
    "details": {},
    "retryable": false
  },
  "request_id": "req_01JABC"
}
```

---

# 17. Códigos HTTP

| HTTP  | Uso                                       |
| ----- | ----------------------------------------- |
| `200` | Operación exitosa                         |
| `201` | Recurso creado                            |
| `202` | Operación aceptada/asíncrona              |
| `204` | Operación exitosa sin contenido           |
| `400` | Solicitud inválida                        |
| `401` | No autenticado                            |
| `403` | Sin permisos                              |
| `404` | Recurso inexistente                       |
| `409` | Conflicto                                 |
| `412` | Condición de concurrencia no cumplida     |
| `422` | Datos semánticamente inválidos            |
| `429` | Rate limit                                |
| `500` | Error interno                             |
| `502` | Error de comunicación con otro componente |
| `503` | Servicio temporalmente no disponible      |
| `504` | Timeout                                   |

---

# 18. Códigos de error

Los códigos de error son estables y no deben depender del texto de `message`.

Ejemplos:

```text
INVALID_REQUEST
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
AUTHORIZATION_DENIED

RESOURCE_NOT_FOUND
ENTITY_NOT_FOUND
DEVICE_NOT_FOUND

ENTITY_UNAVAILABLE
DEVICE_OFFLINE
NODE_UNREACHABLE

COMMAND_REJECTED
COMMAND_FAILED
COMMAND_TIMEOUT
COMMAND_EXPIRED

CONFIGURATION_INVALID
CONFIGURATION_CONFLICT

INTEGRATION_UNAVAILABLE
SYSTEM_BUSY
RATE_LIMIT_EXCEEDED
```

Los clientes deben utilizar `code`, no analizar `message`.

---

# 19. Recursos principales

La API debe proporcionar como mínimo:

```text
/sites
/zones
/groups
/nodes
/devices
/resources
/capabilities
/entities
/functions
/scenes
/automations
/modes
/events
/history
/commands
/integrations
/discovery
/config
/system
/diagnostics
/health
```

---

# 20. Sites

## Obtener instalaciones

```http
GET /api/v1/sites
```

## Obtener una instalación

```http
GET /api/v1/sites/{site_id}
```

## Crear

```http
POST /api/v1/sites
```

## Modificar

```http
PATCH /api/v1/sites/{site_id}
```

## Eliminar

```http
DELETE /api/v1/sites/{site_id}
```

---

# 21. Zones

```http
GET /api/v1/zones
GET /api/v1/zones/{zone_id}
POST /api/v1/zones
PATCH /api/v1/zones/{zone_id}
DELETE /api/v1/zones/{zone_id}
```

Filtros:

```text
?site_id=site_home
?parent_zone_id=zone_floor1
```

---

# 22. Groups

```http
GET /api/v1/groups
GET /api/v1/groups/{group_id}
POST /api/v1/groups
PATCH /api/v1/groups/{group_id}
DELETE /api/v1/groups/{group_id}
```

Un grupo puede contener:

```text
Entities
Devices
Functions
```

según su tipo.

---

# 23. Nodes

Un Node representa una instancia física de firmware/controlador.

Ejemplo:

```text
node.central
node.living
node.garden
node.garage
```

Endpoints:

```http
GET /api/v1/nodes
GET /api/v1/nodes/{node_id}
```

Información:

```json
{
  "node_id": "node.living",
  "name": "Controlador Living",
  "status": "online",
  "firmware_version": "1.4.2",
  "hardware_profile": "NODE_ETH_S3_W5500_REV_A",
  "uptime_s": 152034,
  "last_seen": "2026-10-06T12:00:00Z"
}
```

---

# 24. Devices

```http
GET /api/v1/devices
GET /api/v1/devices/{device_id}
POST /api/v1/devices
PATCH /api/v1/devices/{device_id}
DELETE /api/v1/devices/{device_id}
```

Filtros:

```text
?node_id=node.living
?zone_id=zone.living
?status=online
```

---

# 25. Resources

Los recursos representan hardware o servicios internos.

Ejemplos:

```text
GPIO
PWM
I2C
SPI
UART
RS485
CAN
Ethernet
Wi-Fi
Sensor
Relay
Display
Touch
Camera
Storage
```

Endpoints:

```http
GET /api/v1/resources
GET /api/v1/resources/{resource_id}
```

Los recursos podrán incluir información de hardware.

Sin embargo, la API normal de automatización no debería requerirla.

---

# 26. Capabilities

```http
GET /api/v1/capabilities
GET /api/v1/capabilities/{capability_id}
```

Ejemplos:

```text
on_off
brightness
color
temperature
humidity
pressure
motion
occupancy
position
speed
power
energy
voltage
current
```

---

# 27. Entities

Las entidades son uno de los recursos más importantes de toda la API.

```http
GET /api/v1/entities
GET /api/v1/entities/{entity_id}
```

Ejemplo:

```http
GET /api/v1/entities/light.living
```

---

# 28. Filtros de Entities

La consulta podrá utilizar:

```text
?site_id=site_home
?zone_id=zone_living
?device_id=device_living
?domain=light
?capability=brightness
?available=true
```

Ejemplo:

```http
GET /api/v1/entities?zone_id=zone_living&domain=light
```

---

# 29. Estado de una Entity

```http
GET /api/v1/entities/{entity_id}/state
```

Ejemplo:

```json
{
  "data": {
    "entity_id": "sensor.living.temperature",
    "state": {
      "value": 23.7,
      "unit": "°C"
    },
    "quality": "good",
    "timestamp": "2026-10-06T12:30:00Z"
  }
}
```

---

# 30. Estado deseado y estado real

Los actuadores deben poder distinguir:

```text
Desired State
      │
      ▼
Command
      │
      ▼
Actual State
```

Ejemplo:

```json
{
  "state": {
    "actual": {
      "on": false
    },
    "desired": {
      "on": true
    }
  }
}
```

Esto es importante para sistemas distribuidos.

Puede existir una diferencia temporal entre:

```text
desired = true
actual = false
```

debido a:

* latencia;
* nodo desconectado;
* protección;
* actuador ocupado;
* error físico.

---

# 31. Commands

Los comandos se ejecutan mediante:

```http
POST /api/v1/entities/{entity_id}/commands
```

Ejemplo:

```json
{
  "command": "turn_on",
  "parameters": {}
}
```

Para una luz:

```json
{
  "command": "turn_on",
  "parameters": {
    "brightness": 80
  }
}
```

---

# 32. Respuesta de comandos

Los comandos distribuidos normalmente son asíncronos.

Por ello se recomienda:

```http
202 Accepted
```

Ejemplo:

```json
{
  "data": {
    "command_id": "cmd_01JABC",
    "request_id": "req_01JXYZ",
    "status": "accepted",
    "status_url": "/api/v1/commands/cmd_01JABC"
  }
}
```

---

# 33. Consulta de comando

```http
GET /api/v1/commands/{command_id}
```

Respuesta:

```json
{
  "data": {
    "command_id": "cmd_01JABC",
    "entity_id": "light.living",
    "command": "turn_on",
    "status": "executed",
    "requested_at": "2026-10-06T12:30:00Z",
    "completed_at": "2026-10-06T12:30:01Z"
  }
}
```

---

# 34. Ciclo de vida del comando

Los estados oficiales son:

```text
requested
    ↓
authorized
    ↓
accepted
    ↓
queued
    ↓
executing
    ↓
executed
```

Estados alternativos:

```text
rejected
failed
timeout
cancelled
expired
```

---

# 35. Cancelación

Los comandos que lo permitan podrán cancelarse:

```http
POST /api/v1/commands/{command_id}/cancel
```

No todos los comandos son cancelables.

Por ejemplo:

```text
turn_on
```

podría ser instantáneo.

Mientras que:

```text
move_cover
run_irrigation
run_motor
```

pueden ser cancelables.

---

# 36. Events

Los eventos son hechos ocurridos en el sistema.

```http
GET /api/v1/events
GET /api/v1/events/{event_id}
```

Filtros:

```text
?entity_id=light.living
?device_id=device.living
?event_type=entity.state_changed
?from=...
?to=...
```

---

# 37. Diferencia entre State y Event

Un estado responde:

> ¿Cómo está actualmente algo?

Un evento responde:

> ¿Qué ocurrió?

Ejemplo:

```text
STATE
light.living = ON
```

Evento:

```text
light.living.changed
OFF → ON
```

El evento es histórico e inmutable.

El estado representa la condición actual.

---

# 38. History

```http
GET /api/v1/entities/{entity_id}/history
```

Parámetros:

```text
from
to
limit
offset
interval
aggregation
```

Ejemplo:

```http
GET /api/v1/entities/sensor.living.temperature/history?from=2026-10-06T00:00:00Z&to=2026-10-06T12:00:00Z
```

---

# 39. Agregaciones

Para grandes cantidades de datos:

```text
raw
average
minimum
maximum
sum
count
delta
first
last
```

Ejemplo:

```http
?aggregation=average&interval=5m
```

---

# 40. Telemetría

Los nodos podrán enviar lotes de datos.

```http
POST /api/v1/telemetry
```

Ejemplo:

```json
{
  "source_id": "node.weather",
  "samples": [
    {
      "entity_id": "sensor.weather.temperature",
      "timestamp": "2026-10-06T12:00:00Z",
      "value": 22.4,
      "unit": "°C"
    },
    {
      "entity_id": "sensor.weather.humidity",
      "timestamp": "2026-10-06T12:00:00Z",
      "value": 61.2,
      "unit": "%"
    }
  ]
}
```

La telemetría debe admitir procesamiento por lotes para reducir:

* CPU;
* memoria;
* tráfico;
* consumo energético.

---

# 41. Functions

Las Functions representan funciones lógicas.

```http
GET /api/v1/functions
GET /api/v1/functions/{function_id}
```

Ejemplos:

```text
function.lighting
function.irrigation
function.heating
function.security
function.energy
```

---

# 42. Scenes

Una escena representa un conjunto de estados deseados.

```http
GET /api/v1/scenes
GET /api/v1/scenes/{scene_id}
POST /api/v1/scenes
PATCH /api/v1/scenes/{scene_id}
DELETE /api/v1/scenes/{scene_id}
```

Activación:

```http
POST /api/v1/scenes/{scene_id}/activate
```

Ejemplo:

```json
{
  "entities": [
    {
      "entity_id": "light.living",
      "state": {
        "on": true,
        "brightness": 30
      }
    },
    {
      "entity_id": "light.kitchen",
      "state": {
        "on": false
      }
    }
  ]
}
```

---

# 43. Automations

Endpoints:

```http
GET /api/v1/automations
GET /api/v1/automations/{automation_id}
POST /api/v1/automations
PATCH /api/v1/automations/{automation_id}
DELETE /api/v1/automations/{automation_id}
```

Control:

```http
POST /api/v1/automations/{automation_id}/enable
POST /api/v1/automations/{automation_id}/disable
POST /api/v1/automations/{automation_id}/trigger
POST /api/v1/automations/{automation_id}/validate
```

---

# 44. Estructura de Automation

Una automatización está compuesta por:

```text
Trigger
   ↓
Conditions
   ↓
Actions
```

Ejemplo:

```json
{
  "automation_id": "automation.night_lighting",
  "enabled": true,
  "trigger": [
    {
      "type": "state_change",
      "entity_id": "binary_sensor.hall.motion"
    }
  ],
  "conditions": [
    {
      "type": "mode",
      "mode": "sleep"
    }
  ],
  "actions": [
    {
      "type": "command",
      "entity_id": "light.hall",
      "command": "turn_on",
      "parameters": {
        "brightness": 15
      }
    }
  ]
}
```

---

# 45. Modes

Los modos globales se exponen mediante:

```http
GET /api/v1/modes
GET /api/v1/modes/{mode_id}
```

Ejemplos:

```text
normal
sleep
away
vacation
maintenance
emergency
```

El modo activo podrá consultarse mediante:

```http
GET /api/v1/system/mode
```

Cambiarlo:

```http
POST /api/v1/system/mode
```

Ejemplo:

```json
{
  "mode": "sleep"
}
```

---

# 46. Configuración

La configuración debe separarse del estado operativo.

```http
GET /api/v1/config
GET /api/v1/config/status
PATCH /api/v1/config
POST /api/v1/config/validate
POST /api/v1/config/apply
POST /api/v1/config/rollback
```

---

# 47. Desired Configuration vs Applied Configuration

El sistema debe poder diferenciar:

```text
Desired Configuration
        │
        ▼
Validation
        │
        ▼
Apply
        │
        ▼
Applied Configuration
```

Ejemplo:

```json
{
  "config_version": 15,
  "desired_config_version": 16,
  "applied_config_version": 15,
  "status": "pending"
}
```

Esto es especialmente importante en sistemas distribuidos.

---

# 48. Configuración distribuida

Cuando Central modifica la configuración de un nodo:

```text
Central
  │
  ▼
Desired Config
  │
  ▼
System Bus
  │
  ▼
Node
  │
  ├── Validate
  ├── Apply
  └── Confirm
```

El nodo debe validar localmente la configuración antes de aplicarla.

---

# 49. Hardware Configuration

La API podrá exponer configuración de hardware mediante endpoints administrativos.

Ejemplo:

```http
GET /api/v1/nodes/{node_id}/hardware
```

Podrá mostrar:

```text
MCU
Board
Hardware Profile
Resources
Buses
Expanders
Sensors
Actuators
Displays
Storage
Network
```

Sin embargo, las interfaces normales de automatización no deben depender de estos datos.

---

# 50. Build Profile

El Build Profile representa la configuración utilizada durante compilación.

Ejemplo:

```text
BOARD_ESP32_WROOM
BOARD_ESP32_S3
BOARD_ESP32_C6
```

Puede coexistir con:

```text
ETH_LAN8720
ETH_W5500
CAN_NATIVE
RS485
DISPLAY
CAMERA
```

El Build Profile no debe confundirse con la configuración lógica de la API.

---

# 51. Discovery

El descubrimiento permite identificar nodos nuevos.

```http
GET /api/v1/discovery
POST /api/v1/discovery/scan
GET /api/v1/discovery/nodes
```

Un nodo descubierto puede proporcionar:

```text
node_id
hardware_profile
firmware_version
capabilities
resources
network
status
```

---

# 52. Provisioning

El proceso recomendado:

```text
Unknown Node
     ↓
Discovery
     ↓
Authentication
     ↓
Provisioning
     ↓
Configuration
     ↓
Validation
     ↓
Active
```

Endpoints:

```http
POST /api/v1/discovery/{node_id}/approve
POST /api/v1/discovery/{node_id}/provision
POST /api/v1/discovery/{node_id}/reject
```

---

# 53. Health

Endpoint general:

```http
GET /api/v1/health
```

Endpoints especializados:

```http
GET /api/v1/health/live
GET /api/v1/health/ready
```

### Liveness

Indica:

> ¿El proceso está funcionando?

### Readiness

Indica:

> ¿El sistema está preparado para aceptar operaciones?

---

# 54. Diagnostics

```http
GET /api/v1/diagnostics
GET /api/v1/diagnostics/nodes/{node_id}
GET /api/v1/diagnostics/devices/{device_id}
```

Puede incluir:

```text
uptime
free_heap
minimum_free_heap
cpu_usage
task_status
watchdog
temperature
storage
network
bus_errors
communication_errors
sensor_errors
restart_reason
```

---

# 55. System

```http
GET /api/v1/system
GET /api/v1/system/info
GET /api/v1/system/status
GET /api/v1/system/time
GET /api/v1/system/mode
```

Información:

```json
{
  "data": {
    "name": "Central",
    "firmware_version": "1.0.0",
    "schema_version": "1.0.0",
    "api_version": "v1",
    "uptime_s": 123456,
    "mode": "normal"
  }
}
```

---

# 56. Integrations

```http
GET /api/v1/integrations
GET /api/v1/integrations/{integration_id}
POST /api/v1/integrations
PATCH /api/v1/integrations/{integration_id}
DELETE /api/v1/integrations/{integration_id}
```

Operaciones:

```http
POST /api/v1/integrations/{integration_id}/enable
POST /api/v1/integrations/{integration_id}/disable
POST /api/v1/integrations/{integration_id}/test
POST /api/v1/integrations/{integration_id}/sync
```

---

# 57. Integraciones soportadas

La arquitectura deberá permitir:

```text
Matter
MQTT
Home Assistant
Homey
Apple Home
Google Home
Amazon Alexa
Samsung SmartThings
REST
WebSocket
```

La integración no debe modificar el modelo interno.

```text
Internal Entity
      │
      ▼
Integration Adapter
      │
      ▼
External Platform
```

---

# 58. WebSocket

La API debe proporcionar comunicación en tiempo real.

Endpoint:

```text
/api/v1/ws
```

El WebSocket permite recibir:

```text
state_changed
entity_created
entity_removed
device_online
device_offline
command_status
event
alarm
diagnostic
configuration_changed
```

---

# 59. Suscripciones WebSocket

El cliente podrá solicitar filtros.

Ejemplo:

```json
{
  "type": "subscribe",
  "filters": {
    "entities": [
      "light.living",
      "sensor.living.temperature"
    ]
  }
}
```

O:

```json
{
  "type": "subscribe",
  "filters": {
    "zones": [
      "zone.living"
    ]
  }
}
```

---

# 60. Mensaje WebSocket

```json
{
  "type": "event",
  "event_type": "entity.state_changed",
  "timestamp": "2026-10-06T12:30:00Z",
  "entity_id": "light.living",
  "data": {
    "old_state": {
      "on": false
    },
    "new_state": {
      "on": true
    }
  }
}
```

---

# 61. Paginación

Las colecciones deben admitir:

```text
limit
offset
```

Ejemplo:

```http
GET /api/v1/entities?limit=50&offset=100
```

En implementaciones futuras podrán agregarse:

```text
cursor
next_cursor
previous_cursor
```

Los cursores son preferibles para grandes volúmenes de datos.

---

# 62. Ordenamiento

Las colecciones podrán soportar:

```text
sort
order
```

Ejemplo:

```http
GET /api/v1/events?sort=timestamp&order=desc
```

---

# 63. Filtrado

Los recursos deberán permitir filtros específicos.

Ejemplo:

```http
GET /api/v1/entities?domain=sensor&zone_id=zone.garden
```

Los filtros desconocidos deben generar:

```text
400 INVALID_REQUEST
```

en lugar de ser silenciosamente ignorados cuando puedan cambiar el resultado esperado.

---

# 64. Búsqueda

Podrá existir:

```http
GET /api/v1/search?q=living
```

La búsqueda puede incluir:

```text
entity_id
name
alias
device_id
zone_id
tags
```

---

# 65. Commands vs PATCH State

Un cliente no debería modificar directamente el estado operativo de una entidad mediante:

```http
PATCH /entities/light.living/state
```

Para actuadores debe utilizar:

```http
POST /entities/light.living/commands
```

Esto permite:

* autorización;
* validación;
* trazabilidad;
* idempotencia;
* prioridades;
* ejecución distribuida;
* eventos;
* seguridad.

---

# 66. Operaciones inmediatas

Si una operación puede ejecutarse inmediatamente, puede responder:

```text
200 OK
```

Ejemplo:

```json
{
  "data": {
    "command_id": "cmd_123",
    "status": "executed"
  }
}
```

---

# 67. Operaciones distribuidas

Cuando el resultado depende de otro nodo:

```text
202 Accepted
```

Ejemplo:

```text
API
 ↓
Central
 ↓
Zone Controller
 ↓
Node
 ↓
Actuator
```

El cliente no debe quedarse bloqueado esperando indefinidamente.

---

# 68. Timeouts

Los clientes y servicios deberán utilizar timeouts.

Ejemplo:

```text
API request timeout
       ↓
Command continues
       ↓
Command status available
```

Un timeout HTTP no implica necesariamente que el comando haya fallado.

Por eso debe consultarse:

```http
GET /api/v1/commands/{command_id}
```

---

# 69. Prioridad

Los comandos podrán tener:

```text
critical
high
normal
low
background
```

Prioridad recomendada:

```text
Safety
  ↓
Emergency
  ↓
Critical Automation
  ↓
User Command
  ↓
Normal Automation
  ↓
Telemetry
  ↓
Background
```

---

# 70. Seguridad

La seguridad de la API está definida en:

```text
API-AUTHENTICATION-AUTHORIZATION.md
```

La API deberá soportar, según implementación:

```text
Authentication
Authorization
Roles
Scopes
Tokens
Sessions
API Keys
Device Credentials
```

---

# 71. Principio de mínimo privilegio

Una aplicación que solamente necesita leer temperatura no debería poder:

```text
controlar relays
modificar configuración
crear usuarios
actualizar firmware
```

Ejemplo de scopes:

```text
entities:read
entities:control
devices:read
devices:configure
automation:read
automation:write
system:read
system:admin
```

---

# 72. Roles

Ejemplos:

```text
viewer
user
operator
technician
administrator
system
```

Los roles pueden variar según instalación.

---

# 73. Protección de endpoints críticos

Endpoints como:

```text
/config
/system/reboot
/system/reset
/firmware
/users
/security
/hardware
```

requieren permisos elevados.

---

# 74. Rate Limiting

La API debe poder limitar:

```text
requests/second
commands/minute
login attempts
discovery requests
configuration changes
```

Los límites deben poder variar según:

```text
usuario
cliente
IP
nodo
endpoint
rol
```

---

# 75. Caching

Las consultas de recursos relativamente estáticos pueden utilizar:

```http
ETag
If-None-Match
Cache-Control
```

Especialmente:

```text
devices
resources
capabilities
hardware profiles
integrations
```

El estado en tiempo real no debe depender exclusivamente de cache.

---

# 76. Consistencia distribuida

La API debe diferenciar:

```text
Local State
Central State
Desired State
Actual State
Last Known State
```

Nunca debe presentarse un estado antiguo como si fuera necesariamente actual.

Ejemplo:

```json
{
  "state": {
    "value": true,
    "quality": "stale",
    "timestamp": "2026-10-06T11:20:00Z"
  }
}
```

---

# 77. Calidad del dato

Los estados podrán utilizar:

```text
good
uncertain
stale
invalid
unavailable
```

Esto es especialmente importante para:

* sensores;
* nodos desconectados;
* datos históricos;
* integraciones;
* automatizaciones.

---

# 78. Timestamp

Los timestamps deben utilizar ISO 8601 / RFC 3339.

Ejemplo:

```text
2026-10-06T12:30:00Z
```

Internamente, los dispositivos pueden utilizar:

```text
Unix timestamp
monotonic timer
RTC
```

pero la API debe proporcionar un formato uniforme.

---

# 79. Tiempo sin sincronización

Un nodo que todavía no tenga NTP/RTC válido no debe inventar una fecha.

Debe indicar:

```text
time_valid = false
```

o una calidad apropiada.

El sistema debe poder diferenciar:

```text
timestamp válido
timestamp aproximado
timestamp desconocido
```

---

# 80. Unidades

Las unidades deben ser explícitas.

Ejemplos:

```text
Temperature → °C
Pressure → Pa
Voltage → V
Current → A
Power → W
Energy → Wh
Frequency → Hz
Speed → m/s
Flow → L/min
```

Las integraciones pueden convertir las unidades según sus necesidades.

---

# 81. Identificadores

Los identificadores lógicos deben ser estables.

Ejemplo:

```text
light.living
sensor.living.temperature
switch.irrigation
```

El cambio de:

```text
GPIO12
```

a:

```text
GPIO27
```

no debería cambiar:

```text
switch.irrigation
```

---

# 82. IDs internos

Los objetos también pueden tener IDs internos:

```text
UUID
ULID
```

Ejemplo:

```json
{
  "id": "01K...",
  "entity_id": "light.living"
}
```

El:

```text
id
```

identifica internamente el objeto.

El:

```text
entity_id
```

representa su identidad lógica.

---

# 83. Compatibilidad con hardware

La API no debe depender de:

```text
ESP32
ESP32-S3
ESP32-C6
W5500
LAN8720
MCP23017
74HC595
RS485
CAN
```

Esos datos pertenecen al nivel:

```text
Hardware
Resource
Device
```

No al nivel de automatización.

---

# 84. Ejemplo de abstracción

Un usuario solicita:

```http
POST /api/v1/entities/switch.pump/commands
```

El sistema puede ejecutar:

```text
Entity
   ↓
Capability
   ↓
Device
   ↓
Resource
   ↓
MCP23017
   ↓
I²C
   ↓
GPIO lógico
   ↓
Relay
```

Otro equipo podría ejecutar exactamente el mismo comando mediante:

```text
ESP32 GPIO
74HC595
CAN
RS485
Modbus
Ethernet
```

La API no cambia.

---

# 85. API del Node

Un nodo puede proporcionar una API local.

Ejemplo:

```text
Node
 └── /api/v1
```

Puede funcionar sin Central.

Debe permitir al menos:

```text
GET entities
GET state
POST commands
GET diagnostics
GET configuration status
```

---

# 86. API del Central

Central proporciona una vista agregada.

```text
Central
 │
 ├── Node A
 ├── Node B
 ├── Node C
 └── Node D
```

La API de Central puede devolver:

```http
GET /api/v1/entities
```

con entidades provenientes de múltiples nodos.

---

# 87. Agregación

Ejemplo:

```http
GET /api/v1/entities?zone_id=house
```

Central puede reunir:

```text
Node Living
Node Kitchen
Node Garage
Node Garden
```

y devolver una colección unificada.

El cliente no necesita consultar cada nodo.

---

# 88. Falla del Central

Si Central falla:

```text
Central X
   │
   ├── Zone A ✓
   ├── Zone B ✓
   └── Nodes ✓
```

Los nodos deben continuar con las funciones que puedan ejecutar localmente.

Cuando Central vuelva:

```text
Central
   ↓
Discovery
   ↓
State Synchronization
   ↓
Configuration Reconciliation
   ↓
Normal Operation
```

---

# 89. Reconciliación

Cuando se reconecta un nodo:

```text
Central Desired Configuration
              │
              ▼
        Node Configuration
              │
              ▼
          Compare
          /     \
       Equal   Different
         │        │
         │        ▼
         │      Apply
         │        │
         └────────┘
              │
              ▼
          Confirm
```

La API podrá mostrar:

```text
synchronized
pending
conflict
error
```

---

# 90. API y System Bus

Un comando recibido por API puede convertirse internamente en un mensaje del System Bus.

```text
HTTP
 ↓
API
 ↓
Command
 ↓
System Bus
 ↓
Node
```

Una actualización de estado puede recorrer el camino inverso:

```text
Node
 ↓
Event
 ↓
System Bus
 ↓
Data Model
 ↓
API
 ↓
WebSocket
```

---

# 91. API y MQTT

MQTT es un transporte/integración.

No debe convertirse en el modelo principal.

Ejemplo:

```text
Entity
  ↓
Internal Event
  ↓
MQTT Adapter
  ↓
MQTT Topic
```

La misma entidad puede exponerse mediante:

```text
REST
WebSocket
MQTT
Matter
```

sin crear cuatro modelos diferentes.

---

# 92. API y Matter

Matter debe utilizar el modelo interno como fuente de verdad.

```text
Entity
 ↓
Capability
 ↓
Matter Adapter
 ↓
Matter Device
```

No:

```text
Matter → modelo interno → hardware
```

como modelo principal.

---

# 93. API y Home Assistant

Home Assistant puede conectarse mediante:

```text
Matter
MQTT
REST
WebSocket
```

La integración deberá mapear:

```text
Entity
Capability
State
Command
Event
```

al modelo de Home Assistant.

---

# 94. API para aplicaciones móviles

La misma API deberá poder ser utilizada posteriormente por:

```text
Android
iOS
Web
Desktop
```

Por ello no se debe diseñar una API exclusivamente para la interfaz web.

Ejemplo:

```text
Web UI ─────┐
Mobile ─────┤
Third Party ┤
            ▼
         API v1
            │
            ▼
        Data Model
```

---

# 95. API pública para terceros

La API debe permitir aplicaciones externas sin exponer información innecesaria.

Un tercero puede solicitar:

```text
entities:read
```

y recibir:

```text
temperature
humidity
power
light state
```

sin obtener:

```text
passwords
tokens
GPIO
hardware secrets
network credentials
```

---

# 96. API Tokens

Las aplicaciones externas podrán utilizar tokens limitados.

Ejemplo conceptual:

```json
{
  "token_id": "token_app_01",
  "scopes": [
    "entities:read"
  ],
  "expires_at": "2027-01-01T00:00:00Z"
}
```

Los secretos reales nunca deben almacenarse dentro de respuestas normales.

---

# 97. Secret References

Cuando un recurso necesita una credencial:

```json
{
  "password_ref": "secret:mqtt.password"
}
```

No:

```json
{
  "password": "MiPassword123"
}
```

La API debe evitar devolver secretos salvo operaciones administrativas específicamente autorizadas.

---

# 98. Validación

Toda entrada debe validarse antes de ejecutarse.

```text
Request
  ↓
Syntax Validation
  ↓
Schema Validation
  ↓
Authorization
  ↓
Semantic Validation
  ↓
Safety Validation
  ↓
Execution
```

Ejemplo:

```text
brightness = 150
```

debe rechazarse si el rango permitido es:

```text
0–100
```

---

# 99. Validación de seguridad física

La API no debe ser el único nivel de seguridad.

Ejemplo:

```text
API solicita:
heater = ON
```

Pero el nodo puede rechazarlo debido a:

```text
overtemperature
emergency_stop
hardware_fault
interlock
sensor_invalid
```

Por lo tanto:

> La autorización de software no reemplaza las protecciones locales de hardware o firmware.

---

# 100. Concurrencia

Si dos clientes intentan modificar simultáneamente un recurso:

```text
Client A ──┐
           ├── Entity
Client B ──┘
```

el sistema debe utilizar:

```text
version
ETag
If-Match
command ordering
```

cuando corresponda.

---

# 101. Orden de comandos

Los comandos sobre un mismo recurso pueden requerir orden.

Ejemplo:

```text
OPEN
STOP
CLOSE
```

El sistema no debe ejecutarlos arbitrariamente:

```text
CLOSE
OPEN
STOP
```

si la función requiere orden temporal.

El Command Manager deberá establecer políticas de:

```text
ordering
priority
queue
replacement
cancellation
```

---

# 102. Comandos redundantes

El sistema debería poder evitar comandos innecesarios.

Ejemplo:

```text
Entity already ON
Client → turn_on
```

Puede responder:

```text
executed
```

sin accionar físicamente el dispositivo nuevamente, siempre que el comportamiento de la entidad lo permita.

---

# 103. Escenas y comandos distribuidos

Una escena puede afectar varios nodos:

```text
Scene
 │
 ├── Node A → Light
 ├── Node B → Blind
 ├── Node C → HVAC
 └── Node D → Alarm
```

La API debe proporcionar un identificador de ejecución:

```text
scene_execution_id
```

para poder consultar el resultado global.

---

# 104. Automation Execution

Las automatizaciones también pueden generar múltiples comandos.

Debe poder existir:

```text
automation_execution_id
```

para diagnóstico.

Ejemplo:

```text
Automation triggered
       ↓
Execution ID
       ↓
Action 1
Action 2
Action 3
       ↓
Completed
```

---

# 105. Transactions

Las operaciones complejas podrán utilizar transacciones lógicas.

Ejemplo:

```text
Scene Activation
```

No necesariamente implica una transacción ACID tradicional.

Puede utilizar:

```text
best effort
rollback
compensation
partial success
```

Resultado:

```json
{
  "status": "partial_success",
  "successful": 3,
  "failed": 1
}
```

---

# 106. Estado parcial

En sistemas distribuidos no siempre es posible obtener un estado global simultáneo.

Por ello:

```text
Global State
```

debe entenderse como una vista agregada con timestamps y calidad individual.

Ejemplo:

```text
Living temperature → 22.4 °C → good
Garden temperature → 18.2 °C → good
Garage temperature → 21.1 °C → stale
```

---

# 107. Observabilidad

Toda operación importante debe poder rastrearse mediante:

```text
request_id
command_id
event_id
correlation_id
source
timestamp
```

Ejemplo:

```text
API Request
    │
request_id
    │
    ▼
Command
    │
command_id
    │
    ▼
System Bus
    │
correlation_id
    │
    ▼
Event
```

---

# 108. Logs

Los logs internos podrán asociarse a:

```text
request_id
command_id
event_id
node_id
device_id
entity_id
```

Esto permite reconstruir una operación completa.

---

# 109. Compatibilidad hacia adelante

Los clientes deben ignorar campos desconocidos cuando no sean necesarios para interpretar correctamente el mensaje.

Ejemplo:

```json
{
  "entity_id": "sensor.temp",
  "value": 22.5,
  "unit": "°C",
  "new_future_field": true
}
```

Un cliente antiguo puede ignorar:

```text
new_future_field
```

---

# 110. Campos obligatorios

Los campos obligatorios no deben eliminarse dentro de una versión compatible.

Para eliminar un campo:

```text
Deprecation
   ↓
Warning
   ↓
Migration period
   ↓
New API version
```

---

# 111. Enumeraciones

Las enumeraciones deben diseñarse para permitir futuras extensiones.

Por ejemplo:

```text
quality:
    good
    uncertain
    stale
    invalid
    unavailable
```

Los clientes deben manejar correctamente valores desconocidos.

---

# 112. Límites para dispositivos embebidos

La API debe poder ejecutarse en microcontroladores.

Por ello se deben evitar:

* respuestas gigantes;
* JSON sin límites;
* arrays ilimitados;
* recursividad innecesaria;
* consultas históricas ilimitadas;
* múltiples operaciones bloqueantes;
* asignaciones de memoria impredecibles.

Debe existir:

```text
max_request_size
max_response_size
max_entities_per_request
max_events_per_request
max_command_batch
```

---

# 113. Batch Operations

Cuando sea necesario, podrán agruparse operaciones.

Ejemplo:

```http
POST /api/v1/commands/batch
```

```json
{
  "commands": [
    {
      "entity_id": "light.living",
      "command": "turn_on"
    },
    {
      "entity_id": "light.kitchen",
      "command": "turn_off"
    }
  ]
}
```

La respuesta debe indicar el resultado individual.

---

# 114. API de Firmware

Las operaciones de firmware son administrativas y no forman parte del control normal.

Podrán incluir:

```http
GET /api/v1/firmware
GET /api/v1/nodes/{node_id}/firmware
POST /api/v1/nodes/{node_id}/firmware/update
```

Deben requerir permisos elevados.

---

# 115. Reset

Las operaciones destructivas deben estar separadas.

Ejemplo:

```http
POST /api/v1/system/reboot
POST /api/v1/system/reset/configuration
POST /api/v1/system/reset/factory
```

No debe existir un único endpoint ambiguo como:

```text
/reset
```

---

# 116. Confirmación de operaciones destructivas

Las operaciones críticas deben poder requerir:

```text
confirmation
authorization
re-authentication
```

Ejemplo:

```json
{
  "confirm": true
}
```

---

# 117. Discovery API

La API de discovery debe poder devolver:

```json
{
  "node_id": "node.garden",
  "hardware_profile": "NODE_ETH_WROOM_LAN8720_REV_A",
  "firmware_version": "1.2.0",
  "capabilities": [
    "temperature",
    "humidity",
    "relay"
  ],
  "status": "unprovisioned"
}
```

---

# 118. Health vs Availability

No deben confundirse.

```text
Node health
```

indica el estado del controlador.

```text
Entity availability
```

indica si una entidad puede utilizarse.

Un nodo puede estar:

```text
healthy
```

mientras un sensor conectado esté:

```text
unavailable
```

---

# 119. API de configuración de módulos

Los módulos instalados podrán consultarse:

```http
GET /api/v1/nodes/{node_id}/modules
GET /api/v1/modules
```

Administración:

```http
POST /api/v1/modules/{module_id}/enable
POST /api/v1/modules/{module_id}/disable
GET /api/v1/modules/{module_id}/config
PATCH /api/v1/modules/{module_id}/config
```

Esto coincide con el principio de módulos habilitables/deshabilitables desde la interfaz web.

---

# 120. API de permisos por módulo

Un módulo no debe recibir automáticamente permisos administrativos.

Debe declarar:

```text
required capabilities
required resources
required permissions
```

El sistema valida las dependencias antes de activarlo.

---

# 121. API de dispositivos virtuales

La API debe admitir entidades que no correspondan directamente a hardware.

Ejemplos:

```text
sensor.average_temperature
energy.house.total
binary_sensor.house_occupied
climate.house
scene.night
```

Esto permite crear lógica de alto nivel.

---

# 122. API de entidades calculadas

Ejemplo:

```text
sensor.house.average_temperature
```

puede calcularse a partir de:

```text
sensor.living.temperature
sensor.kitchen.temperature
sensor.bedroom.temperature
```

La API debe tratar el resultado como una entidad normal.

---

# 123. API de alarmas

Podrá utilizarse:

```http
GET /api/v1/alarms
GET /api/v1/alarms/{alarm_id}
POST /api/v1/alarms/{alarm_id}/acknowledge
POST /api/v1/alarms/{alarm_id}/clear
```

Las alarmas deben mantener:

```text
active
acknowledged
cleared
```

y su historial correspondiente.

---

# 124. API de energía

El modelo debe soportar:

```text
voltage
current
power
energy
frequency
power_factor
```

Ejemplos:

```text
sensor.house.power
sensor.house.energy
sensor.garage.current
```

---

# 125. API de agua

Debe poder representar:

```text
flow
volume
pressure
leak
valve
pump
```

Ejemplos:

```text
sensor.garden.flow
sensor.house.water_volume
binary_sensor.bathroom.leak
switch.garden.pump
```

---

# 126. API ambiental

Debe soportar:

```text
temperature
humidity
pressure
air_quality
CO2
PM1
PM2.5
PM10
VOC
illuminance
UV
wind
rain
```

---

# 127. API agrícola

El mismo modelo debe permitir:

```text
soil_moisture
soil_temperature
soil_ec
soil_ph
irrigation
valves
pumps
weather
crop_zone
```

sin modificar la arquitectura principal.

---

# 128. API industrial

También podrá representar:

```text
machine
motor
pump
valve
PLC
Modbus register
production counter
alarm
energy meter
```

Los detalles industriales deben permanecer en:

```text
Resource
Device
Capability
```

y no contaminar el modelo lógico general.

---

# 129. API marina

Podrá representar:

```text
bilge
pump
tank
battery
engine
temperature
pressure
GPS
wind
navigation
```

utilizando las mismas abstracciones.

---

# 130. API de cámara e IA

El sistema podrá exponer entidades relacionadas con visión:

```text
camera.front
binary_sensor.person_detected
binary_sensor.vehicle_detected
sensor.people_count
sensor.object_confidence
```

La API no debe exigir que el cliente conozca el modelo de IA.

Puede incluir:

```text
confidence
model
inference_time
source
```

cuando corresponda.

---

# 131. Compatibilidad con IA

Los resultados de IA deben considerarse datos con:

```text
confidence
timestamp
model_version
source
quality
```

Ejemplo:

```json
{
  "entity_id": "binary_sensor.person_detected",
  "state": {
    "value": true,
    "confidence": 0.94
  }
}
```

---

# 132. API Contract

La implementación debe considerar los siguientes archivos como contrato:

```text
schemas/
    api/
    data-model/
    system-bus/
```

Los endpoints no deben inventar estructuras diferentes a las definidas en `DATA-SCHEMAS.md`.

---

# 133. JSON Schema

Las estructuras oficiales deberán expresarse mediante JSON Schema.

Estructura recomendada:

```text
schemas/
└── v1/
    ├── common/
    ├── sites/
    ├── zones/
    ├── devices/
    ├── resources/
    ├── capabilities/
    ├── entities/
    ├── commands/
    ├── events/
    ├── scenes/
    ├── automations/
    ├── integrations/
    ├── diagnostics/
    └── api/
```

---

# 134. Validación automática

La implementación futura deberá poder validar:

```text
API request
API response
System Bus message
Configuration
Event
Command
Telemetry
```

contra los schemas correspondientes.

---

# 135. Testing de API

Se deben implementar pruebas para:

### Funcionales

```text
GET
POST
PATCH
DELETE
```

### Seguridad

```text
401
403
token expiration
scope validation
rate limiting
```

### Datos

```text
invalid types
missing fields
invalid ranges
unknown enum
invalid IDs
```

### Distribución

```text
node offline
central offline
network timeout
duplicate command
reconnection
state synchronization
```

---

# 136. Contract Testing

Los siguientes componentes deben utilizar contract testing:

```text
Firmware
Central
Web UI
Mobile App
Integrations
Third-party clients
```

El objetivo es evitar que una actualización del firmware rompa la API.

---

# 137. Golden Fixtures

Se recomienda mantener ejemplos oficiales:

```text
tests/
└── fixtures/
    ├── entity.json
    ├── state.json
    ├── command.json
    ├── event.json
    ├── telemetry.json
    ├── scene.json
    ├── automation.json
    └── error.json
```

Estos archivos funcionan como casos de referencia.

---

# 138. Compatibilidad de firmware

Un firmware debe declarar:

```json
{
  "api_version": "v1",
  "schema_version": "1.0.0",
  "firmware_version": "2.4.1"
}
```

Central podrá determinar si el nodo es compatible.

---

# 139. Deprecación

Cuando una función vaya a desaparecer:

```text
Active
   ↓
Deprecated
   ↓
Compatibility period
   ↓
Removed in new major version
```

La API podrá incluir:

```http
Deprecation: true
```

y documentación de migración.

---

# 140. Migración

Las migraciones deben documentar:

```text
old endpoint
new endpoint
old schema
new schema
behavior changes
breaking changes
```

Ejemplo:

```text
/api/v1/entities/{id}/command
```

→

```text
/api/v2/entities/{id}/commands
```

---

# 141. Convención de nombres

Se recomienda:

```text
snake_case
```

para propiedades JSON.

Ejemplo:

```json
{
  "entity_id": "light.living",
  "device_id": "device.living",
  "firmware_version": "1.0.0"
}
```

Los nombres de endpoints utilizarán:

```text
kebab-case
```

cuando sea necesario separar palabras, aunque la mayoría de recursos son nombres simples.

---

# 142. Campos adicionales

Los objetos podrán contener:

```json
{
  "metadata": {}
}
```

para información extensible que no sea parte del contrato principal.

No debe utilizarse `metadata` para ocultar propiedades que deberían formar parte del schema oficial.

---

# 143. Source y Origin

Todo dato relevante debería poder identificar:

```text
source
origin
```

Ejemplo:

```json
{
  "source": "node.garden",
  "origin": "sensor.ph"
}
```

Esto permite distinguir:

```text
sensor físico
entidad calculada
automatización
usuario
integración externa
IA
```

---

# 144. Actor

Las acciones deben poder identificar al actor.

Ejemplo:

```json
{
  "actor": {
    "type": "user",
    "id": "user.alessandro"
  }
}
```

Otros actores:

```text
user
service
automation
scene
integration
device
system
```

---

# 145. Correlation ID

Una operación compleja debe mantener:

```text
correlation_id
```

Ejemplo:

```text
User
 ↓
API Request
 ↓
Scene
 ↓
Command 1
Command 2
Command 3
 ↓
Events
```

Todos pueden compartir:

```text
correlation_id
```

---

# 146. API y automatización local

Una automatización crítica no debe depender de:

```text
HTTP request
Central
Internet
Cloud
```

La API permite administrar la automatización, pero su ejecución debe producirse donde corresponda según la jerarquía de autonomía.

```text
DEVICE
  ↓
ZONE
  ↓
CENTRAL
  ↓
CLOUD
```

---

# 147. API y seguridad funcional

La API nunca debe ser el único mecanismo de seguridad para:

```text
motor
caldera
bomba
puerta
alarma
maquinaria
actuadores peligrosos
```

Las protecciones críticas deben estar implementadas localmente.

---

# 148. Ejemplo completo: sensor

Solicitud:

```http
GET /api/v1/entities/sensor.living.temperature
```

Respuesta:

```json
{
  "data": {
    "entity_id": "sensor.living.temperature",
    "domain": "sensor",
    "name": "Temperatura Living",
    "zone_id": "zone.living",
    "state": {
      "value": 23.7,
      "unit": "°C",
      "timestamp": "2026-10-06T12:30:00Z",
      "quality": "good"
    },
    "availability": "available"
  },
  "request_id": "req_123"
}
```

---

# 149. Ejemplo completo: comando

Solicitud:

```http
POST /api/v1/entities/light.living/commands
Idempotency-Key: light-living-001
```

```json
{
  "command": "turn_on",
  "parameters": {
    "brightness": 70
  }
}
```

Respuesta:

```json
{
  "data": {
    "command_id": "cmd_123",
    "status": "accepted",
    "status_url": "/api/v1/commands/cmd_123"
  },
  "request_id": "req_123"
}
```

---

# 150. Ejemplo completo: evento

```json
{
  "event_id": "event_123",
  "event_type": "entity.state_changed",
  "timestamp": "2026-10-06T12:30:01Z",
  "source": "node.living",
  "entity_id": "light.living",
  "correlation_id": "cmd_123",
  "data": {
    "old_state": {
      "on": false
    },
    "new_state": {
      "on": true,
      "brightness": 70
    }
  }
}
```

---

# 151. Ejemplo completo: pérdida de nodo

Estado anterior:

```text
node.garden = online
```

Después:

```text
node.garden = offline
```

La API debe reflejar:

```text
Node → offline
```

y las entidades dependientes pueden pasar a:

```text
unavailable
```

sin borrar sus configuraciones.

Cuando vuelve:

```text
offline
   ↓
online
   ↓
state synchronization
   ↓
available
```

---

# 152. Principio de no borrado por desconexión

Un dispositivo desconectado no debe eliminarse automáticamente.

Debe mantenerse:

```text
Device
Entity
Configuration
History
Identity
```

y cambiar:

```text
availability
```

---

# 153. API Offline

Si un nodo pierde Central:

```text
Node
 ├── Local API ✓
 ├── Local automation ✓
 ├── Local state ✓
 └── Internet integration ✗
```

Cuando sea posible, la API local continúa operativa.

---

# 154. API Centralizada

Cuando Central está disponible:

```text
Client
  ↓
Central API
  ↓
Distributed System
```

Esto permite una única interfaz para toda la instalación.

---

# 155. API híbrida

Un cliente puede descubrir:

```text
Central API
Node API
```

y elegir según disponibilidad.

La arquitectura recomienda:

```text
Central → administración global
Node   → operación local
```

---

# 156. Descubrimiento de API

El sistema puede proporcionar:

```http
GET /.well-known/automation-api
```

con información como:

```json
{
  "api_version": "v1",
  "base_path": "/api/v1",
  "websocket": "/api/v1/ws",
  "authentication": [
    "token",
    "session"
  ],
  "schema_version": "1.0.0"
}
```

---

# 157. OpenAPI

La API deberá disponer de una especificación OpenAPI.

Ubicación recomendada:

```text
docs/api/openapi.yaml
```

o:

```text
api/openapi.yaml
```

OpenAPI debe generarse/mantenerse alineado con:

```text
API-SPECIFICATION.md
DATA-SCHEMAS.md
```

---

# 158. Documentación automática

A partir de OpenAPI podrán generarse:

```text
Swagger UI
Redoc
SDKs
TypeScript types
client libraries
testing clients
```

Esto permitirá crear posteriormente:

```text
Web App
Android
iOS
Python
C++
Node.js
```

sin redefinir la API manualmente.

---

# 159. SDKs

En el futuro pueden generarse SDKs:

```text
JavaScript / TypeScript
Python
C++
C
Dart
Kotlin
Swift
```

Los SDK deben ser consumidores del contrato OpenAPI y de los schemas.

---

# 160. Arquitectura final

La arquitectura completa queda:

```text
                         ┌──────────────────┐
                         │   Web / Mobile   │
                         └────────┬─────────┘
                                  │
                         ┌────────▼─────────┐
                         │       API        │
                         │ REST / WebSocket │
                         └────────┬─────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │ Authentication / AuthZ    │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │       DATA MODEL          │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │       SYSTEM BUS          │
                    └─────────────┬─────────────┘
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
        ┌─────────┐         ┌───────────┐        ┌─────────┐
        │ Central │         │ Zone Ctrl │        │  Nodes  │
        └─────────┘         └───────────┘        └─────────┘
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  ▼
                         Hardware / Resources
```

---

# 161. Principios de diseño definitivos

La API debe cumplir las siguientes reglas:

### 1. La API trabaja con lógica, no con hardware

```text
Entity ≠ GPIO
Entity ≠ Modbus Register
Entity ≠ CAN ID
```

---

### 2. El modelo interno es la fuente de verdad

```text
Hardware
   ↓
Resource
   ↓
Capability
   ↓
Entity
   ↓
API / Integration
```

---

### 3. REST no reemplaza al System Bus

Son capas diferentes.

---

### 4. Un comando no es un estado

```text
Command → intención
State   → condición
Event   → hecho ocurrido
```

---

### 5. Central no es obligatorio para la autonomía

```text
Local automation > Central dependency
```

---

### 6. Los IDs lógicos son estables

Cambiar hardware no debe cambiar:

```text
light.living
sensor.living.temperature
switch.pump
```

---

### 7. Las integraciones son adaptadores

```text
Internal Model
      ↓
Adapter
      ↓
External Ecosystem
```

---

### 8. Los errores tienen códigos estables

Los clientes no deben interpretar textos.

---

### 9. Las operaciones distribuidas son asíncronas

```text
202 Accepted
    ↓
command_id
    ↓
status
```

---

### 10. La seguridad se aplica en múltiples niveles

```text
API
 ↓
Authorization
 ↓
System Bus
 ↓
Node
 ↓
Hardware Safety
```

---

### 11. La pérdida de conectividad no debe destruir el sistema

Un nodo desconectado conserva:

```text
configuration
identity
local automation
critical state
```

---

### 12. La API debe poder crecer

La misma API debe servir para:

```text
Casa
Oficina
Agricultura
Industria ligera
Invernadero
Estación meteorológica
Barco
Edificio
```

sin crear arquitecturas diferentes.

---

# 162. Estructura recomendada del proyecto

```text
project/
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATA-MODEL.md
│   ├── DATA-SCHEMAS.md
│   ├── SYSTEM-BUS.md
│   ├── API-SPECIFICATION.md
│   └── API-AUTHENTICATION-AUTHORIZATION.md
│
├── api/
│   └── openapi.yaml
│
├── schemas/
│   ├── v1/
│   │   ├── common/
│   │   ├── entities/
│   │   ├── devices/
│   │   ├── commands/
│   │   ├── events/
│   │   ├── scenes/
│   │   ├── automations/
│   │   ├── integrations/
│   │   └── diagnostics/
│
├── tests/
│   ├── api/
│   ├── schemas/
│   └── fixtures/
│
└── src/
    ├── api/
    ├── auth/
    ├── data_model/
    ├── system_bus/
    └── services/
```

---

# 163. Evolución futura

La API podrá incorporar posteriormente:

```text
GraphQL
gRPC
SSE
QUIC
CBOR
Protobuf
Binary RPC
```

pero estos mecanismos deben utilizar el mismo:

```text
Data Model
Data Schemas
Command Model
Event Model
Authorization Model
```

No deben crear modelos paralelos.

---

# 164. Regla de oro

> **La API es la interfaz pública del sistema lógico. El hardware, el transporte y la implementación interna pueden cambiar sin romper la identidad ni el comportamiento lógico de las entidades.**

La arquitectura completa debe permitir:

```text
ESP32
ESP32-S3
ESP32-C6
ESP32-C5
ESP32-H2
ESP32-P4
        │
        ▼
Different Hardware
        │
        ▼
Same Device Model
        │
        ▼
Same API
        │
        ▼
Same Applications
```

Y, al mismo tiempo:

```text
Wi-Fi
Ethernet
CAN
RS485
Zigbee
Thread
Matter
MQTT
        │
        ▼
Same Logical System
```

Por lo tanto:

> **El sistema no debe estar diseñado alrededor de un microcontrolador, un protocolo o una placa determinada. Debe estar diseñado alrededor de un modelo lógico estable, una API versionada y contratos de datos bien definidos.**

---

# 165. Próximo paso recomendado

Una vez establecidos:

```text
DATA-MODEL.md
DATA-SCHEMAS.md
SYSTEM-BUS.md
API-SPECIFICATION.md
API-AUTHENTICATION-AUTHORIZATION.md
```

el siguiente nivel debería ser formalizar:

```text
OpenAPI
   +
JSON Schemas
   +
System Bus Message Schemas
   +
Error Codes
   +
Command/Event Contracts
```

Esto permitirá comenzar a implementar el firmware y el Central sin que cada módulo tenga que inventar su propio formato de comunicación.
