# DATA-MODEL.md

> **Estado:** Planificación
> **Versión:** 1.0.0
> **Tipo:** Arquitectura / Modelo de datos / Contrato del sistema
> **Proyecto:** Distributed Automation Platform
> **Última actualización:** 2026-10-05

---

# 1. Objetivo

Este documento define el modelo de datos común utilizado por toda la plataforma.

El modelo debe ser compartido por:

* Firmware ESP32
* Nodes
* Zone Controllers
* Central
* Web UI
* Aplicaciones móviles
* REST API
* WebSocket API
* System Bus
* Integraciones
* Matter
* MQTT
* Home Assistant
* Alexa
* Google Home
* Apple Home
* Homey
* SmartThings
* Aplicaciones de terceros
* Plugins
* Automatizaciones
* Historial
* Sistema de eventos

El objetivo principal es separar:

```text
HARDWARE
    ↓
RESOURCE
    ↓
CAPABILITY
    ↓
ENTITY
    ↓
FUNCTION
    ↓
ZONE / GROUP
    ↓
SCENE / AUTOMATION
    ↓
API / INTEGRATION
```

---

# 2. Principio fundamental

El usuario, una aplicación externa y una automatización nunca deberían depender directamente del hardware.

No:

```text
GPIO 12
```

ni:

```text
ESP32-001.GPIO12
```

sino:

```text
light.living
```

o:

```text
sensor.living.temperature
```

Por lo tanto:

> **El hardware puede cambiar sin cambiar la identidad lógica del recurso.**

---

# 3. Ejemplo

Inicialmente:

```text
ESP32-WROOM-32
GPIO12
Relay
```

representa:

```text
light.living
```

Posteriormente puede reemplazarse por:

```text
ESP32-S3
GPIO27
PWM
```

y continuar representando:

```text
light.living
```

La API no debería verse afectada.

---

# 4. Capas del modelo

La arquitectura utilizará las siguientes capas:

```text
┌──────────────────────────────┐
│          SITE                │
├──────────────────────────────┤
│           ZONE               │
├──────────────────────────────┤
│           GROUP              │
├──────────────────────────────┤
│          DEVICE              │
├──────────────────────────────┤
│          RESOURCE            │
├──────────────────────────────┤
│        CAPABILITY             │
├──────────────────────────────┤
│          ENTITY              │
├──────────────────────────────┤
│         FUNCTION             │
├──────────────────────────────┤
│      SCENE / AUTOMATION       │
├──────────────────────────────┤
│          EVENT               │
├──────────────────────────────┤
│           STATE              │
└──────────────────────────────┘
```

No todas las relaciones son estrictamente jerárquicas.

---

# 5. Site

Un `Site` representa una instalación física o lógica.

Ejemplos:

```text
Casa
Oficina
Fábrica
Invernadero
Campo
Barco
Edificio
Local comercial
```

Ejemplo:

```json
{
  "id": "site_home_001",
  "type": "site",
  "name": "Casa Principal"
}
```

---

# 6. Site ID

El identificador de Site debe ser:

* único;
* estable;
* independiente del nombre;
* independiente del hardware.

Ejemplo:

```text
site_01JABC...
```

No:

```text
Casa
```

como identificador interno.

---

# 7. Zone

Una `Zone` representa una ubicación o sector lógico.

Ejemplos:

```text
Living
Cocina
Dormitorio
Garage
Jardín
Piscina
Invernadero
Taller
Máquina 01
Cubierta
Sala de máquinas
```

---

# 8. Zone hierarchy

Las zonas pueden anidarse.

Ejemplo:

```text
Casa
├── Planta Baja
│   ├── Living
│   ├── Cocina
│   └── Garage
│
└── Planta Alta
    ├── Dormitorio 1
    └── Dormitorio 2
```

Esto permite instalaciones pequeñas y grandes.

---

# 9. Zone model

Ejemplo:

```json
{
  "id": "zone_living",
  "site_id": "site_home_001",
  "parent_id": "zone_ground_floor",
  "name": "Living",
  "type": "room"
}
```

---

# 10. Zone Type

Tipos posibles:

```text
site
building
floor
room
corridor
outdoor
garden
garage
workshop
industrial_area
machine
vehicle
vessel
greenhouse
custom
```

La lista debe poder ampliarse.

---

# 11. Group

Un `Group` representa una colección lógica de entidades.

No necesariamente representa una ubicación.

Ejemplo:

```text
Todas las luces
Luces exteriores
Persianas
Riego
Sensores de temperatura
Cargas críticas
```

---

# 12. Zone ≠ Group

Una zona representa principalmente:

```text
WHERE
```

Un grupo representa:

```text
WHICH OBJECTS
```

Ejemplo:

```text
Zone:
    Living

Group:
    All Lights
```

Una entidad puede pertenecer simultáneamente a:

```text
Zone = Living
Group = All Lights
Group = Evening Lights
```

---

# 13. Device

Un `Device` representa hardware físico o lógico.

Ejemplos:

```text
ESP32-S3 Central
ESP32-WROOM Node
ESP32-C6 Node
CAN Gateway
RS485 Gateway
Ethernet Node
Touch Panel
Sensor Gateway
```

---

# 14. Device identity

Ejemplo:

```json
{
  "id": "device_01JABC123",
  "name": "Living Controller",
  "manufacturer": "Platform",
  "model": "ESP32-WROOM-32",
  "firmware": {
    "version": "1.0.0"
  }
}
```

---

# 15. Device ID

El `device_id` debe ser estable durante toda la vida útil del dispositivo.

No debería depender de:

```text
IP
MAC
GPIO
hostname
position
```

La IP puede cambiar.

El Device ID no.

---

# 16. Device State

Un dispositivo puede tener:

```text
online
offline
degraded
maintenance
provisioning
error
disabled
retired
```

---

# 17. Device Health

Además del estado funcional, debe existir información de salud.

Ejemplo:

```json
{
  "health": {
    "uptime": 123456,
    "cpu_usage": 21,
    "memory_free": 183240,
    "temperature": 42.3,
    "last_seen": "2026-10-05T15:30:00Z"
  }
}
```

No todos los dispositivos deberán proporcionar todos los campos.

---

# 18. Resource

Un `Resource` representa un recurso físico o lógico disponible dentro de un Device.

Ejemplos:

```text
GPIO
PWM channel
ADC
I2C bus
I2C device
SPI device
UART
RS485
CAN
Ethernet
Wi-Fi
Sensor
Relay
Display
Touch controller
Camera
Storage
```

---

# 19. Resource hierarchy

Ejemplo:

```text
Device
└── GPIO12
    └── Relay
        └── Capability: on_off
            └── Entity: light.living
```

---

# 20. Resource ID

Ejemplo:

```text
resource_gpio12
```

Debe ser único dentro del Device.

---

# 21. Hardware resource

Ejemplo:

```json
{
  "id": "resource_gpio12",
  "type": "gpio",
  "device_id": "device_001",
  "hardware": {
    "pin": 12,
    "direction": "output"
  }
}
```

---

# 22. Resource abstraction

Una API externa no debería necesitar conocer:

```text
GPIO12
MCP23017 pin 7
74HC595 output 3
PWM channel 2
```

Debe recibir:

```text
light.living
```

---

# 23. Capability

Una `Capability` representa algo que un recurso o entidad sabe hacer.

Ejemplos:

```text
on_off
brightness
color
temperature
humidity
pressure
position
speed
power
energy
current
voltage
flow
motion
occupancy
```

---

# 24. Capability ≠ Entity

Una capability describe una capacidad.

Una entity representa un recurso lógico accesible al sistema.

Ejemplo:

```text
Device
    ↓
Relay
    ↓
Capability:
    on_off
    ↓
Entity:
    light.living
```

---

# 25. Capability schema

Ejemplo:

```json
{
  "type": "brightness",
  "unit": "%",
  "min": 0,
  "max": 100,
  "step": 1
}
```

---

# 26. Entity

Una `Entity` es uno de los conceptos más importantes del sistema.

Representa un elemento lógico que puede:

* tener estado;
* recibir comandos;
* generar eventos;
* ser leído;
* ser utilizado por automatizaciones;
* ser expuesto a integraciones.

---

# 27. Entity examples

```text
light.living
switch.pump
sensor.living.temperature
sensor.garden.humidity
binary_sensor.front_door
cover.garage
fan.bedroom
climate.house
lock.front_door
alarm.house
```

---

# 28. Entity ID

El `entity_id` debe ser:

* único;
* estable;
* legible;
* independiente del hardware.

Formato recomendado:

```text
<domain>.<unique_name>
```

Ejemplo:

```text
light.living
sensor.living_temperature
switch.pool_pump
```

---

# 29. Entity ID no debe cambiar

Cambiar:

```text
GPIO12 → GPIO27
```

no debe cambiar:

```text
light.living
```

---

# 30. Entity Domain

El `domain` identifica el tipo lógico.

Ejemplos:

```text
light
switch
sensor
binary_sensor
fan
cover
climate
lock
alarm
camera
media_player
vacuum
valve
button
scene
number
select
text
datetime
counter
timer
energy
water
```

---

# 31. Extensible Domains

Los domains deben ser extensibles.

Ejemplo:

```text
marine.depth
industrial.pressure
agriculture.irrigation
```

No obstante, para interoperabilidad se deben reutilizar domains estándar cuando sea posible.

---

# 32. Entity metadata

Ejemplo:

```json
{
  "id": "light.living",
  "name": "Luz del Living",
  "domain": "light",
  "zone_id": "zone_living",
  "device_id": "device_001"
}
```

---

# 33. Entity capabilities

Una entidad puede tener múltiples capacidades.

Ejemplo:

```json
{
  "capabilities": [
    "on_off",
    "brightness"
  ]
}
```

---

# 34. Light Entity

Ejemplo:

```json
{
  "id": "light.living",
  "domain": "light",
  "capabilities": [
    "on_off",
    "brightness"
  ]
}
```

Estado:

```json
{
  "on": true,
  "brightness": 75
}
```

---

# 35. Sensor Entity

Ejemplo:

```json
{
  "id": "sensor.living_temperature",
  "domain": "sensor",
  "device_class": "temperature",
  "unit": "°C"
}
```

Estado:

```json
{
  "value": 23.7
}
```

---

# 36. Binary Sensor

Ejemplo:

```json
{
  "id": "binary_sensor.front_door",
  "domain": "binary_sensor",
  "device_class": "door"
}
```

Estado:

```json
{
  "value": "open"
}
```

---

# 37. Device Class

La `device_class` aporta semántica.

Ejemplos:

```text
temperature
humidity
pressure
illuminance
motion
occupancy
door
window
smoke
water_leak
power
energy
voltage
current
wind_speed
wind_direction
rain
water_flow
```

---

# 38. Unit

Las mediciones deben tener unidades explícitas.

Ejemplos:

```text
°C
°F
%
Pa
hPa
lux
V
A
W
Wh
kWh
m³
L/min
m/s
km/h
```

---

# 39. Canonical Units

Internamente se recomienda utilizar unidades canónicas.

Ejemplo:

```text
temperature → °C
pressure → Pa
energy → Wh
power → W
voltage → V
current → A
```

La interfaz puede convertirlas para el usuario.

---

# 40. Unit Conversion

Ejemplo:

```text
Internal:
    23.5 °C

User:
    74.3 °F
```

El estado interno sigue siendo:

```text
23.5 °C
```

---

# 41. Precision

Cada entidad puede definir precisión.

Ejemplo:

```json
{
  "precision": 1
}
```

Resultado:

```text
23.7 °C
```

No:

```text
23.70000000001
```

para presentación.

---

# 42. Raw Value vs Display Value

Debe distinguirse:

```text
raw_value
normalized_value
display_value
```

Ejemplo:

```text
ADC
 ↓
Raw = 2874
 ↓
Normalized = 2.31 V
 ↓
Display = 2.3 V
```

---

# 43. State

El `State` representa el estado actual de una entidad.

Ejemplo:

```json
{
  "entity_id": "light.living",
  "state": {
    "on": true,
    "brightness": 75
  }
}
```

---

# 44. State Metadata

Cada estado debe poder incluir:

```text
timestamp
source
quality
confidence
origin
```

Ejemplo:

```json
{
  "timestamp": "2026-10-05T15:30:00Z",
  "source": "device_001",
  "quality": "good"
}
```

---

# 45. State Quality

Valores recomendados:

```text
good
uncertain
stale
invalid
unavailable
```

