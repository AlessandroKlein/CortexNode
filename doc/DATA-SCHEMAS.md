# DATA-SCHEMAS.md

# Data Schemas — Esquemas de Datos de la Plataforma

> **Tipo:** Arquitectura / Convención
> **Estado:** Planificación
> **Versión:** 1.0.0
> **Fecha:** 2026-10-06

---

# 1. Propósito

Este documento define los **esquemas de datos canónicos** utilizados por la plataforma de automatización distribuida.

Los esquemas definidos aquí establecen una representación común para:

* dispositivos;
* recursos;
* capacidades;
* entidades;
* estados;
* comandos;
* eventos;
* escenas;
* automatizaciones;
* zonas;
* grupos;
* funciones;
* modos;
* configuraciones;
* diagnósticos;
* errores;
* mensajes del System Bus;
* sincronización entre nodos;
* API;
* integraciones externas.

El objetivo es que diferentes componentes puedan intercambiar información sin necesidad de conocer cómo fue implementada físicamente.

---

# 2. Relación con otros documentos

La arquitectura de datos queda organizada de la siguiente manera:

```text
ARCHITECTURE.md
       │
       ▼
DEVICE-MODEL.md
       │
       ▼
DATA-MODEL.md
       │
       ▼
DATA-SCHEMAS.md
       │
 ┌─────┴──────────────┐
 ▼                    ▼
SYSTEM-BUS.md     API-SPECIFICATION.md
 │                    │
 ▼                    ▼
Nodes              External Clients
```

Cada documento tiene una responsabilidad diferente.

| Documento                             | Responsabilidad                   |
| ------------------------------------- | --------------------------------- |
| `ARCHITECTURE.md`                     | Arquitectura general              |
| `DEVICE-MODEL.md`                     | Modelo conceptual de dispositivos |
| `DATA-MODEL.md`                       | Entidades y relaciones            |
| `DATA-SCHEMAS.md`                     | Representación concreta de datos  |
| `SYSTEM-BUS.md`                       | Comunicación interna              |
| `API-SPECIFICATION.md`                | API externa                       |
| `API-AUTHENTICATION-AUTHORIZATION.md` | Seguridad de API                  |

---

# 3. Principio fundamental

La plataforma tendrá un **modelo de datos canónico interno**.

Las diferentes interfaces deberán adaptarse a este modelo.

```text
                 CANONICAL DATA MODEL
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
      System Bus        REST          MQTT
          │              │              │
          ▼              ▼              ▼
       Nodes          Clients       Integrations
```

No se deberá crear un modelo diferente para cada transporte o integración.

---

# 4. Formato principal

El formato principal de intercambio será:

```text
JSON
```

por sus características:

* legibilidad;
* facilidad de depuración;
* compatibilidad;
* disponibilidad de librerías;
* facilidad de integración;
* uso directo desde navegadores;
* compatibilidad con REST y WebSocket.

---

# 5. Formatos alternativos

La arquitectura podrá soportar posteriormente:

```text
CBOR
MessagePack
Protobuf
```

Estos formatos podrán utilizarse cuando:

* el ancho de banda sea limitado;
* el mensaje sea muy frecuente;
* el consumo de RAM sea crítico;
* se requiera mayor eficiencia;
* el transporte tenga restricciones de tamaño.

La semántica deberá mantenerse idéntica.

```text
JSON
  ↕
Canonical Model
  ↕
CBOR / MessagePack / Protobuf
```

---

# 6. JSON como representación de referencia

Cuando existan múltiples formatos, JSON será la representación de referencia para documentación.

Ejemplo:

```json
{
  "entity_id": "sensor.living.temperature",
  "state": {
    "value": 24.5,
    "unit": "°C"
  }
}
```

La representación binaria deberá producir exactamente la misma información semántica.

---

# 7. Convenciones generales

Los nombres de propiedades utilizarán:

```text
snake_case
```

Ejemplo:

```json
{
  "device_id": "node_001",
  "hardware_profile": "NODE_ETH_WROOM",
  "firmware_version": "1.0.0"
}
```

No:

```json
{
  "deviceId": "node_001"
}
```

salvo que una integración externa requiera otra convención.

---

# 8. Identificadores

La plataforma utiliza diferentes identificadores.

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
message_id
request_id
correlation_id
```

Cada uno tiene una finalidad diferente.

---

# 9. UUID / ULID

Para identificadores internos se recomienda utilizar:

```text
UUID
```

o:

```text
ULID
```

ULID es especialmente interesante para registros históricos porque mantiene orden temporal.

Ejemplo:

```text
01J...
```

No obstante, los identificadores legibles por el usuario pueden mantenerse separados.

---

# 10. Internal ID vs Entity ID

Se debe distinguir:

```text
internal_id
```

de:

```text
entity_id
```

Ejemplo:

```text
internal_id:
01JABC123...

entity_id:
light.living_room
```

El `internal_id` identifica inequívocamente el registro.

El `entity_id` representa su identidad lógica funcional.

---

# 11. Entity ID

La convención recomendada es:

```text
domain.object
```

Ejemplos:

```text
light.living_room
switch.pool_pump
sensor.living_temperature
sensor.living_humidity
binary_sensor.front_door
cover.garage
fan.bedroom
climate.house
alarm.house
```

---

# 12. Reglas de Entity ID

Un `entity_id` deberá ser:

* único dentro del Site;
* estable;
* independiente del hardware;
* independiente del transporte;
* reutilizable al reemplazar hardware;
* adecuado para API e integraciones.

No deberá depender de:

```text
GPIO
IP
MAC
CAN ID
Modbus address
I2C address
SPI CS
```

---

# 13. Site Schema

El `Site` representa una instalación.

Ejemplo:

```json
{
  "site_id": "site_home_001",
  "name": "Casa",
  "timezone": "America/Argentina/Buenos_Aires",
  "locale": "es-AR",
  "units": "metric",
  "status": "active"
}
```

Campos recomendados:

| Campo      | Tipo   | Requerido |
| ---------- | ------ | --------- |
| `site_id`  | string | Sí        |
| `name`     | string | Sí        |
| `timezone` | string | Sí        |
| `locale`   | string | No        |
| `units`    | string | No        |
| `status`   | enum   | Sí        |
| `metadata` | object | No        |

---

# 14. Zone Schema

Una zona representa un espacio o sector físico/lógico.

```json
{
  "zone_id": "zone_living",
  "site_id": "site_home_001",
  "name": "Living",
  "parent_zone_id": null,
  "status": "active"
}
```

Puede existir jerarquía:

```text
Casa
 ├── Planta Baja
 │    ├── Living
 │    └── Cocina
 └── Planta Alta
      └── Dormitorio
