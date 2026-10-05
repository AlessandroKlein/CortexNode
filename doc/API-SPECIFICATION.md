# API-SPECIFICATION.md

# API Specification — Distributed Automation Platform

> **Tipo:** Especificación de arquitectura y contrato de API
> **Estado:** Diseño
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05
> **Objetivo:** Definir la API REST, WebSocket y mecanismos de interacción externos con la plataforma de automatización distribuida.

---

# 1. Objetivo

La API proporciona una interfaz uniforme para:

* Web UI;
* aplicaciones móviles;
* aplicaciones de escritorio;
* aplicaciones de terceros;
* integraciones;
* servicios externos;
* herramientas de administración;
* sistemas de monitoreo;
* automatizaciones externas.

La API debe permitir trabajar con:

* Sites;
* Zones;
* Groups;
* Devices;
* Resources;
* Capabilities;
* Entities;
* States;
* Commands;
* Events;
* Functions;
* Scenes;
* Automations;
* System Modes;
* History;
* Diagnostics;
* Users;
* Integrations;
* System configuration.

---

# 2. Principio fundamental

> **La API expone el modelo lógico del sistema, no su implementación física.**

Una aplicación debe poder hacer:

```text
GET /api/v1/entities/light.living.main
```

y:

```text
POST /api/v1/entities/light.living.main/commands
```

sin conocer:

```text
GPIO
PWM
I2C
SPI
CAN ID
Modbus register
MQTT topic
Ethernet IP
```

---

# 3. Arquitectura

```text
┌──────────────────────────────┐
│          WEB / APP           │
└──────────────┬───────────────┘
               │
          REST / WebSocket
               │
               ▼
┌──────────────────────────────┐
│          API LAYER           │
│                              │
│ Authentication               │
│ Authorization                │
│ Validation                   │
│ Rate Limiting                │
│ Serialization                │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│        APPLICATION           │
│                              │
│ Automations                  │
│ Scenes                       │
│ Functions                    │
│ Entity Management            │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│         SYSTEM BUS           │
└──────────────┬───────────────┘
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
      Zone    Node    Integration
```

---

# 4. API Protocols

La plataforma utilizará principalmente:

```text
REST
WebSocket
```

Opcionalmente podrá incorporar:

```text
Server-Sent Events
MQTT
WebRTC
gRPC
```

según el caso de uso.

---

# 5. REST API

REST será la interfaz principal para:

* configuración;
* administración;
* consultas;
* comandos;
* historial;
* discovery;
* usuarios;
* integraciones.

Base URL:

```text
/api/v1
```

Ejemplo:

```text
/api/v1/entities
```

---

# 6. Versionado

La API debe versionarse explícitamente.

Formato:

```text
/api/v1
/api/v2
```

No se recomienda modificar de forma incompatible una versión existente.

---

# 7. Compatibilidad

Cambios compatibles:

* agregar campos opcionales;
* agregar nuevos enum values cuando el cliente pueda ignorarlos;
* agregar nuevos endpoints;
* agregar nuevos tipos de entidades;
* agregar nuevos capabilities.

Cambios incompatibles:

* eliminar campos;
* cambiar significado;
* cambiar tipos;
* cambiar comportamiento;
* eliminar endpoints.

Estos cambios requieren nueva versión mayor.

---

# 8. Content Type

JSON será el formato principal:

```http
Content-Type: application/json
```

Para respuestas:

```http
Accept: application/json
```

---

# 9. Character Encoding

Todo JSON debe utilizar:

```text
UTF-8
```

---

# 10. Date and Time

Las fechas deberán utilizar ISO 8601 / RFC 3339.

Ejemplo:

```text
2026-10-05T15:30:00Z
```

Con offset:

```text
2026-10-05T12:30:00-03:00
```

Se recomienda almacenar internamente timestamps en UTC.

---

# 11. Resource IDs

La API utiliza IDs estables.

Ejemplos:

```text
site.home
zone.living
device.living.controller
light.living.main
sensor.living.temperature
```

Los IDs no deben depender de:

```text
GPIO
MAC
IP
CAN ID
Modbus address
MQTT topic
```

---

# 12. Authentication

La API debe integrarse con:

```text
API-AUTHENTICATION-AUTHORIZATION.md
```

Los mecanismos podrán incluir:

```text
Session
Bearer Token
API Key
Refresh Token
mTLS
```

según el tipo de cliente.

---

# 13. Authorization

La autorización debe realizarse antes de ejecutar acciones.

Ejemplo:

```text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Validation
 ↓
Execution
```

---

# 14. Permission Scopes

Ejemplos:

```text
system.read
system.write

sites.read
sites.write

zones.read
zones.write

devices.read
devices.write

entities.read
entities.write

entities.command

states.read

events.read

history.read

scenes.read
scenes.execute
scenes.write

automations.read
automations.write

users.read
users.write

integrations.read
integrations.write

diagnostics.read
diagnostics.write
```

---

# 15. Principle of Least Privilege

Una aplicación debe recibir solamente los permisos necesarios.

Ejemplo:

Una pantalla de monitoreo:

```text
entities.read
states.read
events.read
```

Una aplicación administrativa:

```text
entities.read
entities.write
entities.command
automations.write
```

---

# 16. Root Endpoints

```text
/api/v1
```

Debe proporcionar información básica.

Ejemplo:

```http
GET /api/v1
```

Respuesta:

```json
{
  "api_version": "1.0",
  "system": "Distributed Automation Platform",
  "status": "ok"
}
```

---

# 17. System

```text
GET    /api/v1/system
PATCH  /api/v1/system
```

Información:

* nombre;
* versión;
* uptime;
* estado;
* capabilities;
* modo;
* hora;
* timezone;
* firmware;
* storage;
* network.

---

# 18. System Status

```http
GET /api/v1/system/status
```

Ejemplo:

```json
{
  "status": "ok",
  "uptime": 123456,
  "central": {
    "status": "online"
  },
  "internet": {
    "status": "offline"
  }
}
```

---

# 19. System Health

```http
GET /api/v1/system/health
```

Ejemplo:

```json
{
  "status": "degraded",
  "cpu": {
    "usage": 42
  },
  "memory": {
    "free": 182000
  },
  "storage": {
    "status": "ok"
  },
  "bus": {
    "status": "ok"
  }
}
```

---

# 20. System Modes

```text
GET    /api/v1/system/modes
GET    /api/v1/system/mode
PUT    /api/v1/system/mode
```

Ejemplo:

```json
{
  "mode": "sleep"
}
```

Modos posibles:

```text
normal
sleep
away
vacation
maintenance
emergency
custom
```

---

# 21. Sites

```text
GET    /api/v1/sites
POST   /api/v1/sites
GET    /api/v1/sites/{site_id}
PATCH  /api/v1/sites/{site_id}
DELETE /api/v1/sites/{site_id}
```

---

# 22. Site Example

```json
{
  "id": "site.home",
  "name": "Casa",
  "timezone": "America/Argentina/Buenos_Aires",
  "status": "active"
}
```

---

# 23. Zones

```text
GET    /api/v1/zones
POST   /api/v1/zones
GET    /api/v1/zones/{zone_id}
PATCH  /api/v1/zones/{zone_id}
DELETE /api/v1/zones/{zone_id}
```

---

# 24. Zone Filtering

Ejemplo:

```http
GET /api/v1/zones?site_id=site.home
```

También:

```http
GET /api/v1/zones?parent_id=zone.ground_floor
```

---

# 25. Groups

```text
GET    /api/v1/groups
POST   /api/v1/groups
GET    /api/v1/groups/{group_id}
PATCH  /api/v1/groups/{group_id}
DELETE /api/v1/groups/{group_id}
```

Un Group puede contener:

* entities;
* devices;
* zones;
* functions.

---

# 26. Devices

```text
GET    /api/v1/devices
POST   /api/v1/devices
GET    /api/v1/devices/{device_id}
PATCH  /api/v1/devices/{device_id}
DELETE /api/v1/devices/{device_id}
```

---

# 27. Device Discovery

```text
GET /api/v1/devices/discovered
```

Devuelve dispositivos detectados pero aún no provisionados.

Ejemplo:

```json
{
  "devices": [
    {
      "device_id": "device.node01",
      "status": "discovered",
      "model": "ESP32-S3",
      "firmware": "1.0.0"
    }
  ]
}
```

---

# 28. Device Provisioning

```text
POST /api/v1/devices/{device_id}/provision
```

Ejemplo:

```json
{
  "name": "Controlador Living",
  "zone_id": "zone.living",
  "profile": "standard_node"
}
```

---

# 29. Device Reconfigure

```text
POST /api/v1/devices/{device_id}/reconfigure
```

Permite solicitar reconciliación de configuración.

---

# 30. Device Restart

```text
POST /api/v1/devices/{device_id}/restart
```

Debe requerir permisos administrativos.

---

# 31. Device Firmware

```text
GET  /api/v1/devices/{device_id}/firmware
POST /api/v1/devices/{device_id}/firmware/update
```

La actualización debe implementar mecanismos de seguridad y rollback definidos en la documentación de firmware/OTA.

---

# 32. Resources

```text
GET /api/v1/resources
GET /api/v1/resources/{resource_id}
```

Opcionalmente:

```text
GET /api/v1/devices/{device_id}/resources
```

Los Resources representan:

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

---

# 33. Capabilities

```text
GET /api/v1/capabilities
GET /api/v1/capabilities/{capability_id}
```

También:

```http
GET /api/v1/entities/{entity_id}/capabilities
```

---

# 34. Entities

Las Entities son el principal recurso de la API.

```text
GET    /api/v1/entities
POST   /api/v1/entities
GET    /api/v1/entities/{entity_id}
PATCH  /api/v1/entities/{entity_id}
DELETE /api/v1/entities/{entity_id}
```

---

# 35. Entity Query

Ejemplo:

```http
GET /api/v1/entities?zone_id=zone.living
```

Por dominio:

```http
GET /api/v1/entities?domain=light
```

Por disponibilidad:

```http
GET /api/v1/entities?availability=available
```

---

# 36. Multiple Filters

Ejemplo:

```http
GET /api/v1/entities?site_id=site.home&zone_id=zone.living&domain=light
```

Los filtros deben combinarse mediante AND salvo que se documente lo contrario.

---

# 37. Entity Example

```json
{
  "id": "light.living.main",
  "name": "Luz principal",
  "domain": "light",
  "zone_id": "zone.living",
  "device_id": "device.living.controller",
  "capabilities": [
    "on_off",
    "brightness"
  ],
  "availability": "available"
}
```

---

# 38. Entity State

```http
GET /api/v1/entities/{entity_id}/state
```

Ejemplo:

```json
{
  "entity_id": "light.living.main",
  "state": {
    "on": true,
    "brightness": 80
  },
  "timestamp": "2026-10-05T15:30:00Z",
  "quality": "good"
}
```

---

# 39. Desired State

Para actuadores:

```http
GET /api/v1/entities/{entity_id}/desired-state
```

Puede existir:

```text
desired
actual
```

simultáneamente.

---

# 40. State Comparison

Ejemplo:

```json
{
  "desired": {
    "on": true
  },
  "actual": {
    "on": false
  },
  "status": "mismatch"
}
```

---

# 41. Set State

Para operaciones simples:

```http
PUT /api/v1/entities/{entity_id}/state
```

Ejemplo:

```json
{
  "on": true,
  "brightness": 80
}
```

Internamente esto se transforma en un Command.

---

# 42. Commands

Endpoint principal:

```http
POST /api/v1/entities/{entity_id}/commands
```

Ejemplo:

```json
{
  "action": "turn_on"
}
```

---

# 43. Command with Parameters

```json
{
  "action": "set_brightness",
  "parameters": {
    "brightness": 70
  }
}
```

---

# 44. Command Response

La API debe devolver información suficiente para seguir la operación.

Ejemplo:

```json
{
  "command_id": "cmd_01JXYZ",
  "request_id": "req_01JXYZ",
  "status": "accepted"
}
```

El cliente no debe asumir que `accepted` significa `executed`.

---

# 45. Command Status

```http
GET /api/v1/commands/{command_id}
```

Ejemplo:

```json
{
  "command_id": "cmd_01JXYZ",
  "status": "executed",
  "created_at": "2026-10-05T15:30:00Z",
  "completed_at": "2026-10-05T15:30:00.250Z"
}
```

---

# 46. Command Lifecycle

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
expired
cancelled
```

---

# 47. Command Cancellation

Cuando sea posible:

```http
POST /api/v1/commands/{command_id}/cancel
```

No todos los comandos son cancelables.

---

# 48. Idempotency

Las operaciones POST críticas deben admitir:

```http
Idempotency-Key: <unique-key>
```

Ejemplo:

```http
Idempotency-Key: 8b2f...
```

Si el cliente repite la solicitud con la misma clave, no debe ejecutar dos veces una operación no idempotente.

---

# 49. Events

```text
GET /api/v1/events
GET /api/v1/events/{event_id}
```

Filtros:

```text
entity_id
device_id
zone_id
event_type
from
to
priority
source
```

---

# 50. Event Example

```json
{
  "event_id": "evt_01JXYZ",
  "type": "motion.detected",
  "entity_id": "binary_sensor.hall.motion",
  "timestamp": "2026-10-05T15:30:00Z",
  "source": "device.hall.sensor"
}
```

---

# 51. Event Pagination

Ejemplo:

```http
GET /api/v1/events?limit=100&cursor=abc123
```

Se recomienda cursor pagination para streams grandes.

---

# 52. History

```http
GET /api/v1/history
```

Ejemplo:

```http
GET /api/v1/history?entity_id=sensor.living.temperature
```

Parámetros:

```text
from
to
resolution
aggregation
```

---

# 53. History Resolution

Valores posibles:

```text
raw
1m
5m
15m
1h
1d
```

---

# 54. History Aggregation

Para valores numéricos:

```text
min
max
avg
sum
first
last
count
```

Ejemplo:

```http
GET /api/v1/history?entity_id=sensor.temperature&resolution=1h&aggregation=avg
```

---

# 55. Telemetry

Para datos de alta frecuencia:

```http
GET /api/v1/telemetry
```

No se debe utilizar el endpoint histórico para streaming de alta frecuencia.

---

# 56. Functions

```text
GET    /api/v1/functions
POST   /api/v1/functions
GET    /api/v1/functions/{function_id}
PATCH  /api/v1/functions/{function_id}
DELETE /api/v1/functions/{function_id}
```

Ejemplos:

```text
lighting
heating
cooling
irrigation
security
ventilation
energy
water
```

---

# 57. Scenes

```text
GET    /api/v1/scenes
POST   /api/v1/scenes
GET    /api/v1/scenes/{scene_id}
PATCH  /api/v1/scenes/{scene_id}
DELETE /api/v1/scenes/{scene_id}
POST   /api/v1/scenes/{scene_id}/execute
```

---

# 58. Scene Example

```json
{
  "id": "scene.movie",
  "name": "Modo película",
  "actions": [
    {
      "entity_id": "light.living.main",
      "action": "turn_off"
    },
    {
      "entity_id": "light.living.ambient",
      "action": "set_brightness",
      "parameters": {
        "brightness": 20
      }
    }
  ]
}
```

---

# 59. Scene Execution

```http
POST /api/v1/scenes/scene.movie/execute
```

Respuesta:

```json
{
  "execution_id": "exec_01JXYZ",
  "status": "accepted"
}
```

---

# 60. Automation

```text
GET    /api/v1/automations
POST   /api/v1/automations
GET    /api/v1/automations/{automation_id}
PATCH  /api/v1/automations/{automation_id}
DELETE /api/v1/automations/{automation_id}
POST   /api/v1/automations/{automation_id}/enable
POST   /api/v1/automations/{automation_id}/disable
```

---

# 61. Automation Structure

Conceptualmente:

```text
TRIGGER
   ↓