---

# 46. Stale State

Un sensor que no actualiza su valor durante demasiado tiempo puede marcarse:

```text
stale
```

Esto es diferente de:

```text
value = 0
```

---

# 47. Unknown vs Unavailable

Debe diferenciarse:

```text
unknown
```

de:

```text
unavailable
```

`unknown`:

> El sistema no conoce el estado.

`unavailable`:

> El recurso existe, pero actualmente no puede proporcionar el estado.

---

# 48. State Timestamp

Todo estado medible debería disponer de timestamp.

Ejemplo:

```json
{
  "value": 24.2,
  "timestamp": "2026-10-05T15:30:01Z"
}
```

---

# 49. Monotonic Time

Para medir intervalos locales, el firmware deberá utilizar también un reloj monotónico.

Ejemplo:

```text
last_update
timeout
debounce
duration
```

No depender exclusivamente de UTC.

---

# 50. State Source

El origen puede ser:

```text
device
central
automation
user
third_party
integration
estimated
calculated
```

---

# 51. State Provenance

Ejemplo:

```json
{
  "source": {
    "type": "device",
    "device_id": "device_001"
  }
}
```

Esto permite conocer de dónde proviene un dato.

---

# 52. Command

Un `Command` solicita modificar o actuar sobre una entidad.

Ejemplo:

```json
{
  "entity_id": "light.living",
  "command": "turn_on"
}
```

---

# 53. Command Parameters

Ejemplo:

```json
{
  "entity_id": "light.living",
  "command": "set_brightness",
  "parameters": {
    "brightness": 75
  }
}
```

---

# 54. Command Result

El sistema debe diferenciar:

```text
accepted
executed
rejected
failed
timeout
unavailable
queued
```

---

# 55. Command Lifecycle

```text
REQUESTED
   ↓
AUTHORIZED
   ↓
ACCEPTED
   ↓
QUEUED
   ↓
EXECUTING
   ↓
EXECUTED
```

O:

```text
REJECTED
FAILED
TIMEOUT
```

---

# 56. Command ID

Todo comando importante debe tener un identificador:

```text
cmd_01JABC...
```

Esto permite seguimiento.

---

# 57. Request ID

La solicitud API también debe tener:

```text
request_id
```

Un request puede generar varios comandos.

---

# 58. Idempotency

Las operaciones que puedan repetirse deben poder ser idempotentes.

Ejemplo:

```text
turn_on
```

enviado dos veces:

```text
ON
```

sigue siendo:

```text
ON
```

---

# 59. Idempotency Key

Las APIs que creen recursos o ejecuten operaciones sensibles podrán aceptar:

```http
Idempotency-Key: <key>
```

Esto evita duplicaciones ante reintentos de red.

---

# 60. Event

Un `Event` representa algo que ocurrió.

Ejemplos:

```text
motion_detected
door_opened
temperature_changed
device_online
device_offline
alarm_triggered
command_executed
```

---

# 61. Event ≠ State

Estado:

```text
door = open
```

Evento:

```text
door_opened
```

El estado representa una condición.

El evento representa una ocurrencia.

---

# 62. Event Model

Ejemplo:

```json
{
  "id": "evt_001",
  "type": "door_opened",
  "entity_id": "binary_sensor.front_door",
  "timestamp": "2026-10-05T15:30:00Z"
}
```

---

# 63. Event Data

Los eventos pueden transportar datos.

Ejemplo:

```json
{
  "type": "motion_detected",
  "entity_id": "binary_sensor.garden_motion",
  "data": {
    "confidence": 0.97
  }
}
```

---

# 64. Event Source

Debe existir:

```text
source
```

Ejemplo:

```text
device_001
automation_002
user_001
integration_003
```

---

# 65. Event Correlation

Eventos relacionados pueden compartir:

```text
correlation_id
```

Ejemplo:

```text
motion detected
    ↓
automation triggered
    ↓
light turned on
```

Todos pueden estar relacionados mediante:

```text
correlation_id
```

---

# 66. Function

Una `Function` representa una función lógica del sistema.

Ejemplos:

```text
Lighting
Heating
Cooling
Irrigation
Security
Ventilation
Energy Management
Water Management
Access Control
```

---

# 67. Function ≠ Entity

Una función puede utilizar múltiples entidades.

Ejemplo:

```text
Function:
    Garden Irrigation

Entities:
    switch.pump
    valve.garden_01
    sensor.garden_soil
    sensor.garden_rain
```

---

# 68. Function State

Una función puede tener estado propio.

Ejemplo:

```text
irrigation.garden
```

con:

```text
idle
watering
paused
error
```

---

# 69. Scene

Una `Scene` representa un estado deseado para múltiples entidades.

Ejemplo:

```text
Scene:
    Movie
```

Acciones:

```text
light.living → 20%
cover.living → closed
media_player.tv → on
```

---

# 70. Scene Model

```json
{
  "id": "scene_movie",
  "name": "Película",
  "actions": [
    {
      "entity_id": "light.living",
      "command": "set_brightness",
      "parameters": {
        "brightness": 20
      }
    }
  ]
}
```

---

# 71. Automation

Una `Automation` define lógica:

```text
TRIGGER
+
CONDITION
+
ACTION
```

---

# 72. Automation Example

```text
IF
    motion detected

AND
    HOUSE.MODE == NIGHT

THEN
    light.hallway = 15%
```

---

# 73. Automation Model

```json
{
  "id": "automation_hallway_night",
  "trigger": {
    "type": "event",
    "event": "motion_detected"
  },
  "conditions": [
    {
      "type": "state",
      "entity_id": "house.mode",
      "operator": "equals",
      "value": "night"
    }
  ],
  "actions": [
    {
      "entity_id": "light.hallway",
      "command": "set_brightness",
      "parameters": {
        "brightness": 15
      }
    }
  ]
}
```

---

# 74. Trigger

Un trigger inicia una automatización.

Tipos:

```text
event
state_change
threshold
time
schedule
sunrise
sunset
webhook
manual
device_event
system_event
```

---

# 75. Condition

Una condición determina si la automatización continúa.

Ejemplos:

```text
temperature > 28
mode == sleep
presence == home
time between 22:00 and 06:00
```

---

# 76. Action

Una acción ejecuta algo.

Tipos:

```text
command
scene
notification
delay
condition
script
webhook
event
```

---

# 77. Resource Relationships

Modelo conceptual:

```text
SITE
 │
 ├── ZONES
 │    │
 │    └── ENTITIES
 │
 ├── DEVICES
 │    │
 │    └── RESOURCES
 │
 ├── GROUPS
 │
 ├── FUNCTIONS
 │
 ├── SCENES
 │
 └── AUTOMATIONS
```

---

# 78. Device ↔ Entity

Un Device puede proporcionar múltiples entidades.

Ejemplo:

```text
ESP32 Living
├── light.living
├── sensor.living.temperature
├── sensor.living.humidity
└── binary_sensor.living.motion
```

---

# 79. Entity ↔ Device

Una entidad normalmente tendrá un Device propietario.

Pero una entidad lógica también puede ser:

```text
virtual
calculated
aggregated
```

y no depender directamente de un dispositivo.

---

# 80. Virtual Entity

Ejemplo:

```text
sensor.house.average_temperature
```

Calculado a partir de:

```text
sensor.living.temperature
sensor.bedroom.temperature
sensor.kitchen.temperature
```

---

# 81. Aggregated Entity

Ejemplo:

```text
energy.house.total_power
```

puede sumar:

```text
energy.kitchen
energy.living
energy.garage
```

---

# 82. Calculated Entity

Ejemplo:

```text
sensor.heat_index
```

calculado a partir de:

```text
temperature
humidity
```

---

# 83. Entity Ownership

Cada entidad debe tener un origen definido.

```text
physical
virtual
calculated
aggregated
external
```

---

# 84. External Entity

Una integración puede importar una entidad.

Ejemplo:

```text
weather.external.temperature
```

El sistema debe conservar el origen:

```text
source = integration.weather_api
```

---

# 85. Entity Lifecycle

Una entidad puede estar:

```text
provisioning
active
disabled
unavailable
deprecated
removed
```

---

# 86. Entity Disabled

Una entidad deshabilitada no debe eliminarse automáticamente.

Esto permite:

```text
disable
```

sin perder:

```text
configuration
history
automations
references
```

---

# 87. Entity Removal

La eliminación debe ser una operación explícita.

Antes de eliminar una entidad, el sistema debe detectar referencias:

```text
automations
scenes
groups
integrations
dashboards
```

---

# 88. Soft Delete

Se recomienda utilizar inicialmente:

```text
soft delete
```

para elementos importantes.

Esto permite recuperación.

---

# 89. Entity Alias

Una entidad puede tener aliases.

Ejemplo:

```text
id:
    light.living

name:
    Luz del Living

aliases:
    Luz salón
    Luz principal
```

---

# 90. Friendly Name

El nombre mostrado al usuario no forma parte de la identidad.

Ejemplo:

```text
entity_id:
    light.living

name:
    Luz del Living
```

El usuario puede cambiar:

```text
Luz del Living
```

por:

```text
Lámpara principal
```

sin modificar el ID.

---

# 91. Entity Icon

Las entidades pueden definir iconos.

Ejemplo:

```text
mdi:lightbulb
```

Pero el icono es metadata de presentación.

No forma parte de la lógica.

---

# 92. Entity Visibility

Una entidad puede tener:

```text
visible
hidden
internal
```

---

# 93. Entity Exposure

La entidad puede estar expuesta a integraciones concretas.

Ejemplo:

```json
{
  "exposure": {
    "matter": true,
    "home_assistant": true,
    "alexa": false,
    "google_home": true
  }
}
```

---

# 94. Entity Tags

Se podrán utilizar tags.

Ejemplo:

```text
critical
energy
security
outdoor
lighting
water
```

Esto facilita búsqueda y automatización.

---

# 95. Entity Attributes

Las entidades pueden tener atributos adicionales.

Ejemplo:

```json
{
  "attributes": {
    "manufacturer": "Example",
    "model": "XYZ",
    "installation_date": "2026-01-01"
  }
}
```

Los atributos no deben sustituir las capacidades formales.

---

# 96. Schema Version

Todos los objetos persistentes deben poder indicar versión de schema.

Ejemplo:

```json
{
  "schema_version": "1.0"
}
```

---

# 97. Backward Compatibility

Los cambios de modelo deben procurar compatibilidad hacia atrás.

Nunca modificar silenciosamente:

```text
entity_id
device_id
zone_id
```

---

# 98. Object IDs

Todos los objetos principales deben utilizar IDs estables:

```text
site_id
zone_id
group_id
device_id
resource_id
entity_id
function_id
scene_id
automation_id
event_id
command_id
```

---

# 99. ID Generation

Se recomienda utilizar identificadores suficientemente únicos.

Por ejemplo:

```text
UUID
ULID
```

ULID puede resultar especialmente interesante para:

* orden temporal;
* almacenamiento;
* logs;
* sincronización.

---

# 100. Human-readable IDs

Los nombres legibles no deben utilizarse como única garantía de unicidad.

Ejemplo:

```text
light.living
```

puede ser el identificador lógico público.

Pero internamente deberá existir una identidad estable.

---

# 101. Internal ID vs Entity ID

Se recomienda distinguir:

```text
internal_id
entity_id
```

Ejemplo:

```json
{
  "internal_id": "01JABC...",
  "entity_id": "light.living"
}
```

Esto permite cambiar el nombre lógico con mecanismos de alias/migración sin perder identidad interna.

---

# 102. Naming Rules

Los identificadores deben:

* evitar espacios;
* evitar caracteres ambiguos;
* ser case-insensitive o normalizados;
* ser estables;
* ser legibles.

Ejemplo:

```text
light.living
sensor.garden.temperature
switch.pool_pump
```

---

# 103. Localization

El nombre mostrado debe soportar traducciones.

Ejemplo:

```json
{
  "name": {
    "es": "Luz del Living",
    "en": "Living Room Light",
    "it": "Luce del soggiorno"
  }
}
```

---

# 104. Metadata Language

La lógica no debe depender del idioma.

No:

```text
if name == "Luz del Living"
```

Siempre:

```text
entity_id == "light.living"
```

---

# 105. Entity State Schema

Modelo base:

```json
{
  "entity_id": "sensor.living.temperature",
  "state": {
    "value": 23.7
  },
  "unit": "°C",
  "timestamp": "2026-10-05T15:30:00Z",
  "quality": "good",
  "source": {
    "type": "device",
    "device_id": "device_001"
  }
}
```