```

---

# 15. Group Schema

Un grupo es una agrupación lógica.

No necesariamente representa ubicación física.

```json
{
  "group_id": "group_downstairs_lights",
  "name": "Luces planta baja",
  "entity_ids": [
    "light.living_room",
    "light.kitchen",
    "light.hall"
  ]
}
```

Un grupo puede atravesar varias zonas.

---

# 16. Device Schema

Un Device representa hardware o un dispositivo lógico.

```json
{
  "device_id": "node_001",
  "name": "Nodo Living",
  "manufacturer": "Platform",
  "model": "NODE_ETH_S3",
  "hardware_profile": "NODE_ETH_S3_W5500_REV_A",
  "firmware_version": "1.0.0",
  "status": "online"
}
```

---

# 17. Device Schema completo

```json
{
  "device_id": "01JABC...",
  "name": "Nodo Living",
  "description": "Controlador principal del living",

  "manufacturer": "Platform",
  "model": "NODE_ETH_S3",

  "hardware_profile": "NODE_ETH_S3_W5500_REV_A",
  "firmware_version": "1.0.0",
  "hardware_revision": "A",

  "zone_id": "zone_living",

  "status": "online",

  "capabilities": [
    "ethernet",
    "wifi",
    "gpio",
    "pwm",
    "i2c"
  ],

  "created_at": "2026-10-06T12:00:00Z",
  "updated_at": "2026-10-06T12:00:00Z"
}
```

---

# 18. Device Status

Estados posibles:

```text
provisioning
online
offline
degraded
maintenance
disabled
unavailable
```

---

# 19. Resource Schema

Un Resource representa un recurso físico disponible.

Ejemplos:

```text
GPIO
PWM
ADC
I2C
SPI
UART
RS485
CAN
Ethernet
Wi-Fi
Storage
Camera
Display
Touch
```

Ejemplo:

```json
{
  "resource_id": "gpio_23",
  "device_id": "node_001",
  "type": "gpio",
  "name": "GPIO23",
  "direction": "output",
  "status": "available"
}
```

---

# 20. Resource físico vs lógico

El Resource puede tener propiedades físicas.

Ejemplo:

```json
{
  "resource_id": "gpio_23",
  "type": "gpio",
  "physical": {
    "pin": 23
  }
}
```

Esta información puede existir internamente, pero no debe ser necesaria para consumidores normales de la API.

---

# 21. Capability Schema

Una Capability describe lo que un recurso o entidad puede hacer.

Ejemplo:

```json
{
  "capability_id": "brightness",
  "type": "brightness",
  "min": 0,
  "max": 100,
  "unit": "%"
}
```

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

# 22. Capability con límites

```json
{
  "type": "temperature_setpoint",
  "unit": "°C",
  "min": 10,
  "max": 35,
  "step": 0.1
}
```

Estos límites deben ser validados antes de ejecutar comandos.

---

# 23. Entity Schema

La Entity es el objeto lógico utilizado por aplicaciones y automatizaciones.

Ejemplo:

```json
{
  "entity_id": "light.living_room",
  "name": "Luz Living",
  "domain": "light",
  "device_id": "node_001",
  "zone_id": "zone_living",

  "capabilities": [
    "on_off",
    "brightness"
  ],

  "status": "available"
}
```

---

# 24. Entity completa

```json
{
  "entity_id": "light.living_room",

  "internal_id": "01JABC...",

  "name": "Luz Living",
  "description": "Iluminación principal",

  "domain": "light",

  "device_id": "node_001",
  "zone_id": "zone_living",

  "capabilities": [
    {
      "type": "on_off"
    },
    {
      "type": "brightness",
      "min": 0,
      "max": 100,
      "unit": "%"
    }
  ],

  "device_class": "light",

  "status": "available",

  "enabled": true,

  "visible": true,

  "created_at": "2026-10-06T12:00:00Z",
  "updated_at": "2026-10-06T12:00:00Z"
}
```

---

# 25. Entity Domains

Los dominios son extensibles.

Ejemplos iniciales:

```text
light
switch
fan
cover
sensor
binary_sensor
button
climate
lock
alarm
camera
media_player
vacuum
water_valve
energy_meter
scene
number
select
text
```

No debe existir una lista cerrada que impida agregar nuevos dominios.

---

# 26. Sensor Entity

Ejemplo:

```json
{
  "entity_id": "sensor.living.temperature",
  "domain": "sensor",
  "device_class": "temperature",
  "unit": "°C",
  "state_type": "numeric"
}
```

---

# 27. Binary Sensor

Ejemplo:

```json
{
  "entity_id": "binary_sensor.front_door",
  "domain": "binary_sensor",
  "device_class": "door",
  "state_type": "boolean"
}
```

---

# 28. Actuator Entity

Ejemplo:

```json
{
  "entity_id": "switch.pool_pump",
  "domain": "switch",
  "capabilities": [
    "on_off"
  ]
}
```

---

# 29. State Schema

El estado representa la condición actual conocida.

Estructura recomendada:

```json
{
  "entity_id": "sensor.living.temperature",

  "state": {
    "value": 24.5,
    "unit": "°C"
  },

  "timestamp": "2026-10-06T12:00:00Z",

  "quality": "good",

  "source": "node_001"
}
```

---

# 30. State Metadata

El estado puede incluir:

```text
timestamp
source
quality
confidence
sequence
version
```

Ejemplo:

```json
{
  "state": {
    "value": 24.5,
    "unit": "°C"
  },
  "timestamp": "2026-10-06T12:00:00Z",
  "quality": "good",
  "confidence": 0.98,
  "source": "node_001",
  "sequence": 1054
}
```

---

# 31. Quality

Valores estándar:

```text
good
uncertain
stale
invalid
unavailable
```

---

# 32. Unknown vs Unavailable

Se deben diferenciar.

### Unknown

El sistema todavía no conoce el valor.

```text
value = null
quality = unknown
```

### Unavailable

El recurso normalmente existe, pero actualmente no está disponible.

```text
value = null
quality = unavailable
```

No deben tratarse como equivalentes.

---

# 33. Null

`null` deberá utilizarse únicamente cuando no exista un valor válido.

Ejemplo:

```json
{
  "value": null,
  "quality": "unavailable"
}
```

No se deberá utilizar:

```text
0
-1
9999
```

como sustitutos genéricos de valores desconocidos.

---

# 34. Desired State

Para actuadores se recomienda:

```json
{
  "desired": {
    "power": true
  },

  "actual": {
    "power": false
  }
}
```

Esto permite detectar una diferencia entre intención y realidad.

---

# 35. Command Schema

Un comando representa una acción.

```json
{
  "command_id": "01JCOMMAND...",
  "entity_id": "light.living_room",
  "command": "turn_on",
  "parameters": {},
  "request_id": "01JREQUEST..."
}
```

---

# 36. Command completo

```json
{
  "command_id": "01JCOMMAND...",

  "entity_id": "light.living_room",

  "command": "set_brightness",

  "parameters": {
    "brightness": 75
  },

  "source": {
    "type": "user",
    "id": "01JUSER..."
  },

  "request_id": "01JREQUEST...",
  "correlation_id": "01JCORRELATION...",

  "created_at": "2026-10-06T12:00:00Z",

  "idempotency_key": "abc123"
}
```

---

# 37. Command Status

Estados:

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

Ejemplo:

```json
{
  "command_id": "01JCOMMAND...",
  "status": "executed",
  "completed_at": "2026-10-06T12:00:01Z"
}
```

---

# 38. Event Schema

Un evento representa algo que ocurrió.

```json
{
  "event_id": "01JEVENT...",
  "event_type": "motion_detected",
  "entity_id": "binary_sensor.hall_motion",

  "timestamp": "2026-10-06T12:00:00Z",

  "source": "node_001",

  "data": {}
}
```

---

# 39. Event con datos

```json
{
  "event_id": "01JEVENT...",

  "event_type": "threshold_exceeded",

  "entity_id": "sensor.greenhouse.temperature",

  "timestamp": "2026-10-06T12:00:00Z",

  "data": {
    "value": 31.5,
    "threshold": 30.0,
    "direction": "above"
  }
}
```

---

# 40. Event vs State

Un evento:

```text
door_opened
```

indica:

> La puerta se abrió.

El estado:

```text
door = open
```

indica:

> La puerta está abierta.

Un evento puede producir un cambio de estado, pero ambos conceptos deben mantenerse separados.

---

# 41. Scene Schema

Una escena representa un conjunto de estados deseados.

```json
{
  "scene_id": "scene_night",
  "name": "Noche",

  "actions": [
    {
      "entity_id": "light.living_room",
      "command": "turn_off"
    },
    {
      "entity_id": "light.hall",
      "command": "set_brightness",
      "parameters": {
        "brightness": 15
      }
    }
  ]
}
```

---

# 42. Scene Activation

La activación de una escena produce comandos.

```text
Scene
 ↓
