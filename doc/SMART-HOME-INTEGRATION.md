# Smart Home Integration

> **Tipo:** Arquitectura / Integraciones
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05

---

# 1. Objetivo

Este documento define cómo la plataforma debe integrarse con sistemas externos de automatización y Smart Home.

La plataforma debe permitir que los dispositivos creados mediante ESP32 puedan ser reconocidos y utilizados por:

* Home Assistant;
* Amazon Alexa;
* Google Home;
* Samsung SmartThings;
* Apple Home;
* Homey Pro;
* Matter;
* MQTT;
* REST API;
* WebSocket;
* otras plataformas futuras.

El usuario final debe poder realizar la integración **sin modificar el firmware**.

La configuración debe realizarse desde el Portal Web de la plataforma.

---

# 2. Principio fundamental

> **Los ecosistemas externos no deben conocer el hardware interno del dispositivo.**

Alexa, Google Home o Home Assistant no deberían necesitar saber:

```text
ESP32-C3
ESP32-S3
GPIO 12
MCP23017
LAN8720A
W5500
RS485
CAN
Modbus
```

Deben recibir únicamente información lógica.

Ejemplo:

```text
Device:
"Luz dormitorio"

Type:
Light

Capabilities:
ON/OFF
Brightness
```

---

# 3. Arquitectura

La arquitectura general será:

```text
┌──────────────────────────────────────┐
│              HARDWARE                │
│ ESP32 / Relay / Sensor / CAN / etc. │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│            DEVICE MODEL              │
│                                      │
│ Device / Resource / Capability       │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│         INTEGRATION LAYER            │
│                                      │
│ Entity / Service / Event Mapping     │
└───────────┬───────────┬──────────────┘
            │           │
      ┌─────┴────┐ ┌────┴──────┐
      ▼          ▼ ▼           ▼
   Matter       MQTT          REST
      │          │             │
      ▼          ▼             ▼
  Smart Home  Home Assistant  Apps
```

---

# 4. No crear integraciones dentro de los Nodes

Un ESP32 no debería tener lógica específica para:

```text
Alexa
Google Home
Home Assistant
SmartThings
```

Por ejemplo, un Relay Node no debe implementar:

```text
AlexaRelay
GoogleRelay
HomeAssistantRelay
```

Debe implementar:

```text
Relay
```

La capa de integración realiza la traducción.

---

# 5. Modelo universal

Cada dispositivo debe tener una representación universal.

Ejemplo:

```json
{
  "id": "light_bedroom_01",
  "name": "Luz dormitorio",
  "type": "light",
  "zone": "bedroom",
  "capabilities": [
    "switch",
    "brightness"
  ]
}
```

A partir de esta información pueden generarse las entidades externas.

---

# 6. Device → Entity

La plataforma debe distinguir:

```text
Device
```

de:

```text
Entity
```

Un único dispositivo físico puede contener varias entidades.

Ejemplo:

```text
Device:
"Estación Meteorológica Patio"

Entities:

temperature
humidity
pressure
wind_speed
wind_direction
rain
```

---

# 7. Ejemplo

Un ESP32-S3 puede controlar:

```text
Relay 1
Relay 2
Relay 3
Temperature
Humidity
Power
```

El sistema puede publicar:

```text
Device:
"Controlador cocina"

Entities:

light.kitchen_main
switch.kitchen_fan
switch.kitchen_pump
sensor.kitchen_temperature
sensor.kitchen_humidity
sensor.kitchen_power
```

---

# 8. Entity ID

Cada entidad debe tener un identificador estable.

Formato recomendado:

```text
<domain>.<unique_id>
```

Ejemplos:

```text
light.bedroom_main
switch.garden_pump
sensor.living_temperature
binary_sensor.front_door
cover.living_blind
```

El ID no debe depender del GPIO.

Incorrecto:

```text
relay_gpio_12
```

Preferible:

```text
light.living_main
```

---

# 9. Identidad interna

La plataforma debe mantener un identificador único independiente.

Ejemplo:

```text
Device ID:
dev_01HXYZ...

Resource ID:
resource_01...

Entity ID:
light.living_main
```

Esto permite cambiar:

```text
GPIO
ESP32
PCB
Node
```

sin perder la identidad lógica del dispositivo.

---

# 10. Friendly Name

Cada dispositivo y entidad debe tener un nombre visible.

Ejemplo:

```text
Device:
Controlador Dormitorio

Entity:
Luz principal

Entity:
Temperatura

Entity:
Persiana
```

