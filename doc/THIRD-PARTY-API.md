# Third-Party API

> **Estado:** Planificación
> **Versión:** 1.0.0
> **Tipo:** Arquitectura / Especificación
> **Proyecto:** Distributed Automation Platform
> **Última actualización:** 2026-10-05

---

## 1. Objetivo

La **Third-Party API** permite que aplicaciones, servicios, dispositivos y plataformas externas interactúen con el sistema de automatización sin necesidad de conocer su implementación interna.

Un tercero debe poder:

* descubrir dispositivos;
* consultar entidades;
* consultar capacidades;
* obtener estados;
* enviar comandos;
* publicar datos;
* recibir eventos;
* crear dispositivos virtuales;
* crear sensores virtuales;
* crear actuadores virtuales;
* registrar integraciones;
* ejecutar escenas o acciones permitidas;
* consultar información histórica autorizada;
* monitorizar el estado de la red;
* integrarse utilizando estándares conocidos.

La API debe abstraer completamente:

* GPIO;
* microcontroladores;
* ESP32;
* buses físicos;
* MQTT interno;
* CAN;
* RS485;
* I2C;
* SPI;
* Wi-Fi;
* Ethernet;
* Zigbee;
* Thread;
* Matter;
* implementación del firmware.

El tercero debe trabajar únicamente con el **modelo lógico del sistema**.

---

# 2. Principio fundamental

La API externa nunca debe exponer directamente la implementación física.

La arquitectura debe ser:

```text
┌─────────────────────────────┐
│      THIRD-PARTY APP        │
│                             │
│ Home automation             │
│ Mobile App                  │
│ ERP                         │
│ SCADA                       │
│ IoT Platform                │
│ Custom Software             │
└──────────────┬──────────────┘
               │
               │ HTTPS / WebSocket
               │
┌──────────────▼──────────────┐
│       THIRD-PARTY API       │
├─────────────────────────────┤
│ Authentication              │
│ Authorization               │
│ Rate Limiting               │
│ Validation                  │
│ API Versioning              │
│ Event Management             │
│ Entity Management            │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│       DEVICE MODEL          │
├─────────────────────────────┤
│ Devices                     │
│ Entities                    │
│ Capabilities                │
│ Resources                   │
│ Zones                       │
│ Groups                      │
│ Scenes                      │
│ Automations                 │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│     INTERNAL SYSTEM BUS     │
└──────────────┬──────────────┘
               │
       ┌───────┼────────┐
       │       │        │
      CAN    RS485    Wi-Fi
       │       │        │
      ESP32   ESP32    ESP32
```

La Third-Party API es, por tanto, una **frontera de integración**, no el sistema de control interno.

---

# 3. Objetivos de diseño

La API debe cumplir:

### 3.1 Estabilidad

Una aplicación externa no debería romperse porque:

* se cambió un GPIO;
* se reemplazó un ESP32;
* se cambió Wi-Fi por Ethernet;
* un sensor pasó de I2C a RS485;
* un nodo fue reemplazado;
* una entidad cambió de dispositivo físico.

---

### 3.2 Independencia del hardware

Ejemplo:

```text
Luz Living
```

puede estar físicamente conectada a:

```text
ESP32 #12
GPIO 23
Relay
```

y posteriormente pasar a:

```text
ESP32 #48
GPIO 5
Dimmer
```

La aplicación externa debe continuar utilizando:

```text
entity_id = light.living
```

sin modificaciones.

---

### 3.3 Seguridad

Un tercero nunca debe recibir automáticamente acceso completo al sistema.

Debe existir:

* autenticación;
* autorización;
* scopes;
* permisos por entidad;
* permisos por zona;
* permisos de lectura/escritura;
* expiración de credenciales;
* revocación;
* auditoría.

---

### 3.4 Compatibilidad

La API debe poder utilizar:

* aplicaciones web;
* aplicaciones móviles;
* servidores;
* PLC;
* sistemas SCADA;
* sistemas IoT;
* scripts;
* servicios cloud;
* otros controladores;
* dispositivos embebidos.

---

# 4. API-first

La arquitectura debe diseñarse **API-first**.

El sistema no debe crear primero una interfaz web y después intentar convertirla en API.

El modelo lógico debe ser accesible desde:

```text
Web UI
Mobile App
Third-Party API
Internal API
Automation Engine
Integrations
```

Todas estas interfaces deben utilizar el mismo modelo conceptual.

---

# 5. Tipos de clientes

Se contemplan diferentes tipos de clientes.

## 5.1 Aplicación local

Ejemplo:

```text
http://central.local
```

Una aplicación instalada dentro de la red.

---

## 5.2 Aplicación móvil

Ejemplo:

```text
Android
iOS
```

---

## 5.3 Aplicación web

Aplicación externa ejecutándose en un navegador.

---

## 5.4 Servidor externo

Ejemplo:

```text
ERP
SCADA
Backend IoT
Sistema de gestión energética
```

---

## 5.5 Dispositivo externo

Otro microcontrolador o dispositivo industrial.

Ejemplo:

```text
PLC
ESP32 externo
Gateway Modbus
Gateway CAN
```

---

## 5.6 Integración cloud

Servicios externos que necesitan acceso controlado.

Ejemplo:

```text
Cloud Analytics
Cloud Monitoring
Weather Service
Energy Management
```

---

# 6. Arquitectura de API

La API principal se basará preferentemente en:

```text
HTTPS
REST
JSON
WebSocket
```

Opcionalmente:

```text
SSE
MQTT
Webhooks
mDNS
DNS-SD
```

La API REST será el mecanismo principal de:

```text
discovery
configuration
state
commands
entities
devices
users
permissions
history
```

WebSocket será utilizado principalmente para:

```text
real-time state
events
notifications
subscriptions
```

---

# 7. Base URL

La API deberá utilizar una estructura versionada.

Ejemplo:

```text
https://central.local/api/v1/
```

o:

```text
https://192.168.1.50/api/v1/
```

Nunca debe existir una API externa sin versión.

---

# 8. Versionado

El versionado inicial será:

```text
/api/v1/
```

Las versiones serán independientes.

Ejemplo:

```text
/api/v1/
```

posteriormente:

```text
/api/v2/
```

Una versión nueva no debe modificar silenciosamente el comportamiento de la anterior.

---

# 9. Identificadores

Todos los elementos expuestos por la API deben disponer de identificadores estables.

Ejemplo:

```json
{
  "id": "ent_01JABC123",
  "entity_id": "light.living"
}
```

Se recomienda separar:

```text
internal_id
entity_id
friendly_name
```

Ejemplo:

```json
{
  "id": "ent_01JABC123",
  "entity_id": "light.living",
  "name": "Luz Living"
}
```

El `id` interno nunca debe depender del hardware.

---

# 10. Modelo de entidades

La API utilizará el mismo modelo lógico definido por el proyecto.

Jerarquía:

```text
System
 ├── Zones
 │    ├── Devices
 │    │    └── Entities
 │    └── Groups
 │
 ├── Scenes
 ├── Automations
 └── Virtual Entities
```

---

# 11. Entity

Una Entity representa una función lógica.

Ejemplo:

```json
{
  "id": "ent_001",
  "entity_id": "light.living",
  "type": "light",
  "name": "Luz Living",
  "zone": "living",
  "state": {
    "on": true,
    "brightness": 75
  }
}
```

La entidad puede estar respaldada por:

```text
ESP32
Relay
Dimmer
CAN actuator
Modbus actuator
Virtual device
Cloud service
```

El cliente no necesita saberlo.

---

# 12. Capabilities

Las capacidades indican qué operaciones soporta una entidad.

Ejemplo:

```json
{
  "entity_id": "light.living",
  "type": "light",
  "capabilities": [
    "on_off",
    "brightness"
  ]
}
```

Una entidad puede tener:

```text
on_off
brightness
color
temperature
position
speed
volume
target_temperature
energy_measurement
```

etc.

---

# 13. Discovery

Los terceros deben poder descubrir automáticamente los elementos disponibles.

Endpoint:

```http
GET /api/v1/discovery
```

Respuesta conceptual:

```json
{
  "system": {
    "id": "house_001",
    "name": "Casa"
  },
  "api": {
    "version": "v1"
  },
  "zones": 5,
  "devices": 23,
  "entities": 87
}
```

---

# 14. Discovery de dispositivos

```http
GET /api/v1/devices
```

Ejemplo:

```json
{
  "devices": [
    {
      "id": "dev_001",
      "name": "Controlador Living",
      "manufacturer": "Platform",
      "model": "ESP32-Node",
      "zone": "living",
      "status": "online"
    }
  ]
}
```

---

# 15. Discovery de entidades

```http
GET /api/v1/entities
```

Filtros:

```text
?zone=living
?type=light
?device=dev_001
?group=lights
```

Ejemplo:

```http
GET /api/v1/entities?type=temperature
```

---

# 16. Obtener estado

Endpoint:

```http
GET /api/v1/entities/{entity_id}/state
```

Ejemplo:

```http
GET /api/v1/entities/light.living/state
```

Respuesta:

```json
{
  "entity_id": "light.living",
  "state": "on",
  "attributes": {
    "brightness": 75
  },
  "timestamp": "2026-10-05T20:15:00Z"
}
```

---

# 17. Modificar estado

Los clientes autorizados podrán enviar comandos.

Ejemplo:

```http
POST /api/v1/entities/light.living/command
```

Body:

```json
{
  "command": "turn_on"
}
```

Para brillo:

```json
{
  "command": "set_brightness",
  "value": 75
}
```

---

# 18. Modelo de comandos

Los comandos deben ser abstractos.

No:

```json
{
  "gpio": 23,
  "value": 1
}
```

Sí:

```json
{
  "command": "turn_on"
}
```

Esto protege la arquitectura de cambios de hardware.

---

# 19. Comandos con parámetros

Ejemplo:

```json
{
  "command": "set_brightness",
  "parameters": {
    "value": 75
  }
}
```

Otro ejemplo:

```json
{
  "command": "set_position",
  "parameters": {
    "position": 50
  }
}
```

---

# 20. Respuesta de comandos

La API debe diferenciar entre:

```text
command accepted
command executed
command failed
```

Ejemplo:

```json
{
  "request_id": "req_001",
  "status": "accepted"
}
```

Posteriormente:

```json
{
  "request_id": "req_001",
  "status": "completed"
}
```

Esto permite trabajar correctamente con sistemas distribuidos.

---

# 21. Eventos

Los cambios de estado deberán poder notificarse en tiempo real.

Ejemplo:

```text
light.living
    ↓
state_changed
    ↓
WebSocket
    ↓
Third-party application
```

Evento:

```json
{
  "event": "state_changed",
  "entity_id": "light.living",
  "state": {
    "on": true,
    "brightness": 80
  },
  "timestamp": "2026-10-05T20:15:30Z"
}
```

