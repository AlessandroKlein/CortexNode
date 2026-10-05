# DATA-SCHEMAS.md

# Data Schemas — Distributed Automation Platform

> **Tipo:** Especificación técnica
> **Estado:** Diseño
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05
> **Objetivo:** Definir los esquemas de datos formales utilizados por dispositivos, nodos, zonas, Central, API, System Bus, Web UI, automatizaciones e integraciones externas.

---

## 1. Objetivo

`DATA-SCHEMAS.md` define los contratos estructurales de datos de la plataforma.

Mientras `DATA-MODEL.md` define **qué conceptos existen y cómo se relacionan**, este documento define **cómo se representan esos conceptos técnicamente**.

La plataforma debe utilizar estos esquemas como contrato común entre:

```text
┌───────────────────────────────────────────────┐
│                 DATA SCHEMAS                  │
├───────────────────────────────────────────────┤
│                                               │
│ Firmware       ESP32 / Nodes                  │
│ Central        ESP32-S3                       │
│ Web UI         Administración                 │
│ API            REST / WebSocket               │
│ System Bus     Comunicación interna           │
│ Storage        Persistencia                   │
│ Automations    Reglas                         │
│ Integrations   Matter / MQTT / HA / etc.      │
│ Mobile Apps    Clientes futuros               │
│ Third Party    Aplicaciones externas          │
│                                               │
└───────────────────────────────────────────────┘
```

La misma entidad lógica debe mantener una estructura compatible independientemente de dónde se encuentre almacenada o transportada.

---

# 2. Principios

## 2.1 Source of Truth

El modelo interno de datos es la fuente de verdad de la plataforma.

```text
Hardware
    ↓
Resource
    ↓
Capability
    ↓
Entity
    ↓
State
    ↓
Command / Event
    ↓
Integration
```

Las integraciones externas **no deben modificar directamente el modelo físico**.

---

## 2.2 Independencia del hardware

Los esquemas nunca deben depender directamente de:

* GPIO;
* número de pin;
* dirección I2C;
* registro Modbus;
* CAN ID;
* dirección MAC;
* IP;
* endpoint MQTT;
* endpoint Matter.

Estos datos pertenecen al modelo físico/configuración.

Por ejemplo:

```json
{
  "entity_id": "light.living.main",
  "state": {
    "on": true
  }
}
```

debe seguir siendo válido aunque el dispositivo cambie de:

```text
GPIO 12
```

a:

```text
GPIO 25
```

---

# 3. Formato principal

El formato principal de intercambio será:

```text
JSON
```

Se utilizará para:

* API REST;
* WebSocket;
* configuración;
* archivos de configuración;
* eventos;
* debugging;
* documentación;
* integración externa.

Para transportes de bajo nivel podrán utilizarse representaciones binarias equivalentes:

* CBOR;
* MessagePack;
* Protobuf;
* formato binario propio.

La representación binaria debe conservar la semántica del modelo JSON.

---

# 4. JSON Schema

Los esquemas deberán ser compatibles preferentemente con:

```text
JSON Schema Draft 2020-12
```

Referencia:

```text
https://json-schema.org/draft/2020-12/schema
```

Los esquemas oficiales deberán almacenarse posteriormente en:

```text
schemas/
├── common/
├── site/
├── zone/
├── group/
├── device/
├── resource/
├── capability/
├── entity/
├── state/
├── command/
├── event/
├── automation/
├── scene/
├── function/
├── system/
└── integration/
```

---

# 5. Convenciones generales

## 5.1 Identificadores

Todo objeto persistente debe poseer un identificador estable.

Tipos principales:

```text
site_id
zone_id
group_id
device_id
resource_id
capability_id
entity_id
function_id
scene_id
automation_id
command_id
event_id
request_id
correlation_id
```

Se recomienda:

```text
UUID
```

o:

```text
ULID
```

Los identificadores nunca deben reutilizarse después de eliminar permanentemente un objeto.

---

# 6. Identificadores lógicos

Los objetos pueden tener:

### ID interno

```json
{
  "id": "01JXYZ..."
}
```

### ID lógico

```json
{
  "entity_id": "light.living.main"
}
```

El ID interno identifica el objeto físicamente dentro del sistema.

El ID lógico facilita:

* API;
* automatizaciones;
* integración;
* debugging;
* configuración;
* uso humano.

---

# 7. Nombres

Los nombres visibles no deben utilizarse como identificadores.

Ejemplo:

```json
{
  "entity_id": "light.living.main",
  "name": "Luz principal del living"
}
```

El usuario puede cambiar:

```text
Luz principal del living
```

sin modificar:

```text
light.living.main
```

---

# 8. Versionado

Todos los objetos persistentes deberán permitir conocer la versión de su esquema.

Ejemplo:

```json
{
  "schema": "entity",
  "schema_version": "1.0.0"
}
```

Se utilizará Semantic Versioning:

```text
MAJOR.MINOR.PATCH
```

### MAJOR

Cambios incompatibles.

### MINOR

Nuevos campos o capacidades compatibles.

### PATCH

Correcciones sin cambio estructural.

---

# 9. Timestamps

Los timestamps deben utilizar:

```text
ISO 8601 / RFC 3339
```

Ejemplo:

```text
2026-10-05T15:32:10Z
```

Cuando sea necesario conservar precisión elevada:

```text
2026-10-05T15:32:10.123Z
```

Nunca se deberá interpretar un timestamp sin zona horaria.

---

# 10. Objeto base

Todos los objetos persistentes deberían compartir una estructura conceptual común.

```json
{
  "id": "01J...",
  "schema": "entity",
  "schema_version": "1.0.0",
  "created_at": "2026-10-05T15:00:00Z",
  "updated_at": "2026-10-05T15:30:00Z",
  "metadata": {}
}
```

---

# 11. Metadata

Los objetos pueden incorporar metadata no funcional.

Ejemplo:

```json
{
  "metadata": {
    "manufacturer": "Example",
    "model": "ABC-100",
    "installation": "2026",
    "location_note": "Entrada principal"
  }
}
```

La metadata no debe modificar el comportamiento principal del objeto.

---

# 12. Extensiones

Para permitir extensibilidad se recomienda reservar:

```text
x-*
```

Ejemplo:

```json
{
  "entity_id": "sensor.temperature.room",
  "x-vendor": {
    "custom_parameter": 123
  }
}
```

Las extensiones no deben romper clientes que no las conozcan.

---

# 13. Site

Un `Site` representa una instalación completa.

Ejemplo:

```json
{
  "site_id": "site.home",
  "name": "Casa",
  "timezone": "America/Argentina/Buenos_Aires",
  "locale": "es-AR",
  "units": "metric",
  "status": "active"
}
```

Campos mínimos:

```text
site_id
name
timezone
locale
units
status
```

Estados:

```text
active
disabled
maintenance
```

---

# 14. Zone

Una zona representa una ubicación o sector lógico.

Ejemplo:

```json
{
  "zone_id": "zone.living",
  "site_id": "site.home",
  "parent_zone_id": null,
  "name": "Living",
  "type": "room",
  "status": "active"
}
```

Tipos posibles:

```text
house
floor
room
corridor
garage
garden
office
workshop
warehouse
greenhouse
field
boat
vehicle
industrial_area
custom
```

El modelo debe permitir nuevas categorías.

---

# 15. Group

Un grupo es una colección lógica.

Un grupo no necesariamente representa una ubicación.

Ejemplo:

```json
{
  "group_id": "group.downstairs_lights",
  "site_id": "site.home",
  "name": "Luces planta baja",
  "entity_ids": [
    "light.living.main",
    "light.kitchen.main",
    "light.hall.main"
  ]
}
```

---

# 16. Device

Un `Device` representa un dispositivo físico o lógico.

Ejemplo:

```json
{
  "device_id": "device.living.controller",
  "site_id": "site.home",
  "name": "Controlador Living",
  "manufacturer": "Example",
  "model": "ESP32-S3",
  "firmware": {
    "name": "AutomationNode",
    "version": "1.0.0"
  },
  "status": "online"
}
```

Estados:

```text
online
offline
degraded
maintenance
unknown
```

---

# 17. Resource

Un recurso representa una capacidad física o de comunicación del dispositivo.

Ejemplo:

```json
{
  "resource_id": "resource.device_living.gpio12",
  "device_id": "device.living.controller",
  "type": "gpio",
  "direction": "output",
  "pin": 12,
  "status": "available"
}
```

Tipos:

```text
gpio
pwm
adc
i2c
spi
uart
rs485
can
ethernet
wifi
bluetooth
zigbee
thread
matter
relay
sensor
display
touch
camera
storage
audio
```

Los tipos deben ser extensibles.

---

# 18. Capability

Una capability describe qué puede hacer un recurso o entidad.

Ejemplo:

```json
{
  "capability_id": "capability.light.on_off",
  "type": "on_off",
  "entity_id": "light.living.main",
  "readable": true,
  "writable": true
}
```

Tipos comunes:

```text
on_off
brightness
color
temperature
humidity
pressure
illuminance
motion
occupancy
position
speed
power
energy
voltage
current
flow
co2
pm25
wind_speed
wind_direction
rain
```

---

# 19. Entity

La `Entity` es uno de los objetos centrales del sistema.

Ejemplo:

```json
{
  "entity_id": "light.living.main",
  "device_id": "device.living.controller",
  "zone_id": "zone.living",
  "domain": "light",
  "name": "Luz principal",
  "device_class": "light",
  "capabilities": [
    "on_off",
    "brightness"
  ],
  "state": {
    "on": true,
    "brightness": 75
  },
  "availability": "available"
}
```

---

# 20. Domain

El `domain` determina el tipo funcional principal.

Ejemplos:

```text
light
switch
fan
cover
lock
sensor
binary_sensor
button
climate
alarm_control_panel
camera
media_player
vacuum
water_valve
energy_meter
scene
group
number
select
text
datetime
timer
counter
input_boolean
input_number
virtual
```

El sistema debe permitir nuevos domains.

---

# 21. Device Class

`device_class` permite especificar el significado físico.

Ejemplos:

```text
temperature
humidity
pressure
illuminance
power
energy
voltage
current
motion
door
window
smoke
water_leak
occupancy
presence
wind
rain
water_flow
```

---

# 22. Units

Las unidades deben ser explícitas cuando correspondan.

Ejemplos:

```text
temperature → °C
pressure → Pa
power → W
energy → Wh
voltage → V
current → A
flow → L/min
distance → m
speed → m/s
```

La plataforma deberá almacenar valores en unidades canónicas internas.

Las interfaces pueden convertirlas para presentación.

Ejemplo:

```text
Interno:
20 °C

UI:
68 °F
```

El valor interno permanece:

```text
20 °C
```

---

# 23. State

El estado representa una observación actual.

Ejemplo:

```json
{
  "entity_id": "sensor.living.temperature",
  "state": {
    "value": 23.4,
    "unit": "°C",
    "timestamp": "2026-10-05T15:32:10Z",
    "quality": "good",
    "source": "device.living.controller"
  }
}
```

---

# 24. State Quality

Valores estándar:

```text
good
uncertain
stale
invalid
unavailable
```

### good

Valor válido y actualizado.

### uncertain

Valor disponible pero con alguna incertidumbre.

### stale

El valor es válido pero demasiado antiguo.

### invalid

El valor no es válido.

### unavailable

No existe actualmente una lectura válida.

---

# 25. State Provenance

Todo estado importante debería permitir conocer su origen.

Ejemplo:

```json
{
  "source": {
    "device_id": "device.weather",
    "resource_id": "resource.weather.i2c",
    "method": "sensor"
  }
}
```

También puede indicar:

```text
sensor
calculated
aggregated
manual
automation
integration
external
ai
```

---