---

# 106. Generic Entity Schema

```json
{
  "id": "entity_001",
  "entity_id": "sensor.living.temperature",
  "domain": "sensor",
  "name": "Temperatura Living",
  "zone_id": "zone_living",
  "device_id": "device_001",
  "capabilities": [
    "temperature"
  ],
  "state": {},
  "metadata": {}
}
```

---

# 107. Generic Command Schema

```json
{
  "command_id": "cmd_001",
  "entity_id": "light.living",
  "command": "turn_on",
  "parameters": {},
  "source": {
    "type": "user",
    "id": "usr_001"
  },
  "timestamp": "2026-10-05T15:30:00Z"
}
```

---

# 108. Generic Event Schema

```json
{
  "event_id": "evt_001",
  "type": "motion_detected",
  "source": {
    "type": "device",
    "id": "device_001"
  },
  "entity_id": "binary_sensor.living_motion",
  "timestamp": "2026-10-05T15:30:00Z",
  "data": {}
}
```

---

# 109. Device Schema

```json
{
  "id": "device_001",
  "name": "Living Controller",
  "type": "controller",
  "manufacturer": "Platform",
  "model": "ESP32-WROOM-32",
  "firmware": {
    "version": "1.0.0"
  },
  "status": "online",
  "zone_id": "zone_living",
  "resources": [],
  "entities": []
}
```

---

# 110. Zone Schema

```json
{
  "id": "zone_living",
  "site_id": "site_home_001",
  "parent_id": "zone_ground_floor",
  "name": "Living",
  "type": "room",
  "entities": []
}
```

---

# 111. Group Schema

```json
{
  "id": "group_all_lights",
  "name": "Todas las luces",
  "type": "light",
  "members": [
    "light.living",
    "light.kitchen",
    "light.garden"
  ]
}
```

---

# 112. Function Schema

```json
{
  "id": "function_irrigation",
  "name": "Riego",
  "type": "irrigation",
  "entities": [
    "switch.irrigation_pump",
    "valve.garden_01"
  ]
}
```

---

# 113. Scene Schema

```json
{
  "id": "scene_night",
  "name": "Noche",
  "actions": [
    {
      "entity_id": "light.living",
      "command": "turn_off"
    },
    {
      "entity_id": "light.hallway",
      "command": "set_brightness",
      "parameters": {
        "brightness": 15
      }
    }
  ]
}
```

---

# 114. Automation Schema

```json
{
  "id": "automation_night_motion",
  "name": "Luz nocturna",
  "enabled": true,
  "trigger": {},
  "conditions": [],
  "actions": []
}
```

---

# 115. System Mode

Los modos globales también deben formar parte del modelo.

Ejemplo:

```text
normal
sleep
away
vacation
maintenance
emergency
```

Entidad conceptual:

```text
system.mode
```

---

# 116. Mode State

```json
{
  "entity_id": "system.mode",
  "state": {
    "value": "sleep"
  }
}
```

---

# 117. Modes are not hardcoded

Los dispositivos no deben decidir directamente:

```text
if sleep:
    turn off
```

Cada dispositivo debe exponer capacidades.

Las automatizaciones/policies determinan el comportamiento.

---

# 118. Capability Registry

La plataforma deberá disponer de un registro de capabilities.

Ejemplo:

```text
on_off
brightness
color
temperature
humidity
pressure
position
speed
power
energy
```

Esto evita que cada módulo invente nombres incompatibles.

---

# 119. Capability Schema Registry

Cada capability debe definir:

```text
name
type
data_type
unit
range
commands
events
state_schema
```

---

# 120. Example Capability

```json
{
  "id": "brightness",
  "type": "numeric",
  "unit": "%",
  "range": {
    "min": 0,
    "max": 100
  },
  "commands": [
    "set"
  ]
}
```

---

# 121. Capability Versioning

Las capabilities deben tener versión.

Ejemplo:

```text
brightness@1
temperature@1
energy@1
```

Esto facilita evolución futura.

---

# 122. Capability Composition

Una entidad puede combinar capabilities.

Ejemplo:

```text
Smart Light
├── on_off
├── brightness
├── color
├── color_temperature
```

---

# 123. Device Capability vs Entity Capability

Un dispositivo puede soportar una capability sin necesariamente exponerla como entidad.

Ejemplo:

```text
ESP32-S3
```

puede soportar:

```text
camera
```

pero la cámara puede estar:

```text
disabled
```

o:

```text
internal
```

---

# 124. Resource Mapping

El sistema debe mantener la relación:

```text
Entity
    ↓
Capability
    ↓
Resource
    ↓
Hardware
```

Ejemplo:

```text
light.living
    ↓
brightness
    ↓
PWM
    ↓
GPIO27
```

---

# 125. Hardware Remapping

Si se cambia:

```text
GPIO27
```

por:

```text
MCP23017 pin 4
```

el mapping cambia:

```text
brightness
    ↓
MCP23017.4
```

pero:

```text
light.living
```

permanece.

---

# 126. Mapping Configuration

El mapping debe poder configurarse desde la Web UI.

No debe ser obligatorio modificar firmware.

---

# 127. Multiple Hardware Resources

Una entidad puede utilizar varios recursos.

Ejemplo:

```text
RGB Light
├── GPIO_R
├── GPIO_G
├── GPIO_B
```

y una sola entidad:

```text
light.living
```

---

# 128. External Bus Resource

También puede utilizar:

```text
CAN
RS485
Modbus
Ethernet
I2C
SPI
```

Ejemplo:

```text
sensor.industrial.pressure
    ↓
Modbus
    ↓
RS485
    ↓
Sensor
```

---

# 129. Protocol Independence

La entidad no debe conocer el protocolo físico.

No:

```text
if modbus:
```

en la lógica de usuario.

La capa de comunicación se encarga.

---

# 130. State Synchronization

Cuando un Node cambia de estado:

```text
Node
 ↓
State Update
 ↓
System Bus
 ↓
Central
 ↓
API
 ↓
Integrations
```

---

# 131. State Authority

Cada entidad debe tener un origen de autoridad.

Ejemplo:

```text
physical sensor:
    device

virtual entity:
    central

external entity:
    integration
```

---