---

# 22. WebSocket

Endpoint:

```text
wss://central.local/api/v1/events
```

Al conectarse, el cliente debe autenticarse.

Posteriormente puede suscribirse.

Ejemplo:

```json
{
  "action": "subscribe",
  "events": [
    "state_changed"
  ]
}
```

---

# 23. Suscripciones filtradas

No se debe obligar al cliente a recibir todos los eventos.

Ejemplo:

```json
{
  "action": "subscribe",
  "events": [
    {
      "type": "state_changed",
      "entities": [
        "light.living",
        "sensor.temperature.living"
      ]
    }
  ]
}
```

También:

```text
zone=living
type=temperature
group=security
```

---

# 24. Eventos del sistema

Se contemplan eventos como:

```text
device_online
device_offline
entity_created
entity_removed
state_changed
alarm_triggered
automation_triggered
scene_activated
configuration_changed
central_online
central_offline
```

---

# 25. Publicación de datos externos

Un tercero puede publicar información dentro del sistema.

Ejemplo:

```text
Weather Service
      ↓
Third-Party API
      ↓
Virtual Sensor
      ↓
System
```

Por ejemplo:

```text
Temperatura exterior
Humedad
Pronóstico
Radiación solar
Precio de energía
```

---

# 26. Dispositivos virtuales

Un tercero podrá crear dispositivos virtuales.

Ejemplo:

```http
POST /api/v1/virtual-devices
```

```json
{
  "name": "Servicio Meteorológico",
  "manufacturer": "External Service",
  "model": "Weather API"
}
```

---

# 27. Entidades virtuales

Ejemplo:

```http
POST /api/v1/virtual-devices/dev_weather/entities
```

```json
{
  "entity_id": "sensor.external_temperature",
  "type": "sensor",
  "device_class": "temperature",
  "unit": "°C",
  "name": "Temperatura Exterior"
}
```

Después:

```http
POST /api/v1/entities/sensor.external_temperature/state
```

```json
{
  "value": 24.6
}
```

---

# 28. Virtual actuator

También pueden existir actuadores virtuales.

Ejemplo:

```text
Servicio externo
       ↓
Virtual Switch
       ↓
Automation
       ↓
ESP32
       ↓
Actuator
```

Esto permite que una aplicación externa participe en automatizaciones.

---

# 29. Crear elementos temporales

Debe existir la posibilidad de crear entidades con duración limitada.

Ejemplo:

```text
sensor.weather.alert
```

válido durante:

```text
30 minutos
```

Después puede eliminarse automáticamente.

Esto es útil para:

* servicios externos;
* alarmas;
* integraciones;
* eventos temporales;
* predicciones;
* sistemas externos.

---

# 30. Zones

Los terceros podrán consultar zonas.

```http
GET /api/v1/zones
```

Ejemplo:

```json
{
  "zones": [
    {
      "id": "living",
      "name": "Living"
    },
    {
      "id": "bedroom",
      "name": "Dormitorio"
    }
  ]
}
```

Una entidad puede pertenecer a una zona.

---

# 31. Groups

También podrán consultarse grupos.

Ejemplo:

```text
group.all_lights
group.security
group.garden
group.energy
```

Endpoint:

```http
GET /api/v1/groups
```

---

# 32. Scenes

Los terceros autorizados podrán ejecutar escenas.

```http
POST /api/v1/scenes/night/activate
```

Ejemplo:

```json
{
  "transition": 2
}
```

No deberían poder modificar escenas salvo que tengan permisos específicos.

---

# 33. Automations

Las automatizaciones pueden consultarse:

```http
GET /api/v1/automations
```

Ejecutarse:

```http
POST /api/v1/automations/{id}/trigger
```

Modificar su configuración será una operación privilegiada.

---

# 34. API de lectura

Scopes recomendados:

```text
entities:read
devices:read
zones:read
groups:read
scenes:read
automations:read
history:read
events:read
```

---

# 35. API de escritura

Scopes:

```text
entities:write
devices:write
virtual_devices:create
virtual_entities:create
scenes:execute
automations:trigger
```

---

# 36. API administrativa

Operaciones de alto privilegio:

```text
users:manage
permissions:manage
devices:manage
configuration:manage
integrations:manage
system:manage
```

Estas nunca deben formar parte de un token normal.

---

# 37. Autenticación

Se recomienda soportar inicialmente:

```text
API Keys
Bearer Tokens
```

y posteriormente:

```text
OAuth 2.0
OpenID Connect
```

La implementación debe permitir evolucionar sin cambiar el modelo de permisos.

---

# 38. API Keys

Una aplicación puede recibir:

```text
API Key
```

Ejemplo:

```http
Authorization: Bearer <TOKEN>
```

La clave debe estar asociada a:

```text
application
owner
scopes
zones
expiration
status
```

---

# 39. Scopes

Ejemplo:

```text
weather-service
```

podría recibir:

```text
entities:read
virtual_entities:create
virtual_entities:write
```

pero no:

```text
devices:manage
users:manage
system:manage
```

---

# 40. Restricción por zona

Los permisos también pueden restringirse geográficamente.

Ejemplo:

```json
{
  "scopes": [
    "entities:read",
    "entities:write"
  ],
  "zones": [
    "garden"
  ]
}
```

El tercero puede controlar:

```text
garden
```

pero no:

```text
bedroom
living
security
```

---

# 41. Principio de mínimo privilegio

Cada aplicación debe recibir únicamente los permisos necesarios.