Validate
 ↓
Generate Commands
 ↓
System Bus
 ↓
Entities
```

Una escena no debe modificar directamente GPIO.

---

# 43. Automation Schema

Una automatización se representa como:

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
  "automation_id": "automation_hall_light",

  "name": "Luz del pasillo",

  "enabled": true,

  "trigger": {
    "type": "event",
    "event_type": "motion_detected",
    "entity_id": "binary_sensor.hall_motion"
  },

  "conditions": [
    {
      "type": "time",
      "after": "22:00"
    }
  ],

  "actions": [
    {
      "entity_id": "light.hall",
      "command": "set_brightness",
      "parameters": {
        "brightness": 20
      }
    }
  ]
}
```

---

# 44. Trigger Schema

Tipos de trigger:

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

# 45. Condition Schema

Ejemplos:

```json
{
  "type": "state",
  "entity_id": "binary_sensor.front_door",
  "operator": "eq",
  "value": true
}
```

Otro ejemplo:

```json
{
  "type": "numeric",
  "entity_id": "sensor.house.power",
  "operator": "gt",
  "value": 5000,
  "unit": "W"
}
```

---

# 46. Action Schema

```json
{
  "entity_id": "light.living_room",
  "command": "turn_on",
  "parameters": {}
}
```

Las acciones pueden ser:

```text
entity command
scene activation
notification
event emission
delay
condition
script
automation trigger
```

---

# 47. Function Schema

Una Function representa una función lógica de mayor nivel.

Ejemplos:

```text
lighting
heating
cooling
irrigation
security
ventilation
energy_management
water_management
climate_control
```

Ejemplo:

```json
{
  "function_id": "function_lighting_house",
  "name": "Iluminación",
  "type": "lighting",
  "entity_ids": [
    "light.living_room",
    "light.kitchen",
    "light.hall"
  ]
}
```

---

# 48. Mode Schema

Los modos permiten modificar el comportamiento general.

```json
{
  "mode_id": "sleep",
  "name": "Dormir",
  "active": true
}
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

---

# 49. Mode State

```json
{
  "system_mode": "sleep",
  "changed_at": "2026-10-06T23:00:00Z",
  "changed_by": "automation.sleep_schedule"
}
```

---

# 50. Configuration Schema

La configuración debe diferenciarse del estado.

```text
CONFIGURATION
    ≠
STATE
```

Ejemplo:

```json
{
  "config_version": 15,

  "configuration": {
    "temperature_min": 20,
    "temperature_max": 25
  }
}
```

---

# 51. Desired vs Applied Configuration

La configuración distribuida puede tener:

```json
{
  "desired_config_version": 15,
  "applied_config_version": 14
}
```

Esto indica:

```text
Central:
version 15

Node:
version 14
```

El sistema deberá reconciliar posteriormente.

---

# 52. Configuration Source

Puede indicar:

```text
user
central
node
factory
automation
integration
```

Ejemplo:

```json
{
  "source": {
    "type": "central",
    "id": "central_001"
  }
}
```

---

# 53. Resource Configuration

Los recursos físicos pueden tener configuración.

Ejemplo:

```json
{
  "resource_id": "gpio_23",

  "type": "gpio",

  "configuration": {
    "direction": "output",
    "inverted": false,
    "safe_state": false
  }
}
```

Esta información pertenece a la configuración de hardware y no debe exponerse como abstracción principal a integraciones normales.

---

# 54. Sensor Configuration

Ejemplo:

```json
{
  "resource_id": "i2c_sensor_01",

  "type": "sensor",

  "configuration": {
    "sensor_type": "AHT20",
    "address": "0x38",
    "sampling_interval_ms": 5000
  }
}
```

---

# 55. Calibration Schema

Los sensores pueden tener calibración.

```json
{
  "calibration": {
    "offset": 0.3,
    "scale": 1.0,
    "unit": "°C"
  }
}
```

Puede ampliarse:

```json
{
  "calibration": {
    "method": "linear",
    "points": [
      {
        "raw": 100,
        "reference": 20
      },
      {
        "raw": 500,
        "reference": 40
      }
    ]
  }
}
```

---

# 56. Telemetry Schema

```json
{
  "telemetry_id": "01JTEL...",
  "entity_id": "sensor.house.power",

  "timestamp": "2026-10-06T12:00:00Z",

  "value": 1250.4,
  "unit": "W",

  "quality": "good",

  "source": "node_energy_01"
}
```

---

# 57. Multi-value Telemetry

Algunos sensores generan múltiples valores.

Ejemplo:

```json
{
  "entity_id": "sensor.weather",

  "values": {
    "temperature": 24.5,
    "humidity": 61.2,
    "pressure": 1012.4
  },

  "units": {
    "temperature": "°C",
    "humidity": "%",
    "pressure": "hPa"
  }
}
```

Sin embargo, cuando sea necesario utilizar cada valor independientemente en automatizaciones, se recomienda crear entidades separadas:

```text
sensor.weather.temperature
sensor.weather.humidity
sensor.weather.pressure
```

---

# 58. Units

Las unidades deberán ser explícitas cuando exista riesgo de ambigüedad.

Unidades canónicas recomendadas:

| Magnitud    | Unidad |
| ----------- | ------ |
| Temperatura | °C     |
| Presión     | Pa     |
| Humedad     | %      |
| Tensión     | V      |
| Corriente   | A      |
| Potencia    | W      |
| Energía     | Wh     |
| Flujo       | L/min  |
| Distancia   | m      |
| Velocidad   | m/s    |
| Ángulo      | °      |

Las integraciones podrán convertir unidades para presentar información al usuario.

---

# 59. Canonical Units

El sistema interno debe utilizar unidades canónicas.

Por ejemplo:

```text
pressure = Pa
```

Aunque una UI muestre:

```text
hPa
```

o:

```text
mbar
```

---

# 60. Enum

Los valores enumerados deben utilizar strings estables.

Ejemplo:

```json
{
  "status": "online"
}
```

No:

```json
{
  "status": 1
}
```

Los códigos numéricos dificultan interoperabilidad y debugging.

---

# 61. Extensibilidad de Enums

Los consumidores deberán ignorar valores desconocidos cuando el contexto lo permita.

Ejemplo:

```text
new_status
```

no debería provocar necesariamente un crash.

---

# 62. Metadata

Los objetos podrán contener:

```json
{
  "metadata": {
    "manufacturer_serial": "ABC123",
    "installation_date": "2026-10-01",
    "notes": "Nodo instalado en tablero principal"
  }
}
```

La metadata no deberá ser necesaria para el funcionamiento básico.

---

# 63. Tags

Los objetos pueden tener etiquetas:

```json
{
  "tags": [
    "critical",
    "outdoor",
    "energy"
  ]
}
```

Esto permite búsquedas y agrupaciones dinámicas.

---

# 64. Friendly Name

La identidad técnica no debe confundirse con el nombre mostrado.

```json
{
  "entity_id": "light.living_room",
  "name": "Luz del Living"
}
```

El `entity_id` permanece estable aunque cambie:

```text
name = "Luz principal"
```

---

# 65. Localization

Los nombres podrán localizarse.

Ejemplo conceptual:

```json
{
  "name": {
    "default": "Living Light",
    "es": "Luz del Living",
    "it": "Luce del soggiorno"
  }
}
```

No obstante, para reducir complejidad en dispositivos pequeños, la localización completa puede quedar a cargo de Central/UI.

---

# 66. Visibility

Una entidad podrá tener:

```json
{
  "visible": true,
  "exposed": true
}
```

`visible` y `exposed` no significan necesariamente lo mismo.

### Visible

Aparece en la interfaz local.

### Exposed

Puede ser utilizada por una integración externa.

---

# 67. Entity Lifecycle

Estados:

```text
provisioning
active
disabled
unavailable
deprecated
removed
```

---

# 68. Soft Delete

Una entidad eliminada lógicamente no debería desaparecer inmediatamente de todo el historial.

Puede pasar a:

```text
removed
```

manteniendo sus registros históricos según las políticas de retención.

---

# 69. History Record

Un registro histórico puede tener:

```json
{
  "entity_id": "sensor.house.temperature",
  "timestamp": "2026-10-06T12:00:00Z",
  "value": 24.5,
  "unit": "°C",
  "quality": "good",
  "source": "node_001"
}
```

---

# 70. History Metadata

Puede incluir:

```text
sequence
source
quality
confidence
aggregation
sample_interval
```

---

# 71. Aggregated Data

La plataforma podrá almacenar:

```text
raw
average
minimum
maximum
sum
count
```

Ejemplo:

```json
{
  "period": "1h",
  "average": 24.3,
  "minimum": 22.8,
  "maximum": 26.1,
  "count": 720
}
```

---

# 72. Error Schema

Todos los servicios deberán utilizar un formato uniforme.

```json
{
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity does not exist",
    "details": {},
    "request_id": "01J..."
  }
}
```

---

# 73. Error Codes

Categorías:

```text
AUTH_
DEVICE_
ENTITY_
COMMAND_
CONFIG_
TRANSPORT_
BUS_
VALIDATION_
SYSTEM_
```

Ejemplos:

```text
ENTITY_NOT_FOUND
ENTITY_UNAVAILABLE
COMMAND_INVALID
COMMAND_REJECTED
DEVICE_OFFLINE
TRANSPORT_TIMEOUT
BUS_QUEUE_FULL
CONFIG_INVALID
AUTH_FORBIDDEN
```

---

# 74. Error Details

Los detalles deben ser estructurados.

```json
{
  "error": {
    "code": "VALUE_OUT_OF_RANGE",
    "message": "Brightness is outside the allowed range",
    "details": {
      "field": "brightness",
      "min": 0,
      "max": 100,
      "received": 150
    }
  }
}
```

---

# 75. Request Schema

Las solicitudes deberán incluir:

```text
request_id
timestamp
source
```

cuando sea necesario.

Ejemplo:

```json
{
  "request_id": "01JREQUEST...",
  "operation": "get_state",
  "entity_id": "light.living_room"
}
```

---

# 76. Response Schema

Respuesta estándar:

```json
{
  "success": true,

  "request_id": "01JREQUEST...",

  "data": {}
}
```

En caso de error:

```json
{
  "success": false,

  "request_id": "01JREQUEST...",

  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity not found"
  }
}
```

---

# 77. Bus Message Schema

El System Bus utiliza una envoltura adicional.

```json
{
  "message_id": "01JMESSAGE...",
  "message_type": "command",
  "schema_version": "1.0",

  "timestamp": "2026-10-06T12:00:00Z",

  "source": {
    "device_id": "node_001"
  },

  "destination": {
    "entity_id": "light.living_room"
  },

  "request_id": "01JREQUEST...",
  "correlation_id": "01JCORRELATION...",

  "priority": "normal",
  "qos": "reliable",
  "ttl": 10,

  "payload": {}
}
```

---

# 78. Envelope vs Payload

Se debe diferenciar:

```text
Envelope
```

de:

```text
Payload
```

### Envelope

Información necesaria para transportar y enrutar.

### Payload

Información específica de la operación.

Ejemplo:

```text
Envelope
├── message_id
├── source
├── destination
├── priority
└── qos