El nombre puede modificarse desde el Portal.

---

# 11. Zonas

Las entidades deben poder pertenecer a una zona.

Ejemplo:

```text
Casa
│
├── Living
│   ├── Luz
│   ├── Temperatura
│   └── Persiana
│
├── Dormitorio
│   ├── Luz
│   ├── Temperatura
│   └── PIR
│
└── Jardín
    ├── Riego
    ├── Luz
    └── Humedad suelo
```

La zona debe formar parte del modelo lógico y no de la integración específica.

---

# 12. Device Classes

La plataforma debe definir clases lógicas normalizadas.

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
scene
climate
alarm_control_panel
camera
media_player
vacuum
water_valve
energy_meter
```

También se podrán definir clases propias.

---

# 13. Sensor Types

Los sensores deben declarar:

```text
measurement
unit
device_class
state_class
precision
```

Ejemplo:

```json
{
  "entity": "sensor.living_temperature",
  "type": "temperature",
  "unit": "°C",
  "device_class": "temperature"
}
```

Otros ejemplos:

```text
humidity
pressure
illuminance
co2
pm25
voltage
current
power
energy
wind_speed
wind_direction
rain
water_flow
```

---

# 14. Binary Sensors

Ejemplos:

```text
motion
door
window
presence
smoke
water_leak
tamper
occupancy
```

Ejemplo:

```json
{
  "entity": "binary_sensor.front_door",
  "type": "door",
  "state": "open"
}
```

---

# 15. Actuadores

Los actuadores deben declarar sus capacidades.

Ejemplo:

```text
Light
├── on/off
└── brightness
```

```text
Fan
├── on/off
└── speed
```

```text
Cover
├── open
├── close
├── stop
└── position
```

---

# 16. Servicios

Cada entidad puede declarar comandos.

Ejemplo:

```text
light.turn_on
light.turn_off
light.set_brightness
```

La capa de integración transforma estos comandos al formato correspondiente.

---

# 17. Eventos

Los eventos también deben tener una representación universal.

Ejemplos:

```text
motion.detected
door.opened
button.pressed
alarm.triggered
water.detected
temperature.threshold_exceeded
```

---

# 18. Estados

Los estados deben ser independientes del protocolo.

Ejemplo:

```json
{
  "entity": "light.living_main",
  "state": "on",
  "brightness": 75
}
```

La integración decide cómo representarlo.

---

# 19. Matter

Matter debe considerarse una de las principales tecnologías de integración de Smart Home.

Arquitectura:

```text
ESP32
   │
   ▼
Device Model
   │
   ▼
Matter Adapter
   │
   ▼
Matter
   │
   ├── Apple Home
   ├── Google Home
   ├── Alexa
   └── SmartThings
```

Esto permite reducir la necesidad de implementar una integración individual para cada ecosistema compatible con Matter.

---

# 20. Matter como abstracción

La plataforma debe intentar mapear:

```text
Capability
```

a:

```text
Matter Cluster / Device Type
```

Ejemplo conceptual:

```text
Light
 ↓
Matter Light device
```

```text
Temperature Sensor
 ↓
Matter Temperature Sensor
```

```text
On/Off Relay
 ↓
Matter On/Off device
```

---

# 21. Limitaciones de Matter

No todas las capacidades internas tendrán necesariamente una representación Matter equivalente.

Por ejemplo, una entidad industrial muy específica puede no tener un Device Type estándar.

En ese caso:

```text
Device Model
      │
      ├── Matter
      │
      ├── MQTT
      │
      └── REST
```

La capacidad sigue existiendo aunque Matter no pueda representarla directamente.

---

# 22. MQTT

MQTT será uno de los principales mecanismos de integración con plataformas externas.

Ejemplo:

```text
automation/home/living/light01/state
```

```text
automation/home/living/light01/command
```

```text
automation/home/living/light01/event
```

```text
automation/home/living/light01/availability
```

---

# 23. MQTT Discovery

La plataforma debe poder generar automáticamente configuraciones de discovery cuando el ecosistema lo permita.

El usuario debe poder activar:

```text
Integraciones
   ↓
MQTT
   ↓
Home Assistant Discovery
   ↓