Ejemplo:

### Aplicación meteorológica

```text
entities:read
virtual_entities:create
virtual_entities:write
```

### Aplicación de iluminación

```text
entities:read
entities:write
```

### Aplicación administrativa

```text
system:manage
```

---

# 42. Rate limiting

La API debe proteger al sistema contra exceso de solicitudes.

Ejemplo conceptual:

```text
100 requests/minute
```

por cliente.

Los límites deberán ser configurables.

Para WebSocket:

```text
max_connections
max_subscriptions
max_events_per_second
```

---

# 43. Idempotencia

Las operaciones críticas deben soportar identificadores de solicitud.

Ejemplo:

```http
Idempotency-Key: 123456
```

Esto evita ejecutar dos veces un comando debido a:

* reintentos;
* pérdida de conexión;
* timeout;
* reconexión.

---

# 44. Concurrencia

Dos aplicaciones pueden intentar modificar la misma entidad.

Ejemplo:

```text
App A → light.living = ON
App B → light.living = OFF
```

La API debe registrar:

```text
request_id
client_id
timestamp
priority
result
```

La política de resolución deberá definirse posteriormente.

---

# 45. Prioridades

Los comandos externos no deben poder ignorar las prioridades internas.

Jerarquía:

```text
SAFETY
   ↓
SECURITY
   ↓
LOCAL AUTOMATION
   ↓
ZONE AUTOMATION
   ↓
CENTRAL AUTOMATION
   ↓
THIRD-PARTY COMMAND
```

Una aplicación externa no debe poder:

```text
desactivar una alarma crítica
```

simplemente enviando:

```text
OFF
```

si la política de seguridad lo impide.

---

# 46. Estado versus comando

La API debe diferenciar:

```text
desired state
actual state
```

Ejemplo:

```json
{
  "desired": {
    "on": true
  },
  "actual": {
    "on": false
  }
}
```

Esto es especialmente importante en sistemas distribuidos.

---

# 47. Offline

Una aplicación externa debe poder detectar:

```text
Central unavailable
Device unavailable
Entity unavailable
```

Ejemplo:

```json
{
  "state": "unavailable",
  "reason": "device_offline"
}
```

No debe interpretarse automáticamente como:

```text
OFF
```

---

# 48. Timestamps

Todos los estados y eventos deberán incluir timestamp.

Preferentemente:

```text
ISO 8601 UTC
```

Ejemplo:

```text
2026-10-05T20:15:30Z
```

Opcionalmente:

```text
unix_timestamp
monotonic_timestamp
```

para sistemas donde sea necesario.

---

# 49. Calidad del dato

Los sensores deberían poder proporcionar información adicional.

Ejemplo:

```json
{
  "value": 24.6,
  "unit": "°C",
  "quality": "good"
}
```

Estados posibles:

```text
good
uncertain
stale
invalid
unavailable
```

---

# 50. Datos históricos

Los terceros podrán consultar datos históricos cuando tengan permiso.

```http
GET /api/v1/history
```

Ejemplo:

```text
/api/v1/history?
entity=sensor.temperature.living
&from=2026-10-01T00:00:00Z
&to=2026-10-05T00:00:00Z
```

---

# 51. Agregaciones

La API podrá soportar:

```text
raw
average
minimum
maximum
sum
count
delta
```

Ejemplo:

```http
GET /api/v1/history?
entity=sensor.energy.house
&aggregation=hourly
```

---

# 52. Webhooks

Como alternativa a mantener una conexión WebSocket, los terceros podrán registrar webhooks.

Ejemplo:

```http
POST /api/v1/webhooks
```

```json
{
  "url": "https://example.com/events",
  "events": [
    "state_changed",
    "alarm_triggered"
  ]
}
```

El sistema enviará:

```text
POST https://example.com/events
```

---

# 53. Seguridad de Webhooks

Los webhooks deberán utilizar firma criptográfica.

Ejemplo conceptual:

```text
X-Platform-Signature
```

El receptor debe poder verificar que el evento proviene realmente del sistema.

---

# 54. MQTT

MQTT podrá utilizarse como método adicional de integración.

Sin embargo:

> MQTT no será la arquitectura interna principal del sistema.

El sistema podrá traducir:

```text
Internal System Bus
        ↓
MQTT Adapter
        ↓
Third-party application
```

---

# 55. MQTT Discovery

Para facilitar integración con plataformas externas, podrá existir un mecanismo de discovery basado en MQTT.

Ejemplo conceptual:

```text
platform/discovery/light/living
```

La información deberá derivarse del Device Model.

Nunca se debe mantener manualmente una segunda definición de los dispositivos.

---

# 56. REST + MQTT

Una aplicación podrá utilizar:

```text
REST
```

para:

```text
discovery
configuration
commands
history
```

y:

```text
MQTT
```

para:

```text
telemetry
events
state updates
```

---

# 57. WebSocket + REST

Una aplicación web podrá utilizar:

```text
REST
```

para obtener el estado inicial:

```text
GET /entities
```

y después:

```text
WebSocket
```

para recibir cambios.

Esto evita tener que consultar continuamente el servidor.

---

# 58. Patrón recomendado

```text
1. Authenticate
2. Discovery
3. GET initial state
4. Open WebSocket
5. Subscribe
6. Receive events
7. Send commands
8. Reconnect if necessary
9. Resynchronize state
```

---

# 59. Reconexión

Los clientes deben poder reconectarse automáticamente.

Después de reconectar:

```text
re-authenticate
      ↓
resubscribe
      ↓
request current state
      ↓
continue events
```

Nunca debe asumirse que los eventos perdidos no existen.

---

# 60. Event ID

Cada evento debe tener identificador.

Ejemplo:

```json
{
  "event_id": "evt_001234",
  "sequence": 98231
}
```

Esto permite detectar eventos perdidos.

---

# 61. Sincronización

Un cliente puede indicar:

```text
last_sequence = 98200
```

y solicitar:

```text
events since 98200
```

Si el sistema conserva esos eventos.

Si no:

```text
resync_required
```

y el cliente debe obtener nuevamente el estado completo.

---

# 62. Crear aplicaciones de terceros

Debe existir un mecanismo para registrar aplicaciones.

Ejemplo:

```http
POST /api/v1/applications
```

```json
{
  "name": "Energy Manager",
  "description": "Gestión energética externa"
}
```

Respuesta:

```json
{
  "application_id": "app_001",
  "status": "pending_authorization"
}
```

---

# 63. Consentimiento del usuario

Una aplicación no debe recibir acceso automáticamente.

El usuario debe aprobar:

```text
Energy Manager wants access to:

☑ Read energy
☑ Read power
☑ Control loads

☐ Manage users
☐ Modify security
☐ Modify system configuration
```

---

# 64. Revocación

El usuario debe poder revocar una aplicación.

```text
Settings
 → Integrations
 → Applications
 → Energy Manager
 → Revoke
```

La revocación debe invalidar inmediatamente sus credenciales.

---

# 65. Auditoría

Toda acción externa importante deberá registrarse.

Ejemplo:

```json
{
  "timestamp": "2026-10-05T20:30:00Z",
  "application": "Energy Manager",
  "user": "alessandro",
  "entity": "light.garage",
  "action": "turn_off",
  "result": "success"
}
```

---

# 66. Errores

La API debe utilizar respuestas HTTP estándar.

Ejemplo:

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
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
503 Service Unavailable
```

---

# 67. Formato de error

Ejemplo:

```json
{
  "error": {
    "code": "ENTITY_UNAVAILABLE",
    "message": "Entity is currently unavailable",
    "request_id": "req_001"
  }
}
```

Nunca se debe devolver únicamente:

```text
Error
```

---

# 68. Request ID

Cada solicitud debe poder rastrearse.

Ejemplo:

```text
X-Request-ID: req_01JABC123
```

Esto facilita:

* debugging;
* soporte;
* auditoría;
* logs;
* diagnóstico de instalaciones.

---

# 69. Health API

Debe existir un endpoint para conocer el estado del sistema.

```http
GET /api/v1/health
```

Ejemplo:

```json
{
  "status": "healthy",
  "central": "online",
  "network": "online",
  "devices": {
    "total": 42,
    "online": 40,
    "offline": 2
  }
}
```

---

# 70. Capabilities de la API

El cliente puede consultar qué funcionalidades soporta la instalación.

```http
GET /api/v1/capabilities
```

Ejemplo:

```json
{
  "api": [
    "rest",
    "websocket",
    "webhooks",
    "mqtt"
  ],
  "features": [
    "history",
    "virtual_entities",
    "scenes",
    "automations"
  ]
}
```

Esto permite que diferentes versiones de firmware/software sean compatibles.

---

# 71. API de administración de dispositivos externos

Un tercero autorizado podrá registrar dispositivos externos.

Ejemplo:

```text
External Device
       ↓
Registration
       ↓
Device Model
       ↓
Entities
```

Esto permite que el sistema sea extensible más allá de los ESP32 propios.

---

# 72. External Device

Ejemplo:

```json
{
  "name": "PLC Industrial",
  "manufacturer": "Example",
  "model": "PLC-X100",
  "protocol": "modbus"
}
```

El dispositivo puede posteriormente exponer:

```text
sensor.temperature
switch.pump
sensor.pressure
```

---

# 73. Adaptadores

La integración externa debe utilizar adaptadores.

```text
External System
      ↓
Adapter
      ↓
Third-Party API
      ↓
Device Model
```

Ejemplos:

```text
Modbus Adapter
MQTT Adapter
REST Adapter
CAN Adapter
Cloud Adapter
SCADA Adapter
```

---

# 74. No duplicar lógica

Un adaptador no debe implementar su propio sistema de automatización.

Debe traducir:

```text
External Model
       ↕
Platform Model
```

La lógica central permanece en el sistema.

---

# 75. Compatibilidad futura

La API debe poder evolucionar para soportar:

```text
Matter
Home Assistant
Homey
Alexa
Google Home
Apple Home
SmartThings
SCADA
ERP
MES
BMS
Energy Management
Industrial IoT
```

sin modificar el Device Model.

---

# 76. Matter

Matter se considerará una integración externa especializada.

La arquitectura será:

```text
Device Model
      ↓
Matter Adapter
      ↓
Matter Fabric
```

No:

```text
ESP32
 ↓
Matter-specific application logic
```

El modelo interno seguirá siendo independiente.

---

# 77. API local y API remota

La plataforma debe diferenciar:

```text
Local API
Remote API
```

La API local podrá utilizar:

```text
HTTP
HTTPS
WebSocket
```

La API remota deberá exigir:

```text
HTTPS
authentication
authorization
rate limiting
```

Nunca se debe exponer directamente la API administrativa a Internet.

---

# 78. Acceso remoto

Para acceso desde Internet se recomienda:

```text
Internet
   ↓