Payload
├── command
└── parameters
```

---

# 79. Command Payload

```json
{
  "command": "set_brightness",
  "parameters": {
    "brightness": 75
  }
}
```

---

# 80. Event Payload

```json
{
  "event_type": "motion_detected",
  "data": {
    "confidence": 0.98
  }
}
```

---

# 81. State Payload

```json
{
  "state": {
    "power": true,
    "brightness": 75
  }
}
```

---

# 82. Discovery Payload

```json
{
  "operation": "announce",

  "device": {
    "device_id": "node_001",
    "hardware_profile": "NODE_BASIC_C3_REV_A",
    "firmware_version": "1.0.0"
  }
}
```

---

# 83. Synchronization Schema

La sincronización puede utilizar:

```json
{
  "sync_id": "01JSYNC...",
  "device_id": "node_001",

  "configuration_version": 15,
  "state_version": 104,

  "changes": []
}
```

---

# 84. Configuration Patch

Para evitar transferir toda la configuración:

```json
{
  "operation": "replace",
  "path": "/configuration/temperature_max",
  "value": 26
}
```

También:

```text
add
remove
replace
move
copy
```

si posteriormente se adopta un mecanismo compatible con JSON Patch.

---

# 85. Schema Version

Todos los objetos intercambiables importantes deberán poder identificar su versión.

Ejemplo:

```json
{
  "schema_version": "1.0"
}
```

La versión debe ser independiente de:

```text
firmware_version
API_version
System Bus version
hardware_revision
```

---

# 86. Compatibility Rules

Cambios compatibles:

```text
agregar campos opcionales
agregar enum values
agregar capacidades
```

Cambios potencialmente incompatibles:

```text
eliminar campos
cambiar significado
cambiar tipos
renombrar campos obligatorios
cambiar unidades
```

Los cambios incompatibles requieren una nueva versión de esquema.

---

# 87. Schema Negotiation

Dos dispositivos pueden intercambiar:

```json
{
  "supported_schema_versions": [
    "1.0",
    "1.1"
  ]
}
```

El emisor deberá seleccionar una versión compatible.

---

# 88. Validation

Todo dato recibido deberá validarse.

Validaciones mínimas:

```text
required fields
type
range
enum
format
length
relationships
authorization
```

---

# 89. Numeric Validation

Ejemplo:

```json
{
  "brightness": 75
}
```

Debe validarse:

```text
type = number
min = 0
max = 100
```

---

# 90. String Validation

Los strings deberán poder limitar:

```text
length
charset
format
```

Ejemplo:

```text
entity_id
```

deberá seguir una convención conocida.

---

# 91. Security Classification

Algunos datos podrán clasificarse:

```text
public
internal
sensitive
restricted
secret
```

Ejemplos:

```text
temperature → internal
device diagnostics → internal
API token → secret
credentials → secret
```

Los secretos nunca deben viajar o almacenarse en texto plano sin protección adecuada.

---

# 92. Credentials

Las credenciales deberán mantenerse separadas del modelo normal de entidades.

Nunca:

```json
{
  "device": {
    "password": "..."
  }
}
```

en respuestas normales.

Deberán utilizarse estructuras específicas y controles de acceso.

---

# 93. Secret References

Cuando sea necesario referenciar un secreto:

```json
{
  "credential_ref": "credential_001"
}
```

y no:

```json
{
  "password": "mypassword"
}
```

---

# 94. Provenance Schema

Los datos podrán incluir origen:

```json
{
  "provenance": {
    "device_id": "node_001",
    "module_id": "aht20_01",
    "transport": "i2c"
  }
}
```

La información física puede mantenerse fuera de las respuestas normales.

---

# 95. Virtual Entity

No todas las entidades tienen hardware directo.

Ejemplo:

```json
{
  "entity_id": "sensor.house.average_temperature",
  "domain": "sensor",
  "type": "virtual",
  "source_entities": [
    "sensor.living.temperature",
    "sensor.kitchen.temperature"
  ]
}
```

---

# 96. Calculated Entity

Ejemplo:

```json
{
  "entity_id": "sensor.house.total_power",
  "domain": "sensor",
  "type": "calculated",
  "formula": "sum(power.*)"
}
```

La fórmula podrá restringirse por seguridad.

---

# 97. Aggregated Entity

Ejemplo:

```json
{
  "entity_id": "sensor.zone.temperature_average",
  "domain": "sensor",
  "type": "aggregated",
  "source_entities": [
    "sensor.room1.temperature",
    "sensor.room2.temperature"
  ],
  "aggregation": "average"
}
```

---

# 98. External Entity

Una entidad externa puede provenir de:

```text
Matter
MQTT
Home Assistant
Weather Service
Cloud API
Modbus Gateway
```

Ejemplo:

```json
{
  "entity_id": "weather.external.temperature",
  "domain": "sensor",
  "type": "external",
  "source": "weather_provider"
}
```

---

# 99. Camera / AI Entity

Ejemplo:

```json
{
  "entity_id": "camera.front",
  "domain": "camera",
  "capabilities": [
    "stream",
    "snapshot",
    "object_detection"
  ]
}
```

Una detección:

```json
{
  "event_type": "object_detected",
  "entity_id": "camera.front",
  "data": {
    "object": "person",
    "confidence": 0.96
  }
}
```

---

# 100. Energy Entity

Ejemplo:

```json
{
  "entity_id": "sensor.house.power",
  "domain": "sensor",
  "device_class": "power",
  "unit": "W"
}
```

También:

```text
sensor.house.energy
sensor.house.voltage
sensor.house.current
```

---

# 101. Water Entity

Ejemplo:

```json
{
  "entity_id": "sensor.irrigation.flow",
  "domain": "sensor",
  "device_class": "flow",
  "unit": "L/min"
}
```

---

# 102. Environmental Entity

Ejemplos:

```text
sensor.temperature
sensor.humidity
sensor.pressure
sensor.co2
sensor.pm25
sensor.illuminance
sensor.wind_speed
sensor.wind_direction
sensor.rain
```

---

# 103. Industrial Entity

Ejemplos:

```text
sensor.motor.temperature
sensor.motor.rpm
sensor.pump.pressure
sensor.line.flow
valve.main
alarm.machine
```

---

# 104. Marine Entity

Ejemplos:

```text
sensor.engine.rpm
sensor.battery.voltage
sensor.battery.current
sensor.tank.level
binary_sensor.bilge
alarm.engine
```

---

# 105. State Version

Los estados pueden utilizar un contador:

```json
{
  "state_version": 1054
}
```

Esto ayuda a detectar estados antiguos.

---

# 106. Sequence

Los mensajes de alta frecuencia pueden utilizar:

```json
{
  "sequence": 1054
}
```

Esto permite detectar:

```text
missing
duplicate
out_of_order
```

---

# 107. Ordering

No todos los mensajes requieren orden global.

El orden deberá definirse por:

```text
entity
device
stream
correlation_id
```

Esto evita exigir una secuencia global innecesaria en toda la instalación.

---

# 108. Clock Independence

El sistema no deberá depender exclusivamente de sincronización horaria para determinar orden.

Se recomienda combinar:

```text
timestamp
sequence
boot_id
```

Especialmente durante:

```text
NTP unavailable
device reboot
network outage
```

---

# 109. Boot ID

Cada arranque puede generar:

```json
{
  "boot_id": "01JBOOT..."
}
```

Esto permite distinguir:

```text
sequence 1
```

de diferentes reinicios.

---

# 110. Message Identity

La identidad completa de un mensaje puede interpretarse como:

```text
boot_id
+
message_id
```

o simplemente mediante un ID global suficientemente robusto.

---

# 111. API Compatibility

La API deberá reutilizar estos mismos esquemas.

Por ejemplo:

```text
GET /entities/light.living_room
```

deberá devolver el mismo modelo conceptual utilizado internamente.

No deberá existir:

```text
API Entity Model
```

completamente diferente del:

```text
Internal Entity Model
```

---

# 112. System Bus Compatibility

De igual manera:

```text
System Bus
```

deberá transportar estos objetos o representaciones equivalentes.

Ejemplo:

```text
Command Schema
      ↓