Activar
```

Sin modificar firmware.

---

# 24. Home Assistant

Home Assistant debe poder conectarse mediante:

```text
Matter
MQTT
REST
WebSocket
```

según la función.

Preferencia:

```text
Matter
```

cuando la entidad sea compatible.

```text
MQTT
```

para capacidades avanzadas y entidades personalizadas.

---

# 25. Home Assistant Discovery

La plataforma puede publicar automáticamente:

```text
Device
Entity
Name
Unique ID
Device Class
State Topic
Command Topic
Availability
Units
Capabilities
```

Ejemplo conceptual:

```text
Device:
Controlador Living

Entities:
Light
Temperature
Humidity
Power
```

Home Assistant debe detectar el dispositivo sin que el usuario tenga que crear manualmente cada entidad.

---

# 26. Home Assistant Device Registry

Cuando sea posible, todas las entidades pertenecientes al mismo Node deben aparecer agrupadas.

Ejemplo:

```text
Controlador Living
│
├── Luz principal
├── Luz auxiliar
├── Temperatura
├── Humedad
└── Energía
```

---

# 27. Amazon Alexa

La integración con Alexa debe utilizar preferentemente una arquitectura de alto nivel.

Opciones:

```text
Matter
```

o:

```text
Alexa Smart Home Skill
```

La plataforma debe seleccionar automáticamente la vía adecuada.

---

# 28. Alexa mediante Matter

Cuando un dispositivo sea compatible:

```text
ESP32
 ↓
Matter
 ↓
Alexa
```

El usuario podrá descubrir el dispositivo desde Alexa.

Ejemplo:

```text
"Luz living"
```

Alexa debería reconocerla como:

```text
Light
```

y permitir:

```text
encender
apagar
regular
```

según sus capacidades.

---

# 29. Alexa Skill

Para capacidades que no puedan exponerse adecuadamente mediante Matter, se podrá implementar una integración mediante Skill.

Arquitectura:

```text
Alexa
  │
  ▼
Alexa Skill
  │
  ▼
Cloud/API
  │
  ▼
Platform API
  │
  ▼
Central
```

Esta integración dependerá de conectividad externa.

---

# 30. Regla de autonomía

Las integraciones externas no deben convertirse en dependencia para el funcionamiento local.

Ejemplo:

```text
Internet OFF
```

Debe continuar:

```text
ESP32
├── Sensors
├── Relays
├── Automations
├── Security
└── Local UI
```

Puede dejar de funcionar temporalmente:

```text
Alexa
Google Home
Cloud
Remote access
```

---

# 31. Google Home

Google Home podrá utilizar:

```text
Matter
```

como vía principal cuando corresponda.

Arquitectura:

```text
ESP32
 ↓
Matter
 ↓
Google Home
```

Para funciones avanzadas se podrá utilizar:

```text
Google Home integration
```

mediante servicios compatibles.

---

# 32. Samsung SmartThings

SmartThings podrá utilizar:

```text
Matter
```

o:

```text
SmartThings API / integración específica
```

según las capacidades.

---

# 33. Apple Home

Apple Home debe utilizar preferentemente:

```text
Matter
```

para dispositivos compatibles.

La plataforma no debe implementar un protocolo propietario por dispositivo.

---

# 34. Homey Pro

Homey Pro podrá integrarse mediante:

```text
Matter
MQTT
REST
```

o una aplicación específica cuando sea necesaria.

---

# 35. REST API

La plataforma debe proporcionar una API REST completa.

Ejemplo:

```text
GET /api/devices
GET /api/entities
GET /api/zones
GET /api/scenes
GET /api/automations
```

Comandos:

```text
POST /api/entities/{id}/command
```

Ejemplo:

```json
{
  "command": "turn_on"
}
```

---

# 36. WebSocket

WebSocket se utilizará para:

* dashboards;
* aplicaciones;
* pantallas;
* monitorización;
* eventos en tiempo real.

Ejemplo:

```text
Central
  │
  ▼
WebSocket
  │
  ├── Web UI
  ├── Mobile App
  └── Display Node
```

---

# 37. API y seguridad

La API debe soportar:

```text
Users
Roles
Tokens
API Keys
Permissions
TLS
```

Ejemplo:

```text
Administrator
Installer
User
Viewer
Integration
```

---

# 38. Integraciones activables

Desde el portal:

```text
Integrations

☑ Matter
☑ MQTT
☐ Home Assistant
☐ Alexa
☐ Google Home
☐ SmartThings
☐ Homey
☐ Apple Home
```

El usuario no debe modificar código.

---

# 39. Configuración mediante portal

El usuario debe acceder a:

```text
Portal
 ↓
Integrations
```

y ver:

```text
Matter
[ Enable ]

MQTT
[ Enable ]