# 26. Desired State vs Actual State

Los actuadores deben distinguir entre:

```text
desired_state
```

y:

```text
actual_state
```

Ejemplo:

```json
{
  "desired_state": {
    "on": true,
    "brightness": 80
  },
  "actual_state": {
    "on": false,
    "brightness": 0
  }
}
```

Esto permite detectar:

* fallo del actuador;
* pérdida de comunicación;
* bloqueo;
* discrepancia;
* dispositivo apagado;
* ejecución pendiente.

---

# 27. Command

Un comando representa una solicitud para cambiar un estado o ejecutar una acción.

Ejemplo:

```json
{
  "command_id": "cmd_01JXYZ",
  "request_id": "req_01JXYZ",
  "entity_id": "light.living.main",
  "action": "turn_on",
  "parameters": {
    "brightness": 80
  },
  "source": "web",
  "priority": "normal",
  "created_at": "2026-10-05T15:32:00Z"
}
```

---

# 28. Command Lifecycle

El estado de un comando puede ser:

```text
requested
authorized
accepted
queued
executing
executed
rejected
failed
timeout
cancelled
```

Flujo:

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

Error:

```text
REQUESTED
    ↓
REJECTED
```

o:

```text
EXECUTING
    ↓
FAILED
```

---

# 29. Command Result

Ejemplo:

```json
{
  "command_id": "cmd_01JXYZ",
  "status": "executed",
  "executed_at": "2026-10-05T15:32:01Z",
  "result": {
    "success": true
  }
}
```

En caso de error:

```json
{
  "command_id": "cmd_01JXYZ",
  "status": "failed",
  "error": {
    "code": "DEVICE_UNAVAILABLE",
    "message": "Device is offline"
  }
}
```

---

# 30. Idempotencia

Los comandos deben poder manejar reintentos.

Se recomienda:

```text
Idempotency-Key
```

Ejemplo:

```text
Idempotency-Key: 01JXYZ...
```

Si un cliente reenvía exactamente el mismo comando debido a una pérdida de comunicación, el sistema no debe ejecutar accidentalmente la acción dos veces cuando esta sea no idempotente.

---

# 31. Event

Un evento representa algo que ocurrió.

Ejemplo:

```json
{
  "event_id": "evt_01JXYZ",
  "type": "binary_sensor.state_changed",
  "entity_id": "binary_sensor.front_door",
  "timestamp": "2026-10-05T15:40:00Z",
  "data": {
    "previous": false,
    "current": true
  },
  "source": "device.security"
}
```

Un evento no reemplaza al estado.

```text
STATE
    = situación actual

EVENT
    = algo que ocurrió
```

---

# 32. Event Correlation

Los eventos relacionados deben poder agruparse mediante:

```text
correlation_id
```

Ejemplo:

```json
{
  "event_id": "evt_01",
  "correlation_id": "alarm_123",
  "type": "motion.detected"
}
```

Esto permite seguir una secuencia:

```text
Motion detected
      ↓
Automation triggered
      ↓
Light turned on
      ↓
Alarm notification
      ↓
User acknowledged
```

---

# 33. Function

Una función representa una función lógica del sistema.

Ejemplos:

```text
lighting
heating
cooling
ventilation
irrigation
security
access
energy
water
climate
alarm
```

Ejemplo:

```json
{
  "function_id": "function.living.lighting",
  "name": "Iluminación Living",
  "zone_id": "zone.living",
  "entity_ids": [
    "light.living.main",
    "light.living.ambient"
  ]
}
```

---

# 34. Scene

Una escena representa un conjunto de estados deseados.

Ejemplo:

```json
{
  "scene_id": "scene.movie",
  "name": "Modo película",
  "actions": [
    {
      "entity_id": "light.living.main",
      "command": {
        "action": "turn_on",
        "brightness": 20
      }
    },
    {
      "entity_id": "light.living.ambient",
      "command": {
        "action": "turn_on",
        "brightness": 10
      }
    }
  ]
}
```

---

# 35. Automation

Una automatización está compuesta por:

```text
Trigger
    +
Conditions
    +
Actions
```

Ejemplo:

```json
{
  "automation_id": "automation.hall.motion",
  "name": "Luz del pasillo",
  "enabled": true,
  "trigger": {
    "type": "state_change",
    "entity_id": "binary_sensor.hall.motion",
    "to": true
  },
  "conditions": [
    {
      "type": "time_range",
      "after": "22:00",
      "before": "07:00"
    }
  ],
  "actions": [
    {
      "type": "command",
      "entity_id": "light.hall",
      "action": "turn_on",
      "parameters": {
        "brightness": 20
      }
    }
  ]
}
```

---

# 36. Trigger

Tipos iniciales:

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

Ejemplo:

```json
{
  "type": "threshold",
  "entity_id": "sensor.greenhouse.temperature",
  "operator": ">",
  "value": 30
}
```

---

# 37. Condition

Una condición determina si una automatización puede ejecutarse.

Tipos:

```text
state
comparison
time_range
day_of_week
mode
zone_state
presence
permission
device_availability
```

Ejemplo:

```json
{
  "type": "mode",
  "mode": "sleep"
}
```

---

# 38. Action

Una acción representa una operación.

Tipos:

```text
command
scene
notification
event
delay
condition
automation
webhook
integration
```

Ejemplo:

```json
{
  "type": "command",
  "entity_id": "fan.bedroom",
  "action": "turn_on"
}
```

---

# 39. System Mode

Los modos globales deben representarse como datos.

Ejemplo:

```json
{
  "mode": "sleep",
  "site_id": "site.home",
  "enabled": true
}
```

Modos estándar:

```text
normal
sleep
away
vacation
maintenance
emergency
custom
```

Las automatizaciones deben consultar el modo en lugar de implementar comportamientos específicos directamente en firmware.

---

# 40. Error

Todos los componentes deben utilizar una estructura común de errores.