CONDITIONS
   ↓
ACTIONS
```

---

# 62. Automation Example

```json
{
  "id": "automation.hall_light",
  "name": "Luz del pasillo",

  "enabled": true,

  "trigger": {
    "type": "event",
    "event": "motion.detected",
    "entity_id": "binary_sensor.hall.motion"
  },

  "conditions": [
    {
      "type": "system_mode",
      "equals": "sleep"
    }
  ],

  "actions": [
    {
      "type": "command",
      "entity_id": "light.hall",
      "action": "turn_on"
    }
  ]
}
```

---

# 63. Enable / Disable

Toda automatización debe poder activarse/desactivarse sin eliminarla.

```http
POST /api/v1/automations/{id}/enable
POST /api/v1/automations/{id}/disable
```

---

# 64. Groups Command

Un Group puede recibir comandos.

```http
POST /api/v1/groups/{group_id}/commands
```

Ejemplo:

```json
{
  "action": "turn_off"
}
```

El sistema puede traducirlo en comandos individuales.

---

# 65. Zone Commands

También:

```http
POST /api/v1/zones/{zone_id}/commands
```

Debe existir una política clara sobre qué entidades participan.

---

# 66. Bulk Commands

Para operaciones múltiples:

```http
POST /api/v1/commands/bulk
```

Ejemplo:

```json
{
  "commands": [
    {
      "entity_id": "light.living.main",
      "action": "turn_off"
    },
    {
      "entity_id": "light.kitchen.main",
      "action": "turn_off"
    }
  ]
}
```

La respuesta debe identificar individualmente cada resultado.

---

# 67. Bulk Command Result

```json
{
  "execution_id": "exec_123",
  "results": [
    {
      "entity_id": "light.living.main",
      "status": "executed"
    },
    {
      "entity_id": "light.kitchen.main",
      "status": "failed"
    }
  ]
}
```

---

# 68. Atomicity

Un bulk command no debe considerarse atómico por defecto.

Debe existir:

```text
atomic: false
```

por defecto.

Cuando una operación realmente soporte atomicidad:

```text
atomic: true
```

debe estar explícitamente soportado.

---

# 69. Batch Operations

Para grandes instalaciones:

```http
POST /api/v1/batch
```

puede permitir agrupar operaciones.

Debe limitarse para evitar sobrecargar nodos pequeños.

---

# 70. Discovery API

```text
GET /api/v1/discovery
POST /api/v1/discovery/start
```

Puede devolver:

```text
devices
nodes
capabilities
transports
integrations
```

---

# 71. Network

```text
GET /api/v1/network
GET /api/v1/network/interfaces
GET /api/v1/network/routes
```

La información sensible debe requerir permisos administrativos.

---

# 72. Transports

```text
GET /api/v1/transports
GET /api/v1/transports/{transport_id}
```

Ejemplo:

```json
{
  "id": "transport.ethernet",
  "type": "ethernet",
  "status": "connected",
  "priority": 1
}
```

---

# 73. System Bus API

Opcionalmente:

```text
GET /api/v1/bus/status
GET /api/v1/bus/statistics
GET /api/v1/bus/routes
```

Nunca se debe permitir a un cliente normal publicar mensajes arbitrarios directamente en el bus.

---

# 74. Raw Bus Access

El acceso:

```text
POST /api/v1/bus/messages
```

debe estar restringido a:

```text
system administrator
diagnostic tools
trusted integrations
```

y preferentemente no estar habilitado en instalaciones normales.

La API pública debe trabajar con objetos semánticos.

---

# 75. Integrations

```text
GET    /api/v1/integrations
POST   /api/v1/integrations
GET    /api/v1/integrations/{integration_id}
PATCH  /api/v1/integrations/{integration_id}
DELETE /api/v1/integrations/{integration_id}
POST   /api/v1/integrations/{integration_id}/enable
POST   /api/v1/integrations/{integration_id}/disable
```

---

# 76. Integration Status

```http
GET /api/v1/integrations/{integration_id}/status
```

Ejemplo:

```json
{
  "status": "connected",
  "last_sync": "2026-10-05T15:29:00Z"
}
```

---

# 77. Users

```text
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/{user_id}
PATCH  /api/v1/users/{user_id}
DELETE /api/v1/users/{user_id}
```

El acceso debe depender de roles.

---

# 78. Roles

```http
GET /api/v1/roles
```

Ejemplos:

```text
owner
administrator
operator
user
viewer
integration
service
```

---

# 79. API Keys

```text
GET    /api/v1/api-keys
POST   /api/v1/api-keys
DELETE /api/v1/api-keys/{key_id}
```

La API nunca debe devolver nuevamente un secreto completo después de su creación.

---

# 80. Tokens

Para clientes interactivos:

```text
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
```

---

# 81. Login

Ejemplo:

```json
{
  "username": "admin",
  "password": "********"
}
```

Respuesta:

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "expires_in": 3600
}
```

La implementación exacta dependerá de `API-AUTHENTICATION-AUTHORIZATION.md`.

---

# 82. WebSocket

Endpoint:

```text
/ws/v1
```

El WebSocket está destinado a tiempo real.

Permite:

```text
state updates
events
command results
availability
diagnostics
notifications
```

---

# 83. WebSocket Connection

Flujo:

```text
Client
 ↓
WebSocket Connect
 ↓
Authenticate
 ↓
Subscribe
 ↓
Receive events
```

---

# 84. WebSocket Authentication

La conexión debe autenticarse.

Métodos posibles:

```text
Authorization header
secure cookie
short-lived WebSocket token
```

No se recomienda enviar contraseñas dentro del canal WebSocket después de establecer conexión.

---

# 85. WebSocket Message

Formato:

```json
{
  "type": "event",
  "message_id": "msg_01JXYZ",
  "timestamp": "2026-10-05T15:30:00Z",
  "payload": {}
}
```

---

# 86. WebSocket Subscribe

Ejemplo:

```json
{
  "type": "subscribe",
  "request_id": "req_123",
  "filters": [
    {
      "zone_id": "zone.living"
    }
  ]
}
```

---

# 87. Subscription Response

```json
{
  "type": "subscription_result",
  "request_id": "req_123",
  "subscription_id": "sub_123",
  "status": "active"
}
```

---

# 88. WebSocket Unsubscribe

```json
{
  "type": "unsubscribe",
  "subscription_id": "sub_123"
}
```

---

# 89. WebSocket State Event

```json
{
  "type": "state_changed",
  "entity_id": "light.living.main",
  "state": {
    "on": true,
    "brightness": 80
  },
  "timestamp": "2026-10-05T15:30:00Z"
}
```

---

# 90. WebSocket Command Result

```json
{
  "type": "command_result",
  "command_id": "cmd_123",
  "status": "executed"
}
```

---

# 91. WebSocket Reconnection

El cliente debe poder reconectarse.

Después de reconectar:

```text
CONNECT
 ↓
AUTHENTICATE
 ↓
RESUBSCRIBE
 ↓
STATE SNAPSHOT
 ↓
LIVE EVENTS
```

---

# 92. Event Replay

Cuando sea posible, el cliente puede indicar:

```json
{
  "last_event_id": "evt_123"
}
```

El servidor puede devolver eventos posteriores.

Si el historial ya no está disponible:

```text
RESYNC_REQUIRED
```

y debe enviarse un snapshot.

---

# 93. Server Events

Tipos:

```text
state_changed
event
command_result
device_online
device_offline
entity_available
entity_unavailable
configuration_changed
system_mode_changed
alarm
notification
```

---

# 94. REST vs WebSocket

### REST

Usar para:

* configuración;
* administración;
* consultas;
* creación;
* edición;
* historial;
* operaciones puntuales.

### WebSocket

Usar para:

* tiempo real;
* eventos;
* dashboards;
* estados;
* notificaciones;
* seguimiento de comandos.

---

# 95. Pagination

Los endpoints que devuelvan colecciones deben soportar paginación.

Preferido:

```text
limit
cursor
```

Ejemplo:

```http
GET /api/v1/entities?limit=50&cursor=abc
```

---

# 96. Pagination Response

```json
{
  "items": [],
  "pagination": {
    "limit": 50,
    "next_cursor": "def",
    "has_more": true
  }
}
```

---

# 97. Limit

El servidor debe imponer un máximo.

Ejemplo:

```text
default = 50
maximum = 500
```

Los valores definitivos dependerán del hardware.

---

# 98. Sorting

Colecciones pueden soportar:

```text
sort
order
```

Ejemplo:

```http
GET /api/v1/events?sort=timestamp&order=desc
```

---

# 99. Filtering

Los filtros deben ser explícitos.

Ejemplos:

```text
zone_id
device_id
entity_id
domain
status
availability
created_after
created_before
updated_after
updated_before
```

---

# 100. Search

Puede existir:

```http
GET /api/v1/search?q=living
```

Debe buscar sobre:

* name;
* alias;
* entity_id;
* tags;
* description.

---

# 101. ETags

Los recursos configurables deben soportar ETag cuando sea posible.

Respuesta:

```http
ETag: "v20"
```

Actualización:

```http
If-Match: "v20"
```

Si la versión cambió:

```text
412 Precondition Failed
```

Esto evita sobrescribir cambios realizados por otro usuario.

---

# 102. Resource Version

Los recursos pueden incluir:

```json
{
  "version": 20
}
```

La versión permite detectar conflictos.

---

# 103. Optimistic Concurrency

Ejemplo:

```text
User A reads version 20
User B updates → version 21

User A updates using version 20
        ↓
CONFLICT
```

La API debe evitar sobrescribir silenciosamente la modificación de B.

---

# 104. PATCH

Para modificaciones parciales:

```http
PATCH /api/v1/entities/{entity_id}
```

Ejemplo:

```json
{
  "name": "Luz principal del living"
}
```

---

# 105. PUT

PUT debe utilizarse cuando se pretende reemplazar completamente una representación compatible.

Ejemplo:

```http
PUT /api/v1/system/mode
```

---

# 106. DELETE

DELETE debe utilizarse con cuidado.

Para entidades históricas se recomienda:

```text
soft delete
```

en lugar de eliminación física inmediata.

---

# 107. Soft Delete

Ejemplo:

```json
{
  "status": "removed"
}
```

La información histórica puede conservar:

```text
entity_id
name
device
timestamps
history
```

según las políticas de retención.

---

# 108. Error Model

Todas las respuestas de error deben utilizar una estructura común.

Ejemplo:

```json
{
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity does not exist",
    "request_id": "req_123"
  }
}
```

---

# 109. Error Codes

Ejemplos:

```text
BAD_REQUEST
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
VALIDATION_ERROR
RATE_LIMITED

ENTITY_NOT_FOUND
DEVICE_NOT_FOUND
CAPABILITY_NOT_SUPPORTED

COMMAND_REJECTED
COMMAND_FAILED
COMMAND_TIMEOUT
COMMAND_EXPIRED

TRANSPORT_UNAVAILABLE
NO_ROUTE

CONFIGURATION_INVALID
CONFIGURATION_FAILED

INTERNAL_ERROR
SERVICE_UNAVAILABLE
```

---

# 110. HTTP Status Codes

Usos recomendados:

```text
200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
412 Precondition Failed
422 Unprocessable Entity
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# 111. Validation Errors

Ejemplo:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "fields": [
      {
        "field": "brightness",
        "reason": "must be between 0 and 100"
      }
    ]
  }
}
```

---

# 112. Request ID

Toda solicitud API debe recibir:

```text
request_id
```

Ejemplo:

```http
X-Request-ID: req_01JXYZ
```

Si el cliente proporciona uno válido, el servidor puede conservarlo.

---

# 113. Correlation ID

Para operaciones distribuidas:

```http
X-Correlation-ID: corr_01JXYZ
```

Debe propagarse hacia el System Bus.

---

# 114. Command ID

Las operaciones que generan comandos deben devolver:

```text
command_id
```

Esto permite seguir la ejecución independientemente de la conexión HTTP original.

---

# 115. Traceability

Idealmente:

```text
HTTP Request
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
Node
   │
   ▼
Event / State
```

---

# 116. Rate Limiting

La API debe implementar límites por:

```text
IP
user
token
API key
client
endpoint
```

según el nivel de riesgo.

---

# 117. Rate Limit Response

Cuando se excede:

```http
429 Too Many Requests
```

Puede incluir:

```http
Retry-After: 5
```

---

# 118. Command Rate Limiting

Debe existir una protección específica para comandos.

Ejemplo:

```text
10 commands/s/user
100 commands/s/system
```

Los valores serán configurables según hardware.

---

# 119. Authentication Rate Limiting

Especialmente estricto para:

```text
login
token
password reset
API key
```

---

# 120. CORS

Si existe una Web UI separada:

```text
CORS
```

debe configurarse mediante lista explícita de orígenes permitidos.

Nunca:

```text
Access-Control-Allow-Origin: *
```

para endpoints administrativos autenticados salvo casos específicamente controlados.

---

# 121. CSRF

Las sesiones basadas en cookies deben implementar protección CSRF.

Los tokens Bearer enviados mediante headers tienen un modelo diferente y deben seguir las políticas definidas en la autenticación.

---