Home Assistant
[ Enable ]

Alexa
[ Enable ]

Google Home
[ Enable ]
```

---

# 40. Descubrimiento automático

La plataforma debe poder descubrir Nodes automáticamente.

Proceso:

```text
Node boot
   │
   ▼
Discovery
   │
   ▼
Central / Zone Controller
   │
   ▼
Device Registry
   │
   ▼
Entity generation
   │
   ▼
Integration adapters
```

---

# 41. Auto-provisioning

Cuando se conecta un nuevo Node:

```text
Node detected
      │
      ▼
Hardware identification
      │
      ▼
Capabilities
      │
      ▼
Configuration
      │
      ▼
Entities generated
      │
      ▼
Integrations updated
```

---

# 42. Ejemplo completo

Se conecta:

```text
ESP32-WROOM-32
```

con:

```text
4 relays
1 temperature sensor
1 power meter
```

El sistema detecta:

```text
Device:
Controlador Cocina
```

y crea:

```text
light.cocina_01
light.cocina_02
switch.cocina_03
switch.cocina_04
sensor.cocina_temperature
sensor.cocina_power
```

---

# 43. Exposición selectiva

El usuario debe decidir qué entidades exponer.

Ejemplo:

```text
Controlador Cocina

☑ Luz principal
☑ Temperatura
☑ Energía
☐ Relay técnico
☐ Entrada de mantenimiento
```

Esto evita exponer información innecesaria.

---

# 44. Entidades internas

No todo debe ser visible externamente.

Puede existir:

```text
Internal Entity
```

Ejemplo:

```text
maintenance_mode
firmware_update
diagnostic_state
```

Estas entidades pueden permanecer solamente dentro de la plataforma.

---

# 45. Entidades virtuales

La plataforma también debe poder crear entidades que no correspondan a un GPIO físico.

Ejemplos:

```text
Scene
Group
Virtual Switch
Virtual Sensor
Counter
Timer
Alarm
Presence
Energy Summary
```

Ejemplo:

```text
Virtual Switch:
"Casa en modo noche"
```

Puede activar:

```text
Luces
Persianas
Alarmas
Temperatura
```

---

# 46. Groups

Un grupo puede representar:

```text
All Lights
Ground Floor Lights
Garden Lights
Bedroom Lights
```

El grupo puede exponerse como una entidad lógica.

---

# 47. Scenes

Ejemplo:

```text
Scene:
"Salir de casa"
```

Acciones:

```text
Lights OFF
Climate OFF
Blinds CLOSED
Security ARMED
```

Puede exponerse a:

```text
Matter
Home Assistant
Alexa
Google Home
```

cuando sea compatible.

---

# 48. Automations

Las automatizaciones internas deben permanecer dentro de la plataforma siempre que sea posible.

Ejemplo:

```text
PIR detected
+
HOUSE.MODE == NIGHT
        ↓
Hall Light = 15 %
```

No debe depender de Alexa.

---

# 49. Alexa como interfaz

Alexa puede ejecutar:

```text
Scene
```

en lugar de contener toda la lógica.

Ejemplo:

```text
Alexa:
"Buenas noches"

        ↓

Scene:
NIGHT

        ↓

Platform:
Lights
Blinds
Alarm
Climate
```

Esto mantiene la lógica crítica local.

---

# 50. Google Home como interfaz

Igualmente:

```text
Google Home
       ↓
Command
       ↓
Platform
       ↓
Automation
```

El ecosistema externo actúa principalmente como:

```text
Interface
```

no como:

```text
Core Controller
```

---

# 51. Estados sincronizados

Cuando cambia un dispositivo:

```text
Relay ON
```

el estado debe propagarse:

```text
Node
 ↓
Device Model
 ↓
Integration Layer
 ↓
Matter / MQTT / WebSocket / API
```

Esto evita estados desactualizados.

---

# 52. State of Truth

Debe existir una fuente lógica de verdad.

Propuesta:

```text
Physical Device State
        ↓
Device State Manager
        ↓
Integration adapters
```

Las integraciones no deben mantener una copia independiente que pueda convertirse en autoridad.

---

# 53. Offline

Si Alexa queda desconectada:

```text
Platform
   │
   ├── Local automation ✓
   ├── Sensors ✓
   ├── Actuators ✓
   └── Security ✓
```

Cuando vuelve la conexión:

```text
Integration
   │
   ▼
State synchronization
```

---

# 54. Reconnection

Después de recuperar una conexión:

```text
CONNECTING
    ↓