Bus Envelope
      ↓
Transport
```

---

# 113. Integration Mapping

Las integraciones externas deberán actuar como adaptadores.

```text
Canonical Entity
       │
       ├── Matter mapping
       ├── MQTT mapping
       ├── Home Assistant mapping
       ├── REST mapping
       └── Other mapping
```

Nunca se deberá modificar el modelo interno para adaptarlo a una integración concreta.

---

# 114. Example — Light

```json
{
  "entity_id": "light.living_room",

  "domain": "light",

  "capabilities": [
    "on_off",
    "brightness"
  ],

  "state": {
    "power": true,
    "brightness": 75
  }
}
```

---

# 115. Example — Temperature

```json
{
  "entity_id": "sensor.living.temperature",

  "domain": "sensor",

  "device_class": "temperature",

  "state": {
    "value": 24.5,
    "unit": "°C"
  },

  "quality": "good"
}
```

---

# 116. Example — Door

```json
{
  "entity_id": "binary_sensor.front_door",

  "domain": "binary_sensor",

  "device_class": "door",

  "state": {
    "value": "open"
  }
}
```

---

# 117. Example — Motor

```json
{
  "entity_id": "motor.pool_pump",

  "domain": "motor",

  "capabilities": [
    "on_off",
    "speed"
  ],

  "state": {
    "power": true,
    "speed": 1800,
    "speed_unit": "rpm"
  }
}
```

---

# 118. Example — Climate

```json
{
  "entity_id": "climate.house",

  "domain": "climate",

  "capabilities": [
    "temperature",
    "setpoint",
    "mode"
  ],

  "state": {
    "temperature": 24.2,
    "setpoint": 22,
    "mode": "cool"
  }
}
```

---

# 119. Example — Water Valve

```json
{
  "entity_id": "water_valve.irrigation",

  "domain": "water_valve",

  "capabilities": [
    "open_close",
    "position"
  ],

  "state": {
    "position": 100
  }
}
```

---

# 120. Example — Alarm

```json
{
  "entity_id": "alarm.house",

  "domain": "alarm",

  "state": {
    "status": "armed_away"
  }
}
```

---

# 121. System Snapshot

Central podrá solicitar un snapshot:

```json
{
  "snapshot_id": "01JSNAPSHOT...",
  "site_id": "site_home_001",

  "timestamp": "2026-10-06T12:00:00Z",

  "devices": [],
  "entities": [],
  "zones": [],
  "groups": [],
  "modes": []
}
```

---

# 122. Snapshot Use Cases

Los snapshots pueden utilizarse para:

* sincronización;
* backup;
* diagnóstico;
* restauración;
* onboarding;
* migración;
* comparación de configuración.

---

# 123. Delta Synchronization

Para instalaciones grandes no se recomienda transferir todo el sistema constantemente.

Se podrá utilizar:

```text
snapshot
+
delta changes
```

Ejemplo:

```text
Snapshot version 100
        ↓