# 122. TLS

Para acceso remoto:

```text
HTTPS
WSS
```

debe ser obligatorio.

En instalaciones locales se puede permitir HTTP bajo una política explícita de configuración inicial, pero la plataforma debe poder funcionar completamente con HTTPS.

---

# 123. Local Access

La API local puede estar disponible mediante:

```text
http://device.local
```

o:

```text
https://device.local
```

según el método de instalación y certificados.

El nombre concreto dependerá del hostname/mDNS configurado.

---

# 124. Remote Access

No se recomienda exponer directamente la API administrativa a Internet.

Preferentemente:

```text
Internet
   ↓
VPN / secure gateway / authenticated service
   ↓
Central
```

---

# 125. API Gateway

Central puede actuar como API Gateway.

```text
External Client
      ↓
Central API
      ↓
System Bus
      ↓
Node
```

Esto permite ocultar la topología interna.

---

# 126. Direct Node API

Los nodos también pueden ofrecer API local.

```text
Client
  ↓
Node API
  ↓
Local Entity
```

Debe ser opcional.

---

# 127. Central API vs Node API

### Central

Proporciona:

* sistema completo;
* múltiples zonas;
* usuarios;
* integraciones;
* historial;
* automatizaciones globales.

### Node

Proporciona:

* entidades locales;
* configuración local;
* diagnóstico;
* control local;
* fallback.

Ambas deben utilizar el mismo Data Model.

---

# 128. API Discovery

La API puede proporcionar:

```http
GET /api/v1/capabilities
```

para que clientes conozcan:

```text
supported domains
supported commands
supported features
api version
```

---

# 129. Feature Flags

La respuesta puede incluir:

```json
{
  "features": {
    "websocket": true,
    "history": true,
    "scenes": true,
    "automations": true,
    "matter": false,
    "camera": true
  }
}
```

Esto permite que la UI se adapte al dispositivo.

---

# 130. Capability Discovery

Un cliente nunca debe asumir que todos los dispositivos soportan:

```text
brightness
color
position
speed
```

Debe consultar:

```text
entity.capabilities
```

---

# 131. Domain Discovery

La plataforma puede informar:

```http
GET /api/v1/domains
```

Ejemplo:

```json
{
  "domains": [
    "light",
    "switch",
    "fan",
    "sensor",
    "binary_sensor",
    "cover",
    "climate"
  ]
}
```

---

# 132. Units

Los valores deben incluir unidad cuando sea necesario.

Ejemplo:

```json
{
  "value": 23.4,
  "unit": "°C"
}
```

La API debe utilizar las unidades canónicas definidas en `DATA-MODEL.md`.

---

# 133. Localization

Los IDs no deben depender del idioma.

Ejemplo:

```text
entity_id:
light.living.main
```

El nombre puede cambiar:

```text
Español:
"Luz principal"

English:
"Main light"
```

---

# 134. Friendly Names

La API debe diferenciar:

```text
id
name
aliases
```

Ejemplo:

```json
{
  "id": "light.living.main",
  "name": "Luz principal",
  "aliases": [
    "Luz living",
    "Luz del salón"
  ]
}
```

---

# 135. Tags

Las entidades pueden incluir:

```json
{
  "tags": [
    "lighting",
    "main",
    "downstairs"
  ]
}
```

Esto permite búsquedas y agrupaciones.

---

# 136. Visibility

Una Entity puede indicar:

```text
visible
hidden
internal
```

Las Entities internas no deberían exponerse automáticamente a integraciones externas.

---

# 137. External Exposure

Una Entity puede tener:

```json
{
  "exposure": {
    "home_assistant": true,
    "matter": true,
    "public_api": false
  }
}
```

---

# 138. Virtual Entities

La API debe soportar entidades sin hardware directo.

Ejemplo:

```text
sensor.house.average_temperature
```

Puede calcularse a partir de:

```text
sensor.living.temperature
sensor.bedroom.temperature
sensor.kitchen.temperature
```

---

# 139. Aggregated Entities

Ejemplo:

```text
energy.house.total
```

puede agregar:

```text
energy.kitchen
energy.living
energy.garage
```

La API debe tratarlas como Entities normales.

---

# 140. External Entities

Una integración puede registrar:

```text
sensor.weather.outdoor_temperature
```

aunque el sensor físico esté fuera del sistema.

Debe marcarse:

```text
source: external
```

---

# 141. Source

Estados y eventos pueden incluir:

```json
{
  "source": {
    "type": "device",
    "id": "device.living.sensor"
  }
}
```

Otros tipos:

```text
device
central
automation
user
integration
system
external
```

---

# 142. Quality

Los estados deben soportar:

```text
good
uncertain
stale
invalid
unavailable
```

Ejemplo:

```json
{
  "value": 24.2,
  "unit": "°C",
  "quality": "good"
}
```

---

# 143. Availability

Una Entity puede estar:

```text
available
unavailable
unknown
degraded
disabled
```

No debe confundirse:

```text
unknown
```

con:

```text
unavailable
```

---

# 144. State vs Event

La API debe mantener la diferencia:

```text
State
=
"cómo está ahora"

Event
=
"qué ocurrió"
```

Ejemplo:

```text
State:
door = open

Event:
door.opened
```

---

# 145. Alarm API

Para sistemas de seguridad:

```text
GET  /api/v1/alarms
GET  /api/v1/alarms/{alarm_id}
POST /api/v1/alarms/{alarm_id}/arm
POST /api/v1/alarms/{alarm_id}/disarm
POST /api/v1/alarms/{alarm_id}/acknowledge
```

Las acciones deben requerir permisos adecuados.

---

# 146. Notification API

Opcionalmente:

```text
GET /api/v1/notifications
POST /api/v1/notifications/{id}/acknowledge
```

Puede representar:

* alertas;
* avisos;
* mantenimiento;
* errores;
* eventos importantes.

---

# 147. Diagnostics

```text
GET /api/v1/diagnostics
GET /api/v1/devices/{device_id}/diagnostics
```

Puede incluir:

```text
CPU
RAM
flash
network
bus
tasks
sensors
errors
uptime
watchdog
```

---

# 148. Logs

```http
GET /api/v1/logs
```

Filtros:

```text
level
module
device
from
to
```

El acceso debe estar restringido.

---

# 149. Audit Log

Las acciones administrativas importantes deben registrar:

```text
user
action
resource
timestamp
result
source
request_id
```

Ejemplo:

```json
{
  "user_id": "user.admin",
  "action": "entity.command",
  "entity_id": "lock.front_door",
  "result": "success"
}
```

---

# 150. API Export

La configuración puede exportarse:

```http
GET /api/v1/export
```

Debe poder incluir:

```text
sites
zones
groups
devices
entities
automations
scenes
integrations
configuration
```

Los secretos deben excluirse o cifrarse.

---

# 151. API Import

```http
POST /api/v1/import
```

Debe validar:

```text
schema version
compatibility
IDs
hardware capabilities
dependencies
```

antes de aplicar cambios.

---

# 152. Dry Run

Se recomienda soportar:

```http
POST /api/v1/import?dry_run=true
```

para detectar:

* incompatibilidades;
* entidades faltantes;
* capabilities inexistentes;
* conflictos.

Sin modificar el sistema.

---

# 153. Configuration Transactions

Cambios múltiples de configuración pueden utilizar:

```text
prepare
validate
apply
verify
```