```json
{
  "code": "ENTITY_NOT_FOUND",
  "message": "Entity does not exist",
  "details": {},
  "request_id": "req_01JXYZ"
}
```

Códigos recomendados:

```text
INVALID_REQUEST
INVALID_PARAMETER
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
TIMEOUT
DEVICE_UNAVAILABLE
RESOURCE_UNAVAILABLE
ENTITY_UNAVAILABLE
COMMAND_REJECTED
COMMAND_FAILED
SCHEMA_INVALID
VERSION_UNSUPPORTED
RATE_LIMITED
INTERNAL_ERROR
```

---

# 41. Availability

Los objetos deben diferenciar:

```text
enabled
```

de:

```text
available
```

Ejemplo:

```json
{
  "enabled": true,
  "availability": "unavailable"
}
```

Esto significa:

> El usuario habilitó el objeto, pero actualmente no está disponible.

---

# 42. Lifecycle

Los objetos persistentes pueden utilizar:

```text
provisioning
active
disabled
unavailable
deprecated
removed
```

Flujo típico:

```text
PROVISIONING
     ↓
ACTIVE
     ↓
DISABLED
     ↓
REMOVED
```

---

# 43. Configuration vs State

La configuración y el estado no deben mezclarse.

### Configuration

Define cómo funciona un objeto.

```json
{
  "configuration": {
    "sample_interval": 10,
    "min_temperature": 18,
    "max_temperature": 28
  }
}
```

### State

Describe qué está ocurriendo.

```json
{
  "state": {
    "temperature": 23.4
  }
}
```

---

# 44. Desired Configuration vs Applied Configuration

Los nodos deben poder distinguir:

```text
desired configuration
```

de:

```text
applied configuration
```

Ejemplo:

```json
{
  "configuration": {
    "desired_version": 12,
    "applied_version": 11
  }
}
```

Esto permite detectar:

```text
Central:
config v12

Node:
config v11
```

y comenzar una reconciliación.

---

# 45. Configuration Version

Toda configuración distribuida debe tener versión.

Ejemplo:

```json
{
  "config_version": 12
}
```

Una configuración debe poder compararse mediante:

```text
version
checksum
hash
timestamp
```

---

# 46. Sensor Data

Los sensores deberán utilizar un modelo común.

Ejemplo:

```json
{
  "entity_id": "sensor.weather.temperature",
  "domain": "sensor",
  "device_class": "temperature",
  "state": {
    "value": 24.7,
    "unit": "°C",
    "timestamp": "2026-10-05T15:30:00Z",
    "quality": "good"
  }
}
```

---

# 47. Binary Sensor

Ejemplo:

```json
{
  "entity_id": "binary_sensor.front_door",
  "domain": "binary_sensor",
  "device_class": "door",
  "state": {
    "value": true,
    "timestamp": "2026-10-05T15:30:00Z"
  }
}
```

---

# 48. Energy