Changes 101
Changes 102
Changes 103
```

---

# 124. Change Record

```json
{
  "change_id": "01JCHANGE...",
  "version": 103,

  "operation": "update",

  "entity_id": "light.living_room",

  "path": "/state/brightness",

  "old_value": 50,
  "new_value": 75
}
```

---

# 125. Schema Registry

La plataforma deberá mantener un registro lógico de esquemas.

Conceptualmente:

```text
schemas/
├── site.schema.json
├── zone.schema.json
├── device.schema.json
├── resource.schema.json
├── capability.schema.json
├── entity.schema.json
├── state.schema.json
├── command.schema.json
├── event.schema.json
├── scene.schema.json
├── automation.schema.json
├── bus-message.schema.json
└── error.schema.json
```

---

# 126. JSON Schema

Se recomienda utilizar **JSON Schema** como mecanismo de validación formal.

Por ejemplo:

```text
schemas/
└── v1/
    ├── site.schema.json
    ├── zone.schema.json
    ├── device.schema.json
    ├── entity.schema.json
    ├── command.schema.json
    ├── event.schema.json
    └── bus-message.schema.json
```

Esto permite validar automáticamente:

```text
API
System Bus
Central
Nodes
Tests
CI/CD
```

---

# 127. Schema IDs

Cada esquema puede tener un identificador:

```text
https://platform.local/schema/v1/entity
```

En dispositivos pequeños no será necesario transportar siempre la URL completa; puede utilizarse:

```text
schema_id = entity
schema_version = 1.0
```

---

# 128. Schema Validation en ESP32

No todos los nodos deberán cargar todos los esquemas completos en RAM.

Se recomienda:

```text
Central:
validación completa

Zone Controller:
validación intermedia

Node:
validación específica y ligera
```

Esto permite conservar recursos.

---

# 129. Perfil de capacidad

Cada dispositivo puede anunciar qué esquemas soporta.

```json
{
  "schema_support": {
    "entity": "1.0",
    "command": "1.0",
    "event": "1.0"
  }
}
```

---

# 130. Schema Evolution

Los esquemas deberán evolucionar sin romper instalaciones antiguas.

Estrategia:

```text
1.0
 ↓
1.1
 ↓
1.2
 ↓
2.0
```

Las versiones menores deben mantener compatibilidad siempre que sea posible.

---

# 131. Migration

Cuando una configuración antigua deba transformarse:

```text
Old Schema
   ↓
Migration
   ↓
New Schema
```

Las migraciones deberán ser explícitas.

---

# 132. Backward Compatibility

Un nodo con firmware antiguo podrá seguir funcionando si recibe una estructura compatible.

Ejemplo:

```text
Central 1.2
Node 1.0
```

La Central deberá poder adaptar mensajes cuando sea necesario.

---

# 133. Forward Compatibility

Un nodo antiguo deberá ignorar campos opcionales desconocidos cuando sea seguro hacerlo.

Ejemplo:

```json
{
  "value": 24.5,
  "unit": "°C",
  "confidence": 0.98
}
```

Un nodo que no entiende `confidence` puede seguir procesando:

```text
value
unit
```

si el campo es opcional.

---

# 134. Required vs Optional

Los esquemas deberán diferenciar:

```text
required
optional
deprecated
```

Ejemplo:

```text
entity_id → required
domain → required
name → optional
description → optional
metadata → optional
```

---

# 135. Safety-Critical Fields

Algunos campos deben ser obligatorios en contextos críticos.

Por ejemplo:

```text
safe_state
timeout
priority
authorization
```

cuando se trate de determinados actuadores.

---

# 136. Schema de seguridad para actuadores

Ejemplo:

```json
{
  "safety": {
    "safe_state": "off",
    "communication_loss_action": "off",
    "max_command_duration_ms": 5000
  }
}
```

Los valores concretos dependerán del dispositivo.

---

# 137. Resource Mapping

La relación entre hardware y entidad puede representarse:

```json
{
  "entity_id": "light.living_room",

  "resource_mapping": {
    "resource_id": "gpio_23",
    "channel": 0
  }
}
```

Esta información debe permanecer principalmente en configuración interna.

---

# 138. Expander Mapping

Ejemplo:

```json
{
  "entity_id": "switch.pump",

  "resource_mapping": {
    "resource_type": "mcp23017",
    "resource_id": "mcp23017_01",
    "channel": 4
  }
}
```

La entidad continúa siendo:

```text
switch.pump
```

aunque se cambie el expansor.

---

# 139. Multi-resource Entity

Una entidad puede depender de varios recursos.

Ejemplo:

```text
Motor
 ├── GPIO enable
 ├── PWM speed
 ├── ADC current
 └── feedback input
```

Representación:

```json
{
  "entity_id": "motor.fan",

  "resources": [
    "gpio_enable",
    "pwm_speed",
    "adc_current",
    "feedback"
  ]
}
```

---

# 140. Capability Composition

Una entidad puede combinar varias capacidades.

Ejemplo:

```text
Light
├── on_off
├── brightness
├── color
└── temperature
```

Esto evita crear una entidad diferente para cada característica física.

---

# 141. Device vs Entity

Un dispositivo puede contener muchas entidades:

```text
Node
 ├── sensor.temperature
 ├── sensor.humidity
 ├── light.main
 ├── switch.fan
 └── binary_sensor.motion
```

No se deberá confundir:

```text
device
```

con:

```text
entity
```

---

# 142. Resource vs Entity

Un recurso representa:

> qué hardware existe.

Una entidad representa:

> qué función lógica existe.

Ejemplo:

```text
GPIO23
   ↓
Resource
   ↓
switch.pump
   ↓
Entity
```

---

# 143. Capability vs Entity

Una capability representa:

> qué puede hacer algo.

La entidad representa:

> el objeto lógico que utiliza esa capacidad.

Ejemplo:

```text
light.living_room
   ├── on_off
   └── brightness
```

---

# 144. State Ownership

El estado debe tener un origen.

Ejemplo:

```json
{
  "source": {
    "device_id": "node_001",
    "authority": "device"
  }
}
```

Esto evita que Central sobrescriba arbitrariamente un estado físico real.

---

# 145. State Conflict

Si existen:

```text
desired = ON
actual = OFF
```

el sistema debe poder representar:

```text
state_conflict
```

o:

```text
pending
```

según el caso.

---

# 146. Command Authorization Context

Los comandos pueden transportar:

```json
{
  "source": {
    "type": "user",
    "id": "user_001"
  },

  "authorization": {
    "role": "admin",
    "scope": "device_control"
  }
}
```

No todos los transportes deberán transportar necesariamente todos estos campos.

---

# 147. Audit Data

Operaciones sensibles pueden registrar:

```json
{
  "audit": {
    "actor_type": "user",
    "actor_id": "user_001",
    "action": "unlock",
    "timestamp": "2026-10-06T12:00:00Z"
  }
}
```

---

# 148. Correlation

Una operación completa puede tener:

```text
request_id
command_id
message_id
event_id
correlation_id
```

Ejemplo:

```text
request_id
   │
   └── command_id
          │
          ├── message_id
          ├── ACK
          └── STATE