AUTHENTICATING
    ↓
SYNCING
    ↓
ONLINE
```

La sincronización debe actualizar:

```text
States
Availability
Capabilities
Entities
```

---

# 55. Availability

Cada dispositivo debe informar:

```text
online
offline
degraded
unknown
maintenance
```

Ejemplo:

```json
{
  "availability": "online"
}
```

---

# 56. Availability MQTT

Ejemplo conceptual:

```text
automation/home/living/light01/availability
```

Payload:

```text
online
```

o:

```text
offline
```

---

# 57. Last Will

Cuando MQTT sea utilizado, se recomienda utilizar mecanismos de disponibilidad adecuados, incluyendo Last Will cuando corresponda.

Esto permite detectar Nodes desconectados.

---

# 58. Versionado

Las entidades y capacidades deben tener versiones.

Ejemplo:

```text
Device Model v1
Integration Schema v1
```

Esto permite actualizar el firmware sin romper integraciones existentes.

---

# 59. Compatibilidad

Cada integración debe declarar:

```text
supported
unsupported
partial
```

Ejemplo:

```text
Light
Matter: ✓
MQTT: ✓
Home Assistant: ✓
Alexa: ✓
Google: ✓
```

Una capacidad avanzada:

```text
Industrial Sensor
Matter: partial
MQTT: ✓
REST: ✓
```

---

# 60. Mapeo de capacidades

La plataforma debe mantener un registro de conversiones.

Ejemplo:

```text
Internal Capability
        │
        ├── Matter mapping
        ├── MQTT mapping
        ├── Home Assistant mapping
        ├── Alexa mapping
        └── Google mapping
```

---

# 61. Integration Adapter

Cada integración debe implementar una interfaz común.

Conceptualmente:

```text
IntegrationAdapter
│
├── discover()
├── publishDevice()
├── publishEntity()
├── publishState()
├── receiveCommand()
├── publishEvent()
├── removeEntity()
└── synchronize()
```

---

# 62. Ejemplo

```text
MatterAdapter
MQTTAdapter
RESTAdapter
WebSocketAdapter
AlexaAdapter
GoogleAdapter
SmartThingsAdapter
HomeAssistantAdapter
```

No deben modificar el Device Model.

---

# 63. Registro de integraciones

La plataforma puede utilizar:

```text
Integration Registry
```

Ejemplo:

```json
{
  "id": "mqtt",
  "name": "MQTT",
  "enabled": true,
  "version": "1.0.0"
}
```

---

# 64. Plugins

Las integraciones deben poder implementarse como módulos.

Ejemplo:

```text
modules/
└── integrations/
    ├── matter/
    ├── mqtt/
    ├── home_assistant/
    ├── alexa/
    ├── google_home/
    └── smartthings/
```

Esto permite instalar/desactivar integraciones sin modificar el Core.

---

# 65. No depender de una sola integración

Un dispositivo puede estar simultáneamente disponible mediante:

```text
Matter
MQTT
REST
WebSocket
```

Ejemplo:

```text
ESP32 Relay Node
        │
        ▼
Device Model
   ┌────┼─────┐
   ▼    ▼     ▼
Matter MQTT REST
```

---

# 66. Configuración del usuario

El usuario debe poder configurar:

```text
Nombre
Zona
Icono
Exposición
Integraciones
Permisos
```

sin tocar:

```text
C++
PlatformIO
GPIO
Firmware
MQTT topics
JSON interno
```

---

# 67. Configuración avanzada

Para instaladores se puede ofrecer:

```text
Advanced Integration Settings
```

incluyendo:

```text
Entity ID
MQTT topic
QoS
Retain
Discovery
Matter endpoint
API permissions
```

Pero estas opciones deben estar ocultas para el usuario normal.

---

# 68. Portal de integración

Ejemplo:

```text
┌─────────────────────────────────────┐
│ Integraciones                       │
├─────────────────────────────────────┤
│                                     │
│ Matter                 ● Activo     │
│ MQTT                   ● Activo     │
│ Home Assistant         ● Activo     │
│ Alexa                  ○ Inactivo   │
│ Google Home            ○ Inactivo   │
│ SmartThings            ○ Inactivo   │
│ Apple Home             ○ Inactivo   │
│ Homey Pro              ○ Inactivo   │
│                                     │
└─────────────────────────────────────┘
```

---

# 69. Portal de entidades

```text
Casa
│
├── Living
│   ├── 💡 Luz principal
│   ├── 🌡 Temperatura
│   └── 💧 Humedad
│
├── Cocina
│   ├── 💡 Luz
│   └── ⚡ Energía
│
└── Jardín
    ├── 💧 Riego
    └── 🌡 Temperatura