Secure Gateway
   ↓
Central
```

No:

```text
Internet
   ↓
Port forwarding
   ↓
ESP32
```

---

# 79. Zero Trust

Cada solicitud externa debe considerarse no confiable hasta ser autenticada y autorizada.

La red local no implica automáticamente:

```text
full access
```

---

# 80. API Gateway

En instalaciones grandes puede existir:

```text
Internet
   ↓
API Gateway
   ↓
Central
   ↓
Zone Controllers
```

El Gateway puede encargarse de:

```text
TLS
Authentication
Rate limiting
Routing
Logging
Remote access
```

---

# 81. Escalabilidad

La API debe funcionar tanto para:

```text
1 Central
5 devices
20 entities
```

como para:

```text
1 Central
20 Zone Controllers
500 nodes
5000+ entities
```

La arquitectura debe evitar respuestas gigantescas.

---

# 82. Pagination

Las listas deben soportar paginación.

Ejemplo:

```http
GET /api/v1/entities?limit=100&offset=200
```

Preferentemente se evolucionará hacia paginación mediante cursor.

---

# 83. Filtering

Ejemplo:

```http
GET /api/v1/entities?type=light
```

```http
GET /api/v1/entities?zone=garden
```

```http
GET /api/v1/entities?status=offline
```

---

# 84. Sorting

Ejemplo:

```http
GET /api/v1/entities?sort=name
```

---

# 85. Batch operations

Para instalaciones grandes podrán utilizarse operaciones agrupadas.

Ejemplo:

```http
POST /api/v1/commands
```

```json
{
  "commands": [
    {
      "entity": "light.living",
      "command": "turn_off"
    },
    {
      "entity": "light.kitchen",
      "command": "turn_off"
    }
  ]
}
```

Esto evita cientos de solicitudes individuales.

---

# 86. Transacciones

No todas las operaciones deben ser transaccionales.

Cuando una operación requiera consistencia, debe poder especificarse.

Ejemplo:

```json
{
  "mode": "atomic"
}
```

Si no es posible ejecutar todo:

```text
rollback
```

cuando el tipo de operación lo permita.

---

# 87. Command Queue

Para instalaciones distribuidas, los comandos pueden requerir una cola.

```text
Third Party
    ↓
API
    ↓
Command Queue
    ↓
Zone Controller
    ↓
Node
```

Esto permite:

* reintentos;
* prioridad;
* timeout;
* persistencia;
* confirmación.

---

# 88. Timeout

Todo comando remoto deberá tener timeout.

Ejemplo:

```json
{
  "timeout_ms": 5000
}
```

El timeout no implica necesariamente que el comando nunca se haya ejecutado.

Por ello se utilizará:

```text
request_id
```

para consultar el resultado.

---

# 89. Command status

Endpoint:

```http
GET /api/v1/commands/{request_id}
```

Ejemplo:

```json
{
  "request_id": "req_001",
  "status": "completed",
  "result": {
    "entity": "light.living",
    "state": "on"
  }
}
```

---

# 90. Seguridad de entidades críticas

Algunas entidades podrán clasificarse como:

```text
normal
protected
critical
security
safety
```

Por ejemplo:

```text
door.lock
alarm_control_panel
gas.valve
water.main
emergency.stop
```

Las aplicaciones externas necesitarán permisos especiales.

---

# 91. Safety override

Nunca debe permitirse que una integración externa desactive una función de seguridad crítica salvo que:

```text
policy
+
authorization
+
required confirmation
```

lo permitan.

---

# 92. External data validation

Los datos recibidos de terceros deberán validarse.

Ejemplo:

```text
temperature
```

debe respetar:

```text
tipo numérico
rango permitido
unidad
timestamp
quality
```

Un tercero no debería poder publicar:

```text
temperature = 999999
```

sin que el sistema lo detecte.

---

# 93. Units

Los datos deben indicar unidades.

Ejemplo:

```json
{
  "value": 24.5,
  "unit": "°C"
}
```

Internamente se recomienda utilizar unidades normalizadas.

La API puede convertirlas cuando sea solicitado.

---

# 94. Metadata

Una entidad puede proporcionar metadata.

Ejemplo:

```json
{
  "entity_id": "sensor.temperature.living",
  "metadata": {
    "manufacturer": "AHT20",
    "precision": 0.1,
    "sampling_period_ms": 2000
  }
}
```

---

# 95. Extensibilidad

La API debe permitir atributos adicionales.

Ejemplo:

```json
{
  "state": {
    "value": 24.5
  },
  "attributes": {
    "custom_attribute": "example"
  }
}
```

Los atributos personalizados no deben romper clientes existentes.

---

# 96. SDK

A futuro podrán crearse SDK oficiales.

Ejemplos:

```text
JavaScript / TypeScript
Python
C++
C#
Java
```

El SDK debe ser una capa sobre la API.

No debe definir un modelo diferente.

---

# 97. OpenAPI

La API REST deberá documentarse utilizando:

```text
OpenAPI
```

Ejemplo:

```text
/openapi.json
```

y opcionalmente:

```text
/api/docs
```

Esto permitirá generar automáticamente:

* documentación;
* clientes;
* SDK;
* pruebas;
* validadores.

---

# 98. JSON Schema

Los objetos importantes deberán disponer de esquemas.

Ejemplo:

```text
schemas/
├── device.json
├── entity.json
├── state.json
├── command.json
├── event.json
└── error.json
```

---

# 99. Compatibilidad

Los cambios deberán clasificarse como:

```text
PATCH
MINOR
MAJOR
```

Cambios compatibles:

```text
agregar campo opcional
agregar endpoint
agregar capability
```

Cambios incompatibles:

```text
eliminar campo
cambiar significado
cambiar tipo
eliminar endpoint
```

deberán requerir nueva versión.

---

# 100. API interna versus Third-Party API

No deben ser necesariamente la misma interfaz.

```text
Internal API
     ↓