# 132. Conflict Resolution

Si existen múltiples fuentes:

```text
local device
central
third party
automation
```

debe existir una política clara de prioridad.

Ejemplo:

```text
Safety
>
Local physical input
>
Local automation
>
Zone automation
>
Central automation
>
Third-party command
```

Esta prioridad debe ser configurable cuando corresponda.

---

# 133. Desired vs Actual State

Los actuadores deberían diferenciar:

```text
desired_state
actual_state
```

Ejemplo:

```text
desired:
    75%

actual:
    50%
```

Esto puede ocurrir por:

* fallo;
* saturación;
* dispositivo desconectado;
* protección;
* limitación física.

---

# 134. Actuator State Model

Ejemplo:

```json
{
  "desired": {
    "brightness": 75
  },
  "actual": {
    "brightness": 50
  },
  "status": "degraded"
}
```

---

# 135. State Reconciliation

El sistema debe poder detectar:

```text
desired != actual
```

y decidir:

```text
retry
notify
fallback
ignore
```

según la entidad.

---

# 136. Sensor Quality

Los sensores podrán proporcionar:

```text
quality
confidence
calibration_status
```

Ejemplo:

```json
{
  "value": 25.2,
  "quality": "good",
  "confidence": 0.98
}
```

---

# 137. Calibration

Los sensores pueden disponer de:

```text
offset
gain
calibration_date
calibration_source
```

---

# 138. Calibration should not alter identity

Calibrar:

```text
sensor.living.temperature
```

no cambia su:

```text
entity_id
```

---

# 139. History

El modelo de datos debe permitir guardar estados históricos.

Ejemplo:

```text
timestamp
entity_id
value
quality
source
```

---

# 140. History does not belong to Entity

El historial es un servicio asociado.

No debe duplicarse dentro de cada entidad.

```text
Entity
   ↓
History Service
```

---

# 141. Retention

Cada entidad puede tener política de historial:

```text
none
short
normal
long
custom
```

---

# 142. Telemetry

Telemetry representa datos operativos del sistema.

Ejemplo:

```text
CPU
RAM
temperature
uptime
network
RSSI
packet loss
```

Debe diferenciarse de datos funcionales.

---

# 143. Diagnostics

Diagnostics representa información para mantenimiento.

Ejemplo:

```text
sensor errors
bus errors
CRC errors
restarts
watchdog resets
```

---

# 144. Diagnostics Entities

Pueden existir entidades internas:

```text
diagnostic.device.cpu
diagnostic.device.memory
diagnostic.device.uptime
```

pero no deben exponerse automáticamente a usuarios finales.

---

# 145. Availability

Cada entidad debe poder indicar disponibilidad.

```text
available
unavailable
```

Ejemplo:

```json
{
  "availability": "unavailable",
  "reason": "device_offline"
}
```

---

# 146. Availability Propagation

Si un dispositivo se desconecta:

```text
Device OFFLINE
       ↓
Entities
       ↓
UNAVAILABLE
```

pero las entidades virtuales pueden continuar funcionando si tienen otras fuentes.

---

# 147. Dependency Graph

La plataforma deberá poder representar dependencias.

Ejemplo:

```text
sensor.rain
      ↓
automation.irrigation
      ↓
valve.garden
```

---

# 148. Entity Dependencies

Ejemplo:

```json
{
  "entity_id": "sensor.heat_index",
  "depends_on": [
    "sensor.temperature",
    "sensor.humidity"
  ]
}
```

---

# 149. Dependency Failure

Si una dependencia crítica falla:

```text
heat_index
    ↓
unavailable
```

en lugar de devolver un valor falso.

---

# 150. Groups

Los grupos pueden tener estado agregado.

Ejemplo:

```text
group.lights
```

Estado:

```text
ON
```

si al menos una luz está encendida, según la política del grupo.

---

# 151. Group State Policy

Debe poder definirse:

```text
any
all
majority
aggregate
custom
```

---

# 152. Energy Model

El modelo debe soportar:

```text
voltage
current
power
energy
power_factor
frequency
```

Ejemplo:

```json
{
  "entity_id": "energy.house",
  "voltage": 231.2,
  "current": 4.7,
  "power": 1023,
  "energy": 12.8
}
```

---

# 153. Water Model

Debe soportar:

```text
flow
volume
pressure
level
valve
pump
```

Ejemplo:

```text
sensor.water.flow
sensor.tank.level
valve.garden
switch.water_pump
```

---

# 154. Environmental Model

Debe soportar:

```text
temperature
humidity
pressure
CO2
VOC
PM1
PM2.5
PM10
illuminance
UV
rain
wind
```

---

# 155. Industrial Model

Debe soportar:

```text
pressure
temperature
flow
level
speed
torque
current
voltage
energy
frequency
state
fault
alarm
```

---

# 156. Marine Model

Debe permitir:

```text
depth
water_temperature
battery_voltage
battery_current
engine_rpm
fuel_level
tank_level
bilge
wind
heading
position
```

sin modificar el modelo base.

---

# 157. Agriculture Model

Debe permitir:

```text
soil_moisture
soil_temperature
EC
pH
rain
irrigation_flow
tank_level
valve
pump
```

---

# 158. Cameras

Una cámara puede representarse como:

```text
camera.front
```

con capabilities:

```text
stream
snapshot
motion_detection
person_detection
```

La AI puede agregar:

```text
person_count
vehicle_count
object_detection
```

---

# 159. AI Entities

La IA no debe tener un modelo separado incompatible.

Ejemplo:

```text
camera.garden
        ↓
AI processing
        ↓
binary_sensor.person_garden
```

o:

```text
sensor.garden.person_count
```

---

# 160. AI Confidence

Los resultados de IA deben poder incluir:

```text
confidence
model
timestamp
```

Ejemplo:

```json
{
  "value": true,
  "confidence": 0.94,
  "model": "person_detector_v2"
}
```

---

# 161. Entity Permissions

Las entidades pueden definir requerimientos de seguridad.

Ejemplo:

```json
{
  "security": {
    "classification": "critical",
    "required_scopes": [
      "security:control"
    ]
  }
}
```

---

# 162. Data Classification

Los datos pueden clasificarse:

```text
public
normal
private
sensitive
critical
```

---

# 163. Data Classification Example