```

Cada entidad puede mostrar:

```text
Exponer a:
☑ Matter
☑ Home Assistant
☑ Alexa
☑ Google Home
☐ MQTT
```

---

# 70. Portal de dispositivos

```text
Device
Controlador Living

Hardware
ESP32-S3

Status
ONLINE

Zone
Living

Entities
5

Integrations
Matter ✓
MQTT ✓
Home Assistant ✓
Alexa ✓
Google ✓
```

---

# 71. Vinculación

El sistema debe proporcionar asistentes de configuración.

Ejemplo:

```text
Agregar a Matter

1. Activar Matter
2. Mostrar código QR
3. Escanear con aplicación compatible
4. Seleccionar habitación
5. Finalizar
```

La experiencia debe estar orientada al usuario final.

---

# 72. QR y códigos

Cuando un protocolo lo requiera, el Portal debe poder mostrar:

```text
QR
Pairing Code
Setup Code
Device Information
```

sin que el usuario tenga que editar archivos.

---

# 73. Integración local vs Cloud

Debe diferenciarse:

```text
LOCAL
```

de:

```text
CLOUD
```

Ejemplo:

```text
Matter local
MQTT local
REST local
```

pueden continuar funcionando aunque no exista Internet.

Mientras:

```text
Alexa Cloud
Google Cloud
Remote Access
Cloud Services
```

pueden depender de Internet.

---

# 74. Arquitectura recomendada

```text
                    INTERNET
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Alexa        Google       SmartThings
          │            │            │
          └────────────┼────────────┘
                       │
                    Matter/
                     Cloud
                       │
                       ▼
                 ┌───────────┐
                 │  CENTRAL  │
                 │ ESP32-S3  │
                 └─────┬─────┘
                       │
                 Device Model
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Node           Node           Node
      Relay         Sensor         CAN
```

---

# 75. Arquitectura local

```text
                    LAN
                     │
              ┌──────┴──────┐
              │             │
          Home Assistant  Central
              │             │
              └──────┬──────┘
                     │
                System Bus
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Relay      Sensor      Display
```

---

# 76. Central como Integration Hub

El Central puede actuar como Integration Hub.

Responsabilidades:

```text
Discovery
Entity Registry
Integration Registry
State Synchronization
Authentication
Configuration
API
```

Pero las automatizaciones críticas siguen distribuidas.

---

# 77. Fallo del Central

Si el Central falla:

```text
Matter / external integration
```

puede verse afectada según la arquitectura concreta.

Pero:

```text
Local automation
Node operation
Security
Actuators
Sensors
```

deben continuar.

Los Zone Controllers pueden mantener las automatizaciones de zona.

---

# 78. Integración directa Node ↔ Ecosystem

No debe prohibirse completamente.

Un Node puede integrarse directamente cuando:

```text
hardware
firmware
protocol
security
```

lo permitan.

Ejemplo:

```text
ESP32-C6
 ↓
Matter
 ↓
Google Home
```

Sin necesidad de Central.

Esto es especialmente útil para Nodes sencillos.

---

# 79. Integración mediante Central

Para sistemas más complejos:

```text
Node
 ↓
Zone Controller
 ↓
Central
 ↓
Integration
```

Esto permite:

* administración central;
* configuración;
* agrupación;
* historial;
* permisos;
* traducción de protocolos.

---

# 80. Selección automática

La plataforma debe decidir la mejor ruta.

Ejemplo:

```text
Device compatible Matter
        ↓
Use Matter
```

Si no:

```text
MQTT available
        ↓