Ejemplo:

```text
POST /api/v1/configuration/validate
POST /api/v1/configuration/apply
```

---

# 154. Configuration Version

Toda configuración importante debe tener:

```text
config_version
```

Ejemplo:

```json
{
  "config_version": 42
}
```

---

# 155. Configuration Status

```http
GET /api/v1/configuration/status
```

Ejemplo:

```json
{
  "desired_version": 42,
  "applied_version": 41,
  "status": "pending"
}
```

---

# 156. API and System Bus

Una operación típica:

```text
HTTP POST
     ↓
API Validation
     ↓
Authorization
     ↓
Command creation
     ↓
System Bus
     ↓
Routing
     ↓
Node
     ↓
Entity
     ↓
Command Result
     ↓
API
```

---

# 157. API Does Not Bypass Bus

La API no debe hacer:

```text
HTTP
 ↓
GPIO
```

Debe hacer:

```text
HTTP
 ↓
API
 ↓
Command
 ↓
System Bus
 ↓
Entity
 ↓
Resource
```

---

# 158. Local Command Optimization

Una implementación puede optimizar:

```text
API
 ↓
Local Entity
```

pero semánticamente debe mantener el mismo modelo de Command.

La optimización no debe cambiar el contrato.

---

# 159. API Caching

Los recursos estáticos/configurables pueden usar:

```text
ETag
Cache-Control
Last-Modified
```

Los estados dinámicos deben utilizar políticas de cache apropiadas.

No se debe cachear de manera incorrecta:

```text
live state
security state
command result
```

---

# 160. State Freshness

La API debe poder informar:

```json
{
  "value": 23.4,
  "timestamp": "2026-10-05T15:30:00Z",
  "age_ms": 1200,
  "quality": "good"
}
```

Esto permite que el cliente determine si el dato está actualizado.

---

# 161. Security-Sensitive Entities

Entidades como:

```text
lock
alarm
garage door
security system
```

requieren políticas especiales.

No deben exponerse automáticamente a clientes con permisos genéricos de lectura/escritura.

---

# 162. Safety Limits

La API debe validar límites.

Ejemplo:

```json
{
  "action": "set_temperature",
  "parameters": {
    "value": 100
  }
}
```

Si la capability sólo permite:

```text
0–50 °C
```

debe responder:

```text
VALIDATION_ERROR
```

antes de enviar al dispositivo.

---

# 163. Device-Level Validation

La validación también debe realizarse en el Node.

Por lo tanto:

```text
API validation
+
Node validation
```

Ambas son necesarias.

Nunca debe asumirse que una solicitud validada por Central es segura para el hardware.

---

# 164. Defense in Depth

Validación:

```text
Client
 ↓
API
 ↓
System Bus
 ↓
Node
 ↓
Hardware
```

Cada capa puede rechazar una operación inválida.

---

# 165. API for Mobile Apps

La API debe ser adecuada para:

```text
Android
iOS
Flutter
React Native
Native apps
```

No debe depender de HTML.

---

# 166. Mobile Synchronization

Una app móvil debe poder:

```text
login
 ↓
fetch snapshot
 ↓
open WebSocket
 ↓
subscribe
 ↓
receive state updates
```

---

# 167. Offline Mobile

La app puede mantener:

```text
cached configuration
cached states
pending actions
```

pero nunca debe asumir que un comando offline fue ejecutado hasta recibir confirmación.

---

# 168. Third-Party API

Una aplicación externa debería poder hacer:

```text
GET entities
GET state
POST command
GET history
SUBSCRIBE events
```

sin conocer la arquitectura interna.

---

# 169. Example Third-Party Flow

```text
External App
     │
     │ GET /entities
     ▼
API
     │
     ▼
Entity Model
     │
     ▼
Response
```

Luego:

```text
External App
     │
     │ POST /entities/light.../commands
     ▼
API
     │
     ▼
System Bus
     │
     ▼
Node
```

---

# 170. Web UI Architecture

La Web UI debe utilizar la misma API pública.

```text
Web UI
   ↓
REST
   ↓
WebSocket
   ↓
API
```

No debe existir una API especial que solamente la Web UI conozca.

---

# 171. Zero-Code Principle

El usuario debe poder configurar:

```text
devices
entities
zones
groups
automations
scenes
integrations
```

desde la UI.

La API debe proporcionar todas las operaciones necesarias para hacerlo.

---

# 172. API Extensibility

Nuevos módulos pueden registrar:

```text
new domain
new capabilities
new commands
new event types
```

sin romper la API existente.

---

# 173. Vendor Extensions

Los módulos pueden utilizar:

```text
x-vendor-*
```

para extensiones específicas.

Ejemplo:

```json
{
  "x-vendor-feature": {
    "value": true
  }
}
```

Las extensiones no deben reemplazar campos estándar cuando éstos existen.

---

# 174. Unknown Fields

Los clientes deben ignorar campos desconocidos que no necesiten interpretar.

Esto facilita evolución futura.

---

# 175. Unknown Enum Values

Los clientes robustos deben tratar un enum desconocido como:

```text
unknown
```

y no romper toda la aplicación.

---

# 176. API Documentation

La API deberá documentarse posteriormente mediante:

```text
OpenAPI
```

preferentemente OpenAPI 3.x.

La especificación OpenAPI debe generarse a partir de:

```text
DATA-SCHEMAS.md
+
API-SPECIFICATION.md
```

y no definir modelos contradictorios.

---

# 177. OpenAPI

Se deberá crear posteriormente:

```text
openapi.yaml
```

con:

```text
paths
schemas
responses
securitySchemes
parameters
examples
```

---

# 178. API Testing

Cada endpoint debe disponer de pruebas:

```text
authentication
authorization
validation
success
failure
timeout
offline
concurrency
rate limit
```

---

# 179. Contract Testing

Debe comprobarse que:

```text
API
↔
Data Schemas
```

permanezcan compatibles.

Además:

```text
API
↔
System Bus
```

debe mantener el contrato.

---

# 180. Integration Testing

Ejemplo:

```text
REST
 ↓
Command
 ↓
System Bus
 ↓
Node
 ↓
Entity
 ↓
State
 ↓
WebSocket
```

Debe poder probarse extremo a extremo.

---

# 181. Failure Testing

Debe probarse:

```text
Node offline
Central offline
Internet offline
Transport failure
Timeout
Queue full
Invalid command
Unauthorized command
Configuration mismatch
Duplicate command
```

---

# 182. Security Testing

Debe incluir:

```text
authentication
authorization
token expiration
replay
CSRF
CORS
rate limiting
input validation
injection
privilege escalation
```

---

# 183. API Performance

La API debe estar diseñada para dispositivos con recursos limitados.

Debe evitar:

* respuestas gigantes;
* consultas innecesarias;
* serialización repetida;
* grandes cantidades de memoria;
* polling agresivo.

---

# 184. Prefer WebSocket for Live Data

No se recomienda:

```text
GET /state
cada 100 ms
```

para dashboards.

Preferentemente:

```text
WebSocket
   ↓
state_changed
```

---

# 185. Polling

Polling puede utilizarse cuando:

* WebSocket no está disponible;
* cliente muy simple;
* información de baja frecuencia;
* recuperación.

Debe utilizar intervalos razonables.

---

# 186. Embedded API

En ESP32, la API puede implementar solamente los endpoints compatibles con el hardware.