```text
Outdoor temperature
    → public

Indoor temperature
    → normal

Presence
    → private

Door lock
    → sensitive

Emergency stop
    → critical
```

---

# 164. Data Ownership

Debe poder conocerse:

```text
who created
who modified
when created
when modified
```

---

# 165. Object Audit Metadata

Ejemplo:

```json
{
  "created_by": "usr_001",
  "created_at": "2026-10-05T15:30:00Z",
  "updated_by": "usr_001",
  "updated_at": "2026-10-05T15:35:00Z"
}
```

---

# 166. Configuration vs State

Nunca mezclar:

```text
configuration
```

con:

```text
runtime state
```

Ejemplo:

Configuración:

```text
temperature unit = °C
```

Estado:

```text
temperature = 23.7
```

---

# 167. Desired Configuration

La configuración puede tener:

```text
desired configuration
applied configuration
```

Esto es especialmente importante en sistemas distribuidos.

---

# 168. Configuration Version

Cada configuración debe tener versión.

```text
config_version = 42
```

Esto permite sincronización.

---

# 169. Configuration Revision

Ejemplo:

```json
{
  "config_version": 42,
  "updated_at": "2026-10-05T15:30:00Z"
}
```

---

# 170. Distributed Synchronization

Cuando Central y Node tienen versiones diferentes:

```text
Central config = 42
Node config = 40
```

el sistema detecta:

```text
configuration drift
```

y ejecuta reconciliación.

---

# 171. Configuration Authority

Normalmente:

```text
Central
    ↓
desired configuration
```

y:

```text
Node
    ↓
applied configuration
```

Pero un Node debe poder funcionar con su última configuración válida.

---

# 172. Local Changes

Si un cambio se realiza localmente:

```text
Node local UI
```

debe convertirse en una revisión sincronizable.

---

# 173. Conflict Resolution

Si:

```text
Central config = 42
Node config = 43
```

el sistema debe determinar:

```text
which is authoritative
```

mediante reglas de versión, timestamp, prioridad o aprobación.

---

# 174. Schema Migration

Cuando cambie el modelo:

```text
schema v1
    ↓
migration
    ↓
schema v2
```

La migración debe ser explícita.

---

# 175. API Version vs Data Schema Version

No deben confundirse.

```text
API:
    /api/v1

Data schema:
    schema_version = 2
```

La API y el modelo pueden evolucionar independientemente.

---

# 176. JSON as Primary Exchange Format

Para la API REST y servicios:

```text
JSON
```

será el formato principal.

Para comunicaciones internas de alta eficiencia podrán utilizarse otros formatos.

---

# 177. Internal Binary Protocol

El System Bus podrá utilizar:

```text
CBOR
MessagePack
Protobuf
custom binary
```

según las necesidades.

El modelo lógico debe permanecer igual.

---

# 178. Serialization Independence

No debe definirse el modelo pensando únicamente en JSON.

El mismo objeto debe poder serializarse mediante:

```text
JSON
CBOR
MessagePack
Protobuf
```

cuando corresponda.

---

# 179. Null vs Unknown

Debe definirse una semántica clara.

```text
null
```

significa:

> el valor está explícitamente ausente.

Mientras:

```text
unknown
```

significa:

> el sistema no conoce el valor.

---

# 180. Boolean State

Los estados booleanos deberán evitar ambigüedad.

Preferiblemente:

```text
true
false
```

y para estados con más de dos posibilidades:

```text
enum
```

Ejemplo:

```text
open
closed
unknown
```

---

# 181. Enum Versioning

Los enums no deben cambiar de significado.

Se pueden agregar valores nuevos, pero no reutilizar valores antiguos.

---

# 182. Numeric Validation

Las entidades numéricas deben definir:

```text
min
max
step
unit
precision
```

cuando corresponda.

---

# 183. Safety Limits

Los límites físicos deben estar separados de los límites de UI.

Ejemplo:

```text
hardware max:
    100

user UI max:
    80
```

La UI no debe ser la única protección.

---

# 184. Command Validation

Antes de ejecutar un comando:

```text
authentication
authorization
validation
safety
```

deben completarse.

---

# 185. Invalid Command

Ejemplo:

```json
{
  "command": "set_brightness",
  "parameters": {
    "brightness": 150
  }
}
```

Debe rechazarse si:

```text
max = 100
```

---

# 186. API Error Model

Los errores deberán tener formato uniforme.

Ejemplo:

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Brightness must be between 0 and 100",
    "request_id": "req_001"
  }
}
```

---

# 187. Data Model and Third Parties

Un tercero nunca debería necesitar conocer:

```text
GPIO
I2C address
ESP32 model
Modbus register
CAN ID
```

salvo que esté desarrollando un driver específico.

Su API debería trabajar con:

```text
Entity
Capability
State
Command
Event
```

---

# 188. Third-Party Example

Aplicación:

```text
Energy Manager
```

consulta:

```http
GET /api/v1/entities?domain=energy
```

Obtiene:

```json
{
  "entity_id": "energy.house",
  "state": {
    "power": 1432,
    "energy": 12.4
  },
  "unit": {
    "power": "W",
    "energy": "kWh"
  }
}
```

No necesita conocer el hardware.

---

# 189. Third-Party Command

```http
POST /api/v1/entities/light.living/commands
```

```json
{
  "command": "set_brightness",
  "parameters": {
    "brightness": 50
  }
}
```

---

# 190. WebSocket State

Un cliente puede suscribirse:

```text
subscribe:
    entity = light.living
```

y recibir:

```json
{
  "event": "state_changed",
  "entity_id": "light.living",
  "state": {
    "on": true,
    "brightness": 50
  }
}
```

---

# 191. MQTT Mapping

El modelo puede mapearse a MQTT.

Ejemplo:

```text
platform/entity/light/living/state
```

Payload:

```json
{
  "on": true,
  "brightness": 50
}
```

---

# 192. Matter Mapping

Una entidad:

```text
light.living
```

puede mapearse a:

```text
Matter Light
```

La identidad interna permanece:

```text
light.living
```

---

# 193. Home Assistant Mapping

Una entidad:

```text
sensor.living_temperature
```

puede exponerse como:

```text
Home Assistant sensor
```

sin modificar el modelo interno.

---

# 194. Alexa Mapping

```text
light.living
```

puede exponerse como:

```text
Alexa Light
```

El nombre mostrado puede ser:

```text
Luz del Living
```

---

# 195. Google Home Mapping

La misma entidad:

```text
light.living
```

puede exponerse como:

```text
Google Home Light
```

---

# 196. No Duplicate Entity Models

Las integraciones no deben crear:

```text
AlexaEntity
GoogleEntity
MatterEntity
HomeAssistantEntity
```

como fuentes de verdad independientes.

Debe existir:

```text
Internal Entity
       ↓