Use MQTT
```

Si tampoco:

```text
REST/API
```

---

# 81. Prioridad de integración

Propuesta:

```text
1. Matter
2. MQTT
3. Local API
4. WebSocket
5. Integration-specific API
6. Cloud
```

Esta prioridad puede modificarse por configuración.

---

# 82. Integración con SEMA

Los sensores de SEMA pueden convertirse en entidades:

```text
sensor.outdoor_temperature
sensor.outdoor_humidity
sensor.outdoor_pressure
sensor.wind_speed
sensor.wind_direction
sensor.rain
sensor.uv
```

El mismo Device Model permite integrarlos con:

```text
Home Assistant
Matter
MQTT
REST
```

cuando exista una representación adecuada.

---

# 83. Integración con Invernadero

Ejemplo:

```text
sensor.greenhouse_temperature
sensor.greenhouse_humidity
sensor.soil_moisture
switch.greenhouse_fan
switch.greenhouse_light
cover.greenhouse_roof
```

Las automatizaciones pueden seguir ejecutándose localmente.

---

# 84. Integración energética

Ejemplo:

```text
sensor.house_voltage
sensor.house_current
sensor.house_power
sensor.house_energy
```

Puede exponerse a:

```text
Home Assistant
MQTT
REST
```

y cuando exista soporte adecuado:

```text
Matter
```

---

# 85. Integración de seguridad

Ejemplo:

```text
binary_sensor.front_door
binary_sensor.window_bedroom
binary_sensor.garage_motion
alarm_control_panel.house
```

La lógica de alarma permanece en la plataforma.

Alexa/Google/Home Assistant actúan como interfaces adicionales.

---

# 86. Integración de escenas

Ejemplo:

```text
scene.good_night
```

Puede representar:

```text
Lights OFF
Blinds CLOSED
Security NIGHT
Climate configuration
```

La escena se ejecuta en la plataforma.

---

# 87. Integración de modos

Los modos globales pueden ser entidades lógicas:

```text
HOME
AWAY
SLEEP
VACATION
MAINTENANCE
```

Ejemplo:

```text
HOUSE.MODE = SLEEP
```

Esto puede ser utilizado por:

```text
Automation Engine
Security
Lighting
Climate
External Integrations
```

---

# 88. No duplicar lógica

Incorrecto:

```text
Alexa:
if night -> light 15%

Google:
if night -> light 15%

Home Assistant:
if night -> light 15%

ESP32:
if night -> light 15%
```

Preferible:

```text
Platform:

if HOUSE.MODE == NIGHT
and motion detected
→ light = 15%
```

Alexa/Google/Home Assistant solamente activan o consultan el sistema.

---

# 89. Telemetría

La plataforma puede publicar:

```text
Device status
Sensor state
Energy
Errors
Availability
```

pero debe evitar publicar datos excesivamente frecuentes.

Cada entidad debe tener:

```text
sample rate
publish rate
change threshold
```

cuando corresponda.

---

# 90. Seguridad

Las integraciones deben respetar:

```text
Authentication
Authorization
TLS
Token rotation
API keys
ACL
User roles
Device permissions
```

Un token de Home Assistant no debe proporcionar automáticamente acceso administrativo completo.

---

# 91. Permisos

Ejemplo:

```text
Integration:
Home Assistant

Permissions:

✓ Read sensors
✓ Control lights
✓ Control climate
✓ Execute scenes
✗ Modify firmware
✗ Modify users
✗ Change network
✗ Factory reset
```

---

# 92. Auditoría

Los comandos externos importantes deben poder registrarse.

Ejemplo:

```text
2026-10-05 12:30
Alexa
→ light.living
→ turn_on
→ SUCCESS
```

---

# 93. Diagnóstico

El Portal debe mostrar:

```text
Integration
Status
Last connection
Last synchronization
Errors
Entities
Messages
```

Ejemplo:

```text
Google Home
● Connected

Entities:
24

Last sync:
12:31:24

Errors:
0
```

---

# 94. Actualización de entidades

Si el usuario agrega:

```text
nuevo relay
```

el sistema debe:

```text
Detect
 ↓
Register
 ↓
Create Entity
 ↓
Update integrations
```

sin recompilar firmware.

---

# 95. Eliminación

Si un Node desaparece:

```text
Node offline
```

la entidad debe pasar a:

```text
Unavailable
```

y no eliminarse inmediatamente.

Esto evita perder configuraciones por una desconexión temporal.

---

# 96. Reemplazo de Node

Si se reemplaza un Node físico:

```text
Node antiguo
```

por:

```text
Node nuevo
```

el instalador debe poder asignar:

```text
Logical Device
```

al nuevo hardware.

Ejemplo:

```text
"Luz living"

antes:
ESP32-001

ahora:
ESP32-037
```

Las integraciones no deberían necesitar una reconfiguración completa.

---

# 97. Portabilidad

El modelo lógico debe poder exportarse.

Ejemplo:

```json
{
  "device": "living_controller",
  "entities": [
    {
      "id": "light.living_main",
      "type": "light"
    },
    {
      "id": "sensor.living_temperature",
      "type": "temperature"
    }
  ]
}
```

Esto permite:

* backup;
* migración;
* restauración;
* reemplazo de hardware.

---

# 98. Principio de "Zero Code"

El objetivo para el usuario final es:

```text
Conectar hardware
       ↓