Los medidores de energía deben poder representar:

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
  "entity_id": "sensor.house.power",
  "device_class": "power",
  "state": {
    "value": 1240.5,
    "unit": "W",
    "timestamp": "2026-10-05T15:30:00Z"
  }
}
```

---

# 49. Water

El modelo debe permitir:

```text
water_flow
water_volume
pressure
leak
valve
pump
```

Ejemplo:

```json
{
  "entity_id": "sensor.main_water_flow",
  "device_class": "water_flow",
  "state": {
    "value": 8.2,
    "unit": "L/min"
  }
}
```

---

# 50. Environmental

Debe soportarse:

```text
temperature
humidity
pressure
illuminance
CO2
PM1
PM2.5
PM10
VOC
wind
rain
UV
```

La incorporación de una nueva variable ambiental no debería requerir modificar el núcleo.

---

# 51. AI / Computer Vision

Las entidades generadas mediante IA deben conservar información de confianza.

Ejemplo:

```json
{
  "entity_id": "camera.garage.person_detection",
  "domain": "binary_sensor",
  "state": {
    "value": true,
    "confidence": 0.94,
    "source": "ai"
  }
}
```

Para detecciones múltiples:

```json
{
  "detections": [
    {
      "class": "person",
      "confidence": 0.94,
      "count": 2
    },
    {
      "class": "vehicle",
      "confidence": 0.87,
      "count": 1
    }
  ]
}
```

La IA debe comportarse como una fuente de datos/capabilities, no como una arquitectura paralela.

---

# 52. Virtual Entities

Una entidad no necesita estar asociada a hardware.

Ejemplo:

```json
{
  "entity_id": "sensor.house.average_temperature",
  "domain": "sensor",
  "device_class": "temperature",
  "source": "calculated"
}
```

Puede derivarse de:

```text
sensores
otras entidades
automatizaciones
IA
integraciones externas
datos históricos
```

---

# 53. Aggregated Entities

Ejemplo:

```text
sensor.house.total_power
```

puede representar:

```text
living power
+
kitchen power
+
garage power
```

El modelo debe permitir conservar la relación de origen.

---

# 54. External Entities

Las entidades externas deben poder representarse sin convertirlas necesariamente en dispositivos físicos.

Ejemplo:

```json
{
  "entity_id": "weather.external.temperature",
  "domain": "sensor",
  "source": "external",
  "integration": "weather_service"
}
```

---

# 55. Dependencies

Los objetos pueden declarar dependencias.

Ejemplo:

```json
{
  "entity_id": "fan.greenhouse",
  "dependencies": [
    "sensor.greenhouse.temperature",
    "sensor.greenhouse.humidity"
  ]
}
```

Esto permite:

* diagnóstico;
* propagación de disponibilidad;
* análisis de fallos;
* visualización de dependencias.

---

# 56. Permissions

Los objetos pueden declarar permisos o requerir scopes.

Ejemplo:

```json
{
  "permissions": {
    "read": [
      "entity:read"
    ],
    "write": [
      "entity:control"
    ]
  }
}
```

La autorización completa se define en:

```text
API-AUTHENTICATION-AUTHORIZATION.md
```

---

# 57. Visibility

Las entidades pueden controlar dónde aparecen.

```json
{
  "visibility": {
    "web": true,
    "mobile": true,
    "matter": true,
    "mqtt": true,
    "third_party_api": false
  }
}
```

Esto permite que una entidad exista internamente sin ser necesariamente expuesta externamente.

---

# 58. Tags

Los objetos pueden tener etiquetas:

```json
{
  "tags": [
    "critical",
    "energy",
    "outdoor"
  ]
}
```

Las etiquetas pueden utilizarse para:

* filtros;
* automatizaciones;
* permisos;
* dashboards;
* integraciones.

---

# 59. History

Los cambios de estado deben poder almacenarse como series temporales.

Ejemplo:

```json
{
  "entity_id": "sensor.living.temperature",
  "timestamp": "2026-10-05T15:30:00Z",
  "value": 23.4
}
```

No todas las entidades necesitan almacenar histórico.

La política debe ser configurable:

```text
disabled
changes_only
interval
full
```

---

# 60. Telemetry

Telemetry puede contener:

```text
CPU
RAM
flash
temperature
uptime
Wi-Fi RSSI
Ethernet link
packet loss
task health
watchdog
storage
power consumption
```

Debe distinguirse de las entidades funcionales.

---

# 61. Diagnostics

Los dispositivos pueden generar diagnósticos:

```json
{
  "device_id": "device.node01",
  "diagnostics": {
    "uptime": 123456,
    "free_heap": 184320,
    "cpu_load": 32,
    "wifi_rssi": -54
  }
}
```

---

# 62. API Response Envelope

Las APIs pueden utilizar una envoltura estándar.

Respuesta exitosa:

```json
{
  "success": true,
  "data": {},
  "request_id": "req_01JXYZ"
}
```

Error:

```json
{
  "success": false,
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity not found"
  },
  "request_id": "req_01JXYZ"
}
```

Para listas:

```json
{
  "success": true,
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total": 120
  },
  "request_id": "req_01JXYZ"
}
```

---

# 63. Pagination

Las APIs que devuelvan colecciones deben soportar paginación.

Campos recomendados:

```text
page
page_size
total
next
previous
```

Para grandes instalaciones se recomienda cursor pagination:

```json
{
  "next_cursor": "eyJ..."
}
```

---

# 64. Filtering

Los recursos deberán permitir filtros.

Ejemplos conceptuales:

```text
?zone_id=zone.living
?domain=light
?availability=available
?tag=critical
```

Los filtros específicos pertenecen a `API-SPECIFICATION.md`.

---

# 65. Sorting

Las colecciones podrán ordenar por:

```text
name
created_at
updated_at
timestamp
priority
```

---

# 66. Optimistic Concurrency

Los objetos modificables deberían soportar control de concurrencia.

Ejemplo:

```text
version: 17
```

Cliente:

```text
PATCH entity
If-Version: 17
```

Si el objeto ya está en:

```text
version: 18
```

el servidor debe devolver:

```text
CONFLICT
```

Esto evita sobrescribir cambios realizados por otro administrador.

---

# 67. ETag

Las APIs HTTP podrán utilizar:

```text
ETag
```

para:

* cache;
* sincronización;
* control de cambios;
* reducción de tráfico.

---

# 68. Null, Unknown y Unavailable

Estos conceptos no deben confundirse.

### `null`

El valor explícitamente no existe.

### `unknown`

El sistema no conoce el valor.

### `unavailable`

El recurso no está actualmente disponible.

Ejemplo:

```json
{
  "value": null,
  "quality": "unavailable"
}
```

---

# 69. Boolean States

Los booleanos deben evitar representar simultáneamente múltiples estados.

Incorrecto:

```json
{
  "value": false
}
```

cuando no se sabe si:

```text
OFF
```

o:

```text
UNKNOWN
```

La condición debe acompañarse con:

```json
{
  "value": false,
  "quality": "good"
}
```

o:

```json
{
  "value": null,
  "quality": "unavailable"
}
```

---

# 70. Numeric Validation

Los valores numéricos deben poder definir:

```text
minimum
maximum
step
precision
unit
```

Ejemplo:

```json
{
  "capability": {
    "type": "brightness",
    "minimum": 0,
    "maximum": 100,
    "step": 1,
    "unit": "%"
  }
}
```

Los límites de seguridad deben prevalecer sobre los límites de interfaz.

---

# 71. Safety Limits

Un actuador puede tener:

```json
{
  "limits": {
    "minimum": 0,
    "maximum": 100,
    "safe_minimum": 10,
    "safe_maximum": 80
  }
}
```

El sistema debe impedir comandos fuera de los límites de seguridad aunque la API los acepte sintácticamente.

---

# 72. Priority

Los mensajes y comandos pueden tener prioridad.

Valores iniciales:

```text
emergency
critical
high
normal
low
background
```

Orden:

```text
emergency
    ↓
critical
    ↓
high
    ↓
normal
    ↓
low
    ↓
background
```

---

# 73. Source

Toda operación importante debe poder identificar su origen.

Ejemplos:

```text
local
web
mobile
automation
central
zone
integration
mqtt
matter
api
device
ai
system
```

---

# 74. Request Context

Las operaciones distribuidas pueden contener:

```json
{
  "request_id": "req_01",
  "correlation_id": "corr_01",
  "source": "web",
  "user_id": "user_01"
}
```

Esto permite rastrear una operación desde:

```text
Usuario
 ↓
Web
 ↓
Central
 ↓
Zone Controller
 ↓
Node
 ↓
Actuator
```

---

# 75. Discovery

El proceso de descubrimiento debe utilizar objetos compatibles con este modelo.

Ejemplo:

```json
{
  "device_id": "device.node01",
  "capabilities": [
    "gpio",
    "wifi",
    "temperature_sensor"
  ],
  "firmware": {
    "name": "AutomationNode",
    "version": "1.0.0"
  }
}
```

La detección de hardware debe producir Resources y Capabilities que posteriormente pueden convertirse en Entities.

---

# 76. Schema Validation

Los datos recibidos desde:

* dispositivos;
* API;
* System Bus;
* integraciones;
* archivos;
* plugins;

deben validarse antes de incorporarse al modelo interno.

Flujo:

```text
Input
  ↓