Por ejemplo:

```text
ESP32-C3 Node

✓ system
✓ devices
✓ entities
✓ states
✓ commands
✓ diagnostics

✗ global history
✗ multi-user administration
✗ complex integrations
```

Esto debe descubrirse mediante capabilities.

---

# 187. Central API

El Central puede implementar:

```text
sites
zones
groups
devices
entities
states
commands
events
history
automations
scenes
integrations
users
roles
audit
diagnostics
```

---

# 188. API Capability Profile

Cada servidor API puede anunciar:

```json
{
  "profile": "central",
  "features": [
    "multi_zone",
    "automation",
    "history",
    "websocket",
    "integrations"
  ]
}
```

Un Node:

```json
{
  "profile": "node",
  "features": [
    "local_entities",
    "commands",
    "state",
    "diagnostics"
  ]
}
```

---

# 189. API Profiles

Perfiles iniciales:

```text
node
zone_controller
central
gateway
integration
```

No deben convertirse en tipos rígidos de hardware.

Son perfiles de capacidad.

---

# 190. Multi-Central Future

La API debe poder evolucionar hacia:

```text
Central A
Central B
```

sin cambiar el modelo de Entity.

Las futuras versiones podrán incorporar:

```text
federation
replication
leader election
failover
```

---

# 191. API Availability

La API debe indicar si una operación depende del Central.

Ejemplo:

```json
{
  "operation": "global_history",
  "availability": "central_required"
}
```

Mientras:

```json
{
  "operation": "local_light_command",
  "availability": "local"
}
```

---

# 192. Local vs Global Operations

### Local

```text
entity command
local state
local diagnostics
local configuration
```

### Global

```text
site configuration
cross-zone automation
global history
external integrations
multi-site
```

---

# 193. Central Failure Behavior

Cuando Central no esté disponible:

```text
GET local entity
```

puede seguir funcionando en el Node.

Pero:

```text
GET global history
```

puede responder:

```text
503 SERVICE_UNAVAILABLE
```

con:

```text
dependency: central
```

---

# 194. API Error Dependency

Ejemplo:

```json
{
  "error": {
    "code": "DEPENDENCY_UNAVAILABLE",
    "dependency": "central",
    "request_id": "req_123"
  }
}
```

---

# 195. API Reliability Principle

> **Una API debe reflejar la realidad del sistema distribuido, no ocultar sus fallos.**

No debe devolver:

```text
200 OK
```

cuando el comando simplemente quedó perdido.

---

# 196. Accepted vs Executed

Ejemplo:

```text
202 Accepted
```

significa:

> La solicitud fue aceptada para procesamiento.

No significa:

> El dispositivo ejecutó correctamente la acción.

Para esto se utiliza:

```text
command_id
```

y posteriormente:

```text
command_result
```

---

# 197. Synchronous Commands

Para comandos extremadamente rápidos y locales se puede devolver:

```text
200 OK
```

si la ejecución realmente terminó.

Pero no debe asumirse para operaciones distribuidas.

---

# 198. Asynchronous Commands

Preferidos para:

```text
remote device
firmware update
scene
automation
long-running action
```

Respuesta:

```text
202 Accepted
```

---

# 199. Long-Running Operations

Pueden utilizar:

```http
GET /api/v1/operations/{operation_id}
```

Ejemplo:

```json
{
  "operation_id": "op_123",
  "status": "running",
  "progress": 65
}
```

---

# 200. API Design Rule

La API debe utilizar recursos semánticos:

```text
Entity
Command
Event
State
Scene
Automation
```

y evitar endpoints diseñados alrededor del hardware:

```text
/gpio
/modbus-register
/can-frame
/pwm-channel
```

Estos pueden existir solamente dentro de APIs administrativas/diagnósticas especializadas.

---

# 201. Hardware Diagnostic API

En modo experto puede existir:

```text
GET /api/v1/devices/{device_id}/hardware
GET /api/v1/devices/{device_id}/resources
```

Pero no debe ser la API utilizada por automatizaciones normales.

---

# 202. API Security Boundary

La API representa una frontera de seguridad:

```text
External Client
      │
      ▼
┌───────────────┐
│ API Security  │
└───────┬───────┘
        ▼
System
```

Nunca debe suponerse que un cliente autenticado tiene acceso completo.

---

# 203. Auditability

Las siguientes acciones deben poder auditarse:

```text
login
logout
configuration change
user change
permission change
command
scene execution
automation change
integration change
firmware update
security action
```

---

# 204. Privacy

La API debe minimizar exposición de:

* credenciales;
* tokens;
* claves;
* información innecesaria;
* datos personales;
* cámaras;
* historial sensible.

---

# 205. Camera API

Las cámaras deben utilizar recursos específicos.

Ejemplo:

```text
GET /api/v1/entities/camera.front
```

Para streaming se recomienda separar:

```text
control API
```

de:

```text
media transport
```

La API no debería transportar vídeo completo mediante JSON.

---

# 206. AI API

Entidades de IA pueden exponer:

```text
prediction
classification
confidence
model
timestamp
```

Ejemplo:

```json
{
  "entity_id": "camera.kitchen.ai",
  "prediction": "person",
  "confidence": 0.94
}
```

---

# 207. Energy API

Puede consultar:

```text
power
current
voltage
energy
```

Ejemplo:

```http
GET /api/v1/entities/energy.house/history
```

---

# 208. Water API

Puede soportar:

```text
flow
volume
pressure
leak
```

sin cambiar el modelo general.

---

# 209. Industrial API

Para equipos industriales:

```text
PLC
Modbus
CANopen
RS485
```

la API continúa trabajando con:

```text
Entity
Capability
State
Command
Event
```

---

# 210. Agriculture API

Ejemplo:

```text
soil.moisture
irrigation.valve
weather.temperature
greenhouse.humidity
```

No requiere una API completamente diferente.

---

# 211. Marine API

Ejemplo:

```text
water.temperature
tank.level
pump
bilge_alarm
battery.voltage
```

El mismo modelo continúa siendo válido.

---

# 212. Scalability

La API debe ser válida desde:

```text
1 Node
```

hasta:

```text
1000+ Nodes
```

sin cambiar el modelo conceptual.

La implementación puede utilizar diferentes estrategias de almacenamiento y routing según escala.

---

# 213. Minimal Node

Un ESP32 pequeño puede implementar:

```text
GET /system
GET /entities
GET /entities/{id}/state
POST /entities/{id}/commands
WebSocket
```

---

# 214. Full Central

El Central puede implementar toda la API.

```text
System
Sites
Zones
Groups
Devices
Resources
Capabilities
Entities
States
Commands
Events
History
Functions
Scenes
Automations
Integrations
Users
Roles
Diagnostics
Audit
```

---

# 215. API Evolution

La API debe evolucionar sin obligar a actualizar simultáneamente todos los nodos.

Ejemplo:

```text
Central v2
Node v1
```

pueden coexistir mediante:

```text
schema compatibility
capability negotiation
protocol version
```

---

# 216. Backward Compatibility

Un Central nuevo debe poder comunicarse con Nodes antiguos mientras sean compatibles.

Un Node antiguo debe ignorar funcionalidades que no soporte.

---

# 217. Forward Compatibility

Los clientes deben ignorar:

```text
unknown fields
unknown optional capabilities
```