Node detectado
       ↓
Asignar nombre
       ↓
Asignar zona
       ↓
Seleccionar función
       ↓
Activar integraciones
       ↓
Listo
```

El usuario no debería tener que:

```text
editar C++
editar JSON
editar MQTT topics
compilar
subir firmware
configurar GPIO manualmente
```

para una instalación normal.

---

# 99. Instalador

El instalador puede disponer de un modo avanzado:

```text
Installer Mode
```

para configurar:

```text
GPIO
Modules
Protocols
RS485
CAN
MQTT
Network
Entity IDs
Permissions
```

pero estas opciones no deben ser necesarias para un usuario normal.

---

# 100. Principio definitivo

> **El usuario configura funciones; el sistema se encarga de convertir esas funciones en entidades compatibles con cada ecosistema.**

El usuario debe pensar:

```text
"Luz del living"
```

y no:

```text
GPIO 17
Relay 2
MQTT topic X
Matter endpoint 4
```

---

# 101. Arquitectura final

```text
                         USER
                          │
                          ▼
                    WEB PORTAL
                          │
                          ▼
                ┌──────────────────┐
                │  DEVICE MODEL    │
                │                  │
                │ Device           │
                │ Resource         │
                │ Capability       │
                │ Entity           │
                │ Zone             │
                │ Scene            │
                └────────┬─────────┘
                         │
                  INTEGRATION LAYER
                         │
        ┌────────────────┼─────────────────┐
        │                │                 │
        ▼                ▼                 ▼
      Matter            MQTT              API
        │                │                 │
   ┌────┼────┐           │          ┌──────┴─────┐
   ▼    ▼    ▼           ▼          ▼            ▼
Alexa Google Apple    Home Assistant Apps      Services
```

---

# 102. Regla maestra

La plataforma debe seguir siempre esta separación:

```text
HARDWARE
    ↓
DEVICE MODEL
    ↓
CAPABILITY
    ↓
ENTITY
    ↓
INTEGRATION
    ↓
ECOSYSTEM
```

Nunca:

```text
HARDWARE
    ↓
ALEXA
```

ni:

```text
GPIO
    ↓
GOOGLE HOME
```

---

# 103. Resultado esperado

Una persona compra un sistema basado en esta plataforma.

No necesita conocer:

* FreeRTOS;
* ESP32;
* GPIO;
* MQTT;
* CAN;
* Modbus;
* Matter;
* APIs;
* programación.

El proceso ideal es:

```text
1. Conectar Node.

2. El sistema lo descubre.

3. Aparece en el Portal.

4. El usuario selecciona:
   "Controlador de luces".

5. Asigna:
   "Living".

6. Configura:
   "Luz principal".

7. Activa:
   Matter
   Home Assistant
   Alexa
   Google Home

8. El sistema genera automáticamente
   las entidades necesarias.

9. El usuario puede utilizar:
   "Alexa, enciende la luz del living"

   o

   Google Home

   o

   Home Assistant

   o

   la pantalla local

   o

   la aplicación propia.
```

Todo esto debe funcionar sin modificar el firmware.

---

# 104. Documentos relacionados

Este documento debe mantenerse coordinado con:

```text
ARCHITECTURE.md
SYSTEM-ARCHITECTURE.md
CENTRAL-ARCHITECTURE.md
DEVICE-MODEL.md
COMMUNICATION.md
MODULE-DEVELOPMENT.md
FREERTOS-TASK-ARCHITECTURE.md
HARDWARE-REFERENCE-NODES.md
FAULT-TOLERANCE.md
```

La capa de integración debe considerarse un módulo superior al Device Model y nunca una dependencia del hardware físico.

---

# 105. Principio final

> **El sistema debe ser programable una sola vez y configurable infinitas veces.**

El firmware proporciona:

```text
Capabilities
Services
Communication
Automation
Security
```

El Portal proporciona:

```text
Configuration
Discovery
Naming
Zones
Entities
Integrations
Permissions
```

Y los ecosistemas externos reciben una representación estándar de esas capacidades.

De esta manera, agregar un nuevo ESP32, sensor, relay, dimmer, dispositivo CAN o equipo Modbus no obliga a crear nuevamente las integraciones con Alexa, Google Home, Home Assistant, Matter, etc.