Syntax validation
  ↓
Schema validation
  ↓
Semantic validation
  ↓
Permission validation
  ↓
Safety validation
  ↓
Accept
```

---

# 77. Semantic Validation

Un JSON puede ser estructuralmente válido y aun así ser inválido.

Ejemplo:

```json
{
  "unit": "°C",
  "value": 9999
}
```

Puede cumplir JSON Schema pero superar el rango físico permitido.

Por eso deben existir dos niveles:

```text
Schema validation
+
Domain validation
```

---

# 78. Schema Compatibility

Los clientes deben tolerar campos desconocidos cuando sea posible.

Regla:

> Agregar un campo opcional no debe romper clientes existentes.

Los cambios incompatibles requieren incremento de `MAJOR`.

---

# 79. Backward Compatibility

Ejemplo:

```text
1.0.0
```

puede recibir:

```text
1.1.0
```

si solamente agrega campos opcionales.

No necesariamente puede interpretar:

```text
2.0.0
```

sin negociación o actualización.

---

# 80. Forward Compatibility

Los nodos antiguos deberían ignorar campos que no necesitan cuando sea seguro hacerlo.

Ejemplo:

```json
{
  "state": {
    "value": 22.4,
    "unit": "°C",
    "confidence": 0.98
  }
}
```

Un nodo que no conozca:

```text
confidence
```

puede ignorarlo si no afecta a la seguridad.

---

# 81. Schema Negotiation

Los dispositivos pueden anunciar:

```json
{
  "supported_schemas": {
    "entity": "1.x",
    "command": "1.x",
    "event": "1.x"
  }
}
```

Esto permitirá compatibilidad entre versiones.

---

# 82. Embedded / ESP32 Considerations

El modelo conceptual es amplio, pero un ESP32 no debe cargar necesariamente todos los esquemas.

Se recomienda separar:

```text
Full Schema
```

de:

```text
Embedded Schema
```

Ejemplo:

```text
Central
→ modelo completo

ESP32-S3
→ modelo completo/reducido

ESP32-C3
→ modelo reducido

Sensor Node
→ solamente modelos necesarios
```

---

# 83. Memory Optimization

En dispositivos pequeños:

* evitar strings repetidos;
* utilizar IDs compactos internamente;
* utilizar enums;
* utilizar CBOR/MessagePack cuando sea conveniente;
* reutilizar buffers;
* evitar duplicar estados;
* limitar metadata.

La representación externa puede seguir siendo JSON.

---

# 84. Schema IDs

Cada esquema formal deberá poseer un `$id`.

Ejemplo:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.example.local/entity/v1/entity.json"
}
```

La URL definitiva del proyecto podrá cambiarse posteriormente.

---

# 85. `$defs`

Los elementos reutilizables deben declararse en `$defs`.

Ejemplo:

```json
{
  "$defs": {
    "EntityId": {
      "type": "string",
      "minLength": 1
    },
    "Timestamp": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

Esto evita duplicar reglas.

---

# 86. Ejemplo de Schema Formal

Ejemplo conceptual para una Entity:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.example.local/entity/v1/entity.json",
  "title": "Entity",
  "type": "object",
  "required": [
    "entity_id",
    "domain",
    "name"
  ],
  "properties": {
    "entity_id": {
      "type": "string"
    },
    "domain": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "device_id": {
      "type": [
        "string",
        "null"
      ]
    },
    "zone_id": {
      "type": [
        "string",
        "null"
      ]
    }
  },
  "additionalProperties": true
}
```

Este ejemplo es ilustrativo.

Los esquemas definitivos se almacenarán en:

```text
/schemas/
```

---

# 87. `additionalProperties`

La política dependerá del nivel del objeto.

Para objetos críticos se recomienda:

```text
additionalProperties: false
```

cuando el esquema esté completamente definido.

Para objetos extensibles se podrá utilizar:

```text
additionalProperties: true
```

o un esquema específico de extensiones.

La compatibilidad debe ser evaluada antes de restringir campos adicionales.

---

# 88. Vendor Extensions

Las extensiones propietarias deben utilizar:

```text
x-
```

Ejemplo:

```json
{
  "x-esp32": {
    "cpu_frequency": 240,
    "psram": true
  }
}
```

Ejemplo de integración:

```json
{
  "x-matter": {
    "endpoint": 12
  }
}
```

Estas extensiones no deben convertirse en requisitos del modelo central.

---

# 89. Hardware-Specific Data

Los detalles físicos pertenecen a Resource/Device.

Ejemplo:

```json
{
  "resource_id": "resource.node01.gpio12",
  "type": "gpio",
  "hardware": {
    "pin": 12,
    "mode": "output",
    "active_level": "high"
  }
}
```

Nunca deben aparecer como requisito de una Entity lógica:

```text
light.living.main
```

---

# 90. Transport-Specific Data

Datos como:

```text
MQTT topic
CAN ID
RS485 address
Modbus register
Matter endpoint
IP address
MAC address
```

pertenecen a adaptadores de transporte/integración.

Ejemplo:

```json
{
  "integration": {
    "type": "mqtt",
    "topic": "home/living/light/main"
  }
}
```

El Entity ID continúa siendo:

```text
light.living.main
```

---

# 91. System Bus Messages

Los mensajes del System Bus utilizarán una estructura común.

Ejemplo:

```json
{
  "message_id": "msg_01JXYZ",
  "message_type": "command",
  "timestamp": "2026-10-05T15:30:00Z",
  "source": "device.node01",
  "destination": "entity.light.living",
  "priority": "normal",
  "payload": {}
}
```

El modelo completo será definido en:

```text
SYSTEM-BUS.md
```

---

# 92. Integration Mapping

Las integraciones externas no deben crear un modelo paralelo.

Ejemplo:

```text
Internal Entity
      ↓
Integration Adapter
      ↓
Matter Device
```

o:

```text
Internal Entity
      ↓
MQTT Adapter
      ↓
MQTT Topic
```

o:

```text
Internal Entity
      ↓
Home Assistant Adapter
      ↓
HA Entity
```

---

# 93. API Mapping

La API debe exponer directamente los conceptos del Data Model.

Ejemplo:

```text
GET /api/v1/entities
GET /api/v1/entities/{entity_id}
GET /api/v1/entities/{entity_id}/state
POST /api/v1/entities/{entity_id}/commands
```

No:

```text
GET /api/gpio/12
```

salvo APIs específicamente destinadas a diagnóstico/hardware.

---

# 94. Security

Los datos pueden clasificarse.

Niveles sugeridos:

```text
public
internal
sensitive
critical
```

Ejemplo:

```json
{
  "data_classification": "critical"
}
```

Los datos críticos pueden incluir:

* alarmas;
* cerraduras;
* acceso;
* seguridad;
* configuración;
* credenciales;
* infraestructura.

Las credenciales y secretos nunca deben almacenarse en texto plano dentro de entidades.

---

# 95. Secrets

Los siguientes datos nunca deben aparecer directamente en respuestas normales:

```text
password
API key
private key
access token
refresh token
Wi-Fi password
MQTT password
certificate private key
```

Deben utilizar referencias seguras:

```json
{
  "credential_ref": "credential.mqtt.main"
}
```

---

# 96. Audit Information

Las operaciones administrativas importantes deben poder registrar:

```json
{
  "audit": {
    "user_id": "user_01",
    "source": "web",
    "timestamp": "2026-10-05T15:30:00Z",
    "action": "entity.configuration.updated"
  }
}
```

---

# 97. Data Retention

Cada tipo de información puede tener una política distinta.

Ejemplo:

```text
Realtime state
→ memoria

History
→ días/meses

Audit
→ meses/años

Diagnostics
→ período corto

Events
→ configurable
```

La política concreta pertenece a:

```text
DATABASE-STORAGE.md
```

---

# 98. Offline Operation

Los esquemas deben funcionar sin Central.

Un Node debe poder almacenar localmente:

```text
configuration
critical state
required automations
local entities
pending events
```

La ausencia de Central no debe invalidar el modelo.

---

# 99. Synchronization

Cuando un Node vuelva a conectarse:

```text
Node
 ↓
Identity
 ↓
Schema negotiation
 ↓
Configuration version
 ↓
State synchronization
 ↓
Pending events
 ↓
Health
 ↓
Operational
```

La sincronización debe respetar:

```text
local safety
>
local automation
>
zone
>
central
>
external
```

---

# 100. State Authority

Cuando existan múltiples fuentes, debe definirse quién tiene autoridad.

Ejemplo:

```text
Local safety
    ↓
Local automation
    ↓
Zone controller
    ↓
Central
    ↓
Integration
```

Una integración externa nunca debe sobrescribir silenciosamente una condición de seguridad local.

---

# 101. Conflict Resolution

Cuando existan órdenes simultáneas:

```text
User
Automation
Central
Zone
Safety
Integration
```

debe existir una política de prioridad.

Ejemplo:

```text
EMERGENCY
    >
SAFETY
    >
LOCAL AUTOMATION
    >
ZONE
    >
CENTRAL
    >
USER
    >
INTEGRATION
```

La política final deberá definirse en `SYSTEM-BUS.md` y en la arquitectura de automatizaciones.

---

# 102. Example — Complete Entity

```json
{
  "entity_id": "light.living.main",
  "schema": "entity",
  "schema_version": "1.0.0",

  "device_id": "device.living.controller",
  "zone_id": "zone.living",

  "domain": "light",
  "device_class": "light",
  "name": "Luz principal",

  "capabilities": [
    "on_off",
    "brightness"
  ],

  "state": {
    "on": true,
    "brightness": 75,
    "timestamp": "2026-10-05T15:32:10Z",
    "quality": "good",
    "source": "device.living.controller"
  },

  "availability": "available",

  "visibility": {
    "web": true,
    "mobile": true,
    "matter": true,
    "mqtt": true,
    "third_party_api": true
  },

  "tags": [
    "lighting",
    "living"
  ],

  "status": "active",

  "created_at": "2026-10-01T12:00:00Z",
  "updated_at": "2026-10-05T15:32:10Z"
}
```

---

# 103. Example — Complete Sensor

```json
{
  "entity_id": "sensor.living.temperature",
  "schema": "entity",
  "schema_version": "1.0.0",

  "device_id": "device.living.controller",
  "zone_id": "zone.living",

  "domain": "sensor",
  "device_class": "temperature",
  "name": "Temperatura Living",

  "capabilities": [
    "temperature"
  ],

  "state": {
    "value": 23.4,
    "unit": "°C",
    "timestamp": "2026-10-05T15:32:10Z",
    "quality": "good",
    "source": "sensor"
  },

  "availability": "available",

  "status": "active"
}
```

---

# 104. Example — Complete Command

```json
{
  "command_id": "cmd_01JXYZ",
  "request_id": "req_01JXYZ",
  "correlation_id": "corr_01JXYZ",

  "entity_id": "light.living.main",

  "action": "turn_on",

  "parameters": {
    "brightness": 80
  },

  "source": "web",
  "priority": "normal",

  "created_at": "2026-10-05T15:40:00Z"
}
```

---

# 105. Example — Complete Event

```json
{
  "event_id": "evt_01JXYZ",
  "schema": "event",
  "schema_version": "1.0.0",

  "type": "entity.state_changed",

  "entity_id": "binary_sensor.front_door",

  "timestamp": "2026-10-05T15:42:00Z",

  "source": "device.security",

  "correlation_id": "corr_01JXYZ",

  "data": {
    "previous": false,
    "current": true
  }
}
```