siempre que sea seguro hacerlo.

---

# 218. Deprecation

Los endpoints obsoletos deben marcarse:

```text
deprecated
```

y mantenerse durante un período definido.

Ejemplo:

```http
Deprecation: true
```

---

# 219. API Lifecycle

```text
draft
experimental
stable
deprecated
removed
```

Sólo `stable` debe considerarse contrato de producción.

---

# 220. API Naming

Los nombres deben ser:

* consistentes;
* predecibles;
* semánticos;
* en inglés para el protocolo;
* independientes del idioma de la UI.

Ejemplo:

```text
/entities
/devices
/zones
/automations
```

La interfaz de usuario puede traducir los nombres.

---

# 221. REST Resource Hierarchy

La estructura recomendada:

```text
/sites
/zones
/groups
/devices
/resources
/capabilities
/entities
/functions
/scenes
/automations
/events
/history
/commands
/integrations
/users
/roles
/system
/diagnostics
```

---

# 222. Nested Resources

Se podrán utilizar cuando mejoren la navegación:

```text
/devices/{device_id}/entities
/zones/{zone_id}/entities
/entities/{entity_id}/state
/entities/{entity_id}/commands
```

Pero no debe crearse una jerarquía excesivamente profunda.

---

# 223. Canonical Resource

Cada objeto debe tener un endpoint canónico.

Ejemplo:

```text
/entities/light.living.main
```

Aunque también pueda encontrarse mediante:

```text
/zones/zone.living/entities
```

---

# 224. API Response Envelope

Para recursos individuales:

```json
{
  "data": {}
}
```

Para colecciones:

```json
{
  "data": [],
  "pagination": {}
}
```

La implementación final deberá mantener consistencia en toda la API.

---

# 225. Metadata

Las respuestas pueden incluir:

```json
{
  "meta": {
    "request_id": "req_123",
    "timestamp": "2026-10-05T15:30:00Z"
  }
}
```

---

# 226. Complete Example

Solicitud:

```http
POST /api/v1/entities/light.living.main/commands
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: abc123
X-Request-ID: req_001

{
  "action": "turn_on",
  "parameters": {
    "brightness": 80
  }
}
```

Respuesta:

```http
HTTP/1.1 202 Accepted
```

```json
{
  "data": {
    "command_id": "cmd_001",
    "request_id": "req_001",
    "status": "accepted"
  }
}
```

Posteriormente:

```text
WebSocket
```

recibe:

```json
{
  "type": "command_result",
  "command_id": "cmd_001",
  "status": "executed"
}
```

y:

```json
{
  "type": "state_changed",
  "entity_id": "light.living.main",
  "state": {
    "on": true,
    "brightness": 80
  }
}
```

---

# 227. End-to-End Architecture

```text
┌──────────────┐
│   Web / App  │
└──────┬───────┘
       │
 REST / WS
       │
       ▼
┌──────────────┐
│     API      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Data Model   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ System Bus   │
└──────┬───────┘
       │
       ├───────────────┐
       ▼               ▼
   Zone Node        Integration
       │
       ▼
    Entity
       │
       ▼
    Resource
       │
       ▼
   Hardware
```

---

# 228. Golden Rules

## Rule 1

> La API trabaja con Entities, no GPIO.

## Rule 2

> Los comandos representan intención, no implementación.

## Rule 3

> `202 Accepted` no significa `executed`.

## Rule 4

> Todo comando distribuido debe poder rastrearse.

## Rule 5

> Los estados y eventos son conceptos diferentes.

## Rule 6

> Las capacidades determinan qué puede hacer una Entity.

## Rule 7

> El cliente no debe asumir capabilities.

## Rule 8

> La API nunca debe saltarse las reglas de seguridad del Node.

## Rule 9

> El Central es un coordinador, no el único punto de ejecución.

## Rule 10

> La Web UI debe utilizar la misma API pública.

---

# 229. Flujo completo de una operación

```text
USER
 │
 ▼
WEB / APP
 │
 │ REST
 ▼
API
 │
 ├── Authentication
 │
 ├── Authorization
 │
 ├── Validation
 │
 ├── Idempotency
 │
 └── Command Creation
 │
 ▼
SYSTEM BUS
 │
 ├── Routing
 ├── Priority
 ├── Retry
 ├── Security
 └── Correlation
 │
 ▼
NODE
 │
 ├── Validate
 ├── Authorize
 ├── Safety
 └── Execute
 │
 ▼
HARDWARE
 │
 ▼
STATE
 │
 ▼
EVENT
 │
 ├───────────────► WebSocket
 │
 ├───────────────► Automation
 │
 ├───────────────► Integration
 │
 └───────────────► History
```

---

# 230. Document Status

```text
Estado: Arquitectura base definida

Definido:

- REST API
- WebSocket
- API versioning
- Authentication boundary
- Authorization
- Permissions
- Sites
- Zones
- Groups
- Devices
- Resources
- Capabilities
- Entities
- States
- Desired State
- Commands
- Command lifecycle
- Events
- History
- Functions
- Scenes
- Automations
- System Modes
- Discovery
- Provisioning
- Integrations
- Users
- Roles
- API Keys
- Diagnostics
- Audit
- Pagination
- Filtering
- Sorting
- Search
- ETags
- Optimistic concurrency
- Error model
- Rate limiting
- CORS
- CSRF
- TLS
- WebSocket subscriptions
- Event replay
- API capabilities
- Virtual entities
- External entities
- Third-party API
- Local Node API
- Central API
- Configuration API
- Import/export
- OpenAPI direction
- Testing strategy
- Compatibility strategy
- API evolution

Pendiente:

- OpenAPI definitivo
- JSON Schemas definitivos
- endpoint-by-endpoint schemas
- authentication implementation
- authorization implementation
- WebSocket protocol formal
- pagination schema definitivo
- error code registry
- API rate limits definitivos
- API key lifecycle
- OAuth/OIDC si posteriormente fuese necesario
- API Gateway implementation
- OTA API
- media/camera transport
- federation API
- multi-site API
```

---

# 231. Próximos documentos

La arquitectura documental queda ahora:

```text
ARCHITECTURE.md
       │
       ▼
DATA-MODEL.md
       │
       ▼
DATA-SCHEMAS.md
       │
       ▼
SYSTEM-BUS.md
       │
       ▼
API-SPECIFICATION.md
```

El siguiente documento recomendado es:

```text
DISCOVERY-PROVISIONING.md
```

porque permitirá definir el proceso completo desde que un ESP32 se enciende por primera vez hasta que queda integrado en la instalación:

```text
ESP32 nuevo
    ↓
Boot
    ↓
Identity
    ↓
Network
    ↓
Discovery
    ↓
Authentication
    ↓
Provisioning
    ↓
Hardware discovery
    ↓
Resource discovery
    ↓
Capability discovery
    ↓
Entity creation
    ↓
Zone assignment
    ↓
Configuration
    ↓
Synchronization
    ↓
Operational
```

Después conviene continuar con:

```text
EVENT-MODEL.md
CONFIGURATION-MODEL.md
DATABASE-STORAGE.md
AUTOMATION-ENGINE.md
SECURITY-ARCHITECTURE.md
OTA-UPDATE.md
TESTING-VALIDATION.md
```

Esto dejará prácticamente definida toda la arquitectura antes de comenzar la implementación de firmware, Central y Web UI.