Integration Adapter
```

---

# 197. Source of Truth

El modelo interno de la plataforma será la fuente principal de verdad.

```text
Device Model
    ↓
Entity Model
    ↓
Integration
```

---

# 198. External State

Si una integración modifica una entidad:

```text
Alexa
   ↓
Integration
   ↓
Internal Entity
```

El cambio debe convertirse en una operación interna normal.

---

# 199. No Integration Lock-In

Eliminar:

```text
Home Assistant
```

no debe eliminar:

```text
light.living
```

Eliminar:

```text
Matter
```

no debe eliminar:

```text
light.living
```

---

# 200. Data Model Lifecycle

El ciclo completo será:

```text
Hardware detected
        ↓
Resource created
        ↓
Capability detected
        ↓
Entity created
        ↓
Entity assigned to Zone
        ↓
Entity exposed
        ↓
Entity used by automation
        ↓
Entity integrated
        ↓
Entity state/history
```

---

# 201. Complete Example

```text
SITE
└── Casa
    │
    ├── ZONE
    │   └── Living
    │
    └── DEVICE
        └── ESP32 Living
            │
            ├── RESOURCE
            │   ├── GPIO12
            │   ├── GPIO13
            │   └── I2C
            │
            ├── ENTITY
            │   ├── light.living
            │   ├── sensor.living.temperature
            │   └── binary_sensor.living.motion
            │
            └── CAPABILITIES
                ├── on_off
                ├── temperature
                └── motion
```

---

# 202. Complete Logical Example

```text
light.living
    │
    ├── Domain
    │     light
    │
    ├── Zone
    │     living
    │
    ├── Device
    │     esp32_living
    │
    ├── Resource
    │     gpio12
    │
    ├── Capability
    │     on_off
    │
    ├── State
    │     ON
    │
    ├── Group
    │     all_lights
    │
    ├── Scene
    │     movie
    │
    ├── Automation
    │     night_motion
    │
    └── Integrations
          ├── Matter
          ├── Home Assistant
          ├── Alexa
          └── Google Home
```

---

# 203. Golden Rule

Toda funcionalidad nueva debe intentar integrarse en el modelo existente.

Antes de crear:

```text
NewObject
NewProtocolObject
NewIntegrationObject
```

preguntar:

```text
¿Puede representarse mediante:

Resource
Capability
Entity
Function
Event
Command
State
```

Si la respuesta es sí, se debe reutilizar el modelo existente.

---

# 204. Extensibility

El modelo debe permitir agregar nuevas tecnologías sin cambiar su núcleo.

Ejemplo:

```text
ESP32
↓
CAN
↓
Modbus
↓
Matter
↓
AI
↓
Satellite
```

Todos pueden terminar representados como:

```text
Device
Resource
Capability
Entity
State
Event
Command
```

---

# 205. Separation of Concerns

La plataforma debe mantener:

```text
Hardware
    ↓
Driver
    ↓
Resource
    ↓
Capability
    ↓
Entity
    ↓
Function
    ↓
Automation
    ↓
Integration
```

Cada capa tiene una responsabilidad.

---

# 206. Anti-pattern

No se debe hacer:

```text
GPIO12
   ↓
Alexa
```

ni:

```text
Modbus Register 40001
   ↓
Home Assistant
```

directamente.

Debe ser:

```text
Modbus Register
   ↓
Resource
   ↓
Capability
   ↓
Entity
   ↓
Integration
```

---

# 207. Data Model Rules

## Rule 1

Los IDs lógicos son independientes del hardware.

## Rule 2

El hardware puede cambiar sin romper las entidades.

## Rule 3

Las capabilities deben estar normalizadas.

## Rule 4

Los estados deben tener timestamp.

## Rule 5

Los eventos y estados son conceptos diferentes.

## Rule 6

Las integraciones no son fuentes independientes de verdad.

## Rule 7

Las entidades virtuales son ciudadanos de primera clase.

## Rule 8

Las entidades pueden depender de otras entidades.

## Rule 9

La configuración y el estado deben mantenerse separados.

## Rule 10

Todos los objetos importantes deben ser versionables.

---

# 208. Modelo final

La arquitectura conceptual definitiva es:

```text
                         SITE
                          │
             ┌────────────┼────────────┐
             │            │            │
           ZONE         DEVICE       GROUP
             │            │
             │         RESOURCE
             │            │
             │       CAPABILITY
             │            │
             └─────── ENTITY ─────────┐
                         │             │
                    ┌────┴────┐        │
                    │         │        │
                  STATE      EVENT    COMMAND
                    │         │        │
                    └────┬────┘        │
                         │             │
                     FUNCTION          │
                         │             │
                  SCENE / AUTOMATION   │
                         │             │
                         └──────┬──────┘
                                │
                         SYSTEM BUS
                                │
          ┌─────────────┬───────┼────────┬─────────────┐
          │             │       │        │             │
        REST         WebSocket MQTT    Matter      Third Party
          │             │       │        │             │
          └─────────────┴───────┴────────┴─────────────┘
```

---

# 209. Principio definitivo

> **El hardware proporciona recursos. Los recursos proporcionan capacidades. Las capacidades forman entidades. Las entidades tienen estados, reciben comandos y generan eventos. Las funciones, escenas y automatizaciones utilizan esas entidades. Las integraciones exponen el mismo modelo hacia sistemas externos.**

Por lo tanto:

```text
HARDWARE
    ↓
RESOURCE
    ↓
CAPABILITY
    ↓
ENTITY
    ↓
STATE / EVENT / COMMAND
    ↓
FUNCTION
    ↓
SCENE / AUTOMATION
    ↓
SYSTEM BUS
    ↓
API / INTEGRATIONS
```

Este modelo será el **contrato común de toda la plataforma** y deberá considerarse una de las piezas fundamentales antes de comenzar el desarrollo del firmware.