---

# 106. Example — Complete Automation

```json
{
  "automation_id": "automation.hall.night_light",
  "schema": "automation",
  "schema_version": "1.0.0",

  "name": "Luz nocturna del pasillo",
  "enabled": true,

  "trigger": {
    "type": "state_change",
    "entity_id": "binary_sensor.hall.motion",
    "to": true
  },

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
      "action": "turn_on",
      "parameters": {
        "brightness": 15
      }
    }
  ]
}
```

---

# 107. Recommended Schema Directory

La implementación deberá evolucionar hacia:

```text
schemas/
│
├── common/
│   ├── id.json
│   ├── timestamp.json
│   ├── metadata.json
│   ├── error.json
│   ├── source.json
│   ├── availability.json
│   └── pagination.json
│
├── site/
│   └── site.json
│
├── zone/
│   └── zone.json
│
├── group/
│   └── group.json
│
├── device/
│   ├── device.json
│   ├── firmware.json
│   └── diagnostics.json
│
├── resource/
│   └── resource.json
│
├── capability/
│   └── capability.json
│
├── entity/
│   ├── entity.json
│   ├── sensor.json
│   ├── binary-sensor.json
│   ├── light.json
│   ├── switch.json
│   ├── fan.json
│   ├── cover.json
│   ├── climate.json
│   └── camera.json
│
├── state/
│   └── state.json
│
├── command/
│   ├── command.json
│   └── result.json
│
├── event/
│   └── event.json
│
├── function/
│   └── function.json
│
├── scene/
│   └── scene.json
│
├── automation/
│   ├── automation.json
│   ├── trigger.json
│   ├── condition.json
│   └── action.json
│
├── system/
│   ├── mode.json
│   ├── health.json
│   └── status.json
│
└── integration/
    ├── integration.json
    └── mapping.json
```

---

# 108. Validation Pipeline

Todos los datos externos deben atravesar:

```text
             INPUT
               │
               ▼
       ┌─────────────────┐
       │ JSON / Encoding │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Schema Validation│
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Semantic Check  │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Permission Check│
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Safety Limits   │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Model Accepted  │
       └─────────────────┘
```

---

# 109. What Must Never Happen

El diseño debe impedir que el sistema termine dependiendo de estructuras como:

```json
{
  "gpio": 12,
  "value": true
}
```

como representación principal de una función.

Tampoco:

```json
{
  "mqtt_topic": "home/light/1",
  "value": true
}
```

ni:

```json
{
  "modbus_register": 40001,
  "value": 120
}
```

Estas estructuras pueden existir en capas específicas, pero no son el modelo lógico.

La representación correcta es:

```json
{
  "entity_id": "light.living.main",
  "command": "turn_on"
}
```

y cada capa se encarga de traducirlo.

---

# 110. Modelo definitivo

La relación fundamental queda establecida como:

```text
┌─────────────────────────────┐
│          HARDWARE           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│          RESOURCE           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│         CAPABILITY          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│           ENTITY            │
└──────────────┬──────────────┘
               ↓
        ┌──────┴──────┐
        ↓             ↓
      STATE         EVENT
        ↓             ↓
     COMMAND      AUTOMATION
        │             │
        └──────┬──────┘
               ↓
       FUNCTION / SCENE
               ↓
          INTEGRATION
               ↓
      External Ecosystem
```

---

# 111. Regla de oro

> **El hardware proporciona recursos. Los recursos proporcionan capacidades. Las capacidades forman entidades. Las entidades tienen estados, reciben comandos y generan eventos. Las funciones, escenas y automatizaciones utilizan esas entidades. Las integraciones exponen el mismo modelo hacia sistemas externos.**

---

# 112. Próximos documentos

Este documento habilita formalmente los siguientes:

```text
DATA-MODEL.md
       │
       ▼
DATA-SCHEMAS.md
       │
       ├───────────────┐
       ▼               ▼
SYSTEM-BUS.md      DATABASE-STORAGE.md
       │
       ▼
API-SPECIFICATION.md
       │
       ▼
DISCOVERY-PROVISIONING.md
       │
       ▼
EVENT-MODEL.md
       │
       ▼
CONFIGURATION-MODEL.md
       │
       ▼
AUTOMATION-ENGINE.md
       │
       ▼
TESTING-VALIDATION.md
```

---

# 113. Estado del documento

```text
Estado: Diseño aprobado como base conceptual

Definido:
- modelo de IDs
- versionado
- timestamps
- Site
- Zone
- Group
- Device
- Resource
- Capability
- Entity
- State
- Command
- Event
- Function
- Scene
- Automation
- Trigger
- Condition
- Action
- System Mode
- Error
- Availability
- Configuration
- History
- Diagnostics
- Virtual entities
- External entities
- AI entities
- Validation
- extensibilidad
- compatibilidad
- seguridad conceptual
- sincronización
- estado deseado/real

Pendiente de implementación:
- JSON Schemas definitivos
- validadores
- catálogo de domains
- catálogo de capabilities
- catálogo de device classes
- catálogo de unidades
- políticas definitivas de versionado
- serialización binaria
- persistencia
- System Bus
- API
```

---

# 114. Principio arquitectónico final

La plataforma debe ser capaz de evolucionar de:

```text
ESP32 + Relay
```

a:

```text
ESP32 + sensores
```

a:

```text
ESP32-S3 + cámara + IA
```

a:

```text
múltiples zonas
```

a:

```text
Central + Nodes + Integraciones
```

sin modificar el significado fundamental de los datos.

La arquitectura debe permitir:

```text
Hardware diferente
       ↓
Mismo Data Model
       ↓
Misma API
       ↓
Mismo System Bus
       ↓
Mismas automatizaciones
       ↓
Mismas integraciones
```

> **El modelo de datos es el contrato que permite que toda la plataforma evolucione sin romper su arquitectura.**