System internals

Third-Party API
     ↓
External consumers
```

La API externa debe ser:

* estable;
* segura;
* documentada;
* limitada;
* independiente de implementación.

---

# 101. Qué NO debe hacer un tercero

Un tercero no debería necesitar conocer:

```text
GPIO
I2C address
SPI CS
UART
CAN ID
Modbus register
ESP32 MAC
IP del nodo
FreeRTOS task
driver
firmware version
```

salvo que tenga permisos administrativos específicos.

---

# 102. Ejemplo completo

Un tercero quiere controlar la luz del living.

### Discovery

```http
GET /api/v1/entities?type=light
```

Obtiene:

```json
{
  "entity_id": "light.living",
  "type": "light",
  "capabilities": [
    "on_off",
    "brightness"
  ]
}
```

### Estado

```http
GET /api/v1/entities/light.living/state
```

### Comando

```http
POST /api/v1/entities/light.living/command
```

```json
{
  "command": "turn_on"
}
```

### Evento

```json
{
  "event": "state_changed",
  "entity_id": "light.living",
  "state": {
    "on": true
  }
}
```

El tercero nunca necesita saber qué ESP32 controla la lámpara.

---

# 103. Ejemplo de sensor externo

Una aplicación meteorológica obtiene:

```text
Temperatura = 28.4 °C
```

Publica:

```http
POST /api/v1/entities/sensor.weather.temperature/state
```

```json
{
  "value": 28.4,
  "unit": "°C",
  "quality": "good"
}
```

La plataforma puede utilizar ese dato:

```text
sensor.weather.temperature
             ↓
Automation
             ↓
Climate Controller
             ↓
ESP32
             ↓
Aire acondicionado
```

---

# 104. Ejemplo industrial

Un ERP consulta:

```text
production.machine.status
```

La API responde:

```json
{
  "state": "running",
  "attributes": {
    "rpm": 1450,
    "temperature": 63.4,
    "load": 72
  }
}
```

El ERP no necesita conocer:

```text
Modbus register 40021
```

---

# 105. Ejemplo energético

Una aplicación externa puede leer:

```text
energy.house.power
energy.house.voltage
energy.house.current
energy.solar.production
energy.battery.soc
```

y posteriormente enviar:

```text
set_load_limit
```

si dispone del permiso correspondiente.

---

# 106. Ejemplo de integración completa

```text
┌───────────────────────────┐
│ External Energy Manager   │
└─────────────┬─────────────┘
              │
              │ HTTPS
              ▼
┌───────────────────────────┐
│ Third-Party API           │
│                           │
│ Auth                      │
│ Permissions               │
│ Rate Limit                │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ Device Model              │
│                           │
│ Energy entities           │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ Internal System Bus       │
└─────────────┬─────────────┘
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
     ESP32   CAN    Modbus
```

---

# 107. Third-party plugins

A futuro se podrá permitir que terceros desarrollen plugins.

Ejemplo:

```text
plugins/
├── weather/
├── energy/
├── irrigation/
├── industrial/
└── security/
```

Un plugin podrá:

```text
crear entidades
leer entidades
publicar datos
recibir eventos
ejecutar comandos
```

según sus permisos.

---

# 108. Sandbox

Los plugins de terceros no deben tener acceso directo al sistema operativo o hardware.

Preferentemente:

```text
Plugin
   ↓
Plugin API
   ↓
Platform
```

y no:

```text
Plugin
   ↓
Filesystem
   ↓
GPIO
   ↓
Network
```

---

# 109. Marketplace futuro

La arquitectura podrá permitir un futuro:

```text
Integration Marketplace
```

donde terceros puedan distribuir:

* integraciones;
* drivers;
* dashboards;
* módulos;
* sensores virtuales;
* automatizaciones;
* conectores.

Esto no forma parte de la primera implementación, pero la arquitectura debe evitar bloquearlo.

---

# 110. Principio de independencia

La API debe garantizar:

```text
External Application
        ≠
Hardware
```

y:

```text
Entity ID
        ≠
GPIO
```

y:

```text
Entity
        ≠
Protocol
```

---

# 111. Seguridad por capas

La seguridad completa será:

```text
TLS
 ↓
Authentication
 ↓
Authorization
 ↓
Scopes
 ↓
Zone permissions
 ↓
Entity permissions
 ↓
Command validation
 ↓
Safety policies
 ↓
Audit
```

---

# 112. Reglas fundamentales

### Regla 1

Nunca exponer directamente GPIO.

### Regla 2

Nunca depender de IPs de nodos para identificar entidades.

### Regla 3

Nunca asumir que Internet está disponible.

### Regla 4

Nunca permitir que una integración externa anule automáticamente seguridad local.

### Regla 5

Nunca hacer que MQTT sea una dependencia obligatoria del sistema.

### Regla 6

Nunca exigir que un tercero conozca el hardware.

### Regla 7

Toda integración debe utilizar permisos.

### Regla 8

Los identificadores deben ser estables.

### Regla 9

Los eventos deben poder recuperarse o provocar resincronización.

### Regla 10

La API debe evolucionar sin romper clientes existentes.

---

# 113. Prioridad de funcionamiento

La API externa se encuentra por encima del sistema de automatización local.

Por lo tanto:

```text
SAFETY
   >