```

El `correlation_id` permite relacionarlos.

---

# 149. Idempotency

Los comandos externos deberán poder utilizar:

```text
idempotency_key
```

Ejemplo:

```json
{
  "command": "turn_on",
  "idempotency_key": "user-123-command-456"
}
```

Si la misma operación llega dos veces:

```text
execute once
return existing result
```

---

# 150. Retry Metadata

Los mensajes pueden incluir:

```json
{
  "retry": {
    "attempt": 2,
    "max_attempts": 3
  }
}
```

Esta información puede ser interna y no necesariamente visible para API pública.

---

# 151. Timeout Metadata

```json
{
  "timeout": {
    "deadline": "2026-10-06T12:00:05Z"
  }
}
```

o, para dispositivos sin reloj válido:

```json
{
  "timeout_ms": 5000
}
```

---

# 152. Compression

Para mensajes grandes podrá utilizarse:

```text
gzip
deflate
CBOR
MessagePack
```

según el transporte.

La compresión no deberá modificar el esquema lógico.

---

# 153. Binary Payload

En determinados casos podrá utilizarse:

```json
{
  "encoding": "binary",
  "content_type": "application/octet-stream"
}
```

Esto puede ser útil para:

* imágenes;
* firmware;
* archivos;
* grandes bloques de datos.

No debe utilizarse para mensajes normales.

---

# 154. Image / Camera Data

Una entidad de cámara normalmente no deberá transportar una imagen completa mediante el System Bus.

Preferentemente:

```text
System Bus
    ↓
camera event
    ↓
reference / URI
```

Ejemplo:

```json
{
  "event_type": "object_detected",

  "data": {
    "object": "person",
    "confidence": 0.97,
    "snapshot_ref": "storage://snapshot/01J..."
  }
}
```

---

# 155. Large Data Rule

Regla:

> **El System Bus transporta eventos, estados, comandos y referencias; no debe utilizarse como almacenamiento general de grandes archivos.**

---

# 156. Storage Reference

Los archivos podrán referenciarse mediante:

```text
storage://
http://
https://
local://
```

según la arquitectura.

La seguridad y autorización deberán validarse antes de acceder.

---

# 157. API Pagination

Cuando los esquemas se utilicen en API, las colecciones deberán soportar:

```text
limit
cursor
offset
```

Se recomienda cursor para instalaciones grandes.

Ejemplo:

```json
{
  "items": [],
  "next_cursor": "abc123"
}
```

---

# 158. Filtering

Ejemplo:

```text
entities?zone_id=zone_living
```

o:

```text
entities?domain=light
```

o:

```text
entities?device_id=node_001
```

Los filtros deben operar sobre entidades lógicas.

---

# 159. Sorting

Las colecciones pueden permitir:

```text
name
created_at
updated_at
entity_id
```

La API deberá documentar los campos soportados.

---

# 160. Schema de Collection

```json
{
  "items": [],
  "count": 10,
  "next_cursor": "abc123"
}
```

No es necesario enviar `count` total si calcularlo es costoso.

---

# 161. WebSocket Event Schema

Los eventos enviados por WebSocket deberán utilizar el mismo formato conceptual:

```json
{
  "type": "event",
  "event": {
    "event_id": "01J...",
    "event_type": "state_changed",
    "entity_id": "light.living_room"
  }
}
```

---

# 162. API vs Bus Envelope

La API pública puede ocultar parte del envelope interno.

Por ejemplo, una API puede recibir:

```json
{
  "command": "turn_on"
}
```

y generar internamente:

```text
Bus Envelope
+
Command Payload
```

Esto evita exponer detalles internos innecesarios.

---

# 163. Public vs Internal Schema

Se podrán definir:

```text
Internal Schema
Public API Schema
Integration Schema
```

Pero todos deberán derivar del mismo modelo conceptual.

---

# 164. Regla de no duplicación

No se deberán crear estructuras diferentes para representar el mismo concepto sin necesidad.

Por ejemplo:

```text
TemperatureState
WeatherTemperature
SensorTemperature
ApiTemperature
MqttTemperature
```

no deben convertirse en cinco modelos independientes.

Debe existir un concepto canónico:

```text
Temperature State
```

y adaptadores.

---

# 165. Schema Registry Structure

Estructura recomendada:

```text
schemas/
├── v1/
│   ├── common/
│   │   ├── identifier.schema.json
│   │   ├── timestamp.schema.json
│   │   ├── quality.schema.json
│   │   └── provenance.schema.json
│   │
│   ├── entities/
│   │   ├── entity.schema.json
│   │   ├── sensor.schema.json
│   │   ├── light.schema.json
│   │   └── switch.schema.json
│   │
│   ├── commands/
│   │   └── command.schema.json
│   │
│   ├── events/
│   │   └── event.schema.json
│   │
│   ├── bus/
│   │   └── message.schema.json
│   │
│   └── errors/
│       └── error.schema.json
│
└── v2/
```

---

# 166. Common Schema Components

Se recomienda reutilizar componentes:

```text
identifier
timestamp
entity_reference
device_reference
zone_reference
quality
unit
source
provenance
metadata
```

Esto evita duplicación.

---

# 167. Entity Reference

Cuando no sea necesario incluir una entidad completa:

```json
{
  "entity_id": "light.living_room"
}
```

No deberá repetirse todo el objeto Entity.

---

# 168. Device Reference

```json
{
  "device_id": "node_001"
}
```

---

# 169. Zone Reference

```json
{
  "zone_id": "zone_living"
}
```

---

# 170. Reference vs Embedded Object

Se recomienda utilizar referencias para objetos grandes o reutilizados.

Ejemplo:

```json
{
  "device_id": "node_001"
}
```

en lugar de:

```json
{
  "device": {
    "device_id": "...",
    "name": "...",
    "resources": [...]
  }
}
```

salvo que se solicite explícitamente una expansión.

---

# 171. Expand

La API podrá permitir:

```text
?include=device
```

o:

```text
?include=state
```

según el endpoint.

Esto permite equilibrar tamaño de respuesta y facilidad de uso.

---

# 172. Data Ownership

Cada dato debe tener un propietario lógico.

Ejemplo:

| Dato                   | Autoridad               |
| ---------------------- | ----------------------- |
| GPIO actual            | Node                    |
| sensor value           | Sensor Node             |
| local automation state | Node/Zone               |
| global configuration   | Central                 |
| user account           | Central                 |
| external weather       | Provider                |
| Matter state           | Matter adapter / device |

---

# 173. Source of Truth

La plataforma deberá definir claramente:

```text
Configuration Source of Truth
State Source of Truth
Identity Source of Truth
History Source of Truth
```

Esto evita conflictos.

---

# 174. Configuration Source of Truth

Generalmente:

```text
Central
```

para configuración global.

Pero un nodo deberá mantener una copia local válida para continuar operando.

---

# 175. State Source of Truth

Para hardware físico:

```text
Node / Device
```

Central mantiene una representación sincronizada.

---

# 176. Identity Source of Truth

Normalmente:

```text
Central
```

pero el nodo debe conservar su identidad local.

---

# 177. History Source of Truth

Puede variar:

```text
Node
Zone Controller
Central
External storage
```

según el nivel de almacenamiento disponible.

---

# 178. Data Retention

Cada tipo de información podrá tener una política:

```text
telemetry: 30 days
events: 90 days
critical events: 1 year
diagnostics: 7 days
```

Los valores son configurables y solo representan ejemplos.

---

# 179. Privacy

Los datos que puedan representar información sensible deberán clasificarse.

Ejemplo:

```text
occupancy
camera events
presence
access events
security events
```

La plataforma deberá permitir limitar:

```text
storage
API exposure
integration exposure
user access
```

---

# 180. Performance

En ESP32 deberán evitarse objetos excesivamente grandes.

Preferir:

```text
small messages
references
compact schemas
incremental updates
```

antes que snapshots permanentes.

---

# 181. RAM Management

Los nodos deberán poder trabajar con estructuras parciales.

Por ejemplo:

```text
Node:
Entity State only

Central:
Full Entity Model
```

Esto permite escalar a diferentes capacidades de hardware.

---

# 182. Serialization Strategy

La serialización deberá estar desacoplada:

```text
Object
 ↓
Serializer
 ├── JSON
 ├── CBOR
 ├── MessagePack
 └── Protobuf