SECURITY
   >
DEVICE AUTONOMY
   >
ZONE AUTONOMY
   >
CENTRAL AUTOMATION
   >
THIRD-PARTY API
   >
CLOUD
```

Un servicio externo nunca debe convertirse en un requisito para que el sistema funcione.

---

# 114. Relación con Central

La Third-Party API normalmente será proporcionada por el:

```text
Central ESP32-S3
```

pero conceptualmente pertenece a la plataforma y no al hardware específico.

Arquitectura:

```text
                    INTERNET
                       │
                       ▼
              Third-Party Client
                       │
                       ▼
                Third-Party API
                       │
                       ▼
              Central ESP32-S3
                       │
               Internal System Bus
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Zone Controller  Zone Controller  Node
```

---

# 115. Si el Central falla

La API puede quedar temporalmente indisponible.

Sin embargo:

```text
local automation
zone automation
critical safety
security
device autonomy
```

deben continuar funcionando.

Cuando el Central vuelva:

```text
reconnect
   ↓
state synchronization
   ↓
event synchronization
   ↓
configuration reconciliation
```

---

# 116. Si Internet falla

No debe afectar:

```text
Third-party clients locales
Local Web UI
Local API
Local automation
Zone automation
Device autonomy
```

Solamente las aplicaciones externas que dependan de Internet pueden quedar desconectadas.

---

# 117. Roadmap de implementación

La implementación se dividirá en fases.

## Fase 1 — Core API

```text
GET devices
GET entities
GET state
POST commands
```

---

## Fase 2 — Events

```text
WebSocket
state_changed
device_online
device_offline
```

---

## Fase 3 — Security

```text
API Keys
Bearer Tokens
Scopes
Permissions
Audit
```

---

## Fase 4 — Virtual Entities

```text
create device
create entity
publish state
```

---

## Fase 5 — History

```text
history
aggregation
time ranges
```

---

## Fase 6 — Webhooks

```text
webhook registration
event delivery
signature verification
retry
```

---

## Fase 7 — MQTT

```text
MQTT adapter
MQTT discovery
state publishing
commands
```

---

## Fase 8 — SDK

```text
Python
JavaScript / TypeScript
C++
```

---

## Fase 9 — OAuth / OIDC

Para integraciones externas más complejas.

---

# 118. Estructura propuesta del proyecto

```text
docs/
├── ARCHITECTURE.md
├── DEVICE-MODEL.md
├── COMMUNICATION.md
├── MODULE-DEVELOPMENT.md
├── CENTRAL-ARCHITECTURE.md
├── FREERTOS-TASK-ARCHITECTURE.md
├── HARDWARE-REFERENCE-NODES.md
├── SMART-HOME-INTEGRATION.md
└── THIRD-PARTY-API.md
```

A futuro:

```text
docs/api/
├── OPENAPI.md
├── AUTHENTICATION.md
├── AUTHORIZATION.md
├── EVENTS.md
├── WEBHOOKS.md
├── MQTT.md
└── SDK.md
```

---

# 119. Relación con el resto de la arquitectura

```text
                    THIRD PARTY
                         │
                         ▼
               THIRD-PARTY API
                         │
                         ▼
                INTEGRATION LAYER
                         │
                         ▼
                  DEVICE MODEL
                         │
                         ▼
                INTERNAL SYSTEM BUS
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             CAN       RS485      Ethernet
              │          │          │
              ▼          ▼          ▼
            Nodes      Nodes      Nodes
```

El flujo inverso también debe ser posible:

```text
Sensor
  ↓
Node
  ↓
System Bus
  ↓
Device Model
  ↓
Third-Party API
  ↓
External Application
```

---

# 120. Principio final

La plataforma debe permitir que una empresa o desarrollador externo pueda construir una aplicación completamente independiente del hardware.

Debe poder pensar:

```text
"Quiero leer la temperatura del living"
```

y no:

```text
"Necesito leer GPIO 21 del ESP32 04
que tiene un AHT20 por I2C
con dirección 0x38".
```

Del mismo modo, debe poder pensar:

```text
"Quiero encender la luz del living".
```

y no:

```text
"Necesito escribir HIGH en GPIO 23".
```

La abstracción fundamental será:

```text
┌──────────────┐
│ THIRD PARTY  │
└──────┬───────┘
       │
       │ API
       ▼
┌──────────────┐
│    ENTITY    │
└──────┬───────┘
       │
       │ CAPABILITY
       ▼
┌──────────────┐
│ DEVICE MODEL │
└──────┬───────┘
       │
       │ SYSTEM BUS
       ▼
┌──────────────┐
│  TRANSPORT   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   HARDWARE   │
└──────────────┘
```

> **El tercero interactúa con funciones, no con hardware.**

Este principio permite que la plataforma pueda crecer desde una instalación residencial pequeña hasta sistemas industriales y de automatización distribuida sin obligar a las aplicaciones externas a conocer cómo está construida internamente la red.

## Principio arquitectónico definitivo

> **La Third-Party API debe convertir la plataforma en una infraestructura abierta: cualquier sistema autorizado puede leer, publicar, controlar y crear elementos dentro de la red, pero siempre a través del modelo lógico de la plataforma y respetando sus políticas de seguridad, autonomía y prioridades.**