```

La lógica del objeto no debe conocer el formato.

---

# 183. Deserialization

Los datos externos deberán pasar por:

```text
Raw Data
 ↓
Parser
 ↓
Schema Validation
 ↓
Canonical Object
 ↓
System Bus / Application
```

Nunca:

```text
Raw JSON
 ↓
GPIO
```

directamente.

---

# 184. Security Boundary

La validación deberá realizarse antes de ejecutar acciones.

```text
Receive
 ↓
Parse
 ↓
Validate
 ↓
Authenticate
 ↓
Authorize
 ↓
Validate business rules
 ↓
Execute
```

---

# 185. Business Validation

Un valor puede ser sintácticamente correcto pero lógicamente inválido.

Ejemplo:

```json
{
  "position": 50
}
```

es válido sintácticamente.

Pero si una válvula está bloqueada:

```text
business rule → reject
```

---

# 186. Safety Validation

Incluso un comando autorizado puede ser rechazado por seguridad.

Ejemplo:

```text
set_motor_speed = 5000 rpm
```

si:

```text
max_speed = 3000 rpm
```

Resultado:

```text
COMMAND_REJECTED
```

---

# 187. Data Schema Golden Rules

1. Todo objeto importante debe tener versión.
2. Los IDs deben ser estables.
3. Las unidades deben ser explícitas.
4. Los estados deben diferenciarse de los comandos.
5. Los eventos deben diferenciarse de los estados.
6. Los datos físicos no deben filtrarse innecesariamente a la capa lógica.
7. Los esquemas deben poder validarse.
8. Las extensiones deben ser compatibles.
9. Los errores deben tener formato uniforme.
10. Las integraciones deben adaptar el modelo, no modificarlo.

---

# 188. Flujo completo de datos

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
Canonical Schema
   ↓
System Bus
   ↓
Transport
   ↓
Node / Zone / Central
   ↓
API / Integration
```

---

# 189. Flujo de un sensor

```text
AHT20
 ↓
Driver
 ↓
Resource
 ↓
Capability: temperature
 ↓
Entity:
sensor.living.temperature
 ↓
State Schema
 ↓
System Bus
 ↓
Central
 ↓
REST / WebSocket / MQTT / Matter
```

---

# 190. Flujo de un actuador

```text
API
 ↓
Command Schema
 ↓
Authorization
 ↓
System Bus
 ↓
Transport
 ↓
Node
 ↓
Module
 ↓
Driver
 ↓
GPIO / PWM / Relay
 ↓
Actual State
 ↓
State Schema
 ↓
System Bus
```

---

# 191. Flujo de una automatización

```text
Event
 ↓
Event Schema
 ↓
System Bus
 ↓
Automation Engine
 ↓
Condition
 ↓
Action
 ↓
Command Schema
 ↓
System Bus
 ↓
Actuator
```

---

# 192. Flujo de sincronización

```text
Node reconnect
      ↓
Discovery
      ↓
Device Schema
      ↓
Configuration Version
      ↓
State Version
      ↓
Delta / Snapshot
      ↓
Reconciliation
      ↓
Normal operation
```

---

# 193. Estructura final de esquemas

La implementación deberá evolucionar hacia una estructura similar a:

```text
schemas/
├── v1/
│
├── common/
│   ├── identifiers
│   ├── timestamps
│   ├── units
│   ├── quality
│   ├── provenance
│   └── references
│
├── topology/
│   ├── site
│   ├── zone
│   └── group
│
├── hardware/
│   ├── device
│   ├── resource
│   └── capability
│
├── entities/
│   ├── entity
│   ├── state
│   └── entity-types
│
├── automation/
│   ├── function
│   ├── scene
│   ├── automation
│   ├── trigger
│   ├── condition
│   └── action
│
├── communication/
│   ├── command
│   ├── event
│   ├── telemetry
│   ├── discovery
│   ├── response
│   └── bus-message
│
├── configuration/
│   ├── config
│   ├── config-patch
│   └── synchronization
│
├── diagnostics/
│   ├── health
│   ├── metrics
│   └── error
│
└── integrations/
    └── ...
```

---

# 194. Principio de interoperabilidad

Todos los subsistemas deberán utilizar el mismo lenguaje conceptual:

```text
Device
Resource
Capability
Entity
State
Command
Event
Function
Scene
Automation
Zone
Group
```

Esto permite que:

```text
ESP32
Central
Web UI
Mobile App
REST API
MQTT
Matter
Home Assistant
AI
```

puedan trabajar sobre el mismo modelo.

---

# 195. Principio de evolución

Los esquemas deberán permitir que la plataforma evolucione desde:

```text
1 ESP32
```

hasta:

```text
1 Central
+
10 Zones
+
100 Nodes
```

o incluso instalaciones mayores, sin cambiar los conceptos fundamentales.

---

# 196. Principio de portabilidad

Los esquemas no deben depender de:

```text
ESP32
FreeRTOS
PlatformIO
Wi-Fi
Ethernet
CAN
RS485
```

La plataforma puede implementar posteriormente otros microcontroladores o sistemas.

El modelo de datos deberá permanecer válido.

---

# 197. Principio de abstracción

La capa superior debe poder preguntar:

```text
"¿Cuál es la temperatura del living?"
```

y no:

```text
"¿Qué valor tiene ADC1 del GPIO34 del ESP32 ubicado en node_03?"
```

De la misma manera:

```text
"Enciende la luz del living"
```

y no:

```text
"Pon GPIO23 en HIGH."
```

---

# 198. Principio final

La arquitectura de datos completa queda:

```text
┌────────────────────────────────────┐
│             HARDWARE               │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│             RESOURCE               │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│           CAPABILITY               │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│              ENTITY                │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│          CANONICAL STATE           │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│           SYSTEM BUS               │
└──────────────────┬─────────────────┘
                   ↓
       ┌───────────┼───────────┐
       ↓           ↓           ↓
      API        MQTT        Matter
       ↓           ↓           ↓
   External    External    External
   Clients     Systems     Ecosystems
```

---

# 199. Reglas de oro de DATA-SCHEMAS

> **1. El modelo canónico es único.**

> **2. Hardware y lógica están separados.**

> **3. Entity IDs son estables y no dependen del hardware.**

> **4. Commands expresan intenciones.**

> **5. States representan realidad conocida.**

> **6. Events representan hechos ocurridos.**

> **7. Resources representan hardware.**

> **8. Capabilities representan capacidades.**

> **9. Las unidades deben ser explícitas.**

> **10. Los esquemas deben estar versionados.**

> **11. Los mensajes deben poder validarse.**

> **12. Las integraciones son adaptadores.**

> **13. Los transportes no deben modificar la semántica.**

> **14. La información crítica debe tener autoridad y procedencia claras.**

> **15. El sistema debe poder evolucionar sin romper nodos existentes.**

---

# 200. Principio final de arquitectura

> **Hardware proporciona Resources.**
>
> **Resources proporcionan Capabilities.**
>
> **Capabilities forman Entities.**
>
> **Entities tienen States, reciben Commands y generan Events.**
>
> **Functions, Scenes y Automations utilizan esas Entities.**
>
> **El System Bus transporta estos mensajes.**
>
> **La API y las Integraciones exponen el mismo modelo hacia el exterior.**
>
> **El transporte físico nunca debe convertirse en la lógica de la aplicación.**

Este principio constituye la base común sobre la que deberán construirse:

```text
MODULE-DEVELOPMENT.md
DEVICE-MODEL.md
DATA-MODEL.md
SYSTEM-BUS.md
API-SPECIFICATION.md
SMART-HOME-INTEGRATION.md
```

y las futuras implementaciones de firmware.
